# -
iOS下常用类库整理<br>
* 网络处理  AFNetWorking  Alamofire <br> 
* 数据库    fmdb          sqlite.swift<br>
* 图片异步加载 sdwebimage  kingfisher<br>
* 分享       ShareSDK     <br>
* 菊花君     MBProgressHUD SVProgressHUD<br>
* 轮播       SDCycleScrollView  自己封装吧<br>
* json解析   MJEXension    SwiftJson<br>
* 视频       IJKMediaplayer<br>
* 推送       极光  <br>        



总结学习学习Effective Objective-C 2.0  编写高质量iOS与OS X代码的52个有效方法<br>
* 1. 消息与函数调用 区别<br>
oc 中消息型语言与函数调用  oc其运行的代码由运行环境决定   函数调用 由编译器决定 <br>    
举例说明多态情况下: 	oc                    运行时不管是否多态在运行时去查找这个方法<br>
函数			  是多态的话会运行时按照虚方法表来查到底要执行的方法<br>
明白oc中所有的对象分配是在堆上的，不是在栈上的,延用c  语言的方式<br>
NSString    astackstring   错误    而只能这样使用NSString  *str; 去手动分配<br>
* 2.内存上面的问题<br>
分配在堆上的对象等需要手动管理  栈上的会在弹出时自动清理  如基本类型的变量 包括一系列CG开头的<br>
oc中将内存管理抽象出来，不需要用malloc  和 free 分配释放 而去使用引用计数的方式来管理<br>

* 3.头文件中尽量少引用其他头文件，说不定就会造成头文件引用 <br>
@class  来标明其它类属性  优雅解决<br>
使用协议呢 将其放在单独的文件更好<br>
* 4.多用类型常量少用预处理<br>

a.h中预定义 了个  ＃define  Max    8  那么有b.h 引入这个a.h会把 所有Max 都会换成这个的  <br>
且预处理的不含类型信息（缺点）使用  static   const   double   max = 8 这种方式会更合适点<br>

使用const标记的 ， 想要对它修改 编译就会让它报错  <br>
如果不加static ，编译器会为其创建一个外部符号<br>  如果另一个类中也声明了这样的常量 max 编译器会给出错误提示<br>
用static这个可以来标记这种变量命名一样是否可以在多个类中使用<br>
加了 就不会给他创建一个外部符号，来表示 这个常量就是在本类使用的<br>
static  const  修饰的只会在编译单元内 私有这样来说比较通俗（什么时编译单元内呢， 实际就是.m文件的东西都属于编译单元）<br>
定义这些常量的位置讲究， 是否公开这个常量  不公开  放在实现文件    这样做会更好<br>  
命名规范加类名前缀来避免空间 上的重名冲突<br>

如通知的典范，通知名字字符的定义   <br>
.h  中  extern  NSString   *const   eocstring; <br>
.m NSString  *const    eocstring = @“窗口变化饿了”<br>
怎么看常量定义： 从右向左解读 <br>
eocstring 常量 什么样的常量  是一个指针， 这个指针指向nsstring  <br>
extern   关键标识的常量 会放在 全局符号表中  <br>
只能定义一次，在实现文件定义好了过后  编译器会在数据段为这个字符串分配存储空间<br>
这种方式会让编译器来确保常量值不变 ，且不管在哪个类中定义的这个 所有地方都可以用<br>

* 5.用枚举的时机 ： 状态、选项、状态码
enum  ECname：NSInteger{<br>
ECsucceed =  1,<br>
ECerror,<br>
};<br>
//某种状态可以组合   的方式比如在使用到 animation  选项的时候多种状态组合实现<br>
enum   ECtype:NSInteger{<br>
ECnone = 0 ,<br>
ECsucceed = 1<<0,  //00000001<br>
ECperfect= 1<<1,   //00000010   00000011  3<br>
};<br>
typedef enum  ECtype    ectype;<br>
ectype    atype = ECsucceed | ECperfect ;<br>
//这种不能判断用与ecnone为0这种 与符号为0嘛<br>
if ((atype & ECsucceed )  && (atype & ECperfect) ){<br>
NSLog(@"开启完美 成功模式");<br>
}  <br>       
* 6.关于runtime机制的理解<br>
先说原理不好理解，因此先说说它能干什么<br>
1.  如果是在程序运行过程中 ， 希望动态创建一个类（比如kvo底层就是基于这个特性）<br>
2.程序运行过程中希望动态的生成一些属性或者方法 包括修改<br>
3.遍历一个类的所有成员变量和方法 ，如一个类的属性方法特别多的时候<br>
//这个东西 可以用于深层次开发框架<br>
如：在归档的时候<br>
-(void)encodeWithCoder:(NSCoder )encoder { <br>
unsigned int count = 0; <br>
Ivar ivars = class_copyIvarList([PYPerson class], &count);<br>
for (int i = 0; i<count; i++) {<br>
// 取出i位置对应的成员变量<br>
Ivar ivar = ivars[i];<br>
// 查看成员变量
const char *name = ivar_getName(ivar);<br>
// 归档<br>
NSString *key = [NSString stringWithUTF8String:name];<br>
id value = [self valueForKey:key];<br>
[encoder encodeObject:value forKey:key];<br>
}<br>
-(id)initWithCoder:(NSCoder *)decoder { if (self = [super init]) {<br>
unsigned int count = 0;<br>
Ivar *ivars = class_copyIvarList([PYPerson class], &count);<br>
for (int i = 0; i<count; i++) {<br>
// 取出i位置对应的成员变量<br>
Ivar ivar = ivars[i];<br>
// 查看成员变量<br>
const char *name = ivar_getName(ivar);<br>
// 归档<br>
NSString *key = [NSString stringWithUTF8String:name];<br>
id value = [decoder decodeObjectForKey:key];<br>
// 设置到成员变量身上<br>
[self setValue:value forKey:key];<br>
  }<br>

	free(ivars);<br>
	//tips  这个是c语言上集成的 因此虚呀手动释放<br>
}<br> 
return self;<br>
}<br>
	
	常用函数
	objc_msgSend : 给对象发送消息
        class_copyMethodList : 遍历某个类所有的方法
	class_copyIvarList : 遍历某个类所有的成员变量
	class_..... 这是我们学习runtime必须知道的函数！

	7.oc运行特性的简单运用,即简单函数式编程与响应式编程的实现
	@interface Person : NSObject
	//1. 传统方式
	//- (void)run;
	//- (void)study;
	//-(void)sleep;
	//2.稍微像原始oc的方式
	//-(Person*)run;
	//-(Person*)study;
	//3.最终形态
	@property(nonatomic,strong) NSString  *name;
	-(Person* (^) ())runblock;
	-(Person*  (^) (NSString*studytime))studyblock;
	-(Person*  (^) (NSString *sleeptime))sleepblock;
	
	+(void )adetails;
	@implementation Person
	{
	    NSInteger   age;
	    NSString    *pcode;
	}
	//-(void)run{
	//    NSLog(@"i    run over");
	//}
	//
	//-(void)study
	//{
	//    NSLog(@"i   study  over");
	//}
	//-(void)sleep
	//{
	//      NSLog(@"i go sleep");
	//}
	//-(Person *)run
	//{
	//    NSLog(@"i    run over");
	//    return  [[Person alloc]init];
	//}
	//-(Person *)study{
	//    NSLog(@"i    study over");
	//    return  [[Person alloc]init];
	//}
	-(Person *(^)())runblock{
	    Person* (^block)() = ^() {
        NSLog(@"我跑完步了");
	        return  self;
	    };
	    return  block;
	}
	-(Person *(^)(NSString*studytime))studyblock{
	    Person* (^block)(NSString*studytime) = ^(NSString*studytime) {
	        NSLog(@"我学习了%@小时，终于学完了",studytime);
	        return  self;
    };
	    return  block;
	}
	-(Person *(^)(NSString * sleeptime))sleepblock
	{
	    Person *  (^block)(NSString *sleeptime) = ^ (NSString *sleeptime)
	    {
	        NSLog(@"我在%@,准时入睡",sleeptime);
       return  self;
    };
	    return  block;
	}
	+(void)adetails
	{
	    //这里我们获取函数名字
	    id   classobj  = objc_getClass([@"Person" UTF8String]);
	    //存储属性的property 属性的个数
	    unsigned  int   count = 0 ;
	    //成员变量的个数
    unsigned   int  icount = 0 ;
    class_copyPropertyList(classobj, &count);
	    class_copyIvarList(classobj, &icount);
	//    objc_property_t  *properties = class_copyPropertyList(classobj, &count);
	//    Ivar   *ivars =  class_copyIvarList(classobj, &icount);
	    
	    //打印属性
    NSLog(@"属性%d,成员变量%d",count,icount);
	    
   
	}
	-(NSString *)description{
	    
	    
	    return @"本身的person描述";
	}
	-(NSString*) currentdescription
	{
	    return @"你没有权限来查看我";
	}
	
	-(instancetype)init{
    self = [super init];
    Method   orgdescription =  class_getInstanceMethod([Person class], @selector(description));
	    Method   currentdecription =  class_getInstanceMethod([Person  class], @selector(currentdescription));
	    method_exchangeImplementations(orgdescription, currentdecription);
	    
	    return  self ;
	}
	调用       //这种方式就叫传统的函数式编程
	    Person   * p = [[Person  alloc]init];
	    
	    //第三方自动布局的思想就是响应式的 .sd_layout().left(90).right(80)
	    //模仿实现的最终目标  p.studyblock().runblock().sleepblock()
	    //1. 先实现这种调用 [[p  run]study];
	    
	   // [[p  run] study];
	    //2. 思考 上面这种实际肯定就是block方式调用的
	    p.studyblock(@"8").runblock().sleepblock(@"10");
	    
	    [Person adetails];
	  
	     //这样 就让外部
	    NSLog(@"%@",p);
	tips : 
	一个oc类的+load方法是在app开始运行时首先会被调用执行的方法，也就是说，当app被点击，再被系统加载app程序进入内存后，首先会实例化所 有类到代码或全局区,就会调用类的load方法，如果要给一个类做方法交换，则一般情况放在load方法中来操 作。方法交换一旦完成，则程序运行中全局生效。






