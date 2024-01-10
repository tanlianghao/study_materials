#### egg 1.x 跟 egg 2.x的区别

- egg 1.x 底层是基于koa 1.x 进行开发，异步解决方案是基于co封装的generator function，而egg 2.x是基于koa 2.x进行开发，异步解决方案也是通过async function

- egg 1.x 插件是以及 Egg 核心使用 generator function 编写，而egg 2.x插件以及核心都是使用的async function进行编写。

- egg 2.x 只支持node.js 8及以上的版本。

#### 初步尝鲜

##### 编写controller

```javascript
//app/controller/home.js
const Controller = require('egg').COntroller;
//使用class继承Controller基类
class HomeController extends Controller{
    //使用async进行异步编程
    async index(){
    // this.body直接用来渲染到HTML页面上
    this.body = 'Hello world';
}
}
//通过module.export 暴露一个接口
module.export = HomeController;
```

##### 编写路由

```javascript
//app/router.js
//暴露一个函数，并且该函数的参数是app对象
module.export = app => {
    const {router, controller} = app;
    router.get('/', controller.home.index);
}
```

##### 使用插件

```javascript
// 通过npm install 插件名（egg-view-nunjucks）
// 开启插件
// config/plugin.js
exports.nunjucks = {
    enable:true,
    package:'egg-view-nunjucks'
}
// 开启插件完之后还需要在 config/config.default.js 添加view配置
exports.view = {
    defaultViewEngine:'nunjucks',
    mapping:{'.tpl':'nunjucks'}
}
```

##### view层

```javascript
// 视图层一般约定是放在app/view目录下面
// app/view/news/index.tpl
<html>
    <head></head>
    <body><div></div></body>
</html>
```

##### 编写service层

```javascript
// service层是被抽象出来的一个数据处理机构，复杂的逻辑被放到这个地方进行处理
// app/service/new.js
const Service = require('egg').Service;
// 同controller一样是继承而来，不过继承的是Service基类
class NewsService extends Service {
  async list(page = 1) {
    // this.config能够访问到config配置中的数据
    const { serverUrl, pageSize } = this.config.news;
}
}
module.export = NewsService;
// 同时在controller结构中，可以使用this.ctx.service.new.list 访问到对应的service函数
```

##### 编写扩展

存在的疑问：中间件，扩展，插件之间的使用方式的区别？

##### 编写middleware

```javascript
// app/middleware/robot.js
module.export = (options,app)=>{
    return async function robotMiddlewar(){
    const source = ctx.get('user-agent');
    const match = options.us.some(ua = >{
        ua.test(source);
})
    if(match){
    ctx.status = 403;
    ctx.message = "hello world";
    
}else{
    await next()
}
}
}
// 另外编写完中间件之后，需要在config.default配置文件中开启
exports.middleware = ['robot']
```

#### 尝鲜之后的思考

egg项目的简单流程可以如下：

首先创建一个egg项目，如果需要添加新的页面，1）在app/router.js文件下面创建好路由名跟对应的controller函数名，2）在app/controller下面创建新的页面，在controller确认需要渲染的html页面。3）在app/view下面创建需要的html页面。4）如果页面需要从后台传送数据过来，可以再app/service下面进行数据的获取跟处理并传回给controller，controller然后再通过渲染到html上面。

#### egg内置对象

##### Application

在整个egg项目中几乎所有编写的应用上面都能获取到该对象，主要原因是因为所有被框架加载的文件（controller等）都可以exports一个函数，而该函数则接受一个app作为参数。只是根据所在的位置不同获取方式存在些许的差别。

##### context

context是作为一个请求级别的对象，所以每次用户有请求的时候都会实例化一个该对象，框架会把所有的service都挂载到这个对象上面（所以在昨天的笔记中controller跟service的关系是通过this.ctx.service来进行联系的） ，context是继承koa-context,在koa-context中context封装了request和response对象。其中也提供了许多的使用方法[链接](https://koa.bootcss.com/#context)

##### Request && Response

因为context是继承于koa-context，获取方式ctx.request && ctx.response,此外在context上面也有部分的request&&response的方法，具体可以参考[链接](https://koa.bootcss.com/#context) request跟reponse别名。

##### Controller

所有编写的controller都应该是继承框架提供的Controller基类上面，而且基类上面提供了许多方便的属性，比如ctx，app，service等等，继承Controller基类，有两种编写方式，不过在现在的项目中大部分用的都是如下的格式

```javascript
module.exports = app => {
    return Class HomeController extends Controller{
    
}
}
```

##### Service

service的结构跟写法同Controller的写法差不多。参考controller

##### helper

helper的作用是各个组件需要用的某个函数的时候可以直接获取，获取的方式因为获取的文件位置不同而不同。

- 在context实例上面获取helper

```javascript
module.exports = app => {
    return Class HomeController extends Controller{
    async index(){
    this.ctx.helper.formatUser()
}   
}
}
```

- 在模版中获取的方式

```javascript
{{helper.formater()}}
```

egg.js提供了自定义扩展helper的结构，如果想要进行自定义helper对象，可以在 app/extend/helper.js文件里面进行扩展

```javascript
module.exports = {
    formatuser(){
}
}
```

#### egg运行环境总结

egg框架给开发者提供了两种不同的设置环境的模版，config/env && EGG_SERVER_ENV=prod，从已加入的项目中发现，使用的更多的是后面这种方式，直接在package.json文件中进行配置就行了。在应用中可以通过app.config.env获取到运行环境。

#### egg中间件（middleware）详解

中间件的写法通过async/await来进行编写,其中函数有两个参数，第一个是上下文ctx，第二个则是next，当条件满足的时候，通过next()进行下一步操作。

```javascript
async function gzip(ctx,next){
    await next()
}
```

而在框架中，中间件通常都是伴随着配置文件绑定的。而在之前的学习中知道，框架中默认把middleware都统一放在了app/middleware下面。并且该文件下面写的middleware是作为一个函数的返回值，而这个函数则接受两个参数，一个是config的配置数据(options)，一个是app当前的应用实例。

```javascript
module.exports = (options, app)=>{
    return async function gizp(ctx,next){
    //中间件的开启跟配置文件的数据如下，直接可以通过options.threshold进行获取。
    if(options.threshold && options.threshold>1000){
    console.log(.....)
}
}
}
```

config中的配置文件如下。

```javascript
module.exports = {
    middleware:['gizp'],
    gizp:{
    threshold:1024
}
}
```

上面middleware是在应用上的使用（全局起作用），下面说到的是在router上的使用（只针对使用中间件的路由），应用上定义的中间件（app.config.appMiddleware）最后都会被加载到app.middleware上面。而在router上面使用中间件则正好是通过app.middleware进行实例化，再把实例传入到render的第二个参数上面。

```javascript
module.exports = app => {
    const gzip = app.middleware.gzip({threshold:1024});
    app.get('/',gizp,app.controller,handler);
}
```

需要注意的是应用定义的中间件名不要跟框架和插件上面的中间件名相同个，因为应用层上的中间件比框架上面的中间件后加载，而且如果同名的话也不能进行覆盖，导致报错。

#### egg路由详解

egg路由的完整定义，实际情况可以根据自己的需要进行可选。

```javascript
router.verb('router-name','path-match','middleware1'...,app.controller.action);
```

ps：路由中的中间件是串联而不是并联。

参数获取

- Query string方式

```javascript
// https://127.0.0.1:7001/search?name=egg
// app/controller/search
module.exports = app => {
    return Class Home extends Controller{
     const params = this.ctx.query.name;
}   
}
```

- 参数命名方式

在路由中设置参数，在应用中通过this.ctx.param获取

```javascript
// app/router.js
module.exports = app => {
    app.get('/user/:id/:name',app.controller.user.info);
}
// app/controller/user.js
exports.info=async ctx=> {
    ctx.body = `user: ${ctx.params.id}, ${ctx.params.name}`;
}
// http://127.0.0.1:7001/user/123/xiaoming
```

上面的只是常见的两种。



## egg启动的步骤

```javascript
// {app.root}/index.js

require('@ali/egg').startCluster({
    baseDir: __dirname,
    port,
    workers
});
// 调用egg框架中的startCluster,一直追踪下去
// 
exports.startCluster = require('./lib/cluster/index').startCluster;

// /lib/cluster/index.js
exports.startCluster = function(options, callback) {
  options = options || {};
  options.eggPath = path.join(__dirname, '../..');
  startCluster(options, callback);
};

// 追踪到最后发现是调用egg-cluster文件index.js中的startCluster方法。并且是实例化例Master对象
exports.startCluster = function(options, callback) {
  new Master(options).ready(callback);
};
// master.js 主要是启动master agent appworker等进程。待所有的agent、appworker进程启动完成之后
// egg会先去做一些准备工作
// 加载插件：会把本地插件，egg内置插件以及app的框架全部都集成到allplugin中。并之后对已经开启的插件进行整合。
// 扩展内置对象： 也是找出所有的application文件。并添加到内置对象中。
// 加载中间件
// 加载控制器
// 加载service
// 加载路由
// 最后才执行业务逻辑。
```

#### egg-security插件

提供了请求post请求的ctoken校验，egg.js是一个很基础的框架，提供了插件跟中间件的加载方式。使得多种多样化。

## egg启动方式

在frontend项目中，本地开发执行的script命令： npm run dev；该命令其实调用的gulp start任务。

```javascript
gulp.task('start', [ 'dev' ], function () {
// 其中nodemon是调用插件gulp-nodemon
// script：将要运行的文件；ext：需要监控的文件；env：开发模式跟端口。调用nodemon返回的是nodejs stream
// on（），在执行完上面的流程之后，会执行on方法中的gulp任务。
    nodemon({
        script: 'index.js',
        ext: 'js html',
        ignore: [
            '../'
        ],
        env: { NODE_ENV: 'development', PORT: 9080 }
    }).on('start', [ 'open:dev', 'watch' ]);
});
```

上面的重点是在index.js文件。事实上index.js文件就只是调用了egg框架中的startCluster()方法。

```javascript
require('@ali/egg').startCluster({
    baseDir: __dirname,
    port,
    workers
});

// 定位到egg框架中startCluster，进一步解析。
exports.startCluster = require('./lib/cluster/index').startCluster;

// .lib/cluster/index.js，最终调用的egg-cluster插件startCluster；
const startCluster = require('@ali/egg-cluster').startCluster;
exports.startCluster = function(options, callback) {
  options = options || {};
  options.eggPath = path.join(__dirname, '../..');
  startCluster(options, callback);
};

// egg-cluster插件
const Master = require('./lib/master');
exports.startCluster = function(options, callback) {
  new Master(options).ready(callback);
};
```



通过上述的解析，可以看到最终调用了master.js文件。并把options参数传递过去。查看源码，master完整参数可以看到。在master文件中，最开始会启动master进程，成功之后会通过自定义的Messenger类把消息传递给parent、appworker、agent；接着通过调用forkAgentWorker（）函数fork出agent进程。待agent启动完成之后，会触发agent-start事件，从而调用forkAppWorkers（）来启动appworker进程。这里需要区分agent和appworker进程的启动方式不一样。agent是通过child_process方法来启动的，而appworker则是通过插件cfork来启动的。并且work启动完成之后，就会开始做一些项目启动的准备工作，比如加载中间件、插件等。

```javascript
    // 告知父进程启动成功
    this.ready(() => {
      this.isStarted = true; // 启动完成。
      ...
      const action = 'egg-ready';
      this.messenger.send({ action, to: 'parent' });
      this.messenger.send({ action, to: 'app', data: this.options });
      this.messenger.send({ action, to: 'agent', data: this.options });
    });

    this.on('agent-exit', this.onAgentExit.bind(this));
    this.on('agent-start', this.onAgentStart.bind(this));
    this.on('app-exit', this.onAppExit.bind(this));
    this.on('app-start', this.onAppStart.bind(this));
    this.on('reload-worker', this.onReload.bind(this));

    // agent 启动完成后再启动 app，forkAppWorkers 只能调用一次，work进程fork完成之后，就开始干活。
    this.once('agent-start', this.forkAppWorkers.bind(this));

    // 处理 master 进程退出
    this.onExit();
    // 启动agent进程函数
    this.forkAgentWorker()
```



在上步中，启动worker进程是通过cfork插件来启动的，部分代码如下：

```javascript
const appWorkerFile = path.join(__dirname, 'app_worker.js');
cfork({
      exec: appWorkerFile, // 执行文件的路径
      args,// 参数
      silent: false,
      limit: 10, // 一分钟异常退出 10 次，就不再尝试重启
      duration: 60000,
      count: this.options.workers,
      // 本地开发环境，不自动重启意外退出的 worker，方便开发察觉 bug 的出现
      refork: this.isProduction,
    });
```



而app_worker.js文件中的代码主要是起到两个作用，第一个是跟koa一样，利用http插件来启动服务。第二个就是实例话application

```javascript
// options.customEgg 就是egg框架的根目录。
const Application = require(options.customEgg).Application;
const app = new Application({
  customEgg: options.customEgg,
  antxpath: options.antxpath,
  baseDir: options.baseDir,
  plugins: options.plugins,
});

// app实例化是继承EggApplication类，在父类中会给子类实例化Loader

// application.js
 constructor(options) {
    options = options || {};
    options.type = 'application';
    super(options);
    this.loader.loadApplication();
// 调用app_worker_loader.js文件下面的load()函数，该函数会调用app_worker_loader.js文件下面的
// load()方法，
    this.loader.load(); 
    this.on('server', server => this.onServer(server));
  }

// app_worker_loader.js文件
 load() {
// 下面的函数，都会去做对应的工作，比如聚合中间件等。
    // app > plugin > core
    this.loadRequest();
    this.loadResponse();
    this.loadContext();
    this.loadHelper();

    // app > plugin
    this.loadCustomApp();
    // app > plugin
    this.loadProxy(); // 需要依赖 hsf
    // app > plugin
    this.loadService();
    // app > plugin > core
    this.loadMiddleware();
    // app
    this.loadController();
    // app
    this.loadRouter(); // 依赖 controller
  }
```



上一步中app_worker_loader.js文件中load()函数中调用的函数，继承于base_loader.js -> @ali/egg-loader插件 -> @ali/egg-loader/lib/base_loader 。比loadController()函数

```javascript
  /**
   * 加载 app/controller 目录下的文件
   *
   * @param {Object} opt - loading 参数
   */
  loadController(opt) {// opt参数为undefined
    const app = this.app; // 指向的是app实例
    opt = Object.assign({ lowercaseFirst: true }, opt);
    // 判断项目中controller目录路径
    let controllerBase = path.join(this.options.baseDir, 'app/controller');
    if (!fs.existsSync(controllerBase)) {
      // 兼容复数目录
      controllerBase = path.join(this.options.baseDir, 'app/controllers');
    }
    delete app.controllers;
    delete app.controller;
    app.controllers = app.controller = {};
    // loading()引用的是loading插件，先判断是否是new一个实例，再调用类的into方法
    loading(controllerBase, opt).into(app, 'controller');
    app.coreLogger.info('[egg:loader] Controller loaded: %s', controllerBase);
  }


// loading插件，into方法，最终返回的是经过一系列处理的app实例。
proto.into = function (target, field, options) {
  this._load(target, field, options); // 调用这个的目的是为了把诸如controller函数，重新赋值到对应的target[field]中
  console.log('最终得到的targe', target);
  return target;
};
```



待上一步controller、proxy、middleware等文件处理完成之后，在egg-cluseter/lib/app_worker.js中会通过调用http模块启动服务，并通过app调用koa中的callback()函数处理中间件。

## 简易流程图

![img](https://cdn.nlark.com/lark/0/2018/png/89927/1534765548630-f436baef-d153-4fb3-9b7f-8ae58e519629.png)



## Master.js源码分析

```javascript
constructor(options) {
    // master类是基于EventEmitter来进行构建的
    super()
// parseOptions() 函数是在options对象中再添加一些值，比如customEgg
    this.options = parseOptions(options);
// Messenger实例化，是定义了master、agent、appworker之间的通信方式
    this.messenger = new Messenger(this);
    ready.mixin(this);
// MasterLoader类，继承的BaseLoader基类，并添加loadConfig方法，loadConfig调用父类中的
// loadAntx() 跟 loadConfig() 这两个方法。
    const loader = new MasterLoader(this.options);
// 调用函数，会去加载baseloader中的loadAntx() 跟loadConfig()
    const config = loader.loadConfig();
    this.isProduction = config.env !== 'local' && config.env !== 'unittest';
    this.isDebug = process.execArgv.indexOf('--debug') !== -1 || typeof v8debug !== 'undefined';
// 读取真实的框架名和版本信息
    const frameworkPath = options.customEgg || options.eggPath;
    const frameworkPkg = require(path.join(frameworkPath, 'package.json'));
// 在日志里面留下容易识别的启动位置标记
    this.logger.info(`[master] =================== ${frameworkPkg.name} start =====================`);
    ...

    const startTime = Date.now();

// 当父进程启动完成之后，会调用ready（）方法
    this.ready(() => {
        this.isStarted = true;
        this.logger.info('[master] %s started on %s://127.0.0.1:%s (%sms)',
    frameworkPkg.name, this.options.https ? 'https' : 'http', this.options.port, Date.now() - startTime);

        const action = 'egg-ready';
        通过message通知各分支情况。
        this.messenger.send({ action, to: 'parent' });
        this.messenger.send({ action, to: 'app', data: this.options });
        this.messenger.send({ action, to: 'agent', data: this.options });
        });

        this.on('agent-exit', this.onAgentExit.bind(this));
        this.on('agent-start', this.onAgentStart.bind(this));
        this.on('app-exit', this.onAppExit.bind(this));
        this.on('app-start', this.onAppStart.bind(this));
        this.on('reload-worker', this.onReload.bind(this));

    // agent 启动完成后再启动 app，forkAppWorkers 只能调用一次
        this.once('agent-start', this.forkAppWorkers.bind(this));

    // 处理 master 进程退出
        this.onExit();
    // forkAgentWorker() 函数会定义一些事件，通过这些事件来通知各部分自身的开启情况
        this.forkAgentWorker();
}
```

在上面代码中会调用到baseLoader基类，这个类中，会写上一些loader的方法。![img](https://cdn.nlark.com/lark/0/2018/png/89927/1535715526700-8f8ec3ae-fe74-4ba6-8cce-9da3994b8dde.png)

## config配置文件加载

通过上一章知道，egg在fork出work进程之后，会去做启动前的一些准备工作，比如加载中间件，配置文件等，这次分析配置文件的启动，egg是怎样默认加载config.default.js和根据不同的运行环境加载不同的config文件。

```javascript
// worker进程文件
const Application = require(options.customEgg).Application;
// 实例化application
const app = new Application({
    customEgg: options.customEgg,
    antxpath: options.antxpath,
    baseDir: options.baseDir,
    plugins: options.plugins
});
// application构造函数，继承egg框架
constructor(options) {
        options = options || {};
        options.type = 'application';
        super(options);
        this.loader.loadApplication();
        this.loader.load(); // 调用app_worker_loader.js文件下面的load()函数，该函数会
        this.on('server', (server) => this.onServer(server));
    }
get [Symbol.for('egg#loader')]() {
        return AppWorkerLoader;// egg/lib/core/loader/app_worker_loader.js
    }
// egg.js框架中，
const Loader = this[Symbol.for('egg#loader')]; // this指向的是子类的实例，即application实例。返回的是appWorkerLoader
// 实例化app_worker_loader.js文件中的构造函数。
const loader = this.loader = new Loader(options);
// 调用loader的loadconfig方法
loader.loadConfig();

// app_worker_loader.js文件，继承于同目录下的base_loader.js文件。而base_loader文件继承egg-loader插件
    loadConfig() {
        super.loadPlugin();
        super.loadAntx();
        super.loadConfig();
    }
// egg-loader文件中的loaderconfig() 返回是通过同目录下的config_loader.js来注入的
loadConfig(){
    const names = [
        'config.default.js',
        `config.${this.serverEnv}.js`
    ]
    const appConfig = this._preloadAppConfig();// 优先加载应用的配置文件
    通过循环把配置都合并起来
    for (const filename of names) {
            for (const dirpath of this.loadDirs()) {
                const config = this._loadConfig(dirpath, filename, appConfig);

                if (!config) {
                    continue;
                }

                debug('Loaded config %s/%s, %j', dirpath, filename, config);
                extend(true, target, config);
            }
        }
}
```

## 插件plugin加载

在上一节中可以看到，在app_worker_loader.js文件下的loadconfig()会调用父类的loadPlugin、loadConfig等方法，这次主要分析loadPlugin()的过程。app_worker_loader.js是继承于base_loader.js，base继承于@ali/egg-loader插件。并在base中会去调用插件的loadConfig()![img](https://cdn.nlark.com/lark/0/2018/png/89927/1536116743956-47d900df-e1fb-4b58-b327-3af999486323.png)

最后调用的plugin_loader.js文件

```javascript
// plugin_loader.js文件，该文件暴露了loadPlugin()方法。

loadPlugin () {
    // readPluginConfigs()是实例的方法，读取应用的plugin.js文件。
    const appPlugins = this.readPluginConfigs(path.join(this.options.baseDir, 'config/plugin.js'));
    // 读取框架的plugin.js文件
    const eggPluginConfigPaths = this.eggPaths.map((eggPath) => path.join(eggPath, 'lib/core/config/plugin.js'));
    const eggPlugins = this.readPluginConfigs(eggPluginConfigPaths);
    ... // 会去读取自定义插件的配置plugin.js

    // 会把应用，框架，插件的plugin.js文件都合并到this.allplugins属性上面
    this.allPlugins = {};
    extendPlugins(this.allPlugins, eggPlugins);
    extendPlugins(this.allPlugins, appPlugins);
    extendPlugins(this.allPlugins, customPlugins);

    // 集合了所有的插件之后，需要把应用enbale关闭的，不满足运行环境的插件给剔除掉
    // 循环this.allPlugins对象
    const enabledPluginNames = []; // 入口开启的插件列表，不包括被依赖的
    const plugins = {}; // 经过循环拿到的是enable：true的插件。
    const env = this.serverEnv;
    for (const name in this.allPlugins) {
        const plugin = this.allPlugins[name];
        ...
        // 因为插件的引用分两种形式，如果是packgage格式需要转化成路径的形式。path格式就可以直接调用。
        plugin.path = this.getPluginPath(plugin, this.options.baseDir);
        // 转换path路径之后，通过mergePluginConfig()函数，需要根据package.json文件来核对plugin信息
        // 首先会根据pkg中的version来确认plguin的version，并且查看pkg.eggPlugin会赋值给config
        // 接着判断config如果为空的话，会先查找plugin目录下的config/plugin.js文件，并且赋值给config，用来查看是否存在依赖插件
        // 根据plugin 跟config来判断，如果config[dep,[env]]存在，则plugin存在依赖插件
        this.mergePluginConfig(plugin);
        // 只有符合插件env中能找到当前环境env，且应用中的插件是开启状态的，这些才会写人到plugins对象中去。
        // 只允许符合服务器环境 env 条件的插件开启
        if (env && plugin.env.length && plugin.env.indexOf(env) === -1) {
            debug('Disabled %j as env is %j, but got %j', name, plugin.env, env);
            plugin.enable = false;
            continue;
        }

        // app 的配置优先级最高，切不允许隐式的规则推翻 app 配置
        if (appPlugins[name] && !appPlugins[name].enable) {
            debug('Disabled %j as disabled by app', name);
            continue;
        }
        plugins[name] = plugin;
        // enable为true
        if (plugin.enable) {
            enabledPluginNames.push(name);
        }
    }
    this.plugins = enablePlugins; //为开启状态的plugins。
}

readPluginConfigs(configPaths){
    ...
    const plugins = {};
    for (const configPath of configPaths) {
        // 判断configPath目录是否存在
        if (!fs.existsSync(configPath)) {
            continue;
        }
        // loadFile()方法作用：返回传入的参数路径中暴露的对象，并且会在函数内部做进行错误的捕获
        const config = utils.loadFile(configPath);
        for (const name in config) {
            // normalizePluginConfig() 函数会对config中的对象进行标准化
            this.normalizePluginConfig(config, name);
        }

        // 拷贝一个新对象，不修改原来的对象，根据config来生成plugins
        extendPlugins(plugins, config);
    }
    return plugins
}

normalizePluginConfig(config, name) {
    // 判断逻辑是根据，如果config[name]的对象是boolean，则标准化
    if (typeof plugin === 'boolean') {
        plugins[name] = {
            name,
            enable: plugin,
            dep: [],
            env: []
        };
        return plugins[name];
    }
    // 同时如果enable没写，默认是true
    if (!('enable' in plugin)) {
        plugin.enable = true;
    }
}
```

### 插件能做什么

1. #### 扩展内置对象接口

   在插件相应的文件内对框架内置对象进行扩展和在应用中扩展是一样的。比如app/extend下扩展request.js用来扩展Koa#Request类。

1. #### 插入自定义中间件

   可以在插件中的app/middleware文件中，设置中间件。并在${plugin_root}/app.js文件中设置中间件的应用。

1. #### 在应用启动时做一些初始化工作

- - 我在启动前想读取一些本地配置

    `// ${plugin_root}/app.js const fs = require('fs'); const path = require('path'); module.exports = app => {  app.customData = fs.readFileSync(path.join(app.config.baseDir, 'data.bin'));   app.coreLogger.info('read data ok'); };`

- - 如果有异步启动逻辑，可以使用 `app.beforeStart` API

    `// ${plugin_root}/app.js const MyClient = require('my-client'); module.exports = app => {  app.myClient = new MyClient();  app.myClient.on('error', err => {    app.coreLogger.error(err);  });  app.beforeStart(async () => {    await app.myClient.ready();    app.coreLogger.info('my client is ready');  }); };`

- - 也可以添加 agent 启动逻辑，使用 `agent.beforeStart` API

    `// ${plugin_root}/agent.js const MyClient = require('my-client'); module.exports = agent => {  agent.myClient = new MyClient();  agent.myClient.on('error', err => {    agent.coreLogger.error(err);  });  agent.beforeStart(async () => {    await agent.myClient.ready();    agent.coreLogger.info('my client is ready');  }); };`

1. #### 设置定时任务

### proxy文件的源码分析

### 疑问点

1. proxy是怎么加载到的

1. 为什么在service上面可以直接通过this.proxy.proxy文件名.方法名这样的方式进行调用。

### 源码解析

从上面的plugin源码分析中可以知道，worker进程启动之后，会主动去加载proxy、plugin、service等文件。追踪this.loadProxy()方法可以得到最终调用的是Proxy_loader.js文件中暴露出来的方法。

```javascript
module.exports = {
        loadProxy(opt) {
            // app实例对象
            const app = this.app;
            // opt为undefined
            opt = Object.assign({ call: true, lowercaseFirst: true }, opt);
            // this.loadDirs()函数返回的是数组，包括框架的目录，已开启插件的跟目录，应用根目录。
            // arr返回的是数组，上面对应的app/proxy目录
            const arr = this.loadDirs().map((dir) => join(dir, 'app/proxy'));
            // load proxy classes to app.proxyClasses
            delete app.proxyClasses;
            // loading()会一次加载对应的app/proxy文件，并把生成的对象赋值到app.proxyClasses对象上。
            loading(arr, opt).into(app, 'proxyClasses');
            // 得到的是一个ClassLoader类，且prototype原型上面添加了app/proxy文件下暴露出来的方法
            app.ProxyClassLoader = createClassLoader(app.proxyClasses, null);
  
            // this.proxy.cifQuery.getUser(uid)
            // Object.defineProperty()函数会直接在一个对象上定义一个新属性，或者修改一个已经存在的属性， 并返回这个对象
            Object.defineProperty(app.context, 'proxy', {
                get() {
                    let loader = this[classLoader];
                    if (!loader) {
                        this[classLoader] = loader = new this.app.ProxyClassLoader(this);
                    }
                    return loader;
                }
            });
    
            app.coreLogger.info('[egg:loader] Proxy loaded from %j', arr);
        }
  
  };
```

