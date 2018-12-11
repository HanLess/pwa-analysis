# pwa-analysis

参考文章：https://huangxuan.me/2017/02/09/nextgen-web-pwa/

pwa 的五个内容

<ul>
  <li>manifest：web app 的桌面化</li>
  <li>service worker</li>
  <li>cache 缓存</li>
  <li>push 消息推送</li>
  <li>notification 消息提醒</li>
</ul>

业务流程

<ul>
  <li>service worker 是主体，其他三个功能都依赖 service worker</li>
  <li>利用 cache api 缓存静态资源</li>
  <li>利用 push api，由服务端向客户端主动推送消息</li>
  <li>利用 notification api 对用户弹出消息</li>
</ul>

