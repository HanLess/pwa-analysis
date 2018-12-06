#### service worker的主要特点

<ul>
  <li>独立线程</li>
  <li>一旦 install 则永远存在，除非手动 uninstall </li>
  <li>可编程拦截代理请求和返回，缓存文件，缓存的文件可以被网页进程取到（包括网络离线状态）</li>
  <li>能向客户端推送消息</li>
  <li>不能直接操作 DOM</li>
  <li>必须在 HTTPS 环境下才能工作</li>
  <li>异步实现，内部大都是通过 Promise 实现</li>
</ul>

#### service worker 依赖：Promise，html5 fetch，Cache API，目前主流浏览器已经支持

#### 注册工作线程要在window onload之后，以免阻塞页面的加载，影响用户体验

<a href="#life">service worker 的生命周期</a>

<a href="#cache">配合 caches 做离线缓存</a>

<h2 id="life">service worker 的生命周期</h2>

#### 初始化时

<ul>
  <li>installing：注册成功后到达 installing，执行 install 回调函数</li>
  <li>installed：安装成功，当 install 的回调函数执行成功后到达 installed</li>
  <li>activating：这是一个过渡状态，如果当前没有其他的service worker 则跳过</li>
  <li>activate：激活成功</li>
</ul>

示例如下：

<img src="https://github.com/HanLess/pwa-analysis/blob/master/imgs/init.png" />

#### 当 service worker 注册的 JS 发生了变化

<ul>
  <li>waiting：这时会新开一个 service workder，而旧的 service worker正在运行，这时候新的 service worker 会停在 waiting 状态（可以理解为已经经过了 installed，等待激活的状态，可也以认为是卡在 activating 状态），而新线程等待的是：把旧线程注销，替换掉</li>
</ul>

示例如下：

<img src="https://github.com/HanLess/pwa-analysis/blob/master/imgs/change.png" />

#### 这时候可以调用 self.skipWaiting() 来激活新线程，流程图如下

<img src="https://github.com/HanLess/pwa-analysis/blob/master/imgs/ws-update.png" />

<h2 id="cache">配合 caches 做离线缓存</h2>

#### fetch 的流

fetch 中的 request / response 都使用了流的概念，而流只能使用一次，如果再次使用，需要 response.clone() 来得到流的副本

event.respondWith 方法接收一个 promise 对象，且携带 response 信息，这个 response 就是返给页面的内容

```
var VERSION = 'v1';
// 缓存
self.addEventListener('install', function(event) {
  /*
    event.waitUntil 的作用是延迟线程安装，状态处于 installing ，只有传入的 promise 处于 resolved 状态才会转为 installed
  */
  event.waitUntil(
    caches.open(VERSION).then(function(cache) {
        console.log("installing start")
      return cache.addAll([
        './index.html',
        'https://static-fe.daojia.com/pt/project/h5-login-components/dist/main.css',
      ]);
    })
  );
});

// 捕获请求并返回缓存数据
self.addEventListener('fetch',function(event) {
    console.log("start fetch")

    // 监听 fetch 事件拦截请求，event.respondWith 可以返回指定的资源
    event.respondWith(
        // caches.match 用来根据请求匹配缓存资源
        caches.match(event.request).then(function(response){
            // 如果匹配到了直接返回
            if(response){
                return response
            }else{
                // 没有匹配到，response是undefined，则进行http请求，并将结果返回
                return fetch(event.request);
            }
        }).then(function(_response){
            // 这里把缓存更新
            caches.open(VERSION).then(function(cache) {
                cache.put(event.request, _response);
            });  
            // 返回结果，这里返回一个 response 的副本，因为 cache.put 已经使用过了 response 
            return _response.clone();
        }).catch(function(err){
            return caches.match('./index.html')
        })
    )
});
```

#### 缓存效果

<img src="https://github.com/HanLess/pwa-analysis/blob/master/imgs/cache.png" />



