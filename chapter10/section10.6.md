# 10.6 iOS https 协议 配置


## 前言：

  自苹果宣布2017年1月1日开始强制使用https以来，htpps慢慢成为大家讨论的对象之一，不是说此前https没有出现，只是这一决策让得开发者始料未及， 
虽然在2016年底，苹果突然宣布无限期推迟这个计划，不过随着网络世界的日益发达，将自己开发的产品从 http 升级到 https 已经是势在必行了，
最近在元翔专车项目中已经将 https 应用其中，在这里记录下来，以帮助其他开发者快速接入 https 。

## https原理简述

https的作用是为了客户端与服务器之间通讯的安全性，实际上https是工作在SSL上的http协议。

SSL握手协议包含4个阶段：

#### ＊  第一阶段：建立安全能力

    由客户端发起，向服务器发送Client Hello消息，其中包含SSL的版本，客户端随机数（用于生成密钥），会话号
    ，加密算法清单，压缩算法清单。 服务器返回Server Hello消息，其中包含SSL版本，服务器随机数（用于生成密钥）
    ，会话号，选择的加密算法，选择的压缩算法。

    总之就是互相先打招呼，告诉对方自己是谁干嘛的。

#### ＊  第二阶段：服务器认证与密钥交换

 本阶段服务器是唯一发送消息方，这里共分三步： 

      第一步：证书，服务器将服务器数字证书以及整个CA证书链发给客户端，客户端由此获得服务器公钥；
      客户端还可以在这一步选择是否验证服务器是否可信，不验证的话就相当于跳过证书验证这一步，直接进行通讯
      ，前提是服务器支持，不错这样一来https将失去其本来的安全作用，
      所以还是需要进行验证的，如果不可信则客户端可以停止连接，并提醒用户；
      
      第二步：证书请求，服务器请求客户端的数字证书，客户端认证在SSL中是可选的，因此这一步也是可选的。

      第三步：服务器握手完成，发送这个消息后，服务器等待客户端响应。

#### ＊   第三阶段 ：客户端认证与密钥互换

    本阶段客户端是唯一消息发送方，共分三步。

    第一步：证书，客户端将客户端数字证书发送给服务器，里面包含了客户端的公钥。
    通常来说这个证书应该是由服务提供者分发给客户端，由指定的CA签发的
    ，因此服务器可以验证客户端证书的合法性，并决定是否继续。
    
    第二步：密钥交换，客户端生成“预主密码”，用服务器的公钥加密，然后发送给服务器。这个加密过的“预主密码”只有客户端和服务器才能知道，
    与后面对称加密的对称密钥有关。
    在这一步也间接验证了服务器，服务器必须拥有与证书对应的私钥才能解密预主密码”。
    
    第三步：证书验证，客户端还需要向服务器验证自己是真正的客户端（数字证书只是证明客户端拥有合法的公钥，
    但无法证明它就是客户端，拥有与公钥对应的私钥才是关键），为此客户端把已经加密的“预主密码”、客户端含有签名的随机数和自己的证书，
    发送给服务器。服务器通过验证CA的有效性合法性，随机数等等，
    若通过，服务器将用自己的私钥解开加密的“预主密码”，然后执行一系列步骤来产生主通讯密码。
    （客户端获取主通讯密码的方式和服务器是一样的）

#### ＊   第四阶段：完成

   在这个阶段，客户端和服务器各自独立的生成相同的主通讯密码，也就是所谓的对称密钥。

   SSL握手之后就进入会话阶段，客户端和服务器使用握手协议中生成的对称密钥进行加密和解密以保证通信的安全。

综上所述，如果要完成HTTPS的双向认证需要以下密钥和证书：

 服务器端 ：

    1.服务器私钥 ；
     
    2.由CA签发的含有服务器公钥的数字证书 ；

    3.CA的数字证书。在双向认证的实践中，通常服务器可以自己作为证书机构，并且由服务器CA签发服务器证书和客户端证书。

 客户端 ：

    1.客户端私钥，因为公钥是通过服务端的数字证书和CA证书获得的 ；

    2.由CA签发的含有客户端公钥的数字证书。为了避免中间人攻击，
     客户端还需要内置服务器证书，用来验证所连接的服务器是否是指定的服务器。

## 使用第三方库AFNetworking，配置证书

 由于苹果在2016年8月强制要求APP 内的网络需要适配 IPv6 ，而 AFNetworking 在 3.0版本之后开始支持 IPv6，所以以下的证书配置建立在 AFnetworking 3.x 版本。

#### （一）单向认证方式

写工具类#import "AFNetworking.h"

以下例子建立在项目已经将 AFNetworking 导入的情况下，自己建立一个网络请求的方法。

基本原理： 在请求的方法里面，将证书作为协议安全参数进行数据传输，由后端进行单向校验。

### .h

```
#import <Foundation/Foundation.h>

@interface HttpRequest : NSObject
/**
 *  发送一个POST请求
 *
 *  @param url     请求路径
 *  @param params  请求参数
 *  @param success 请求成功后的回调（请将请求成功后想做的事情写到这个block中）
 *  @param failure 请求失败后的回调（请将请求失败后想做的事情写到这个block中）
 */
+ (void)post:(NSString *)url params:(NSDictionary *)params success:(void (^)(id responseObj))success failure:(void (^)(NSError *error))failure;

@end
```


### .m


```
#import "HttpRequest.h"
#import "AFNetworking.h"

/**
 *  是否开启https SSL 验证
 *
 *  @return YES为开启，NO为关闭
 */
#define openHttpsSSL YES
/**
 *  SSL 证书名称，仅支持cer格式。“app.bishe.com.cer”,则填“app.bishe.com”
 */
#define certificate @"adn"

@implementation HttpRequest

+ (void)post:(NSString *)url params:(NSDictionary *)params success:(void (^)(id))success failure:(void (^)(NSError *))failure
{
      //对应域名的校验我认为应该在url中去逻辑判断。--》冯龙腾写
      //通过对url字符串的切割对子域名进行逻辑验证处理。

    // 1.获得请求管理者
    AFHTTPRequestOperationManager *mgr = [AFHTTPRequestOperationManager manager];
    // 2.申明返回的结果是text/html类型
    mgr.responseSerializer = [AFHTTPResponseSerializer serializer];
    mgr.responseSerializer.acceptableContentTypes = [NSSet setWithObjects:@"application/json",@"text/json",@"text/javascript",@"text/html",nil];
    // 3.设置超时时间为10s
    mgr.requestSerializer.timeoutInterval = 10;

    // 加上这行代码，https ssl 验证。
    if(openHttpsSSL)
    {
        [mgr setSecurityPolicy:[self customSecurityPolicy]];
    }

    // 4.发送POST请求
    [mgr POST:url parameters:params
      success:^(AFHTTPRequestOperation *operation, id responseObj) {
          if (success) {
              success(responseObj);
          }
      } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
          if (failure) {
              failure(error);
          }
      }];
}

+ (AFSecurityPolicy*)customSecurityPolicy
{
    // /先导入证书
    NSString *cerPath = [[NSBundle mainBundle] pathForResource:certificate ofType:@"cer"];//证书的路径
    NSData *certData = [NSData dataWithContentsOfFile:cerPath];

    // AFSSLPinningModeCertificate 使用证书验证模式
    AFSecurityPolicy *securityPolicy = [AFSecurityPolicy policyWithPinningMode:AFSSLPinningModeCertificate];

    // allowInvalidCertificates 是否允许无效证书（也就是自建的证书），默认为NO
    // 如果是需要验证自建证书，需要设置为YES
    securityPolicy.allowInvalidCertificates = YES;

    //validatesDomainName 是否需要验证域名，默认为YES；
    //假如证书的域名与你请求的域名不一致，需把该项设置为NO；如设成NO的话，即服务器使用其他可信任机构颁发的证书，也可以建立连接，这个非常危险，建议打开。
    //置为NO，主要用于这种情况：客户端请求的是子域名，而证书上的是另外一个域名。因为SSL证书上的域名是独立的，假如证书上注册的域名是www.google.com，那么mail.google.com是无法验证通过的；当然，有钱可以注册通配符的域名*.google.com，但这个还是比较贵的。
    //如置为NO，建议自己添加对应域名的校验逻辑。
    //对应域名的校验我认为应该在url中去逻辑判断。
    securityPolicy.validatesDomainName = NO;
    if (certData) {
        securityPolicy.pinnedCertificates = [[NSSet alloc] initWithArray: @[certData]];
    }
    return securityPolicy;
}
@end
```


#### （二）双向认证方式

进行双向认证的时候，需要新建一个封装了  AFHTTPSessionManager  类的方法，用于将认证的证书封装在网络请求管理者里面，在传输层中，客户端将会根据证书进行校验。

#### .h

```
#import <Foundation/Foundation.h>
#import "AFNetworking.h"

@interface YQLHTTPSManager : NSObject


/**
 生成 https 控制器

 @return 控制器
 */
+ (AFHTTPSessionManager *)manager;

@end
```


#### .m


```
#import "YQLHTTPSManager.h"

@implementation YQLHTTPSManager

+ (AFHTTPSessionManager *)manager {
    
    NSString *certFilePath = [[NSBundle mainBundle] pathForResource:@"server" ofType:@"cer"];
    NSData *certData = [NSData dataWithContentsOfFile:certFilePath];
    NSSet *certSet = [NSSet setWithObject:certData];
    AFSecurityPolicy *policy = [AFSecurityPolicy policyWithPinningMode:AFSSLPinningModeCertificate withPinnedCertificates:certSet];
    policy.allowInvalidCertificates = YES;
    policy.validatesDomainName = NO;
    __weak AFHTTPSessionManager *_manager = [AFHTTPSessionManager manager];
    _manager.securityPolicy = policy;
    _manager.requestSerializer = [AFHTTPRequestSerializer serializer];
    _manager.responseSerializer = [AFHTTPResponseSerializer serializer];
    _manager.responseSerializer.acceptableContentTypes =  [NSSet setWithObjects:@"application/json", @"text/json", @"text/javascript",@"text/plain", nil];
    //关闭缓存避免干扰测试r
    _manager.requestSerializer.cachePolicy = NSURLRequestReloadIgnoringLocalCacheData;
    [_manager setSessionDidBecomeInvalidBlock:^(NSURLSession * _Nonnull session, NSError * _Nonnull error) {
        NSLog(@"setSessionDidBecomeInvalidBlock");
    }];
    //客户端请求验证 重写 setSessionDidReceiveAuthenticationChallengeBlock 方法
    __weak typeof(self)weakSelf = self;
    [_manager setSessionDidReceiveAuthenticationChallengeBlock:^NSURLSessionAuthChallengeDisposition(NSURLSession*session, NSURLAuthenticationChallenge *challenge, NSURLCredential *__autoreleasing*_credential) {
        NSURLSessionAuthChallengeDisposition disposition = NSURLSessionAuthChallengePerformDefaultHandling;
        __autoreleasing NSURLCredential *credential =nil;
        if([challenge.protectionSpace.authenticationMethod isEqualToString:NSURLAuthenticationMethodServerTrust]) {
            if([_manager.securityPolicy evaluateServerTrust:challenge.protectionSpace.serverTrust forDomain:challenge.protectionSpace.host]) {
                credential = [NSURLCredential credentialForTrust:challenge.protectionSpace.serverTrust];
                if(credential) {
                    disposition =NSURLSessionAuthChallengeUseCredential;
                } else {
                    disposition =NSURLSessionAuthChallengePerformDefaultHandling;
                }
            } else {
                disposition = NSURLSessionAuthChallengeCancelAuthenticationChallenge;
            }
        } else {
            // client authentication
            SecIdentityRef identity = NULL;
            SecTrustRef trust = NULL;
            NSString *p12 = [[NSBundle mainBundle] pathForResource:@"client"ofType:@"p12"];
            NSFileManager *fileManager =[NSFileManager defaultManager];
            
            if(![fileManager fileExistsAtPath:p12])
            {
                NSLog(@"client.p12:not exist");
            }
            else
            {
                NSData *PKCS12Data = [NSData dataWithContentsOfFile:p12];
                
                if ([[weakSelf class]extractIdentity:&identity andTrust:&trust fromPKCS12Data:PKCS12Data])
                {
                    SecCertificateRef certificate = NULL;
                    SecIdentityCopyCertificate(identity, &certificate);
                    const void*certs[] = {certificate};
                    CFArrayRef certArray =CFArrayCreate(kCFAllocatorDefault, certs,1,NULL);
                    credential =[NSURLCredential credentialWithIdentity:identity certificates:(__bridge  NSArray*)certArray persistence:NSURLCredentialPersistencePermanent];
                    disposition = NSURLSessionAuthChallengeUseCredential;
                }
            }
        }
        *_credential = credential;
        return disposition;
    }];
    return _manager;
}

+ (BOOL)extractIdentity:(SecIdentityRef*)outIdentity andTrust:(SecTrustRef *)outTrust fromPKCS12Data:(NSData *)inPKCS12Data {
    OSStatus securityError = errSecSuccess;
    //client certificate password
    NSDictionary*optionsDictionary = [NSDictionary dictionaryWithObject:@"certificate password"
                                                                 forKey:(__bridge id)kSecImportExportPassphrase];
    
    CFArrayRef items = CFArrayCreate(NULL, 0, 0, NULL);
    securityError = SecPKCS12Import((__bridge CFDataRef)inPKCS12Data,(__bridge CFDictionaryRef)optionsDictionary,&items);
    
    if(securityError == 0) {
        CFDictionaryRef myIdentityAndTrust =CFArrayGetValueAtIndex(items,0);
        const void*tempIdentity =NULL;
        tempIdentity= CFDictionaryGetValue (myIdentityAndTrust,kSecImportItemIdentity);
        *outIdentity = (SecIdentityRef)tempIdentity;
        const void*tempTrust =NULL;
        tempTrust = CFDictionaryGetValue(myIdentityAndTrust,kSecImportItemTrust);
        *outTrust = (SecTrustRef)tempTrust;
    } else {
        NSLog(@"Failedwith error code %d",(int)securityError);
        return NO;
    }
    return YES;
}


@end
```


总结： 目前 https 只有 单向验证，双向验证 这两种方式，一般的项目，不涉及重大机密和金钱的一般情况下使用单向认证方式就足够了。

#### 注意事项：

- 将证书拉进项目中，但是获取不到证书的数据

解决方案：点击项目列表中的证书，show in Find ，查看证书是否在项目的根目录下，最好在导入证书的时候使用 Add Files to，以免没有正确导入证书

- 据以上的方式之一配置了 https 之后请求不成功

解决方案： 需要先检查是否在请求的地址中将 http 改成 https 了，改了之后还是不能请求成功，一般情况时后台配置出了问题，需要将后台的配置升级。

## 生成证书

配置 https 证书的时候，需要后端和前端的证书保持一致，在这种情况下，证书一般会由后端给我们前端开发者。给的证书有两份，一份是 .pem 格式的公钥证书，另一份是以 .pfx 格式的私钥。这两份证书我们不可以直接拿来用，需要将其格式分别转化成 .cer 和 .p12 格式的证书。具体的转化方法可以先将 后端给的证书导入钥匙串，然后分别将证书在钥匙串里面导出：

> .pem  ->  .cer
> 
> .pfx   ->  .p12

导出之后就可以将其放入项目中使用。
