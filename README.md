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
  var url = query.domain + '?appid=' + query.appid + '&redirect_uri='+_self.$basePath+'/auth' +'&response_type=code&scope=snsapi_base&state='+query.state+'#wechat_redirect'
 window.location.href = url//路径跳转
```
* 判断参数有code（根据code获取用户信息）  
获取用户信息后跳转页面（最好replace）
### 微信支付的调用
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
### 微信的sdk的信息分享配置
``` javascript
 wx.config({
    debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
    appId: data.appId, // 必填，公众号的唯一标识
    timestamp: data.timeStamp, // 必填，生成签名的时间戳
    nonceStr: data.nonceStr, // 必填，生成签名的随机串
    signature: data.signature, // 必填，签名，见附录1
    jsApiList: ['checkJsApi',
      'onMenuShareTimeline',
      'onMenuShareAppMessage',
      'onMenuShareQQ',
      'onMenuShareWeibo',
      'showMenuItems'
    ] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
  });
  
  wx.ready(function () {
        var link = window.location.href;
        var protocol = window.location.protocol;
        var host = window.location.host;
        //分享朋友圈
        wx.onMenuShareTimeline({
            title: '这是一个自定义的标题！',
            link: link,
            imgUrl: protocol+'//'+host+'/resources/images/icon.jpg',// 自定义图标
            trigger: function (res) {
                // 不要尝试在trigger中使用ajax异步请求修改本次分享的内容，因为客户端分享操作是一个同步操作，这时候使用ajax的回包会还没有返回.
                //alert('click shared');
            },
            success: function (res) {
                //alert('shared success');
                //some thing you should do
            },
            cancel: function (res) {
                //alert('shared cancle');
            },
            fail: function (res) {
                //alert(JSON.stringify(res));
            }
        });
        //分享给好友
        wx.onMenuShareAppMessage({ //onMenuShareQQ    onMenuShareTimeline
            title: '这是一个自定义的标题！', // 分享标题
            desc: '这是一个自定义的描述！', // 分享描述
            link: link, // 分享链接，该链接域名或路径必须与当前页面对应的公众号JS安全域名一致
            imgUrl: protocol+'//'+host+'/resources/images/icon.jpg', // 自定义图标
            type: 'link', // 分享类型,music、video或link，不填默认为link
            dataUrl: '', // 如果type是music或video，则要提供数据链接，默认为空
            success: function () {
                // 用户确认分享后执行的回调函数
            },
            cancel: function () {
                // 用户取消分享后执行的回调函数
            }
        });
        wx.error(function (res) {
            alert(res.errMsg);
        });
    });
    
    //配置信息data
    data:{
      "URLLLLL":"https://m.tamaidan.com/testOAserver/deposit",  
      "appId":"wxa69d932fd28deb71",
      "nonceStr":"cde254ba68a44b8c86ef17727fbf1f1e",
      "signature":"66da685aeed95f309c7feee7c00863ac80f29805",
      "timeStamp":"1519617353"
    }
```
* [官方链接](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115)
