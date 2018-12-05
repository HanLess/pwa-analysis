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

<h2>service worker 的生命周期</h2>

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
