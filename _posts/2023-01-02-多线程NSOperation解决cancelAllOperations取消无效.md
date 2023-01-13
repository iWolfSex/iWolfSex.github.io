---
layout: post
title: 多线程NSOperation使用cancelAllOperations取消无效的解决办法
date: 2023-01-12
tag: iOS
---
### 多线程NSOperation解决cancelAllOperations取消无效

在开发中使用NSOperation的封装性可以方便实现线程的优先级、依赖关系、取消等功能。我在使用NSOperation来实现下载任务时。调用cancelAllOperations后队列里面的数量依旧没有发生变化。在几番搜索下发现如果你取消了一个操作。它不会马上被取消。调用cancelAllOperations相当于把你加入队列中的所有线程都cancel。只有某个时刻在调用main函数中[self isCancelled]函数检查是否被取消。如果此时取消就return。所以要真正的取消队列中的所有任务需要继承NSOperation或者是NSBlockOperation。来重新- (void)main函数。在main函数中实现你的逻辑。然后在每一个退出的地方调用if ([self isCancelled]) return;。这用就可以实现取消全部的任务。

开始调用ancelAllOperations无法取消队列里面任务实现的代码片段

```
    NSBlockOperation * requestOperation = [NSBlockOperation blockOperationWithBlock:^{

        //使用信号量 控制添加到 NSOperationQueue 里面的任务一个一个的执行 (按照先后顺序执行)
        dispatch_semaphore_t sema = dispatch_semaphore_create(0);
        NSString *url = nil;
        if (custom.length>0) {
            url = [KBaseUrl stringByAppendingString:[NSString stringWithFormat:@"custom=%@",custom]];
        }
        if (cmd.length>0) {
            url=[url stringByAppendingString:[NSString stringWithFormat:@"&cmd=%@",cmd]];
        }
        if (par.length>0) {
            url=[url stringByAppendingString:[NSString stringWithFormat:@"&par=%@",par]];
        }
        if (str.length>0) {
            url=[url stringByAppendingString:[NSString stringWithFormat:@"&str=%@",str]];
        }
        YQ_LOG(@"CMD***%@指令请求开始:%@", cmd,url);
        [self.netTool yq_get:url params:@{} cacheType:YQNetCacheTypeNone successed:^(id  _Nonnull object, BOOL isCache) {
            YQ_LOG(@"CMD指令请求返回数据%@:%@",url,[self yq_convertToJsonData:object]);
            if (![object isKindOfClass:[NSDictionary class]]){
                YQ_LOG(@"CMD***%@指令请求失败", cmd);
                YQ_BLOCK(callback,nil,NO)
                dispatch_semaphore_signal(sema);
                return;
            }
            if(![cmd isEqualToString:@"3014"]){//3014 指令没有Status状态值
                if ([object[@"Status"] intValue] != 0) {
                    YQ_LOG(@"CMD***%@指令请求失败", cmd);
                    YQ_BLOCK(callback,nil,NO)
                    dispatch_semaphore_signal(sema);
                    return;
                }
            }
            YQ_LOG(@"CMD***%@指令请求成功", cmd);
            YQ_BLOCK(callback,object,YES);
            dispatch_semaphore_signal(sema);

        } failed:^(NSError * _Nonnull error) {
            YQ_LOG(@"CMD***%@指令请求失败:%@", cmd,error);
            YQ_BLOCK(callback,nil,NO);
            dispatch_semaphore_signal(sema);

        }];
        dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);
    }];
    //如果最后的参数是NO，那么会不等待上面的任务直接执行最后任务，如果是YES会等待上面执行完毕。
    [[YQOperationQueueManager shareYQOperationQueueManager] yq_addOperations:@[requestOperation] waitUntilFinished:NO];
    [requestOperation setCompletionBlock:^{
        YQ_LOG(@"CMD指令请求队列数量%lu", (unsigned long)[[YQOperationQueueManager shareYQOperationQueueManager] yq_operationCount]);
    }];
```
修改后调用ancelAllOperations可以取消队列里面任务实现的代码片段

```
- (void)main {
    if ([self isCancelled]) return;
    //使用信号量 控制添加到 NSOperationQueue 里面的任务一个一个的执行 (按照先后顺序执行)
    dispatch_semaphore_t sema = dispatch_semaphore_create(0);
    NSString *url = nil;
    if (self.custom.length>0) {
        url = [KBaseUrl stringByAppendingString:[NSString stringWithFormat:@"custom=%@",self.custom]];
    }
    if (self.cmd.length>0) {
        url=[url stringByAppendingString:[NSString stringWithFormat:@"&cmd=%@",self.cmd]];
    }
    if (self.par.length>0) {
        url=[url stringByAppendingString:[NSString stringWithFormat:@"&par=%@",self.par]];
    }
    if (self.str.length>0) {
        url=[url stringByAppendingString:[NSString stringWithFormat:@"&str=%@",self.str]];
    }
    YQ_LOG(@"CMD***%@指令请求开始:%@", self.cmd,url);
    [self.netTool yq_get:url params:@{} cacheType:YQNetCacheTypeNone successed:^(id  _Nonnull object, BOOL isCache) {
        if ([self isCancelled]) return;
        YQ_LOG(@"CMD指令请求返回数据%@:%@",url,[NSString yq_convertToJsonData:object]);
        if (![object isKindOfClass:[NSDictionary class]]){
            YQ_LOG(@"CMD***%@指令请求失败",  self.cmd);
            YQ_BLOCK(self.callback,nil,NO)
            dispatch_semaphore_signal(sema);
            return;
        }
        if(![self.cmd isEqualToString:@"3014"]){//3014 指令没有Status状态值
            if ([object[@"Status"] intValue] != 0) {
                YQ_LOG(@"CMD***%@指令请求失败", self.cmd);
                YQ_BLOCK(self.callback,nil,NO)
                dispatch_semaphore_signal(sema);
                return;
            }
        }
        YQ_LOG(@"CMD***%@指令请求成功", self.cmd);
        YQ_BLOCK(self.callback,object,YES);
        dispatch_semaphore_signal(sema);
        
    } failed:^(NSError * _Nonnull error) {
        if ([self isCancelled]) return;
        YQ_LOG(@"CMD***%@指令请求失败:%@", self.cmd,error);
        YQ_BLOCK(self.callback,nil,NO);
        dispatch_semaphore_signal(sema);
       
    }];
    dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);
    
}

```

<br>
转载请注明：[iWolf的博客](https://iwolfsex.github.io/) » [多线程NSOperation解决cancelAllOperations取消无效](http://iWolf.com/2023/01/多线程NSOperation解决cancelAllOperations取消无效/)  
