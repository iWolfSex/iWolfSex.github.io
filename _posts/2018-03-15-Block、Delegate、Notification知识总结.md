---
layout: post
title: Block、Delegate、Notification知识总结
date: 2018-03-15
tag: iOS
---

从事iOS开发工作4年多了，开发过程中对Block、Delegate和Notification使用了无数次。今天就把这方面的知识做一个梳理。在实际项目中如何选择使用相应的模式来实现通信方式。在各种应用场景中应该注意事项。

###  Block

block 顾名思义就是代码块，将同一逻辑的代码放在一个块，使代码更简洁紧凑，易于阅读。带有局部变量的匿名函数，与C语言中的函数指针类似，可以当着参数进行传值，并且可以没有名字。


1、block定义
```
returnType (^blockName)(parameterTypes) = ^returnType(parameters) {...};
即可以解读
返回值类型(^block变量名)(形参列表) = ^(形参列表) {

};
```
2、block的使用

```
void (^myBlock1)(void);  //无返回值，无参数
void (^myBlock2)(NSObject, int); //无返回值，有参数
NSString* (^myBlock3)(NSString* name, int age); //有返回值和参数，并且在参数类型后面加入了参数名(仅为可读性)
int(^myBlock4)(void) = ^{return 45;};//无参数，有返回值
```
3、block的定义和属性的声明。

为避免使用同类型block时都要编辑大量代码，可以使用typedef 定义,代码如下。

```
typedef int (^MyBlock)(int , int);
@property (nonatomic,copy) MyBlock myBlockOne;
等同于
@property (nonatomic,copy) int (^MyBlock)(int , int);
```

注意：声明block属性时要用copy，只有copy后的Block才会在堆中，栈中的Block的生命周期是和栈绑定的。在声明Block属性时需要确认“在调用Block时另一个线程有没有可能去修改Block？如果确定不会有这种情况发生的话，那么Block属性声明可以用nonatomic。如果会有这种情况发生的话，那么你首先需要声明Block属性为atomic；在非ARC下则需要手动retain一下，否则如果属性被置空，本地变量就成了野指针了，也要记着release。

4、用__block修饰符修饰，改变局部变量的值.
block可以获取获取变量值，但不可以直接赋值。需要改变变量的值时需要用__block.

```
__block int val = 0;
void (^block)(void) = ^{
      val = 1;
}
block();
NSLog(@"val = %d",val); //val的值为1
```

5、 使用__weak关键字，避免循环使用造成内存泄漏

ARC下这样防止：
```
__weak typeof(self) weakSelf = self;
  [yourBlock:^(NSArray *repeatedArray, NSArray *incompleteArray) {
       [weakSelf doSomething];
    }];
```

非ARC
```
__block typeof(self) weakSelf = self;
  [yourBlock:^(NSArray *repeatedArray, NSArray *incompleteArray) {
       [weakSelf doSomething];
    }];
```
6、block的应用。

情景一：作为参数实现回调
block的声明

```
typedef void (^Success)(id responseObject);     // 成功Block
typedef void (^Failure)(NSError *error);        // 失败Blcok

```

block的调用

```
- (void)POST:(NSString *)serviceApi parameters:(NSDictionary *)parameters withToken:(BOOL)token success:(Success)success failure:(Failure)failure;
```

```
- (void)POST:(NSString *)serviceApi isNetCar:(BOOL)isNetCar parameters:(NSDictionary *)parameters  withToken:(BOOL)token  success:(Success)success failure:(Failure)failure {
    
    [self checkAccToken:^{
        
        NSString        *urlString = [NSString stringWithFormat:@"%@%@",isNetCar ? NetHttpService:HttpService,APIService];

        [netManager POST:urlString parameters:[self api:serviceApi andParameter:parameters andToken:token] constructingBodyWithBlock:^(id<AFMultipartFormData>  _Nonnull formData) {
            
        } progress:^(NSProgress * _Nonnull uploadProgress) {
            
        } success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
            
            if (success){
                
                if([[responseObject objectForKey:@"status"] integerValue] == SuccessCode){
                    success([responseObject objectForKey:@"result"]);
                }else{
                    NSDictionary *userInfo = [NSDictionary dictionaryWithObject:[responseObject objectForKey:@"errmsg"]?[responseObject objectForKey:@"errmsg"]:@"未知错误"                                                                      forKey:NSLocalizedDescriptionKey];
                    NSError *error = [NSError errorWithDomain:HttpService code:[[responseObject objectForKey:@"status"] integerValue] userInfo:userInfo];
                    
                    failure(error);
                }
            }
            
        } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
            
            if (failure){

                NSDictionary *userInfo = [NSDictionary dictionaryWithObject:@"网络访问失败"                                                                      forKey:NSLocalizedDescriptionKey];
                error = [NSError errorWithDomain:HttpService code:HttpErrorCode userInfo:userInfo];
                failure(error);
                
            }
            
        }];
        
    } andFail:^(NSError *error) {
        
        if (failure){
            failure(error);
        }
        
    }];
```

block 回调处理

```
+(void)invoiceListWithSuccess:(void(^)(NSArray *resultArray))success withFailure:(Failure)fail{
    [[HttpRequstManager shareManager] NetCarPOST:InvoiceList_API parameters:nil withToken:YES success:^(id responseObject) {
        NSLog(@"responseObject%@",responseObject);
        
        NSArray  *resultArray = [CTInvoiceHistoryModel mj_objectArrayWithKeyValuesArray:responseObject];
        success(resultArray);
        
        
    } failure:^(NSError *error) {
        fail(error);
    }];
}
```

情景二：作为事件实现回调

block声明
```
typedef void(^ClickshareViewBlock)(NSInteger Index);
@property (nonatomic,copy)ClickshareViewBlock clickshareViewBlock;
```

block调用
```
_drawSignCallBackBlock(self.frame.size.height-40,self.frame.size.width-90,_lineArray,[self saveScreen]);
```

block 回调处理
```
```
7、使用block的优点

```
1.一对多的消息传递
2.需先声明Block，调用block，实现block，相比于delegate简直是不要太简洁。
3.能实现方法回调
```

8、使用block的缺点

```
使用时需要注意避免循环使用的问题（使用__weak关键字修饰词可避免）。
```
###  Delegate

Delegate (委托/代理)是 iOS 开发中常用的设计模式，表示将一个对象的部分功能委托给另一个对象处理。

1、声明 Delegate

```
@protocol CTDriverViewDelegate <NSObject>

-(void)cancelCar:(CTMessageContentModel*)messageContentModel;
-(void)callDriverPhone:(CTMessageContentModel*)messageContentModel;

@end

```

```
@property(nonatomic,weak)id<CTDriverViewDelegate> delegate;
注意：3.有非常严格的语法（比如定义delegate不能使用强应用，只能使用弱引用）
```

2、设置 Delegate

```
CTDriverView   *driverView = [[CTDriverView alloc] initWithFrame:CGRectMake(Scale(15), WinSize_Height - 110 -Scale(15), WinSize_Width-Scale(30), 110)];
    driverView.delegate = self;
    [driverView setDriverInfoForMessageContentModel:self.contentModel];
    [self.view addSubview:driverView];
```
3、回调
```
if ([_delegate respondsToSelector:@selector(cancelCar:)]) {
        [self.delegate cancelCar:_messageContentModel];
    }
```
4、回调的处理
```
- (void)cancleOrder {
    [self cancelCar:nil];
}
```

5、Delegate的优点
```
1.能够接收调用的协议方法的返回值。这意味着delegate能够提供反馈信息给controller。（这是利用代理传值的一个使用）
2. 实现一对一的消息传递
3. 在一个类中可以定义定义多个不同的协议，每个协议有不同的delegates
4. 协议中的方法有必须实现的，也可以有不必须实现的，但要有@optional修饰
5. 一个委托里可以有多个回调事件。
6. delegate运行成本低。block成本很高的。block出栈需要将使用的数据从栈内存拷贝到堆内存，当然对象的话就是加计数，使用完或者block置nil后才消除；delegate只是保存了一个对象指针，直接回调，没有额外消耗。相对C的函数指针，只多做了一个查表动作 
```


6、Delegate缺点
```
1.定义代码太多，步骤繁琐
2.只能实现一对一的消息传递
3.当实现跨层传值监听的时候将加大代码的耦合性，并且程序的层次结构将变的混乱。 
4.当对多个对象同时传值响应的时候，委托的易用性将大大降低。
```

### Notification

NSNotification 是iOS中一个调度消息通知的类,采用单例模式设计,在程序中实现传值、回调等地方应用很广。


1.发送通知
```
[[NSNotificationCenter defaultCenter] postNotificationName:RESERVATION_FAIL_NOTICE object:model.netCarModel];
```

2.创建监听者
```
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(reservationFail:) name:RESERVATION_FAIL_NOTICE object:nil];
```

3.接收通知
```
- (void)reservationFail:(NSNotification *)notification {
    
    [self.reservationTimer invalidate];
    self.reservationTimer = nil;
    
    self.startPointView.contentlab.text = @"派车失败";
    self.titlelab.text = @"预约详情";
    self.successView.titlelab.text = @"很抱歉，";
    self.successView.messagelab.text = @"暂时没有空闲车辆，我们客服会联系您。";
    
}
```

4.移除监听者
```
[[NSNotificationCenter defaultCenter] removeObserver:self];
或者
[[NSNotificationCenter defaultCenter]removeObserver:self name:RESERVATION_FAIL_NOTICE object:nil];
```

5、Notification的优点

```
1.不需要写多少代码，实现比较简单。
2.一个对象发出的通知，多个对象能进行反应，一对多的方式实现很简单。
3.controller能够传递context对象（dictionary），context对象携带了关于发送通知的自定义的信息
```

6、Notification的缺点
```
3.在编译期不会检查通知是否能够被观察者正确的处理； 
4.在释放注册的对象时，需要在通知中心取消注册；
5.在调试的时候应用的工作以及控制过程难跟踪；

```

<br>
转载请注明：[iWolf的博客](http://iWolf.com) » [Block、Delegate、Notification知识总结](http://iWolf.com/2018/03/Block、Delegate、Notification知识总结/)  


🈳️