## 微信公众号的总结
### 用户授权
* 判断无code参数（重定向后会获取code）
 ``` javascript
  var query = {
      domain: 'https://open.weixin.qq.com/connect/oauth2/authorize',//请求code的域名
      appid: 'wxa69d932fd28deb71',//appid
      scope: 'snsapi_userinfo',//snsapi_userinfo(非静默)
      state: saveSharingPerson(),//自定义分享的参数
      redirect_uri: encodeURIComponent(window.location.href),//wx重定向地址 redirect_uri
      response_type: 'code'
  }
  var url = query.domain + '?appid=' + query.appid + '&redirect_uri=' + 'https://m.tamaidan.com'+_self.$basePath+'/auth' +       '&response_type=code&scope=snsapi_base&state='+query.state+'#wechat_redirect'
 window.location.href = url//路径跳转
```
* 判断参数有code（根据code获取用户信息）  
获取用户信息后跳转页面（最好replace）
### weixin-js-sdk的配置
``` javascript 
import wx from 'weixin-js-sdk'
function payApi(data){
   return new Promise((resolve, reject) => {
      wx.config({
        debug: false, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
        appId: data.appId, // 必填，公众号的唯一标识
        timestamp: data.timeStamp, // 必填，生成签名的时间戳
        nonceStr: data.nonceStr, // 必填，生成签名的随机串
        signature: data.signature, // 必填，签名，见附录1
        jsApiList: ["chooseWXPay", "getLocation"] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
    });
    wx.ready(function() {
        if (typeof WeixinJSBridge == "undefined") {
            if(('typeof WeixinJSBridge == undefined')){
                resolve(res)
                return;
            }
            if(document.addEventListener) {
                document.addEventListener(
                    "WeixinJSBridgeReady",
                    onBridgeReady,
                    false
                );
            }else if (document.attachEvent) {
                document.attachEvent("WeixinJSBridgeReady", onBridgeReady);
                document.attachEvent("onWeixinJSBridgeReady", onBridgeReady);
            }
        }else{
            WeixinJSBridge.invoke(
                "getBrandWCPayRequest", {
                    appId: data.appId, //公众号名称，由商户传入
                    timeStamp: data.timeStamp, //时间戳，自1970年以来的秒数
                    nonceStr: data.nonceStr, //随机串
                    package: data.package,//订单详情扩展字符串  统一下单接口返回的prepay_id参数值，提交格式如：prepay_id=***
                    signType: "MD5", //微信签名方式：
                    paySign: data.paySign //微信签名
                },
                function(res) {
                    if (res.err_msg != "get_brand_wcpay_request:ok") {// 使用以上方式判断前端返回,微信团队郑重提示：res.err_msg将在用户支付成功后返回ok，但并不保证它绝对可靠。
                        reject(res)
                    } else {
                        resolve(res)
                    }
                }
            )}
        })
    })   
}

//返回data
 {
    "appId":"wxa69d932fd28deb71",
    "nonceStr":"0a1317ac6d5c4cf18cd223b5088d3fdd",
    "orderId":"102268847030590",
    "package":"prepay_id=wx20180226111446966f8718dc0989102255",
    "paySign":"9182F2CB8AAAFB3C694693FF5FF36BFC",
    "signType":"MD5",
    "signature":"b15aa9625efb36cda79d6d9e631f1bba8ad53e20",
    "timeStamp":"1519614886"
 }
```

* 调用  
``` javascript
payApi(options)
.then((res)=>{})//支付成功
.catch((err)=>{})//支付失败
```
