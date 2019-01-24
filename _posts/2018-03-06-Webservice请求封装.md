---
layout: post
title: iOS关于Webservice请求封装
date: 2018-03-06
tag: iOS
---

在[计算机网络基础知识总结](http://iWolf.com/2018/02/计算机网络基础知识总结/) 这篇中提到过WebService。什么是WebService？WebService:soap请求 (Simple Object Access Protocol，简单对象访问协议) 是HTTP POST的一个专用版本，遵循一种特殊的xml消息格式Content-type设置为: text/xml任何数据都可以xml化。生成的SOAP请求会被嵌入在一个HTTP POST请求中.
SOAP简单的理解，就是这样的一个开放协议SOAP=RPC+HTTP+XML：采用HTTP作为底层通讯协议；RPC作为一致性的调用途径，ＸＭＬ作为数据传送的格式，允许服务提供者和服务客户经过防火墙在INTERNET进行通讯交互。
SOAP协议则定义了怎么把一个对象变成XML文本
这篇文章我们重点不深究什么是WebService。我重点讲解如何通过iOS的NSURLConnection实现数据请求。为了避免NSURLConnection的协议函数在每一个VC都重复写。所以我把这几个函数提出来封装了一下。也就是RMCQNSXMLParser这个类。这个类里面有两个回调函数对应亲求成功和失败。代码如下。

###  自己封装NSURLConnection网络请求调用webService接口

头文件

```

//
//  RMCQNSXMLParser.h
//  CMSMobile
//
//  Created by LiuChao on 13-11-27.
//
//

#import <Foundation/Foundation.h>
@protocol RMCQNSXMLParserDelegate;
@interface RMCQNSXMLParser : NSObject<NSXMLParserDelegate>{
    NSMutableString*result;
    NSString*aElementName;
    NSMutableData *recvData;
    BOOL recordResults;
    NSXMLParser *xmlParse;
    id<RMCQNSXMLParserDelegate>delegate;
}
@property(nonatomic,retain)NSMutableString*result;
@property(nonatomic,retain)NSString*aElementName;
@property(nonatomic,retain)NSMutableData *recvData;
@property (nonatomic,retain)NSXMLParser *xmlParse;
@property(nonatomic,assign)id<RMCQNSXMLParserDelegate>delegate;
-(id)initWithElementName:(NSString*)aElementName;
@end
@protocol RMCQNSXMLParserDelegate
-(void)getDownloadData:(NSMutableString*)aResult addElementName:(NSString*)aElementName;
-(void)netRequestFailed:(NSURLConnection *)request didRequestError:(NSError *)error;
@end


```

源文件

```

//
//  RMCQNSXMLParser.m
//  CMSMobile
//
//  Created by LiuChao on 13-11-27.
//
//

#import "RMCQNSXMLParser.h"

@implementation RMCQNSXMLParser
@synthesize result;
@synthesize aElementName;
@synthesize delegate;
@synthesize recvData;
@synthesize xmlParse;
-(id)initWithElementName:(NSString*)elementName{
    if (self=[super init]) {
        self.aElementName=elementName;
    }
    return self;
}
-(void)dealloc{
    [aElementName release];
    [recvData release];
    [xmlParse release];
    [super dealloc];
}
-(void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response
{
    if (self.recvData == nil) {
        self.recvData = [[NSMutableData data] retain];
    }
    [recvData setLength:0];
    NSLog(@"1");
}

-(void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data
{
    recordResults = NO;
    [recvData appendData:data];
    NSLog(@"2");
}

-(void)connectionDidFinishLoading:(NSURLConnection *)connection
{
    NSLog(@"3-recv len = %d",[recvData length]);
    NSString *theXML = [[NSString alloc] initWithBytes: [recvData mutableBytes] length:[recvData length] encoding:NSUTF8StringEncoding];
    [theXML release];
    
    //重新加載xmlParser
    if( xmlParse )
    {
        [xmlParse release];
    }
    
    xmlParse = [[NSXMLParser alloc] initWithData: recvData];
    [xmlParse setDelegate:self];
    [xmlParse setShouldResolveExternalEntities: YES];
    [xmlParse parse];
    
    [connection release];
}

-(void)parser:(NSXMLParser *)parser didStartElement:(NSString *)elementName namespaceURI:(NSString *)namespaceURI qualifiedName:(NSString *)qName attributes:(NSDictionary *)attributeDict
{
    NSLog(@"4");
    if ([elementName isEqualToString:aElementName]) {
        if (!result) {
            result=[[NSMutableString alloc]init];
        }
        recordResults =YES;
    }
}

-(void)parser:(NSXMLParser *)parser foundCharacters:(NSString *)string
{
    NSLog(@"5");
    if (recordResults) {
        [result appendString:string];
    }
}


-(void)parser:(NSXMLParser *)parser didEndElement:(NSString *)elementName namespaceURI:(NSString *)namespaceURI qualifiedName:(NSString *)qName
{
    NSLog(@"6");
    if ([elementName isEqualToString:self.aElementName]) {
        NSLog(@"%@",result);
        [delegate getDownloadData:result addElementName:self.aElementName];
        [result release];
        result=nil;
    }

}
-(void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error
{
    if ([self.aElementName isEqualToString:self.aElementName]) {
        [delegate netRequestFailed:connection didRequestError:error];
    }
    [connection release];
    [recvData release];
}
@end

```

然后就是对下载的请求进行封装。一个下载请求对应一个方法。

头文件

```

//
//  RMCQWebserviceEntity.h
//  RMCQMobile
//
//  Created by  on 11-12-23.
//  Copyright (c) 2011年 __MyCompanyName__. All rights reserved.
//

#import <Foundation/Foundation.h>

@interface RMCQWebserviceEntity : NSObject
{
    NSString *_server;
    NSString *_username;
    NSString *_passwrod;
    
    id delegate;
}

@property (nonatomic,retain)NSString *server;
@property (nonatomic,retain)NSString *username;
@property (nonatomic,retain)NSString *password;
@property (nonatomic,retain)id       delegate;

-(id)initWithServer:(NSString*)serverName USER:(NSString*)name PWD:(NSString*)password;
-(BOOL)login;
-(BOOL)addLogInfo:(NSString*)logType Device:(NSString*)deviceName Channels:(NSArray*)channelAarry Directions:(NSString*)direction

@end

```

源文件

```

//
//  RMCQWebserviceEntity.m
//  RMCQMobile
//
//  Created by  on 11-12-23.
//  Copyright (c) 2011年 __MyCompanyName__. All rights reserved.
//

#import "RMCQWebserviceEntity.h"

@implementation RMCQWebserviceEntity

@synthesize server=_server;
@synthesize username=_username;
@synthesize password=_passwrod;
@synthesize delegate;

-(id)initWithServer:(NSString*)serverName USER:(NSString*)name PWD:(NSString*)password
{
    self = [super init];
    self.server = serverName;
    self.username = name;
    self.password = password;
    
    return self;
}

-(BOOL)login
{
    //封装soap请求消息
    NSString *soapMessage = [NSString stringWithFormat:
                             @"<?xml version=\"1.0\" encoding=\"utf-8\"?>\n"
                             "<soap:Envelope xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\" xmlns:xsd=\"http://www.w3.org/2001/XMLSchema\" xmlns:soap=\"http://schemas.xmlsoap.org/soap/envelope/\">\n"
                             "<soap:Body>\n"
                             "<Login xmlns=\"cq-video.cn\">\n"
                             "<_userName>%@</_userName>\n"
                             "<_newPassword>%@</_newPassword>\n"
                             "</Login>\n"
                             "</soap:Body>\n"
                             "</soap:Envelope>\n",self.username,self.password
                             ];
    NSLog(@"%@",soapMessage);
    
    //请求发送到的路径
    NSString *strUrl = [NSString stringWithFormat:@"%@/cms_service.asmx",self.server];
    NSURL *url = [NSURL URLWithString:strUrl];
    NSMutableURLRequest *theRequest = [NSMutableURLRequest requestWithURL:url];
    [theRequest setTimeoutInterval:10];
    NSString *msgLength = [NSString stringWithFormat:@"%d", [soapMessage length]];
    
    NSString* strCommand = [NSString stringWithFormat:@"cq-video.cn/Login"];
    [theRequest addValue: @"text/xml; charset=utf-8" forHTTPHeaderField:@"Content-Type"];
    [theRequest addValue: strCommand forHTTPHeaderField:@"SOAPAction"];
    
    [theRequest addValue: msgLength forHTTPHeaderField:@"Content-Length"];
    [theRequest setHTTPMethod:@"POST"];
    [theRequest setHTTPBody: [soapMessage dataUsingEncoding:NSUTF8StringEncoding]];
    
    //请求
    NSURLConnection *theConnection = [[NSURLConnection alloc] initWithRequest:theRequest delegate:self.delegate];
    
    //如果连接已经建好，则初始化data
    if( theConnection )
    {
        NSLog(@"the connect is ok!!");
        return YES;
    }
    else
    {
        NSLog(@"theConnection is NULL");
        return NO;
    }
}

-(BOOL)addLogInfo:(NSString*)logType Device:(NSString*)deviceName Channels:(NSArray*)channelAarry Directions:(NSString*)direction {
    //封装soap请求消息
    NSString*logContent;
    int channel=0;
    NSString* channels=@"0";
    if ([channelAarry count]>0) {
       channels=[channelAarry objectAtIndex:0];
        for (int i=1; i<[channelAarry count]; i++) {
            channels=[NSString stringWithFormat:@"%@,%@",channels,[channelAarry objectAtIndex:i]];
        }
    }
    if ([logType isEqualToString:@"1000"]) {
        logContent=@"";
        deviceName=@"";
    }else if ([logType isEqualToString:@"1001"]){
        logContent=@"";
        deviceName=@"";
    }else if ([logType isEqualToString:@"1002"]){
        logContent=[NSString stringWithFormat:@"%@:%@",NSLocalizedString(@"Channel",nil),channels];
        deviceName=[NSString stringWithFormat:@"%@:%@",NSLocalizedString(@"dns",nil),deviceName];
    }else if ([logType isEqualToString:@"1003"]){
        logContent=[NSString stringWithFormat:@"%@:%@",NSLocalizedString(@"Channel",nil),channels];
        deviceName=[NSString stringWithFormat:@"%@:%@",NSLocalizedString(@"dns",nil),deviceName];
    }else if ([logType isEqualToString:@"1004"]){
        logContent=[NSString stringWithFormat:@"%@:%@",NSLocalizedString(@"Channel",nil),channels];
        deviceName=[NSString stringWithFormat:@"%@:%@",NSLocalizedString(@"dns",nil),deviceName];
    }else if ([logType isEqualToString:@"1005"]){

        logContent=[NSString stringWithFormat:@"%@:%@;%@",NSLocalizedString(@"Channel",nil),channels,direction];
        deviceName=[NSString stringWithFormat:@"%@:%@",NSLocalizedString(@"dns",nil),deviceName];
    }
    
    NSDate* date = [NSDate date];
    NSDateFormatter* formatter = [[[NSDateFormatter alloc] init] autorelease];
    [formatter setDateFormat:@"yyyy-MM-dd HH:mm:ss"];
    NSString* str = [formatter stringFromDate:date];
    
    NSString *soapMessage = [NSString stringWithFormat:
                             @"<?xml version=\"1.0\" encoding=\"utf-8\"?>\n"
                             "<soap:Envelope xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\" xmlns:xsd=\"http://www.w3.org/2001/XMLSchema\" xmlns:soap=\"http://schemas.xmlsoap.org/soap/envelope/\">\n"
                             "<soap:Body>\n"
                             "<addLogInfo xmlns=\"cq-video.cn\">\n"
                             "<logType>%@</logType>\n"
                             "<logContent>%@</logContent>\n"
                             "<channelIndex>%d</channelIndex>\n"
                             "<logTime>%@</logTime>\n"
                             "<remark>%@</remark>\n"
                             "<username>%@</username>\n"
                             "<userpassword>%@</userpassword>\n"
                             "</addLogInfo>\n"
                             "</soap:Body>\n"
                             "</soap:Envelope>\n",logType,logContent,channel,str,deviceName,self.username,self.password
                             ];
    NSLog(@"%@",soapMessage);
    
    //请求发送到的路径
    NSString *strUrl = [NSString stringWithFormat:@"%@/cms_service.asmx",self.server];
    NSURL *url = [NSURL URLWithString:strUrl];
    NSMutableURLRequest *theRequest = [NSMutableURLRequest requestWithURL:url];
    [theRequest setTimeoutInterval:10];
    NSString *msgLength = [NSString stringWithFormat:@"%d", [soapMessage length]];
    
    NSString* strCommand = [NSString stringWithFormat:@"cq-video.cn/addLogInfo"];
    [theRequest addValue: @"text/xml; charset=utf-8" forHTTPHeaderField:@"Content-Type"];
    [theRequest addValue: strCommand forHTTPHeaderField:@"SOAPAction"];
    
    [theRequest addValue: msgLength forHTTPHeaderField:@"Content-Length"];
    [theRequest setHTTPMethod:@"POST"];
    [theRequest setHTTPBody: [soapMessage dataUsingEncoding:NSUTF8StringEncoding]];
    
    //请求
    NSURLConnection *theConnection = [[NSURLConnection alloc] initWithRequest:theRequest delegate:self.delegate];
    
    //如果连接已经建好，则初始化data
    if( theConnection )
    {
        NSLog(@"the connect is ok!!");
        return YES;
    }
    else
    {
        NSLog(@"theConnection is NULL");
        return NO;
    }
    
}
@end


```



然后就是如何调用相应的请求方法

```
RMCQNSXMLParser*carInfo=[[RMCQNSXMLParser alloc]initWithElementName:@"GetTerminalInfoResult"];
    carInfo.delegate=self;
     if (carInfo) {
        
        [RMCQWebserviceManager shareWebservice].currentWebservice.delegate=carInfo;
        [[RMCQWebserviceManager shareWebservice].currentWebservice getCarInfo]
        [carInfo release];
    }

```


请求成功后的协议函数回调，对数据进行处理。

```
-(void)getDownloadData:(NSMutableString *)aResult addElementName:(NSString *)aElementName{
    if ([aElementName isEqualToString:@"GetTerminalInfoResult"]) {//5
        NSLog(@"****************1");
        [RMCQWebserviceManager shareWebservice].carList = [[NSMutableArray alloc]init];
        [RMCQWebserviceManager shareWebservice].carDict = [[NSMutableDictionary alloc]init];
        [RMCQWebserviceManager shareWebservice].statusDict = [[NSMutableDictionary alloc]init];
        NSLog(@"%d",[[RMCQWebserviceManager shareWebservice].carList retainCount]);
        NSString* serverWebIP = [[RMCQWebserviceManager shareWebservice].currentWebservice.server substringFromIndex:7];
        NSArray *temparray = [serverWebIP componentsSeparatedByString:@":"];
        NSString* strReplaceIP = [temparray objectAtIndex:0];
        NSArray *returnArray = [aResult componentsSeparatedByString:@";"];
        
        for (int i=1; i<[returnArray count]-1; i++) {
            NSString* carstring = [returnArray objectAtIndex:i];
            NSArray *carArray = [carstring componentsSeparatedByString:@","];
            RMCQDeviceEntity* device = [[RMCQDeviceEntity alloc]init];
            device.deviceIP = [carArray objectAtIndex:0];
            device.deviceName = [carArray objectAtIndex:1];
            NSString* transIP = [carArray objectAtIndex:2];
            if (transIP == nil || [transIP length]<3) {
                device.serverIP = strReplaceIP;
            }else{
                device.serverIP = transIP;
            }
            
            device.serverPort = [NSNumber numberWithInt:[[carArray objectAtIndex:3] intValue]];
            device.deviceType = [NSNumber numberWithInt:[[carArray objectAtIndex:4] intValue]];
            device.channel = [NSNumber numberWithInt:[[carArray objectAtIndex:5] intValue]];
            
            [[RMCQWebserviceManager shareWebservice].carList addObject:device];
            [[RMCQWebserviceManager shareWebservice].carDict setObject:device forKey:device.deviceName];
            
            [device release];
        }
    }
}

```

请求失败后的协议函数回调。

```

-(void)netRequestFailed:(NSURLConnection *)request didRequestError:(NSError *)error{
   
}

```

###  AFN也实现Soap请求webService接口

当然AFN也实现Soap请求webService接口的封装。我们大可不必自己去实现SURLConnection这些方法。我在网上找了一段代码示例如下。会更加简单便捷

```
+ (void)SOAPData:(NSString *)url soapBody:(NSString *)soapBody success:(void (^)(id responseObject))success failure:(void(^)(NSError *error))failure {
     
    NSString *soapStr = [NSString stringWithFormat:
                         @"<?xml version=\"1.0\" encoding=\"utf-8\"?>\
                         <soap:Envelope xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\" xmlns:xsd=\"http://www.w3.org/2001/XMLSchema\"\
                         xmlns:soap=\"http://schemas.xmlsoap.org/soap/envelope/\">\
                         <soap:Header>\
                         </soap:Header>\
                         <soap:Body>%@</soap:Body>\
                         </soap:Envelope>",soapBody];
     
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
    manager.responseSerializer = [AFXMLParserResponseSerializer serializer];
     
    // 设置请求超时时间
    manager.requestSerializer.timeoutInterval = 30;
     
    // 返回NSData
    manager.responseSerializer = [AFHTTPResponseSerializer serializer];
     
    // 设置请求头，也可以不设置
    [manager.requestSerializer setValue:@"application/soap+xml; charset=utf-8" forHTTPHeaderField:@"Content-Type"];
    [manager.requestSerializer setValue:[NSString stringWithFormat:@"%zd", soapStr.length] forHTTPHeaderField:@"Content-Length"];
 
    // 设置HTTPBody
    [manager.requestSerializer setQueryStringSerializationWithBlock:^NSString *(NSURLRequest *request, NSDictionary *parameters, NSError *__autoreleasing *error) {
        return soapStr;
    }];
     
    [manager POST:url parameters:soapStr success:^(NSURLSessionDataTask * _Nonnull task, id  _Nonnull responseObject) {
        // 把返回的二进制数据转为字符串
         NSString *result = [[NSString alloc] initWithData:responseObject encoding:NSUTF8StringEncoding];
         
        // 利用正则表达式取出<return></return>之间的字符串
        NSRegularExpression *regular = [[NSRegularExpression alloc] initWithPattern:@"(?<=return\\>).*(?=</return)" options:NSRegularExpressionCaseInsensitive error:nil];
         
        NSDictionary *dict = [NSDictionary dictionary];
        for (NSTextCheckingResult *checkingResult in [regular matchesInString:result options:0 range:NSMakeRange(0, result.length)]) {
             
            // 得到字典
            dict = [NSJSONSerialization JSONObjectWithData:[[result substringWithRange:checkingResult.range] dataUsingEncoding:NSUTF8StringEncoding] options:NSJSONReadingMutableLeaves error:nil];
        }
        // 请求成功并且结果有值把结果传出去
        if (success && dict) {
            success(dict);
        }
         
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        if (failure) {
            failure(error);
        }
    }];
}
```

<br>
转载请注明：[iWolf的博客](http://iWolf.com) » [iOS关于Webservice请求封装](http://iWolf.com/2018/03/iOS关于Webservice请求封装/)  


