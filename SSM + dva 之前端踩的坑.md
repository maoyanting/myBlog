---
title: SSM + dva 之前端踩的坑
date: 2017-12-13 18:02:50
categories: 
- 前端
tags:
---

## 技术栈

后端：SSM + Tomcat 
前端：dva + roadhog + antd + React

**提示**：dva的坑很多，react的体系很大（学习成本很高）不过github上Q&A很多也比较全面 
然后其实这个适合做后台页面，所以才封装得那么像java吧

------

# 前后端衔接部分：

*网上找了很多都没有我能用的，我自己最后弄了个比较蠢的方法*

首先往.roadhogrc里面添加

```
"publicPath": "./",
"outputPath": "你的webapp的路径",12
```

`publicPath` 修改是为了保证到时候build出来的文件的静态目录不出错

*其实这里很奇怪，为什么webpack自己本身不去弄好这个*

前端的代码写好以后，直接npm build到webapp的目录下 
**这步操作先会让webapp里面所有的文件清空** 
我自己比较蠢的方法就是把web.xml文件复制出来，每次build后再复制进去

更新：后来觉得这样做实在是太傻逼兼太麻烦了 
在.roadhog里面加上了这个代理

```
  "proxy": {
      "/api": {
        "target": "http://localhost:8080/exampleone/",
        "changeOrigin": true,
        "headers":{
        "host":"localhost:8080/exampleone/"
        },
        "pathRewrite": { "^/api" : "" }
      }
    },12345678910
```

这里面的内容可以参考：[https://www.jianshu.com/p/3bdff821f859](https://www.jianshu.com/p/3bdff821f859) 
service里面和后端的交互：

```
export async function login(data) {
  return request('/api/login', {
  /* 之前这里是/exampleone/login。我是build后使用这个方式*/
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(data),
  });
}12345678910
```

Spring MVC里面 
对应的`@requestMapping（value="/login"）` 
只要开着Tomcat，前端这边npm start后都能访问到后端的数据。开心ヾ(❀╹◡╹)ﾉ~

emmmmm……不过这个其实导致了后面很多的问题…… 
首先就是cookies丢失的问题……

------

# 问题Q&A：

## Q：build完的文件，启动tomcat，开始还能使用，后来再次build的时候就url显示无法获取，各种404

A：我在index.js里面修改了history为`browserHistory` 
默认是`hashHistory`，就是会在url里面加了一个#的那种，build出来的文件是可以在tomcat里面看见的，路由跳转也没问题； 
⚠️但是！发起request的时候就不行了

看了下资料： 
**官方推荐的是browserHistory**，但是需要是服务器的支持。 
hashHistory在开发阶段比较好用，不需要服务器 
服务器的支持，官方表示请**把所有请求都转发到index.html**（喵喵喵？总觉得是一个很不优雅的办法）

但是，转发后还是有问题，只有背景图片显示了，检查下，js和css应该都是加载了的，但是就是不正常显示。

这里我花了两天时间来搞这个问题……

## BrowserHistory+Tomcat build后页面空白情况下的最终解决方案：

**后端Java部分**

参考：[https://stackoverflow.com/questions/41246261/react-routing-is-able-to-handle-different-url-path-but-tomcat-returns-404-not-av/41249464#41249464](https://stackoverflow.com/questions/41246261/react-routing-is-able-to-handle-different-url-path-but-tomcat-returns-404-not-av/41249464#41249464) 
在web.xml文件内加上：

```
<error-page>
    <error-code>404</error-code>
    <location>/index.html</location>
</error-page>1234
```

**前端部分**

.roadhogrc文件：

```
"publicPath": "/exampleone/"1
```

index.js文件

```
import dva from 'dva';
import createHistory from 'history/createBrowserHistory';

// 1. Initialize
const app = dva({
  history: createHistory({ basename: '/exampleone' }),
  ...
})12345678
```

参考：[https://github.com/sorrycc/roadhog/issues/187](https://github.com/sorrycc/roadhog/issues/187) 
这里参考链接里面的是用的react-router3，但是最新的dva带的是react-router4，所以一开始一直走不通，提示useRouterHistory is not a function 
后来参考了[https://github.com/ReactTraining/react-router/issues/4006](https://github.com/ReactTraining/react-router/issues/4006)

至此，前后端服务器下页面都能正常显示了 
这里有个小问题：因为这里其实我是把basename写死了，在项目迁移的时候肯定有问题。

其实不写basename也有一种方法，就是把这个项目tomcat的路径写到根目录就是ROOT里面。 
这样只需要修改后端的web.xml，启动tomcat的时候，localhost:8000就能正常加载了。 
一般来说，一个tomcat下面只有一个项目，所以其实这样最好。

------

## Q：提交表单

A： 
routes：return表单，需要提交的时候dispatch一models里面的effect 
models：effect会yield call 已经import进来的service里面的action 
service：action会调用request，把url和数据等传输给对应的后端接口 
models：yield完毕后，能获取到responce传过来的数据

------

## Q：request里面的url指向的是后端的对应的url吗？就是controller里面的RequestMapping？

A： 
RequestMapping里面的是api请求的路径 
在service里面request的就是这个。

------

## Q：显示chatList，正在聊天的列表

A： 
遍历app/sendMessage里面的消息，获取所有的对话用户id。 
放入chatList 
展示chatList的时候，根据用户id，去friendList寻找用户数据。

------

## Q：每次request没有cookies，导致每次的sessionId不同

A：开始认为是跨域的锅 
同源：[http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html](http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html) 
同源： 
协议相同、域名相同、端口相同 
导致的问题： 
1、Cookie、LocalStorage 和 IndexDB 无法读取。 
2、DOM 无法获得。 
3、AJAX 请求不能发送。

跨域：[http://www.ruanyifeng.com/blog/2016/04/cors.html](http://www.ruanyifeng.com/blog/2016/04/cors.html) 
上网查了下，按照：[https://github.com/dvajs/dva/issues/139](https://github.com/dvajs/dva/issues/139) 
来写，添加了 
`const options= { method: "GET",mode: 'cors',credentials: 'include'};` 
再次运行的时候，cookies是有了，但是依旧没有sessionId。 
这里有个点：虽然**普通的请求没有sessionId，但是webSocket的请求里面出现了sessionId。**

webSocket的请求和其他请求不同的地方就在，webSocket是直接设置了 
`const socket = new WebSocket('ws://localhost:8080/exampleone/ws');` 
我们之前设置的proxy，其实应该是已经做了代理，不存在跨域问题。 
所以这个是为什么内？(っ╥╯﹏╰╥c)

最终方案： 
前端： 
`const options= { method: "GET",mode: 'cors',credentials: 'include'};` 
后端： 
Tomcat/conf/context.xml里面修改`<Context sessionCookiePath="/">` 
responce加上

```
response.setHeader("Access-Control-Allow-Origin", "http://localhost:8000");
response.setHeader("Access-Control-Allow-Methods", "POST, GET, OPTIONS, DELETE");
response.setHeader("Access-Control-Allow-Credentials","true");123
```

我觉得是因为我应用部署在了tomcat的子目录下，导致返回的sessionId的path是/exampleone。但是前端在request的时候是去根目录找的cookies，就错过了/exampleone下已经存在的sessionId눈_눈

------

## Q：验证是否登录状态

A：本来打算验证登录慢慢做的，毕竟其实我就一个页面。 
后来发现我还是太naive了，页面一刷新就报错了。

我自己脑洞的方法： 
登录后，每次的sessionId是一样的，后端session保存了你的user数据 
只要后端发现你是sessionId是一致的，就能知道你是在登录的状态。

登出后，后端删除这个session，即使是同一个浏览器过来的数据，它没有找到session，就会新给一个sessionId。

前端好像一般会用token 
[https://tkvern.com/20170204/Dva%20+%20Ant%20Design%20%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB%E4%B9%8B%20React%20%E5%BA%94%E7%94%A8%E5%AE%9E%E8%B7%B5/](https://tkvern.com/20170204/Dva%20+%20Ant%20Design%20%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB%E4%B9%8B%20React%20%E5%BA%94%E7%94%A8%E5%AE%9E%E8%B7%B5/)

实现： 
每次刷新的时候state里面的内容会不见。 
每次request的时候，拦截，先查看sessionId有没有保存。

------

## Q：webSocket的实现

A： 
*先说一下别的轮询方式：* 
ajax轮询：让浏览器隔个几秒就发送一次请求，询问服务器是否有新信息。 
long poll：浏览器发起连接后，如果没消息，就一直不返回Response给浏览器。直到有消息才返回，返回完之后，浏览器再次建立连接，周而复始。

ajax轮询 需要服务器有很快的处理速度和资源。 
long poll 需要有很高的并发。

*再说webSocket，*先必须记住，**webSocket是一个协议** 
我们使用的应该说是webSocket API

WebSocket是基于TCP的独立的协议。 
和HTTP的唯一关联就是HTTP服务器需要发送一个“Upgrade”请求，即101 Switching Protocol到HTTP服务器，然后由服务器进行协议转换 
//简单来说，就是我是从TCP那边继承过来的，干活需要依靠HTTP先帮我连接上

**最简单的实现：**

```
var ws = new WebSocket("wss://echo.websocket.org");

ws.onopen = function(evt) { 
  console.log("Connection open ..."); 
  ws.send("Hello WebSockets!");
};

ws.onmessage = function(evt) {
  console.log( "Received Message: " + evt.data);
  ws.close();
};

ws.onclose = function(evt) {
  console.log("Connection closed.");
}; 123456789101112131415
```

其实看起来很简单，实际上好像也就这些。 
//但是我疑问的是，java中，api是需要导入的。但是js中好像没有这个操作。

还有一个问题， 
前端是用socket.io比较多，因为部分浏览器不支持websocket 
java后端用socket的我就找到了一个netty-socket.io，但是我用的是tomcat。

下面在写我的项目的过程中，首先需要知道的就是，这个webSocket需要写在哪里？

**有人建议单独创建连接实例，在subscription中设置连接的相关回调方法。** 
**在入口Component的componentDidMount中dispatch action开始连接，componentWillUnmount中断开连接。** 
//上面的componentDidMount和componentWillUnmount与组件生命周期有关，请自行百度。

最终方法： 
routes/chat 写onmessage 
models/app 写一个effect，const一个webSocket和相关方法 
login后，调用这个effect

## Q:webSocket403错误

A:加上setAllowedOrigins

------

## Q:webSocket没有session

A:跨域问题导致的 
[https://github.com/dvajs/dva/issues/139](https://github.com/dvajs/dva/issues/139)

------

## Q:群聊通过url分享

A:自己搞得一个小功能 
群聊没办法加用户，只能把群分享一个链接出去 
链接：群id 
用户在浏览器输入这个链接， 
进入一个页面，按钮：加入群。 
登录情况下，会自动加入这个群。 
非登录情况下，会先跳转到登录界面，再加入这个群