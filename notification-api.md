浏览器消息提醒可以分以下几步

<ul>
  <li>请求用户授权</li>
  <li>注册service worker</li>
  <li>showNotification 展示提示栏</li>
  <li>service worker 监听 notificationclick 事件</li>
</ul>

#### 请求用户授权

用户授权有三种状态：

<ul>
  <li>denied：拒绝</li>
  <li>default：默认，即处于未知状态，跟 denied 一个效果</li>
  <li>granted：授权成功</li>
</ul>

```
Notification.requestPermission().then(function(result) {
  console.log(result)
  if (result === 'denied') {
    console.log('Permission wasn\'t granted. Allow a retry.');
    return;
  }
  if (result === 'granted') {
    console.log('The permission request was dismissed.');
    return;
  }
});
```

#### 注册 service worker 后调用 showNotification

```
navigator.serviceWorker.register("./type.js",{scope : "/"}).then(function(registration){
  var title = 'PWA即学即用';
  var options = {
    body: '邀请你一起学习',
    icon: '/img/icons/book-128.png',
    actions: [{
      action: 'show-book',
      title: '去看看'
    }, {
      action: 'contact-me',
      title: '联系我'
    }],
    tag: 'pwa-starter',
    renotify: true
  };
  registration.showNotification(title, options);

})
```

#### 监听 notificationclick 事件

```
self.addEventListener('notificationclick', function (e) {
    var action = e.action;
    console.log(`action tag: ${e.notification.tag}`, `action: ${action}`);
    
    switch (action) {
        case 'show-book':
            console.log('show-book');
            break;
        case 'contact-me':
            console.log('contact-me');
            break;
        default:
            console.log(`未处理的action: ${e.action}`);
            action = 'default';
            break;
    }
    e.notification.close();
});
```

#### 以配合push api，监听到 push 事件后调用 showNotification，弹出推送消息

#### 在监听到 notificationclick 事件后，可以通过 service worker 与主线程通信（详见 service worker 内容），对用户的动作进行处理





