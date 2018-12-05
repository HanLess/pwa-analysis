### service worker的主要特点

<ul>
  <li>独立线程</li>
  <li>一旦 install 则永远存在，除非手动 uninstall </li>
  <li>可编程拦截代理请求和返回，缓存文件，缓存的文件可以被网页进程取到（包括网络离线状态）</li>
  <li>能向客户端推送消息</li>
  <li>不能直接操作 DOM</li>
  <li>必须在 HTTPS 环境下才能工作</li>
  <li>异步实现，内部大都是通过 Promise 实现</li>
</ul>
