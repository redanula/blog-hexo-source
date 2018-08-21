title: iOS腾讯云人脸识别
date: 2018-08-08 22:12:15
tags: [腾讯云,Objective-C SHA1, 人脸识别, 鉴权]
---

### 前言
工作需要做了一个简易的、通过API第三方人脸识别、人脸对比分析的App Demo，对比了一下各家提供的服务、SDK，最后选择了腾讯云的智能图像服务。
调用API的流程为：

    1.鉴权签名 
    2.调用人脸识别、人脸对比等API。
    
这里主要在iOS上模拟生成鉴权签名识别的流程，实际生产环境应该是服务器生成鉴权签名，APP调用接口识别。

### 鉴权签名

云端开通服务后的Key：

    #define kAPPID @"your APPID"
    #define kSecretId @"your SecretId"
    #define kSecretKey @"your SecretKey"


拼接签名串：

    - (NSString *)getSign{
    
        NSDate* date = [NSDate dateWithTimeIntervalSinceNow:0];//获取当前时间0秒后的时间
        NSDate* edate = [NSDate dateWithTimeIntervalSinceNow:60*60*24*30];//一个月
        NSTimeInterval time=[date timeIntervalSince1970];// *1000 是精确到毫秒，不乘就是精确到秒
        NSTimeInterval etime=[edate timeIntervalSince1970];// *1000 是精确到毫秒，不乘就是精确到秒
        NSString *timeString = [NSString stringWithFormat:@"%.0f", time];
        NSString *etimeString = [NSString stringWithFormat:@"%.0f", etime];
        int ranInt = arc4random() %100000;
        NSString *ranString = [NSString stringWithFormat:@"%d", ranInt];
        NSString *abketr = [NSString stringWithFormat:@"a=%@&b=%@&k=%@&e=%@&t=%@&r=%@",
                            kAPPID,
                            @"tencentyun",
                            kSecretId,
                            etimeString,
                            timeString,
                            ranString
                            ];
        NSString *signString = [self HmacSha1:kSecretKey data:abketr];
    
        return signString;
    }
    
HMAC-SHA1 算法加密，注意最后需要拼接签名串到NSMutableData末尾：

    //HmacSHA1加密；
    - (NSString *)HmacSha1:(NSString *)key data:(NSString *)data
    {
        const char *cKey  = [key cStringUsingEncoding:NSASCIIStringEncoding];
        const char *cData = [data cStringUsingEncoding:NSASCIIStringEncoding];

        unsigned char cHMAC[CC_SHA1_DIGEST_LENGTH];
        CCHmac(kCCHmacAlgSHA1, cKey, strlen(cKey), cData, strlen(cData), cHMAC);
      
        NSData *HMAC = [[NSData alloc] initWithBytes:cHMAC length:sizeof(cHMAC)];
        NSData *dData = [[NSData alloc] initWithBytes:cData length:strlen(cData)];
        
        NSMutableData *mData = [[NSMutableData alloc] init];
        
        [mData appendData:HMAC];
        [mData appendData:dData];
       
        NSString *hash = [mData base64EncodedStringWithOptions:0];//将加密结果进行一次BASE64编码。
        return hash;
    }

如果是SHA256，可以：

    unsigned char cHMAC[CC_SHA256_DIGEST_LENGTH];
    CCHmac(kCCHmacAlgSHA256, cKey, strlen(cKey), cData, strlen(cData), cHMAC);

### 人脸检测

通过生成的签名添加到请求头authorization里面，请求如下：

    NSString *apiString = [NSString stringWithFormat:@"%@%@",@"https://recognition.image.myqcloud.com",@"/face/detect"];
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
    [manager setRequestSerializer:[AFHTTPRequestSerializer serializer]];
    [manager.requestSerializer setValue:@"recognition.image.myqcloud.com" forHTTPHeaderField:@"host"];
    [manager.requestSerializer setValue:[self getSign] forHTTPHeaderField:@"authorization"];
    [manager POST:apiString parameters:params constructingBodyWithBlock:^(id<AFMultipartFormData>  _Nonnull formData) {
        [formData appendPartWithFileData:imageData name:@"image" fileName:@"image.jpg" mimeType:@"image/jpg"];
    } progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        block(responseObject);
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        block(nil);
    }];

其他接口可以查看：
[1.腾讯云鉴权签名](https://cloud.tencent.com/document/product/867/17719)
[2.人脸识别接口文档](https://cloud.tencent.com/document/product/867/17588)

