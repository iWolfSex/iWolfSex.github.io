---
layout: post
title: Ojective_c语言的消息机制与运行时(Runtime)
date: 2016-08-23
tag: iOS
---
### Runtime概述

Objective_c是一种动态语言，因此很多行为在运行时决定的。对于静态语言来说，函数的调用在编译时已经确定。动态语言则不然，动态语言使得函数的调用推迟到运行时决定。Objective_c主要是运用消息机制实现运行时特性。Runtime阐述了objective_c语言消息传递、和消息转发机制。

### 使用消息发送代替函数调用

Objective_c语言的方法调用与其他静态语言类似。但其实有本质上的不同。Objective_c语言在编译时会将方法的调用转换为消息的发送。处理消息的对象和消息处理的方式可以在运行时灵活确定。因此消息的传递是Objective_c语言运行时动态的基础。

```

#import <UIKit/UIKit.h>
#import "AppDelegate.h"
#import <Foundation/Foundation.h>
#import <objc/message.h>

@interface MyObject : NSObject
-(void)hello:(NSString*)name;
@end

@implementation MyObject

-(void)hello:(NSString*)name{
    NSLog(@"HelloWorld:%@",name);
}

@end

int main(int argc, char * argv[]) {
    NSString * appDelegateClassName;
    @autoreleasepool {
        // Setup code that might create autoreleased objects goes here.
        appDelegateClassName = NSStringFromClass([AppDelegate class]);
    }
    
    MyObject *obj = [[MyObject alloc] init];
    [obj hello:@"LIli"];
    ((void(*)(id, SEL,NSString*))objc_msgSend)(obj, @selector(hello:),
                                               @"LIli");
    
    
    return UIApplicationMain(argc, argv, nil, appDelegateClassName);
}

```

控制台输出结果

```
2016-08-23 20:13:23.075112+0800 LCTestDemo[36303:420127] HelloWorld:LIli
2016-08-23 20:13:23.075975+0800 LCTestDemo[36303:420127] HelloWorld:LIli

```

对MyObject类进行实例化，并通过[obj hello]调用方法，在编译时会被转换成((void(*)(id, SEL))objc_msgSend)(obj, @selector(hello))，C语言风格的消息发送函数。
上面调用的objc_msgSend函数用来发送消息，这个函数的第一个参数为消息要发送给的对象，第二个参数为要执行的方法选择器，后面还可以添加任意个的参数，后面添加的参数都会作为方法选择器对应方法中的参数。
### 消息传递过程
在Objective_c中，所有Objective_c类最终都将继承自NSObject类。在NSObject类中定义了一个名为isa的成员变量：

```
@interface NSObject <NSObject> {
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wobjc-interface-ivars"
    Class isa  OBJC_ISA_AVAILABILITY;
#pragma clang diagnostic pop
}
```

isa为Class类型，表示当前对象所属于的类。Class实际上是Objective_c中定义的一个结构体，其中封装了类的基本信息，具体如下

```
struct objc_class {
	//元类指针
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
	//父类
    Class _Nullable super_class                              OBJC2_UNAVAILABLE;
    //类名
    const char * _Nonnull name                               OBJC2_UNAVAILABLE;
    //类的版本
    long version                                             OBJC2_UNAVAILABLE;
    //信息
    long info                                                OBJC2_UNAVAILABLE;
    //内存布局
    long instance_size                                       OBJC2_UNAVAILABLE;
    //变量列表
    struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;
    //函数列表
    struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE;
    //缓存方式
    struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;
    //协议列表
    struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
#endif

```
可以看到，其中封装了类的名字，其父类结构体、类中的变量与函数列表等，消息发送实际上时通过对象的isa指针找到对应的类，在类对应的方法类表中搜索对应签名的函数进行调用，并且消息处理是基于继承链的，向对象发送消息后。首先会从对象所属类的方法类表中寻找对应的方法，如果当前类中没有找到，就向其父类中继续寻找，如果父类中依然没有找到对应的方法，则会继续向上寻找，直到在继承类中找到对应的方法，或者直到选择到基类都没有找到再结束，如果最终没有找到要执行的方法，则Objective_c的默认处理会使程序抛出异常。
在消息的传递过程中，还会发送一些有趣的事情。首先，如果对象对某个消息在整个继承链中都没有找到对应的方法，则之后会调用类的resolveInstanceMethod方法，这个方法的作用是动态处理不能识别的实例方法与之对应的还有一个resolveClassMethod方法，这个方法用来动态处理不能被识别的类方法。
resolveInstanceMethod给开发者提供了一种动态处理未被识别选择器方法。如果我们不对这个方法做额外的处理，则之后会进行消息的转发流程，会调用类的实例方法forwardingTargetForSelector。我们可以通过这个方法返回一个实例对象。当前对象无法处理的消息会被转发给被返回的实例对象。通过forwardingTargetForSelector方法进行的消息转发也被称为直接转发。methodSignatureForSelector和doesNotRecognizeSelector方法可以用来进行简接的消息转发.

Objective_c消息机制的流程图
<img src="/images/posts/Runtime/Runtime.jpeg" > 

### 关于super关键字

super是我们开发中常用到的一个关键字，在重写父类方法时，通常需要使用super关键字调用父类的方法。使用super关键字调用方法实际上也是进行消息的发送。其使用的是objc_megSendSuper函数来发送消息，这个函数的第一个参数为objc_super结果体类型的指针，后面的参数与objc_msgSend函数完成一致。
Objective_c消息机制的本质，我们可以明确，对于任何方法的调用实际上都会转换成消息的发送，影响消息发送的三要素为消息接收者、方法选择器、方法参数。
[self className]在调用时会采用前面介绍的消息发送机制从当前类中找className，当前类中并没有提供className韩素，所以消息会随着继承链向上传递，最终在NSObject类中会找到这个方法。调用【super ClassName】的过程类似，最终在NSObject类中会找到这个方法对于这两条消息来说，其接收者对象是一样的，执行的方法选择器也是一样的，只是寻找方法选择器起始的类不同。因此运行结果是一样的。

### Objective_c的运行时技术
了解了Objective_c的消息机制，运行时就非常容易理解。因此Objective_c中任何方法的调用都是消息的发送，因此可以在程序运行的时候动态的修改消息、改变消息接收的对象、动态添加属性与方法等。 运行时技术最强大的地方在于可以在运行时动态替换实现方法，通过这种Hook技术可以开发iOS运行时的各种插件，也可以对程序运行的过程进行监控。


<br>
转载请注明：[iWolf的博客](http://iWolf.com) » [Ojective_c语言的消息机制与运行时(Runtime)](http://iWolf.com/2016/08/Objective_c语言的消息机制与运行时(Runtime)/)  