---
layout: post
title: objiective_c属性的声明规则
date: 2016-01-11
tag: iOS
---

### @property 

@property = ivar + getter + setter;
“属性” (property)有两大概念：ivar（实例变量）、存取方法（access method ＝ getter + setter）。
原子性：nonatomic、atomic
atomic
是默认的
会保证 CPU 能在别的线程来访问这个属性之前，先执行完当前流程
速度不快，因为要保证操作整体完成
nonatomic
不是默认的
更快
线程不安全
如有两个线程访问同一个属性，会出现无法预料的结果
如果该对象无需考虑多线程的情况，这个属性会让编译器少生成一些互斥代码，可以提高效率。

### 读写权限：readwrite(读写)、readonly (只读)

readwrite 是可读可写特性；需要生成getter方法和setter方法时（补充：默认属性，将生成不带额外参数的getter和setter方法（setter方法只有一个参数））
readonly 是只读特性  只会生成getter方法 不会生成setter方法 ;不希望属性在类外改变
内存管理：assign、weak 、retain 、strong、copy、unsafe_unretained
assign与retain
>* 1、assign: 简单赋值，不更改索引计数；
>* 2、assign的情况：NSString *newPt = [pt assing]; 
此时newPt和pt完全相同 地址都是0Xaaaa 内容为0X1111 即newPt只是pt的别名，对任何一个操作就等于对另一个操作， 因此retainCount不需要增加；
>* 3、assign就是直接赋值；
>* 4、retain使用了引用计数，retain引起引用计数加1, release引起引用计数减1，当引用计数为0时，dealloc函数被调用，内存被回收；

### weak and strong (强引用和弱引用的区别)：
>*  1、 weak 和 strong 属性只有在你打开ARC时才会被要求使用，这时你是不能使用retain release autorelease 操作的，因为ARC会自动为你做好这些操作，但是你需要在对象属性上使用weak 和strong,其中strong就相当于retain属性，而weak相当于assign。
>* 2、只有一种情况你需要使用weak（默认是strong），就是为了避免retain cycles（就是父类中含有子类{父类retain了子类}，子类中又调用了父类{子类又retain了父类}，这样都无法release）    
>* 3、声明为weak的指针，指针指向的地址一旦被释放，这些指针都将被赋值为nil。这样的好处能有效的防止野指针。 

什么情况下用weak
>* 1、ARC情况下有可能出现循环引用的时候 。可以使用weak 。比如delegate
>* 2、IBoutlet:拖出来的控件属性一般也是weak。当然也可以用strong。

###  weak与assign 的不同
>* 1、weak为这种属性设置新值时，设置方法既不保留新值，也不释放旧值。然而在属性所指的对象遭到摧毁时，属性值也会清空。
assign基础数据的简单赋值。
>* 2、assign 可以用非 OC 对象,而 weak 必须用于 OC 对象。


怎么用或者什么情况用copy关键字
>* 1、NSString、NSArray、NSDictionary 等等经常使用copy关键字，是因为他们有对应的可变类型：NSMutableString、NSMutableArray、NSMutableDictionary；
>* 2、block 也经常使用 copy 关键字。
注意 1：只有copy后的Block才会在堆中，栈中的Block的生命周期是和栈绑定的。
>* 2、另一个需要注意的问题是关于线程安全。


```
    在声明Block属性时需要确认“在调用Block时另一个线程有没有可能去修改Block？如果确定不会有这种情况发生的话，那么Block属性声明可以用nonatomic。如果会有这种情况发生的话，那么你首先需要声明Block属性为atomic；在非ARC下则需要手动retain一下，否则如果属性被置空，本地变量就成了野指针了，也要记着release。
```

### copy 与 retain的区别
>* 1、copy其实是建立了一个相同的对象，而retain不是
>* 2、copy是内容拷贝，retain是指针拷贝；
>* 3、copy是内容的拷贝 ,对于像NSString,的确是这样，但是如果copy的是一个NSArray呢?这时只是copy了指向array中相对应元素的指针.这便是所谓的"浅复制”.

### @synthesize和@dynamic分别有什么作用？
>*  1、 @property有两个对应的词，一个是 @synthesize，一个是 @dynamic。如果 @synthesize和 @dynamic都没写，那么默认的就是@syntheszie var = _var;
>* 2、 @synthesize 的语义是如果你没有手动实现 setter 方法和 getter 方法，那么编译器会自动为你加上这两个方法。
>* 3、 @dynamic 告诉编译器：属性的 setter 与 getter 方法由用户自己实现，不自动生成。（当然对于 readonly 的属性只需提供 getter 即可）。假如一个属性被声明为 @dynamic var，然后你没有提供 @setter方法和 @getter 方法，编译的时候没问题，但是当程序运行到 instance.var = someVar，由于缺 setter 方法会导致程序崩溃；或者当运行到 someVar = var 时，由于缺 getter 方法同样会导致崩溃。编译时没问题，运行时才执行相应的方法，这就是所谓的动态绑定。

### __block和__weak修饰符的区别

>* 1、__block不管是ARC还是MRC模式下都可以使用，可以修饰对象，还可以修饰基本数据类型。 
>* 2、__weak只能在ARC模式下使用，也只能修饰对象（NSString），不能修饰基本数据类型（int）。 
>* 3、__block对象可以在block中被重新赋值，__weak不可以。 
GCD里面用 __weak 防止内存释放不了，循环引用。



<br>
转载请注明：[iWolf的博客](http://iWolf.com) » [objiective_c属性的声明规则](http://iWolf.com/2016/01/objiective_c属性的声明规则/)  


