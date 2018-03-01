---
title: express源码阅读(1)——路由结构
date: 2018-02-5 17:38:02
tags: express
---
由于最近2个面试都提到了express，需要了解中间件怎么运作，内部原理是什么，所以特意去看了[express源码](https://github.com/expressjs/express)，最后出来了这篇文章。 
***

我们平时项目引入express的源文件index.js，

```
module.exports = require('./lib/express');
```

知道`lib/express.js`为主文件
我们先回顾下怎么用express启用服务器的

```
var app = require('express')();
var server = http.createServer(app);
server.listen(port);
```

了解的人应该知道这个http方法，`http.createServer([requestListener])`,
`requestListener` 是一个函数，会被自动添加到 [`'request'`](http://nodejs.cn/api/http.html#http_event_request) 事件

了解这些后，再来看看`index.js`的代码

```
// index.js
var bodyParser = require('body-parser')   // 解析body参数的
var EventEmitter = require('events').EventEmitter;  // 事件监听对象
var mixin = require('merge-descriptors');   // 合并对象
var proto = require('./application');     // app的大部分方法
var Route = require('./router/route');
var Router = require('./router'); 
var req = require('./request');
var res = require('./response');

exports = module.exports = createApplication;

function createApplication() {
  // 这个app就是返回的 requestListener 函数, 用于http.createServer([requestListener])
  var app = function(req, res, next) {
    app.handle(req, res, next);
  };
 
  mixin(app, EventEmitter.prototype, false);  // 把EventEmitter的原型方法合并到app
  mixin(app, proto, false);                   // 把proto的方法方法合并到app

  // expose the prototype that will get set on requests
  app.request = Object.create(req, {  // 在指定原型对象上添加新属性后的对象。
    app: { configurable: true, enumerable: true, writable: true, value: app }
  })

  // expose the prototype that will get set on responses
  app.response = Object.create(res, {
    app: { configurable: true, enumerable: true, writable: true, value: app }
  })

  app.init(); // 初始化，在proto(application.js)里面
  return app;
}
```

主要把一些方法都集成在app当中，再看看看`app.init()`

```
// application.js
var app = exports = module.exports = {};

var trustProxyDefaultSymbol = '@@symbol:trust_proxy_default';

app.init = function init() {
  this.cache = {};
  this.engines = {};
  this.settings = {};

  this.defaultConfiguration();
};

app.defaultConfiguration = function defaultConfiguration() {
  var env = process.env.NODE_ENV || 'development';

  // default settings
  this.enable('x-powered-by');
  this.set('etag', 'weak');
  this.set('env', env);
  this.set('query parser', 'extended');
  this.set('subdomain offset', 2);
  this.set('trust proxy', false);

  // trust proxy inherit back-compat
  Object.defineProperty(this.settings, trustProxyDefaultSymbol, {
    configurable: true,
    value: true
  });

  debug('booting in %s mode', env);
 
  this.on('mount', function onmount(parent) {
    // inherit trust proxy
    if (this.settings[trustProxyDefaultSymbol] === true
      && typeof parent.settings['trust proxy fn'] === 'function') {
      delete this.settings['trust proxy'];
      delete this.settings['trust proxy fn'];
    }

    // inherit protos
    setPrototypeOf(this.request, parent.request)
    setPrototypeOf(this.response, parent.response)
    setPrototypeOf(this.engines, parent.engines)
    setPrototypeOf(this.settings, parent.settings)
  });

  // setup locals
  this.locals = Object.create(null);

  // top-most app is mounted at /
  this.mountpath = '/';

  // default locals
  this.locals.settings = this.settings;

  // default configuration
  this.set('view', View);
  this.set('views', resolve('views'));
  this.set('jsonp callback name', 'callback');

  if (env === 'production') {
    this.enable('view cache');
  }
};
```

这部分主要设置了一些初始化的东西

在使用当中，我们主要使用的`app.use`中间件、`app.get`和`app.post`等路由方法。
现在我们来看看这两个问题__express怎么生成路由__，__中间件是怎么运作的__

#### 1. express怎么生成路由
查到了一些app没有get，post之类的方法，但从`application.js`可以找到这段代码
```
// application.js
methods.forEach(function(method){
  app[method] = function(path){
    if (method === 'get' && arguments.length === 1) {
      // app.get(setting)
      return this.set(path);
    }

    this.lazyrouter();

    var route = this._router.route(path);
    route[method].apply(route, slice.call(arguments, 1));  
    return this;
  };
});
```
methos是一个引入的库`var methods = require('methods');`，查看github上的methods源码知道它就是一个请求方法的数组`['get, 'post', ...]`, 所以这段代码就是注册了这些`app.get`, `app.post`等方法

`route[method].apply(route, slice.call(arguments, 1));`
可以知道我们注册的路由`function`就是`Array.prototype.slice.call(arguments, 1)`;

我们主要看看`route`到底是什么
从`this.lazyrouter();` 可以找到`this._router`的值，就是一个router实例。
```
app.lazyrouter = function lazyrouter() {
  if (!this._router) {
    this._router = new Router({
      caseSensitive: this.enabled('case sensitive routing'),
      strict: this.enabled('strict routing')
    });

    this._router.use(query(this.get('query parser fn')));
    this._router.use(middleware.init(this));
  }
};
```
Router是导入的`lib/router/index.js`
```
var proto = module.exports = function(options) {
  var opts = options || {};

  function router(req, res, next) {
    router.handle(req, res, next);
  }
 
  // mixin Router class functions
  setPrototypeOf(router, proto);   // 把proto作为router的原型

  router.params = {};
  router._params = [];
  router.caseSensitive = opts.caseSensitive;
  router.mergeParams = opts.mergeParams;
  router.strict = opts.strict;
  router.stack = [];

  return router;
};
```

`setPrototypeOf(router, proto);`   这句话把proto作为router的原型
我们来看看proto有没有route这个方法,答案是有的

```
proto.route = function route(path) {
  var route = new Route(path);
  
 // 一个layer层实例，存着route，方便以后回调方法
  var layer = new Layer(path, {
    sensitive: this.caseSensitive,
    strict: this.strict,
    end: true
  }, route.dispatch.bind(route));

  layer.route = route;
 
  this.stack.push(layer);   // router的stack数组里添加layer， 以后匹配路由遍历使用
  return route;
};
```
创建了Route实例和Layer实例；
把route作为属性引入在layer实例中，并在stack加入layer实例（这里主要是当路由触发，遍历的时候用到，以后再讲）
所以之前的route也就是一个Route实例。
我们再看看`/lib/router/route.js`
```
// /lib/router/route.js
module.exports = Route;

function Route(path) {
  this.path = path;
  this.stack = [];

  debug('new %o', path)

  // route handlers for various http methods
  this.methods = {};
}

methods.forEach(function(method){
  Route.prototype[method] = function(){
    var handles = flatten(slice.call(arguments));

    for (var i = 0; i < handles.length; i++) {
      var handle = handles[i];

      if (typeof handle !== 'function') {
        var type = toString.call(handle);
        var msg = 'Route.' + method + '() requires a callback function but got a ' + type
        throw new Error(msg);
      }

      debug('%s %o', method, this.path)

      var layer = Layer('/', {}, handle);
      layer.method = method;

      this.methods[method] = true;
      this.stack.push(layer);
    }

    return this;
  };
});
```
route实例也会存在一个stack, 主要用于一个路由可能有多个回调函数。
如`app.post('/', authFn, cb)`
所以之前的这段代码`route[method].apply(route, slice.call(arguments, 1));`
其实就是执行这个原型方法。
`handles`就是我们路由的回调函数。
每个Route都会创建一个Layer实例，并在layer层存储在route的stack里面。
我们再看看Layer构造函数
```
function Layer(path, options, fn) {
  if (!(this instanceof Layer)) {
    return new Layer(path, options, fn);
  }

  debug('new %o', path)
  var opts = options || {};

  this.handle = fn;
  this.name = fn.name || '<anonymous>';
  this.params = undefined;
  this.path = undefined;
  this.regexp = pathRegexp(path, this.keys = [], opts);

  // set fast path flags
  this.regexp.fast_star = path === '*'
  this.regexp.fast_slash = path === '/' && opts.end === false
}
```
所以可以知道，layer实例存储着我们的handle方法，也就是我们路由的回调函数。



现在我们总结下express的大体结构，大概是这样的：

![express大体结构](/img/express/express_1.png)

今天就到这里吧。下次再写写中间件`app.use`










