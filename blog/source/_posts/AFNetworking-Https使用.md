---
title: AFNetworking+Https使用
date: 2016-11-17 14:09:18
tags: [AFNetworking,SSL]
---

 iOS9中新增App Transport Security（简称ATS）特性, 主要使到原来请求的时候用到的HTTP，都转向TLS1.2协议进行传输。这也意味着所有的HTTP协议都强制使用了HTTPS协议进行传输。最近公司app从HTTP转向HTTPS。简单记录如下。

## Https是什么

>HTTPS就是将HTTP协议数据包放到SSL/TSL层加密后，在TCP/IP层组成IP数据报去传输，以此保证传输数据的安全；而对于接收端，在SSL/TSL将接收的数据包解密之后，将数据传给HTTP协议层，就是普通的HTTP数据。HTTP和SSL/TSL都处于OSI模型的应用层。

具体可查看参考文档。

## App Transport Security

之前我们如果还是使用的http,针对IOS9及以上情况，通过在 Info.plist 中声明，倒退回不安全的网络请求。

<!-- more -->

```
<key>NSAppTransportSecurity</key>
  <dict>
        <key>NSAllowsArbitraryLoads</key>
        <true/>
 </dict>
```
## IOS使用HTTPS注意

>These are the App Transport Security requirements:

> The protocol Transport Security Layer (TLS) must be at least version 1.2.

>Connection ciphers are limited to those that provide forward secrecy (see the list of ciphers below.)

>Certificates must use at least an SHA256 fingerprint with either a 2048 bit or greater RSA key, or a 256 bit or greater Elliptic-Curve (ECC) key.

根据原文描述，首先必须要基于TLS 1.2版本协议。再来就是连接的加密方式要提供Forward Secrecy。

最后就是证书至少要使用一个SHA256的指纹与任一个2048位或者更高位的RSA密钥，或者是256位或者更高位的ECC密钥。如果不符合其中一项，请求将被中断并返回nil。

但是也要注意证书的合法性,注意是否有效，iOS要求连接的HTTPS站点必须为CA签名过的合法证书。

## 为啥AFNetworking?

iOS开发中，AFNetworking以其优雅的结构设计和简便的调用方式，使其成为了最流行的网络开源库之一（另一个应该算是ASI了，但经久失修不维护的原因，已经不是首选）。

## 使用AFNetworking（3.0.0以后）

```
AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
AFSecurityPolicy *securityPolicy = [AFSecurityPolicy defaultPolicy];
//allowInvalidCertificates 是否允许无效证书（也就是自建的证书），默认为NO//如果是需要验证自建证书，需要设置为YES
securityPolicy.allowInvalidCertificates = YES;
//validatesDomainName 是否需要验证域名，默认为YES；
securityPolicy.validatesDomainName = YES;
manager.securityPolicy  = securityPolicy;
manager.responseSerializer = [AFHTTPResponseSerializer serializer];
[manager POST:urlString parameters:dic success:finishedBlock failure:failedBlock];
```
之前版本只需要把AFHTTPSessionManager替换成AFHTTPRequestOperationManager就可以了，其他大同小异。

## 防止中间人攻击

中间人攻击者通过伪造的ssl证书使app连接到了伪装的假冒的服务器上，这是个严重的问题！那么如何防止中间人攻击呢？

首先web服务器必须提供一个ssl证书，需要一个 .crt 文件，然后设置app只能连接有效ssl证书的服务器。

在开始写代码前，先要把 .crt 文件转成 .cer 文件，然后在加到xcode 里面


 * crt文件转成.cer文件

  使用openssl进行转换

  ` openssl x509 -in 你的证书.crt -out 你的证书.cer -outform der `

* 安装crt文件，电脑导出

  * 先打开“钥匙串访问”
  * 选中你安装的crt文件证书，选择“文件”--》“导出项目”
  * 选择.cer证书，存储即可。

## 添加使用

* 添加一个安全方法

```objective-c
- (AFSecurityPolicy*)customSecurityPolicy
{
    // /先导入证书
    NSString *cerPath = [[NSBundle mainBundle] pathForResource:@"cheguo" ofType:@"cer"];//证书的路径
    NSData *certData = [NSData dataWithContentsOfFile:cerPath];

    NSString *cerPath2 = [[NSBundle mainBundle] pathForResource:@"cgw360" ofType:@"cer"];//证书的路径
    NSData *certData2 = [NSData dataWithContentsOfFile:cerPath2];

    // AFSSLPinningModeCertificate 使用证书验证模式
    AFSecurityPolicy *securityPolicy = [AFSecurityPolicy policyWithPinningMode:AFSSLPinningModeCertificate];

    // allowInvalidCertificates 是否允许无效证书（也就是自建的证书），默认为NO
    // 如果是需要验证自建证书，需要设置为YES
    securityPolicy.allowInvalidCertificates = NO;

    //validatesDomainName 是否需要验证域名，默认为YES；
    //假如证书的域名与你请求的域名不一致，需把该项设置为NO；如设成NO的话，即服务器使用其他可信任机构颁发的证书，也可以建立连接，这个非常危险，建议打开。
    //置为NO，主要用于这种情况：客户端请求的是子域名，而证书上的是另外一个域名。因为SSL证书上的域名是独立的，假如证书上注册的域名是www.google.com，那么mail.google.com是无法验证通过的；当然，有钱可以注册通配符的域名*.google.com，但这个还是比较贵的。
    //如置为NO，建议自己添加对应域名的校验逻辑。
    securityPolicy.validatesDomainName = NO;

    securityPolicy.pinnedCertificates = [[NSSet alloc] initWithObjects:certData, certData2,nil];

    return securityPolicy;
}

```

 * 接口请求中添加安全校验策略


```objective-c
- (void) NetRequestGETWithRequestAction: (NSString *) action
                          WithParameter: (NSDictionary *) parameter
                   WithReturnValeuBlock: (ReturnValueBlock) block
                     WithErrorCodeBlock: (ErrorCodeBlock) errorBlock
                       WithFailureBlock: (FailureBlock) failureBlock
                             requestURL: (NSString *)requestURL
{
    NSString *requestURLString = [NSString stringWithFormat:@"%@%@", requestURL, action];

    AFHTTPSessionManager *manager = [self afCommonRequestManager];
     manager.responseSerializer = [AFHTTPResponseSerializer serializer];

    // 加上这行代码，https ssl 验证。
    [manager setSecurityPolicy:[self customSecurityPolicy]];

     [manager GET:requestURLString parameters:parameter progress:^(NSProgress * _Nonnull downloadProgress) {

     } success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
         NSDictionary *dic = [NSJSONSerialization JSONObjectWithData:responseObject options:NSJSONReadingAllowFragments error:nil];
         if ([self processingRequestCode:dic]) {
             block(dic);
         }else{
             errorBlock([self unknowExeceptionNotify]);
         }
     } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
          failureBlock();
     } ];
}

```

关于验证策略可参考：

[如何正確設定 AFNetworking 的安全連線](http://nelson.logdown.com/posts/2015/04/29/how-to-properly-setup-afnetworking-security-connection/)

[正确使用AFNetworking的SSL保证网络安全](http://www.cocoachina.com/ios/20160224/15394.html)


参考文章
1. [iOS中AFNetworking HTTPS的使用](http://www.jianshu.com/p/20d5fb4cd76d)
2. [AFNetworking https SSL认证]http://www.cnblogs.com/jys509/p/5001566.html
