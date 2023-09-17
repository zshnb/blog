---
title: 微信公众号开发踩坑记
date: 2023-09-17 14:17:49
tags:
- Java
- Wechat
---

# 背景
最近业务经常需要针对微信公众号开发，比如处理用户关注公众号事件，微信网页扫码登录及绑定帐号等。在开发过程中遇到许多问题，搜索了许多前人的文章，加上自己开发过程中的一些思考，特此来做个总结。
<!--more-->
# 过程

## 准备

1. 首先我们需要申请一个公众号，申请成功后进入**开发/基本配置**页面

![](img1.png)

AppID，AppSecret：用来获取accessToken，accessToken是调用微信一切接口的凭证

IP白名单：当服务端在本地调试时想调用微信接口时，需把本地ip地址加入白名单中

URL：服务器上暴露的接口地址，必须是域名形式，不能是IP

令牌：任意值，后续解析加密消息用

密钥：随机生成值，后续解析加密消息用。如果要使用[weixin-mp-java](https://github.com/Wechat-Group/WxJava) sdk开发，不能直接使用公众号网页上生成的随机key，解决办法是使用`Base64.encodeBase64String(UUID.randomUUID().toString().replaceAll("-","").getBytes()); `生成44位的字符串，然后去除末尾=号作为key

消息加密方式：是否加密传输的消息，可选择明文，明文密文，仅密文

准备工作已完成，然后就可以开发了

## 开发

2. 在[springboot初始化项目生成页](https://start.spring.io/)生成一个项目，然后加入以下依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   
   <dependency>
       <groupId>com.github.binarywang</groupId>
       <artifactId>weixin-java-mp</artifactId>
       <version>3.8.0</version>
   </dependency>
   ```

   公众号接入指南中提到微信会往上一步填写的URL上发送校验签名请求，服务器必须对请求作出响应，才能完成接入。新建`WechatController`

   ```java
   // 此接口的作用是接收微信校验签名以及消息请求
   // msgSignature：加密的消息参数，当加密方式设置为密文时，微信会在请求里带上此参数
   // 其余参数请参考https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/Access_Overview.html接入指南
   @RequestMapping("/wechat")
   @ResponseBody
   public String wechat(HttpServletRequest request,
                        @RequestParam(value = "msg_signature", required = false) String msgSignature,
                        @RequestParam("signature") String signature,
                        @RequestParam("timestamp") String timestamp,
                        @RequestParam("nonce") String nonce,
                        @RequestParam(value = "echostr", required = false) String echostr) throws IOException {
   
       if (wxMpService.checkSignature(timestamp, nonce, signature)) {
           // GET请求味校验签名接口
           if (request.getMethod().equals("GET")) {
               System.out.println("success");
               return echostr;
           } else {
               // 接收微信消息请求根据WxMpXmlMessage的msgType判断消息类型，然后根据类型处理相应业务逻辑
               WxMpXmlMessage inMessage = WxMpXmlMessage.fromEncryptedXml(request.getInputStream(), wxMpConfigStorage,
                                                                          timestamp, nonce, msgSignature);
               System.out.println(inMessage.toString());
               return "";
           }
       } else {
           System.out.println("failed");
           return "";
       }
   }
   ```

   3. 接下来是网页授权和扫码功能的开发，场景一般是微信内访问网页以及网页上提供微信扫码服务

      根据[网页授权](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/Wechat_webpage_authorization.html)里的说明，需要以下几个接口

      - 生成提示用户授权的链接

        ```java
        // redirectUrl：用户同意授权后重定向的链接，可以为html地址或者服务端处理后续业务接口，重定向链接中会携带code，作为获取accessToken的凭据
        // state：随机数，可以用来存储用户状态，扫码登录的时候会用到
        @RequestMapping("/authorization-url")
        @ResponseBody
        public String getAuthorizationUrl(@RequestParam("redirectUrl") String redirectUrl,
        								  @RequestParam("state") String state) {
            return wxMpService.oauth2buildAuthorizationUrl(redirectUrl, "snsapi_userinfo", state);
        }
        ```

      - 获取用户信息

        ```java
        @RequestMapping(value = "", method = RequestMethod.GET)
        @ResponseBody
        public String xxx(@RequestParam("code") String code) throws WxErrorException {
            WxMpOAuth2AccessToken accessToken = wxMpService.oauth2getAccessToken(code);
            WxMpUser wxMpUser = wxMpService.oauth2getUserInfo(accessToken, "zh_CN");
            String openId = wxMpUser.getOpenId();
            String unionId = wxMpUser.getUnionId();
            // 拿到openId和unionId之后，可以做相应的业务逻辑，比如绑定微信帐号，用微信登录等等
        
            return "success";
        }
        ```

   4. 扫码功能。关于扫码具体代码就不贴了，说下思路，结合上述代码实现也很简单

      1. 前端调用生成提示用户授权的链接，然后把链接变成二维码，展示给用户，同时生成一个随机数，存入map中
      2. 服务端提供一个检查扫码状态接口，检查随机数对应的微信unionId是否为空，并返回相应的状态给前端
      3. 如果用户确认授权，微信会调用服务端确认授权的回调接口，把随机数对应的微信unionId设置好
      4. 前端轮询扫码状态接口，并在网页上作出相应的提示

