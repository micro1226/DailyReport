# SSL Pinning

## 关键词 
---------
**SSL Pinning**   
**AFNetworking**  



## 内容
---------
### 1.获取证书
openssl s_client -connect www.website.com:443 </dev/null 2>/dev/null | openssl x509 -outform DER > myWebsite.cer

### 2.把证书加进项目中
把生成的.cer证书摁住直接拖进你的项目相关文件中，记得勾选Copy items if needed和你的targets

### 3.配置AFSecurityPolicy
重点是设置 AFSSLPinningMode
- `AFSSLPinningMode`: 信任任何证书，没有安全性可言，是默认值。
- `AFSSLPinningModePublicKey`: 只比对证书的Public Key，而一般更新服务器证书，公钥是不会变的，只要公钥没有改变，证书的其他变动都不会影响使用。
- `AFSSLPinningModeCertificate`: 最安全的比对模式。但是也比较麻烦，因为证书是打包在APP中，如果服务器证书改变或者到期，旧版本无法使用了，我们就需要用户更新APP来使用最新的证书。

```swift
func testAPI() {
    let para = ["wd": "hello"]
    let manager = AFHTTPSessionManager.init(baseURL: URL.init(string: "https://www.baidu.com"))
    manager.responseSerializer = AFJSONResponseSerializer.init()
    manager.responseSerializer.acceptableContentTypes = ["application/json", "text/json", "text/javascript","text/html","text/css","text/plain"]
    
    let cerPath = Bundle.main.path(forResource: "baidu", ofType: "cer")!

    do {
        let data = try Data.init(contentsOf: URL.init(fileURLWithPath: cerPath))
        
        var dataSet: Set<Data> = Set.init()
        dataSet.insert(data)
        let securityPolicy = AFSecurityPolicy.init(pinningMode: .certificate, withPinnedCertificates: dataSet)
        securityPolicy.allowInvalidCertificates = false
        securityPolicy.validatesDomainName = true
        
        manager.securityPolicy = securityPolicy
    } catch {
        print(error)
    }
    
    manager.get("s", parameters: para, progress: nil) { (task, result) in
        print("task", task)
        print("result", result)
    } failure: { (task, error) in
        print("task", task)
        print("error", error)
    }
}
```

```objc
+ (AFHTTPSessionManager *)manager
{
    static AFHTTPSessionManager *manager = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        
        //1.创建manager对象
        NSURLSessionConfiguration *config = [NSURLSessionConfiguration defaultSessionConfiguration];
        manager =  [[AFHTTPSessionManager alloc] initWithBaseURL:[NSURL URLWithString:@"你的访问的地址的domian"] sessionConfiguration:config];
        //2.设置接收的response类型
        [[manager responseSerializer]setAcceptableContentTypes:[NSSet setWithObjects:@"application/json",@"text/plain",@"text/html", nil]];
        
        //3.https 证书配置
        //3.1 将证书拖进项目
        //3.2 获取证书路径
        NSString *certPath = [[NSBundle mainBundle] pathForResource:@"你的证书名字" ofType:@"cer"];
        //3.3 获取证书data
        NSData *certData = [NSData dataWithContentsOfFile:certPath];
        //3.4 创建AFN 中的securityPolicy
        AFSecurityPolicy *securityPolicy = [AFSecurityPolicy policyWithPinningMode:AFSSLPinningModePublicKey withPinnedCertificates
                                                                                  :[[NSSet alloc] initWithObjects:certData,nil]];
        //3.5 这里就可以添加多个server证书
        NSSet *dataSet = [[NSSet alloc]initWithObjects:certData, nil];
        //3.6 绑定证书（不止一个证书）
        [securityPolicy setPinnedCertificates:dataSet];
        //3.7 是否允许无效证书
        [securityPolicy setAllowInvalidCertificates:NO];
        //3.8 是否需要验证域名
        /*
        validatesDomainName 是否需要验证域名，默认为YES；
        假如证书的域名与你请求的域名不一致，需把该项设置为NO；
        如设成NO的话，即服务器使用其他可信任机构颁发的证书，也可以建立连接，这个非常危险，建议打开。
        置为NO，主要用于这种情况：客户端请求的是子域名，而证书上的是另外一个域名。
        因为SSL证书上的域名是独立的，假如证书上注册的域名是www.google.com，那么mail.google.com是无法验证通过的；
        当然，有钱可以注册通配符的域名*.google.com，但这个还是比较贵的。
        如置为NO，建议自己添加对应域名的校验逻辑。
         */
        [securityPolicy setValidatesDomainName:YES];

        manager.securityPolicy = securityPolicy;
    });
    return manager;
}
```

## 参考
- [iOS安全防护--在AFNetworking中实现 SSL pinning](https://www.jianshu.com/p/6b0172e42b29) 主要参考
- [如何正確設定 AFNetworking 的安全連線](http://nelson.logdown.com/posts/2015/04/29/how-to-properly-setup-afnetworking-security-connection/)