消息推送允许服务端向浏览器推送消息

<img src="https://github.com/HanLess/pwa-analysis/blob/master/imgs/pushFlow.png" />

#### push service

push service 是一个中间服务，由浏览器提供，它有两个作用

<ul>
  <li>提供 PushSubscription 对象，对象中的关键参数是 endpoint ，endpoint 是一个 url，代表这个用户端的地址</li>
  <li>服务端推送消息时，充当中转站，完成 服务端 -> push service -> 浏览器 的流程 </li> 
</ul>

#### 如何保证push的安全性，即服务端如何准确地找到对应的浏览器

<ul>
  <li>PushSubscription</li>
  <li>自定义私钥、公钥；客户端拿公钥，服务端拿私钥，通过比对秘钥实现对应关系</li> 
</ul>

#### 流程

<ul>
  <li>
    serviceWorker.register 完成注册，返回 registration 对象，通过 registration.pushManager.subscribe 完成订阅（向 push service 订阅），push service 返回 pushSubscription 对象
  </li>
  <li>把 pushSubscription 发给服务端，由服务端统一存储（post请求即可）</li>
  <li>服务端存储 pushSubscription 对象</li>
  <li>可以通过运营后台，发送推送消息：通过npm模块 web-push ，使用 sendNotification 发送推送消息</li>
  <li>发送后前端的接收：service worker 监听 self.addEventListener('push')</li>
  <li>监听到 push 事件后，可以通过 self.registration.showNotification 来展示推送消息，或做其他业务处理</li>
</ul>

#### 实现代码

```
// 用到的工具方法


function sendSubscriptionToServer(option){
  return new Promise(function(resolve){
    $.ajax({
      type : "post",
      data : {
        subscription : JSON.stringify(option)
      },
      dataType : "json",
      url : "http://127.0.0.1:3000/pushSub",
      success : function(_data){
        console.log(_data)
        resolve(_data)
      }
    })
  })
}

function urlBase64ToUint8Array(base64String) {
  const padding = '='.repeat((4 - base64String.length % 4) % 4);
  const base64 = (base64String + padding)
    .replace(/\-/g, '+')
    .replace(/_/g, '/')
  ;
  const rawData = window.atob(base64);
  return Uint8Array.from([...rawData].map((char) => char.charCodeAt(0)));
}
function subscribeUserToPush(registration, publicKey) {
  var subscribeOptions = {
    userVisibleOnly: true,
    applicationServerKey: urlBase64ToUint8Array(publicKey)
  }; 
  return registration.pushManager.subscribe(subscribeOptions).then(function (pushSubscription) {
    return pushSubscription;
  });
}


/* 

一 前端订阅 

*/

if ('serviceWorker' in navigator && 'PushManager' in window) {
var publicKey = 'BOEQSjdhorIf8M0XFNlwohK3sTzO9iJwvbYU-fuXRF0tvRpPPMGO6d_gJC_pUQwBT7wD8rKutpNTFHOHN3VqJ0A';
// 注册service worker
navigator.serviceWorker.register('./type.js').then(function (registration) {
  console.log('Service Worker 注册成功');
  // 开启该客户端的消息推送订阅功能
  return subscribeUserToPush(registration, publicKey);
}).then(function (subscription) {
  console.log(subscription)
  var body = {subscription: subscription};
  // 为了方便之后的推送，为每个客户端简单生成一个标识
  body.uniqueid = new Date().getTime();
  // 将生成的客户端订阅信息存储在自己的服务器上
  return sendSubscriptionToServer(body);
}).catch(function (err) {
  console.log(err);
});
}
  
  
/*

  二 服务端用 nodejs 实现
    存储 subscription 对象
    发推送消息
*/
var express = require('express');
var router = express.Router();
const webpush = require('web-push');

var subscription 

const vapidKeys = {
  publicKey: 'BOEQSjdhorIf8M0XFNlwohK3sTzO9iJwvbYU-fuXRF0tvRpPPMGO6d_gJC_pUQwBT7wD8rKutpNTFHOHN3VqJ0A',
  privateKey: 'TVe_nJlciDOn130gFyFYP8UiGxxWd3QdH6C5axXpSgM'
};

// 设置web-push的VAPID值
webpush.setVapidDetails(
  'mailto:alienzhou16@163.com',
  vapidKeys.publicKey,
  vapidKeys.privateKey
);

function pushMessage(data) {
  webpush.sendNotification(subscription, data).then(data => {
      console.log('-------------------------------------push service的相应数据:', data);
      return;
  }).catch(err => {
        console.log(subscription);
        console.log(err);
  })
}

/* 在这个方法里存 subscription 对象，相当于初始化  */
router.post('/', function(req, res, next) {
    res.header("Access-Control-Allow-Origin", "*");
    subscription = JSON.parse(req.body.subscription).subscription
    console.log(JSON.stringify(subscription))
    res.send("subcribe success")
});

/*
  这里处理推送，运营后台可以调这个接口
*/
router.get("/push",function(req,res){
    var data = req.param('name')
    pushMessage(data);
    res.send("ok" + JSON.stringify(data))
})



/*

  三 前端接收消息

*/

self.addEventListener('push', function (e) {
    var data = e.data;
    if (e.data) {
        data = data.json();
        console.log('push的数据为：', data);
        // 展示推送
        self.registration.showNotification(data.text);        
    } 
    else {
        console.log('push没有任何数据');
    }
});
```

### 国内用不了 push service，被墙了，会报错：connect ETIMEDOUT xxx.xxx.xxx.xxx:443 




