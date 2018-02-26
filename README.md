## 微信公众号的总结
### 授权
* 判断无code参数
 ``` javascript
  var query = {
      domain: 'https://open.weixin.qq.com/connect/oauth2/authorize',//请求code的域名
      appid: 'wxa69d932fd28deb71',
      scope: 'snsapi_userinfo',//snsapi_userinfo(非静默)
      state: saveSharingPerson(),
      redirect_uri: encodeURIComponent(window.location.href),//wx重定向地址 redirect_uri
      response_type: 'code'
  }
  var url = query.domain + '?appid=' + query.appid + '&redirect_uri=' + 'https://m.tamaidan.com'+_self.$basePath+'/auth' +       '&response_type=code&scope=snsapi_base&state='+query.state+'#wechat_redirect'
window.location.href = url
```
