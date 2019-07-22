---
layout: post
title:  "iOS客户端的微信支付接入"
tags: [iOS]
---



对于一个iOS的APP，如果有一些虚拟的商品或者服务需要通过在线支付来收费的话，有两种主流的选择：1.调起支付宝客户端；2.调起微信支付。本文将记录客户端里面，如何支持微信支付。



### 文档阅读

实际上，从代码的角度，调起支付APP就是把一些关键的参数通过一定方式打包成为一个订单，然后发送到支付平台的服务器。所以，只要搞清楚了参数设置，搞清楚了每个支付平台的SDK里面一些关键API的使用，基本上就可以很简单的支持支付。

首先。我们要仔细阅读一下微信SDK的[开发文档](http://pay.weixin.qq.com/wiki/doc/api/app.php?chapter=8_1)，了解一下整个支付的大概流程。然后根据提示，把相应的SDK下载下来，所谓的SDK，也就是一个链接库和两个头文件。下载完毕，需要把SDK导入到工程里面，并且配置一下工程。因为[开发者文档](http://pay.weixin.qq.com/wiki/doc/api/app.php?chapter=8_5)已经有详细描述，这里就不再复述。



##### 核心代码段

从文档看到，调起微信支付其实最核心的是一下这么一段

```
PayReq *request = [[[PayReq alloc] init] autorelease];
request.partnerId = @"10000100";
request.prepayId= @"1101000000140415649af9fc314aa427";
request.package = @"Sign=WXPay";
request.nonceStr= @"a462b76e7436e98e0ed6e13c64b4fd1c";
request.timeStamp= @"1397527777";
request.sign= @"582282D72DD2B03AD892830965F428CB16E7A256";
[WXApi sendReq:request];
```

这里的范例是一段hardcode，真正使用的时候，参数都需要自行传入。



##### 示例代码

为了搞清楚如何使用API，我们可以下载Sample代码。不过，这个sample代码应该是微信的实习生写的，而且应该是一个对于C++比较熟悉，对于ObjectC比较陌生的实习生。。。代码风格可以看出很多东西哈。。所以这个sample读起来总觉得有点奇怪。当然，写出这个demo也是需要不错的水平，因为这个sample不仅仅是一些API的调用，还包括了一些算法的实现，MD5之类的。

看懂了sample之后，一般可以自己重构一下，成为自己APP里面的一个Manager类。
我是在2015.5.23下载的微信Sampel代码，里面包括有

```objective-c
- ApiXml.h
- ApiXml.m
- WXUtil.h
- WXUtil.m
- payRequestHandler.h
- payRequestHandler.m
```

如果比较看重命名规范(也就是强迫症)的OC程序猿，就会觉得这个payRequestHandler类非常别扭，与一般对类的命名规则不一致，而且handler这个词更偏向于c++风格。我就以这个类为原型，重构了一下，并改装成一个传参的方法，供自己的APP调用。APP里面卖商品，一般就是商品名字，价格两个关键参数。所以这个重构的方法也只是提供这两个参数的接口。

ApiXml.h && ApiXml.m && WXUtil.h && WXUtil.m不变



###### WechatPayManager.h

```objective-c
//
//  WechatPayManager.h
//
//  Created by HuangCharlie on 5/24/15.
//
//

#import <Foundation/Foundation.h>
#import "WXUtil.h"
#import "ApiXml.h"
#import "WXApi.h"

// 账号帐户资料
// 更改商户把相关参数后可测试
#define APP_ID          @"wx@@@@@@@@@@@@@@@@"        //APPID
#define APP_SECRET      @""                          //appsecret,看起来好像没用
//商户号，填写商户对应参数
#define MCH_ID          @"@@@@@@@@@@"
//商户API密钥，填写相应参数
#define PARTNER_ID      @"12345678901234567890123456789012"
//支付结果回调页面
#define NOTIFY_URL      @"http://wxpay.weixin.qq.com/pub_v2/pay/notify.v2.php"
//获取服务器端支付数据地址（商户自定义）(在小吉这里，签名算法直接放在APP端，故不需要自定义)
#define SP_URL          @"http://wxpay.weixin.qq.com/pub_v2/app/app_pay.php"


@interface WechatPayManager : NSObject
{
}


//预支付网关url地址
@property (nonatomic,strong) NSString* payUrl;

//debug信息
@property (nonatomic,strong) NSMutableString *debugInfo;
@property (nonatomic,assign) NSInteger lastErrCode;//返回的错误码

//商户关键信息
@property (nonatomic,strong) NSString *appId,*mchId,*spKey;


//初始化函数
-(id)initWithAppID:(NSString*)appID
             mchID:(NSString*)mchID
             spKey:(NSString*)key;

//获取当前的debug信息
-(NSString *) getDebugInfo;

//获取预支付订单信息（核心是一个prepayID）
- (NSMutableDictionary*)getPrepayWithOrderName:(NSString*)name
                                         price:(NSString*)price
                                        device:(NSString*)device;

@end
```



###### WechatPayManager.m

```objective-c
//
//  WechatPayManager.m
//
//  Created by HuangCharlie on 5/24/15.
//
//

#import "WechatPayManager.h"

@implementation WechatPayManager

//初始化函数
-(id)initWithAppID:(NSString*)appID mchID:(NSString*)mchID spKey:(NSString*)key
{
    self = [super init];
    if(self)
    {
        //初始化私有参数，主要是一些和商户有关的参数
        self.payUrl    = @"https://api.mch.weixin.qq.com/pay/unifiedorder";
        if (self.debugInfo == nil){
            self.debugInfo  = [NSMutableString string];
        }
        [self.debugInfo setString:@""];
        self.appId = appID;//微信分配给商户的appID
        self.mchId = mchID;//
        self.spKey = key;//商户的密钥
    }
    return self;
}

//获取debug信息
-(NSString*) getDebugInfo
{
    NSString *res = [NSString stringWithString:self.debugInfo];
    [self.debugInfo setString:@""];
    return res;
}

//创建package签名
-(NSString*) createMd5Sign:(NSMutableDictionary*)dict
{
    NSMutableString *contentString  =[NSMutableString string];
    NSArray *keys = [dict allKeys];
    //按字母顺序排序
    NSArray *sortedArray = [keys sortedArrayUsingComparator:^NSComparisonResult(id obj1, id obj2) {
        return [obj1 compare:obj2 options:NSNumericSearch];
    }];
    //拼接字符串
    for (NSString *categoryId in sortedArray) {
        if (   ![[dict objectForKey:categoryId] isEqualToString:@""]
            && ![categoryId isEqualToString:@"sign"]
            && ![categoryId isEqualToString:@"key"]
            )
        {
            [contentString appendFormat:@"%@=%@&", categoryId, [dict objectForKey:categoryId]];
        }
       
    }
    //添加key字段
    [contentString appendFormat:@"key=%@", self.spKey];
    //得到MD5 sign签名
    NSString *md5Sign =[WXUtil md5:contentString];
   
    //输出Debug Info
    [self.debugInfo appendFormat:@"MD5签名字符串：\n%@\n\n",contentString];
   
    return md5Sign;
}

//获取package带参数的签名包
-(NSString *)genPackage:(NSMutableDictionary*)packageParams
{
    NSString *sign;
    NSMutableString *reqPars=[NSMutableString string];
    //生成签名
    sign        = [self createMd5Sign:packageParams];
    //生成xml的package
    NSArray *keys = [packageParams allKeys];
    [reqPars appendString:@"<xml>\n"];
    for (NSString *categoryId in keys) {
        [reqPars appendFormat:@"<%@>%@</%@>\n", categoryId, [packageParams objectForKey:categoryId],categoryId];
    }
    [reqPars appendFormat:@"<sign>%@</sign>\n</xml>", sign];
   
    return [NSString stringWithString:reqPars];
}

//提交预支付
-(NSString *)sendPrepay:(NSMutableDictionary *)prePayParams
{
    NSString *prepayid = nil;
   
    //获取提交支付
    NSString *send      = [self genPackage:prePayParams];
   
    //输出Debug Info
    [self.debugInfo appendFormat:@"API链接:%@\n", self.payUrl];
    [self.debugInfo appendFormat:@"发送的xml:%@\n", send];
   
    //发送请求post xml数据
    NSData *res = [WXUtil httpSend:self.payUrl method:@"POST" data:send];
   
    //输出Debug Info
    [self.debugInfo appendFormat:@"服务器返回：\n%@\n\n",[[NSString alloc] initWithData:res encoding:NSUTF8StringEncoding]];
   
    XMLHelper *xml  = [[XMLHelper alloc] autorelease];
   
    //开始解析
    [xml startParse:res];
   
    NSMutableDictionary *resParams = [xml getDict];
   
    //判断返回
    NSString *return_code   = [resParams objectForKey:@"return_code"];
    NSString *result_code   = [resParams objectForKey:@"result_code"];
    if ( [return_code isEqualToString:@"SUCCESS"] )
    {
        //生成返回数据的签名
        NSString *sign      = [self createMd5Sign:resParams ];
        NSString *send_sign =[resParams objectForKey:@"sign"] ;
       
        //验证签名正确性
        if( [sign isEqualToString:send_sign]){
            if( [result_code isEqualToString:@"SUCCESS"]) {
                //验证业务处理状态
                prepayid    = [resParams objectForKey:@"prepay_id"];
                return_code = 0;
               
                [self.debugInfo appendFormat:@"获取预支付交易标示成功！\n"];
            }
        }else{
            self.lastErrCode = 1;
            [self.debugInfo appendFormat:@"gen_sign=%@\n   _sign=%@\n",sign,send_sign];
            [self.debugInfo appendFormat:@"服务器返回签名验证错误！！！\n"];
        }
    }else{
        self.lastErrCode = 2;
        [self.debugInfo appendFormat:@"接口返回错误！！！\n"];
    }
   
    return prepayid;
}

- (NSMutableDictionary*)getPrepayWithOrderName:(NSString*)name
                                         price:(NSString*)price
                                        device:(NSString*)device
{
    //订单标题，展示给用户
    NSString* orderName = name;
    //订单金额,单位（分）
    NSString* orderPrice = price;//以分为单位的整数
    //支付设备号或门店号
    NSString* orderDevice = device;
    //支付类型，固定为APP
    NSString* orderType = @"APP";
    //发器支付的机器ip,暂时没有发现其作用
    NSString* orderIP = @"196.168.1.1";
   
    //随机数串
    srand( (unsigned)time(0) );
    NSString *noncestr  = [NSString stringWithFormat:@"%d", rand()];
    NSString *orderNO   = [NSString stringWithFormat:@"%ld",time(0)];
   
    //================================
    //预付单参数订单设置
    //================================
    NSMutableDictionary *packageParams = [NSMutableDictionary dictionary];
   
    [packageParams setObject: self.appId  forKey:@"appid"];       //开放平台appid
    [packageParams setObject: self.mchId  forKey:@"mch_id"];      //商户号
    [packageParams setObject: orderDevice  forKey:@"device_info"]; //支付设备号或门店号
    [packageParams setObject: noncestr     forKey:@"nonce_str"];   //随机串
    [packageParams setObject: orderType    forKey:@"trade_type"];  //支付类型，固定为APP
    [packageParams setObject: orderName    forKey:@"body"];        //订单描述，展示给用户
    [packageParams setObject: NOTIFY_URL  forKey:@"notify_url"];  //支付结果异步通知
    [packageParams setObject: orderNO      forKey:@"out_trade_no"];//商户订单号
    [packageParams setObject: orderIP      forKey:@"spbill_create_ip"];//发器支付的机器ip
    [packageParams setObject: orderPrice   forKey:@"total_fee"];       //订单金额，单位为分
   
    //获取prepayId（预支付交易会话标识）
    NSString *prePayid;
    prePayid = [self sendPrepay:packageParams];
   
    if(prePayid == nil)
    {
        [self.debugInfo appendFormat:@"获取prepayid失败！\n"];
        return nil;
    }
   
    //获取到prepayid后进行第二次签名
    NSString    *package, *time_stamp, *nonce_str;
    //设置支付参数
    time_t now;
    time(&now);
    time_stamp  = [NSString stringWithFormat:@"%ld", now];
    nonce_str = [WXUtil md5:time_stamp];
    //重新按提交格式组包，微信客户端暂只支持package=Sign=WXPay格式，须考虑升级后支持携带package具体参数的情况
    //package       = [NSString stringWithFormat:@"Sign=%@",package];
    package         = @"Sign=WXPay";
    //第二次签名参数列表
    NSMutableDictionary *signParams = [NSMutableDictionary dictionary];
    [signParams setObject: self.appId  forKey:@"appid"];
    [signParams setObject: self.mchId  forKey:@"partnerid"];
    [signParams setObject: nonce_str    forKey:@"noncestr"];
    [signParams setObject: package      forKey:@"package"];
    [signParams setObject: time_stamp   forKey:@"timestamp"];
    [signParams setObject: prePayid     forKey:@"prepayid"];
   
    //生成签名
    NSString *sign  = [self createMd5Sign:signParams];
   
    //添加签名
    [signParams setObject: sign         forKey:@"sign"];
   
    [self.debugInfo appendFormat:@"第二步签名成功，sign＝%@\n",sign];
   
    //返回参数列表
    return signParams;
}

@end
```






然后，在需要调用微信支付的Controller里面，新建一个方法。在合适的地方调用。这个方法里面利用WechatPayManager这个类进行了初始化和参数封装，然后调用上述的核心代码（PayReq那一段）

```
- (void)wxPayWithOrderName:(NSString*)name price:(NSString*)price
{
    //创建支付签名对象 && 初始化支付签名对象
    WechatPayManager* wxpayManager = [[[WechatPayManager alloc]initWithAppID:APP_ID mchID:MCH_ID spKey:PARTNER_ID] autorelease];
   
    //获取到实际调起微信支付的参数后，在app端调起支付
    //生成预支付订单，实际上就是把关键参数进行第一次加密。
    NSString* device = [[UserManager defaultManager]userId];
    NSMutableDictionary *dict = [wxpayManager getPrepayWithOrderName:name
                                                               price:price
					                                        device:device];
   
    if(dict == nil){
        //错误提示
        NSString *debug = [wxpayManager getDebugInfo];
        return;
    }
    
    NSMutableString *stamp  = [dict objectForKey:@"timestamp"];
   
    //调起微信支付
    PayReq* req             = [[[PayReq alloc] init]autorelease];
    req.openID              = [dict objectForKey:@"appid"];
    req.partnerId          = [dict objectForKey:@"partnerid"];
    req.prepayId            = [dict objectForKey:@"prepayid"];
    req.nonceStr            = [dict objectForKey:@"noncestr"];
    req.timeStamp          = stamp.intValue;
    req.package            = [dict objectForKey:@"package"];
    req.sign                = [dict objectForKey:@"sign"];
   
//        BOOL flag = [WXApi sendReq:req];
    BOOL flag = [WXApi safeSendReq:req];
}
```

---

再者，支付完成了需要调用一个delegate，这个delegate方便个性化显示支付结果。一般直接把这两个delegate放在AppDelegate就好了。因为有一些其他内容也是需要在AppDelegate里面实现，省的分开找不到。


```objective-c
-(void) onResp:(BaseResp*)resp
{  
    //启动微信支付的response
    NSString *strMsg = [NSString stringWithFormat:@"errcode:%d", resp.errCode];
    if([resp isKindOfClass:[PayResp class]]){
        //支付返回结果，实际支付结果需要去微信服务器端查询
        switch (resp.errCode) {
            case 0:
                strMsg = @"支付结果：成功！";
                break;
            case -1:
                strMsg = @"支付结果：失败！";
                break;
            case -2:
                strMsg = @"用户已经退出支付！";
                break;
            default:
                strMsg = [NSString stringWithFormat:@"支付结果：失败！retcode = %d, retstr = %@", resp.errCode,resp.errStr];
                break;
        }
    }
}
```



>  注意事项：
> 1）如果APP里面已经使用了ShareSDK，就有一些地方要注意。不要再重复导入微信的SDK，因为shareSDK里面的extend已经包括了微信的SDK。
> 2）微信本身是鼓励客户APP把签名算法放到服务器上面，这样信息就不容易被破解。但是如果客户APP本身没有服务器端，或者认为不需要放到服务器端，也可以直接把签名（加密）的部分直接放在APP端。Sample代码的注释有点乱，多次提到服务器云云，但是其实可以不这么做。
> 3）微信的price单位是分。注意下即可。
> 4）暂时想不到，以后想到了再记录。。

