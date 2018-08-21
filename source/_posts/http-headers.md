title: .Net Core 添加 HTTP Headers
date: 2018-08-01 20:43:12
tags: [Http Header]
---

### 前言
基于安全的需求，要在API（基于.Net Core）交互的过程中除了特有的鉴权外，还添加额外的Header。

### Strict-Transport-Security 
#### 1.概述    
<u>HTTP Strict Transport Security is an excellent feature to support on your site and strengthens your implementation of TLS by getting the User Agent to enforce the use of HTTPS. Recommended value strict-transport-security: max-age=31536000; includeSubDomains.</u>

HTTP Strict Transport Security（HTTP严格安全传输） 通常简称为HSTS，要求浏览器只能通过HTTPS访问当前资源，而不是HTTP。浏览器会自动把所有尝试使用HTTP的请求自动替换为HTTPS请求，在一定范围内（如钓鱼网站等）防止中间人攻击。

#### 2.示例
    
    Strict-Transport-Security: max-age=31536000
    
    //includeSubDomains适用于该网站的所有子域名
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    
    //Google维护 首次浏览器加载后可以预加载到缓存，需要向Google申请
    Strict-Transport-Security: max-age=31536000; preload

### Public-Key-Pins
#### 1.概述
<u>HTTP Public Key Pinning protects your site from MiTM attacks using rogue X.509 certificates. By whitelisting only the identities that the browser should trust, your users are protected in the event a certificate authority is compromised.</u>

HTTP公钥固定（又称HTTP公钥钉扎，英语：HTTP Public Key Pinning，缩写HPKP）是HTTPS网站防止攻击者利用数字证书认证机构（CA）错误签发的证书进行中间人攻击的一种安全机制，用于预防CA遭受入侵或其他会造成CA签发未授权证书的情况。

#### 2.示例

    /*
        pin-sha256 即证书指纹，允许出现多次（实际上最少应该指定两个）；
        max-age 和 includeSubdomains (可选) 与HSTS一致；
        report-uri(可选) 用来指定验证失败时的上报地址，格式和含义跟 CSP（Content Security Policy）中的同名字段一致；
    */
    
    Public-Key-Pins: pin-sha256="base64=="; max-age=expireTime [; includeSubdomains][; report-uri="reportURI"]
    

### X-Frame-Options
#### 1.概述
<u>X-Frame-Options tells the browser whether you want to allow your site to be framed or not. By preventing a browser from framing your site you can defend against attacks like clickjacking. Recommended value x-frame-options: SAMEORIGIN.</u>

X-Frame-Options是用来给浏览器指示允许一个页面可否在 frame,iframe 或者 object 标签中展现的标记。避免了点击劫持 (clickjacking) 的攻击。

#### 2.示例
在IIS中配置如下：

    <system.webServer>
      ...
    
      <httpProtocol>
        <customHeaders>
          <add name="X-Frame-Options" value="SAMEORIGIN" />
        </customHeaders>
      </httpProtocol>
    
      ...
    </system.webServer>

### X-Content-Type-Options 
#### 1.概述
<u>X-Content-Type-Options stops a browser from trying to MIME-sniff the content type and forces it to stick with the declared content-type. The only valid value for this header is X-Content-Type-Options: nosniff</u>

通常浏览器会根据Content-Type字段来区分类型，X-Content-Type-Options用来禁用浏览器的 MIME 类型嗅探。
#### 2.示例   
    
    X-Content-Type-Options: nosniff
    
### Referrer-Policy
#### 1.概述
<u>Referrer Policy is a new header that allows a site to control how much information the browser includes with navigations away from a document and should be set by all sites.</u>

HTTP来源地址（referer，或HTTP referer）是HTTP表头的一个字段，用来表示从哪儿链接到目前的网页，采用的格式是URL。换句话说，借着HTTP来源地址，目前的网页可以检查访客从哪里而来，这也常被用来对付伪造的跨网站请求。[Referrer维基百科](https://zh.wikipedia.org/wiki/HTTP參照位址)

#### 2.示例

    Referrer-Policy: no-referrer
    Referrer-Policy: no-referrer-when-downgrade
    Referrer-Policy: origin
    Referrer-Policy: origin-when-cross-origin
    Referrer-Policy: same-origin
    Referrer-Policy: strict-origin
    Referrer-Policy: strict-origin-when-cross-origin
    Referrer-Policy: unsafe-url

同源、跨域等策略详细参考：[同源策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)

### Content-Security-Policy
#### 1.概述
<u>Content Security Policy is an effective measure to protect your site from XSS attacks. By whitelisting sources of approved content, you can prevent the browser from loading malicious assets.</u>

Content Security Policy，网页安全政策，缩写 CSP。CSP可以添加来源白名单，防止XSS攻击，更进一步可以通过`report-uri`记录异常行为。

#### 2.示例

    Content-Security-Policy: default-src 'self'; ...; report-uri /my_amazing_csp_report_parser;
    

### X-XSS-Protection
#### 1.概述
<u>X-XSS-Protection sets the configuration for the cross-site scripting filter built into most browsers. Recommended value "X-XSS-Protection: 1; mode=block".</u>

X-XSS-Protection 是浏览器默认开启的，防止XSS的保护机制。可以为不支持 CSP 的旧版浏览器的用户提供保护。

#### 2.示例
    
    //关闭
    X-XSS-Protection: 0
    //默认开始，清除页面
    X-XSS-Protection: 1
    //开启，不加载
    X-XSS-Protection: 1; mode=block
    //CSP 通过`report-uri`记录异常行为
    X-XSS-Protection: 1; report=<reporting-uri>

### .Net Core的 Header 添加
了解以上Header后，对应就要添加.Net Core的Header了，详细参考：[How to add default security headers in ASP.NET Core using custom middleware](https://andrewlock.net/adding-default-security-headers-in-asp-net-core/)
构建对应的SecurityHeadersMiddleware，然后在`Startup.cs`的`Configure`添加：

    app.UseSecurityHeadersMiddleware(new SecurityHeadersBuilder().AddDefaultSecurePolicy());

也可以参考：[ADDING HTTP HEADERS TO IMPROVE SECURITY IN AN ASP.NET MVC CORE APPLICATION](https://damienbod.com/2018/02/08/adding-http-headers-to-improve-security-in-an-asp-net-mvc-core-application/)，在 NuGet Package 里添加 `NWebsec.AspNetCore.Middleware`
然后在`Startup.cs`的`Configure`添加：

    app.UseHsts(hsts => hsts.MaxAge(365).IncludeSubdomains());
    app.UseXContentTypeOptions();
    app.UseReferrerPolicy(opts => opts.NoReferrer());
    app.UseXXssProtection(options => options.EnabledWithBlockMode());
    app.UseXfo(options => options.Deny());
    app.UseCsp(opts => opts
        .BlockAllMixedContent()
        .StyleSources(s => s.Self())
        .StyleSources(s => s.UnsafeInline())
        .FontSources(s => s.Self())
        .FormActions(s => s.Self())
        .FrameAncestors(s => s.Self())
        .ImageSources(s => s.Self())
        .ScriptSources(s => s.Self())
    );
上述文章还推荐了一个在线扫描Header的网站：https://securityheaders.com/ 可以试试网站的安全评分。
    
按上述添加完后的响应的Header，Done：
    ![2018080101](/images/2018080101.png)




### 参考
[HTTP Strict Transport Security](https://developer.mozilla.org/zh-CN/docs/Security/HTTP_Strict_Transport_Security)
[HTTP Public Key Pinning 介绍](https://imququ.com/post/http-public-key-pinning.html)
[X-Frame-Options 响应头](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/X-Frame-Options)
[Reducing MIME type security risks](https://docs.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/compatibility/gg622941(v=vs.85))
[Content Security Policy 入门教程](http://www.ruanyifeng.com/blog/2016/09/csp.html)
[X-Frame-Options 响应头](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/X-Frame-Options)


