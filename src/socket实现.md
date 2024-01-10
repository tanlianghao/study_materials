

## 背景

项目需要构建用户双方的直接沟通。故需要构建单对单的聊天室功能。

## 项目初探问题集结

1. 技术选型。
2. 项目构建打包。
3. 怎么实现单对单的聊天室。
4. 页面中的下拉刷新。

...

### 项目构建遇到的问题

1. 为了调通九游rpc跟hsf接口，使用了@ali/egg-aliplay-apiproxy跟@ali/egg-aliplay-sfclient这两个插件，由于插件版本问题，导致在调用接口的时候不能使用路径的形式，最后更换到跟frontend相同版本。使用@ali/egg-hsfclient插件来调用hsf接口，由于jar包版本没有修改，导致一直不能拉取到jar包。
2. 在把项目提交到git上的时候，因为没有规定好.gitignore文件，导致后面很多不需要提交的文件都被提交上。
3. 定义eslintrc规则的时候，忘记继承一些规则。

### 编码过程遇到的问题

1. 因为要实现的是单对单的聊天，所以需要加入一个标识，socket.io提供创建room的能力，房间号需要后端同学给出。

```javascript
var roomNumber = $.trim($('#chatroomNumber').val());
var socket = io('/', {
    query: {
        room: roomNumber, // 房间号
        userId: 'client_' + Math.random()
    },
    transports: ['websocket']
});
```

1. 在使用ctx.cookies获取值的时候，一直获取不到。原因是egg框架中，默认cookie值是加签不加密，如果在获取的时候，不加上signed属性，则获取不到值。`const usedTicket   = cookies.get('used_ticket', { signed: false });`
2. 应用启动的时候，连接上服务器之后，socket.io则马上进行连接。会触发egg-socket.io中的中间件。当进入页面的时候，会先触发egg服务。而后socket.io先进行http连接，后在转化成websocket连接。
3. 由于连接的服务不同，http/websocket，走的路线不同从而导致在egg框架中间件中ctx.locals添加的值，在websocket中是获取不到的。
4. 在使用iscroll.js插件出现问题。因为聊天室的性质，使得用户在进入页面的时候，会话是拉在最下面的，最开始是算出需要移动的高度，然后使用translate属性来移动，导致的结果就是，当用户在点击滚动区域的时候，会重新定位到 0，0的位置。解决方案是使用自带的startY属性来设置。当发送消息的时候，需要最下面的消息把上面的消息顶上去，也可以使用iscroll.js实例的scrollTo()方法来设置。（ps：文档看的不仔细）
5. 输入框键盘调起问题。不要使用fixed定位。存在某些浏览器兼容性问题，比如uc浏览器在调起输入框的时候会缩小页面比例导入输入框别键盘遮住问题。

### 项目上线之后遇到的问题

#### 问题：

线上测试的时候，会出现能发信息，当不一定能收到信息。查看相关资料发现是因为线上服务是集群的原因，当两个用户在加入到聊天室的时候，可能会被分配到不同的机器上面，而socket信息是暂存在机器上的。当在发送消息的时候，socket.io会从当前的服务器上面向该房间的成员广播信息，由于双方不是在同一个机器，导致查询不到其他成员。从而出现这种情况，同理多台tengine时也会出现同样的问题。

#### 解决方案：

egg-socket.io推荐的方案是使用内置的socket.io-redis来保存并推送消息。redis是一个VK类型的服务器。在确定方案之后，当连接redis服务集群的时候，出现连接失败的。原因是redis服务有多种连接模式，经过确认，集团使用的redis使用的是sendine（哨兵模式）,随后又出现socket.io-redis插件目前不支持哨兵模式连接。所以只能把插件内置到应用中，在plugin.js文件中，使用path来指定插件的路径。在egg-socket.io中使用ioredis插件，该插件是支持sendine模式的。代码如下：



```javascript
exports.io = {
    init: {},
    namespace: {
        '/': {
            connectionMiddleware: [ 'auth' ],
            packetMiddleware: []
        }
    },
  // redis多实例情况下，password需要放在外面。
    redis: {
        name: 'gcmall_chat_6398',
        password: '1lN2ie4j9x6rMx2hpre16rXG',
        sentinels: [
            {
                host: '11.12.72.32',
                port: 26398
            },
            {
                host: '11.12.72.33',
                port: 26398
            },
            {
                host: '11.12.72.38',
                port: 26398
            }
        ]
    }
};
```

### 申请redis机器遇到的问题

在申请redis服务的时候，需要发送申请邮件。需要标明容量，QPS等数据。峰值QPS计算方式如下，80%的访问量，将在20%的时间内进行访问；公式：( 总PV数 * 80% ) / ( 每天秒数 * 20% ) = 峰值时间每秒请求数(QPS)，根据机器的QPS值能知道总共需要多少台机器：峰值时间每秒QPS / 单台机器的QPS = 需要的机器数。

二期主要的功能点在于 已读/未读消息状态、消息卡片推送，先聊后买等功能。遇到的难点主要是集中在消息状态跟卡片推送，下面总计这两点遇到的困难。

### 消息状态变更

要使状态变更，是通过把曝光过的commendid发送给后端，后端返回保存成功的commendid，前端再根据数据通过js进行修改展示的状态。难点在于怎么算曝光过的commendid,调用ele.getBoundingClientRect()函数来进行判断。后端返回的数据，必须通过socket.io广播来通知对方。这样才能在对方测把消息状态修改。

### 卡片推送

卡片推送的难点在于实时推送。即当用户在线，且购买了商品之后，此时系统需要给买卖家双方都发送卡片消息。因为买卖家双方需要推送的卡片内容不一样，所以需要区分开推送。经过跟后端同学的讨论最终采用的是双方使用redis进行推送。前端服务通过实例化一个redis对象，在该对象的基础上订阅某个channel，后端在需要推送卡片的时候对这个channel进行推送消息。在redis对象上面监听message事件。来获得卡片消息。然后把消息通过socket推送出去，如果把实例化redis操作跟用户行为绑定起来的话，最终用户每发送一个行为都将导致redis实例化一次。因此需要把实例化操作，跟用户行为解偶。

由此，首先想到的把实例化的操作放到app.js文件上。因为app.js文件只执行一次。在本地跟后端联调很完美，没有任何的问题，然而，在app.js文件中是拿不到用户的socketid信息。因此就导致了我们后面采用redis来存储socketid的信息，当用户链接的时候，在io中间件中把用户的socketid存储起来。当有卡片消息推送的时候，在app.js通过redis获取出来。redis.get方法是异步方式。

在打包上测试环境后，偶尔出现用户不能实时收到卡片消息。之前一直以为是代码问题。最后才发现，原来是因为app.js文件没有打包到。原因是在app.js是新增文件。而打包脚本是移植frontend项目的。而在项目根目录下的文件打包不是通配符的形式，而是写死的，导致新增文件没有在打包脚本中。为什么在第一次打包上测试环境的时候没有发现，主要是因为在本地起了服务，链接上了redis服务器，导致在有推送的是，本地的服务接收到信息之前，成功推送了，所以一直没发现问题。

在app.js成功打包上去之前。再次测试的时候，发现用户会一次收到多张相同的卡片信息。经过一番自查。发现是因为egg项目在本地启动只会启动一个进程。而在测试环境上面启动会根据cpu的数量启动不同的进程。而app.js是运行在进程上面的。从而导致，每个进程都触发了redis的推送。经过查询，可以把代码移植到agent进程上面，并把接受到的卡片消息通过agent.messenger.sendRandom()方法随机推送个一个worker进程来广播出去。不过需要评估下，agent适不适合做这些逻辑。agent是每台机器启动一次。可是线上一般都是多台机部署，同样到时候agent也会跟agent一样。因此，到时候socketid的信息是不能存储在redis服务上面。不然这个信息变成来公用的，最后还是会导致用户收到几张不同的卡片。

### socket源码

### 1、概述及特点

socket.io是基于websocket的实时通信，底层是基于engine.io，engine.io使用的是websocket，xhr-polling(或者jsonp)封装的一套协议。在不支持websocket的情况下，是降及使用长轮询，socket.io在engine.io基础上添加了namespace(命名空间),room(频道),自动重连等功能。

1.准确性，可靠性。就算是在负载均衡，代理，个人防火墙及有杀毒软件的情况下都能连接成功。

2.自动重连。

3.掉线能及时发现。

4.二进制支持。

5.多路复用。

6.房间支持。



socket.io不是websocket的实现。虽然socket.io是使用websocket来作为传输的，但在基础上增加来元数据，数据包类型，命名空间。这也是为什么websocket的客户端不能连接上socket.io的服务端，反之亦然。



socekt.io执行的底层逻辑，基于engine.io-client、engine.io、engine.io-parser构成传输通道。





```javascript
client
# 首先客户端使用engine.io-client创建一个新的engine.io-client实例。
const client = io('https://myhost.com');
# engine.io-client实例尝试创建一个polling长轮询的传输通道。
GET https://myhost.com/socket.io/?EIO=3&transport=polling&t=ML4jUwU&b64=1
"EIO=3" # 当前Engine.IO的协议版本号
"transport=polling" # 正在创建的传输通道
"t=ML4jUwU&b64=1" # 用于缓存清除的哈希时间戳
# engine.io 服务端响应请求，返回对应的response
{
  "type": "open",
  "data": {
        "sid": "36Yib8-rSutGQYLfAAAD", # 唯一的session id
      "upgrades": ["websocket"], # 可能能升级的传输通道列表
      "pingInterval": 25000, # 心跳机制的第一个参数
      "pingTimeout": 5000, # 心跳机制的第二个参数
    }
}
# 上面的response通过engine.io-parser进行数据包装后（编码），返回给客户端。
# 客户端获取到的数据，通过engine.io-parser进行反向解析（解码）。获取到数据。并且通过engine.io-client
# 会触发open、connect事件。
# 一旦刷新了现有的缓存区域数据，就会在侧边尝试升级传输通道。
GET wss://myhost.com/socket.io/?EIO=3&transport=websocket&sid=36Yib8-rSutGQYLfAAAD
"EIO=3"                     # engine.io的协议版本
"transport=websocket"       # 一个新的通道正在被创建
"sid=36Yib8-rSutGQYLfAAAD"  # 唯一的session ID
```



### 2、双向通信方式

短轮询：定期向服务端发送请求询问是否有数据，定期的时间过长，不利于实时性，时间过短，对服务器压力大。

长轮询：向服务端发送请求之后，服务端作出相应的修改来，请求发送过来之后，会处于等待的状态，一直等到有数据或者超时才会返回。

websocket是基于TCP的一个独立的协议。采用了http协议的握手。同http使用同一个端口。

### 3、websocket通信

engine.io服务器维护了一个socket的数据结构用于管理连接到该机的客户端。客户端的标识是sid，如果有多个worker，则需要保证同一个客户端的请求落在同一台worker上，因为每一个worker只维护了部分客户端连接。如果要支持广播，room，需要使用redis。通过使用redis的订阅发布机制在多台机器或者多个worker进程之间进行消息的推送。