#### wep app 的桌面应用

```
<link rel="manifest" href="/manifest.json">
```

```
{
  "short_name": "Manifest Sample",
  "name": "Web Application Manifest Sample",
  "icons": [{
      "src": "launcher-icon-2x.png",
      "sizes": "96x96",
      "type": "image/png"
   }],
  "scope": "/sample/",
  "start_url": "/sample/index.html",
  "display": "standalone",
  "orientation": "landscape"
  "theme_color": "#000",
  "background_color": "#fff",
}
```

#### 目前兼容性不好
