### 什么是PWA？

PWA全称是渐进式网络应用（progressive web apps），如果一个浏览器是支持pwa，则使用pwa模式，如果不支持则可以按照普通的方式进行解析。

### PWA有哪些优点

- 响应式

- 独立于网络连接

- 类似原生应用的交互体验

- 始终保持更新

- 安全

- 可发现

- 可重连

- 可安装

- 可链接



### PWA的组成

pwa就是最普通的网页的构成元素构成，比如html、css、js等，只是在基础上增加一些功能文件。pwa会指向一个清单文件（mainfest）文件，其中包含网站的相关信息、包括图标、背景屏幕、颜色和默认方向；pwa同时也使用了service worker的重要新功能，它可以令开发者深入网络请求并构建更好的web体验，同时使用service worker pwa就可以离线工作。

### service worker的理解

service workers（sw）可以让你全权控制网站发起的每一个请求，所以开发者可以重定向请求甚至停止请求。service workers也是有javascript编写的，不过有点不同的是，它是运行在它自己的全局脚本的上下文中，并不绑定到具体的页面，所以没办法修改页面中的DOM元素，由于权限太强大，为了安全考虑只能使用https。sw是由事件控制的，学习sw重点也是学习其事件的类型。

#### service workers的生命周期

下图展示了一个sw的生命周期，它会在用户访问该网站的时候发生。

![img](https://cdn.yuque.com/lark/0/2018/png/89927/1526638366282-3e90c704-a5d5-4275-a1ba-3c4158792663.png)

上面的生命周期大致是这样实现的：

当用户导航到页面的时候，服务器会返回响应的页面，如上图第1步所示，当调用register()函数的时候，sw开始下载，在注册的过程中，浏览器会下载、解析并执行sw（第2步）。如果在这个过程中出现任何的错误，register()返回的promise都会执行reject操作，并且sw会被遗弃。一旦sw成功执行了，install事件就会被激活(3)，一旦安装这步完成，sw便会激活并控制在其范围的一切，

总结：sw可以当成一组交通信号灯，在注册的时候sw处于红灯，因为它需要下载和安装，接下来是黄灯，因为它在执行，还没有完全准备好。等所有的工作都准备好之后，sw处于绿灯状态。



需要注意的一点，从上面的生命周期图可以看出，当你在第一次访问该网站的时候，并不会有激活的service worker来控制页面，只有当service worker安装完成之后并且用户刷新页面或者跳转至网站的其他页面的时候，servicer worker才会激活并开始拦截请求。这个情况并不是我们想要的。service worker已经有了完美的解决办法。通过使用skipwaiting()函数强制等待中的service worker被激活。skipwaiting()可以与self.clients.claim()函数一起使用。

```javascript
self.addEventListener('install', function(event){
    event.waitUntil(self.skipWaiting());
})
self.addEventListener('activate', function(event){
    event.waitUntil(self.clients.claim());
})
```



#### service workers示例

因为sw是运行在后台线程上的javascript文件，所以可以在html页面中直接引用，使用sw之前，需要注册。

```javascript
<html>
    <script>
     // 注册 service worker
    if ('serviceWorker' in navigator) {                                                                ❶
      navigator.serviceWorker.register('/sw.js').then(function(registration) {                         ❷
        // 注册成功
        console.log('ServiceWorker registration successful with scope: ', registration.scope);         ❸
      }).catch(function(err) {                                                                         ❹
        // 注册失败 :(
        console.log('ServiceWorker registration failed: ', err);
      });
    }
    </script>
</html>

❶ 检查当前浏览器是否支持 Service Workers
❷ 如果支持，注册一个叫做 'sw.js' 的 Service Worker 文件
❸ 如果成功则打印到控制台
❹ 如果发生错误，捕获错误并打印到控制台

//sw.js文件
// fetch 作用相当于ajax
self.addEventListener('fetch', function(event) {     ❶
  if (/\.jpg$/.test(event.request.url)) {            ❷
    event.respondWith(fetch('/images/unicorn.jpg')); ❸
  }
});

❶ 为 fetch 事件添加事件监听器
❷ 检查传入的 HTTP 请求是否是 JPEG 类型的图片
❸ 尝试获取独角兽的图片并用它作为替代图片来响应请求
```



### pwa的清单文件(manifest.json)

清单文件的作用主要可以包括开发者自定义启动画面，模版颜色等，清单文件只是一份简单的json配置文件。

在页面中使用link引用manifest.json文件<link rel='manifest' href='./manifest.json'>。

```javascript
{
  "name": "progressive web app",//用作当用户被提示安装应用时出现的文本。
  "short_name":"web app",//用作当应用安装后出现在用户主屏幕上的文本。
  "start_url":"/index.html",//决定了当用户从设备的主屏幕开启web应用时出现的第一个页面。
  "display":"standalone",//表示开发者希望web应用如何向开发者展示。
  "theme_color":"red",// 可以对浏览器的地址栏进行着色。
  "background_color":"yellow",
//决定了当web应用被添加到主屏幕时所显示的图标。
  "icons": [
    {
      "src": "./images/homescreen.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "./images/homescreen_1.png",
      "sizes": "144x144",
      "type": "image/png"
    }
  ]
}
```



#### 推送通知

发送推送通知可以分成三个步骤，1）首先我们需要提示用户并获得他们的订阅信息，2）然后把这些订阅信息保存到服务器上面，3）最后在需要的时候发送任何消息。在向用户发送通知前，我们要显示的提示用户，这一功能浏览器都是默认自带的。

```javascript
var endpoint;
      var key;
      var authSecret;
      var vapidPublicKey = 'BAyb_WgaR0L0pODaR7wWkxJi__tWbM1MPBymyRDFEGjtDCWeRYS9EF7yGoCHLdHJi6hikYdg4MuYaK0XoD0qnoY'; ❷

      function urlBase64ToUint8Array(base64String) {                                    ❸
        const padding = '='.repeat((4 - base64String.length % 4) % 4);
        const base64 = (base64String + padding)
          .replace(/\-/g, '+')
          .replace(/_/g, '/');
        const rawData = window.atob(base64);
        const outputArray = new Uint8Array(rawData.length);
        for (let i = 0; i < rawData.length; ++i) {
          outputArray[i] = rawData.charCodeAt(i);
        }
        return outputArray;
      }
      if ('serviceWorker' in navigator) {
        navigator.serviceWorker.register('sw.js').then(function (registration) {
          return registration.pushManager.getSubscription()                              ❹
            .then(function (subscription) {
              if (subscription) {                                                        ❺
                return;
              }
              return registration.pushManager.subscribe({                                ❻
                  userVisibleOnly: true,
                  applicationServerKey: urlBase64ToUint8Array(vapidPublicKey)
                })
                .then(function (subscription) {
                  var rawKey = subscription.getKey ? subscription.getKey('p256dh') : ''; ❼
                  key = rawKey ? btoa(String.fromCharCode.apply(null, new Uint8Array(rawKey))) : '';
                  var rawAuthSecret = subscription.getKey ? subscription.getKey('auth') : '';
                  authSecret = rawAuthSecret ?
                    btoa(String.fromCharCode.apply(null, new Uint8Array(rawAuthSecret))) : '';
                  endpoint = subscription.endpoint;
                  return fetch('./register', {                                           ❽
                    method: 'post',
                    headers: new Headers({
                      'content-type': 'application/json'
                    }),
                    body: JSON.stringify({
                      endpoint: subscription.endpoint,
                      key: key,
                      authSecret: authSecret,
                    }),
                  });
                });
            });
        }).catch(function (err) {
          // 注册失败 :(
          console.log('ServiceWorker registration failed: ', err);
        });
      }

❷ 客户端和服务端都需要公钥，以确保消息是加密过的
❸ 需要将 VAPID 钥从 base64 字符串转换成 Uint8 数组，因为这是 VAPID 规范要求的
❹ 获取任何已存在的订阅
❺ 如果已经订阅过了，则无需再次注册
❻ 还没有订阅过，则显示一个提示
❼ 需要从订阅对象中获取 key 和 authSecret
❽ 最后，将详细信息发送给服务器以注册用户
```

服务器的代码如下

```javascript
const webpush = require('web-push');                   ❶
const express = require('express');
var bodyParser = require('body-parser');
const app = express();
webpush.setVapidDetails(                               ❷
  'mailto:contact@deanhume.com',
  'BAyb_WgaR0L0pODaR7wWkxJi__tWbM1MPBymyRDFEGjtDCWeRYS9EF7yGoCHLdHJi6hikYdg4MuYaK0XoD0qnoY',
  'p6YVD7t8HkABoez1CvVJ5bl7BnEdKUu5bSyVjyxMBh0'
);
app.post('/register', function (req, res) {            ❸
  var endpoint = req.body.endpoint;
  saveRegistrationDetails(endpoint, key, authSecret);  ❹
  const pushSubscription = {                           ❺
    endpoint: req.body.endpoint,
    keys: {
      auth: req.body.authSecret,
      p256dh: req.body.key
    }
  };
  var body = 'Thank you for registering';
  var iconUrl = 'https://example.com/images/homescreen.png';
  webpush.sendNotification(pushSubscription,           ❻
      JSON.stringify({
        msg: body,
        url: 'http://localhost:3111/',
        icon: iconUrl
      }))
    .then(result => res.sendStatus(201))
    .catch(err => {
      console.log(err);
    });
});
app.listen(3111, function () {
  console.log('Web push app listening on port 3111!')
});

❶ 添加必要的依赖
❷ 设置 VAPID 详情
❸ 监听指向 '/register' 的 POST 请求
❹ 保存用户注册详情，这样我们可以在稍后阶段向他们发送消息
❺ 建立 pushSubscription 对象
❻ 发送 Web 推送消息
```



## PWA踩坑经历

1、需要被注册的service worker文件（sw.js）文件需要放在项目的根目录下面，否则在sw监听http请求的时候（fetch）事件的时候，不能监听到页面的请求。或者通过在nginx中添加响应头add_header Service-Worker-Allowed "设置scope作用域"。

2、scope作用域指的是path路径，页面路径。而不是资源路径。比如如果要监听http:localhost:9080/g2/这个路径下面的资源。则scope指定的是/g2/，之后就会监听指定页面的所有fetch事件。

3、使用cacheAPI来缓存资源的使用，不支持post请求，需要过滤。

