---
title: express源码分析(2)——中间件
date: 2018-02-08 13:36:17
tags: express
---
express主要有一下几种中间件：应用级中间件、路由级中间件、错误处理中间件、内置中间件、第三方中间件

应用级中间件和路由级中间件其实差不多，下面分析会涉及。

我们先看看`app.use`是怎么实现的
```
// application.js
app.use = function use(fn) {
  var offset = 0;
  var path = '/';

  // default path to '/'
  // disambiguate app.use([fn])
  if (typeof fn !== 'function') {
    var arg = fn;

    while (Array.isArray(arg) && arg.length !== 0) {
      arg = arg[0];
    }

    // first arg is the path
    if (typeof arg !== 'function') {
      offset = 1;
      path = fn;
    }
  }

  // 我们的参数fns
  var fns = flatten(slice.call(arguments, offset));

  if (fns.length === 0) {
    throw new TypeError('app.use() requires a middleware function')
  }

  // setup router
  this.lazyrouter();
  var router = this._router;

  fns.forEach(function (fn) {
    // non-express app
    if (!fn || !fn.handle || !fn.set) {
      return router.use(path, fn);
    }

    debug('.use app under %s', path);
    fn.mountpath = path;
    fn.parent = this;

    // restore .app property on req and res
    router.use(path, function mounted_app(req, res, next) {
      var orig = req.app;
      fn.handle(req, res, function (err) {
        setPrototypeOf(req, orig.request)
        setPrototypeOf(res, orig.response)
        next(err);
      });
    });

    // mounted an app
    fn.emit('mount', this);
  }, this);

  return this;
};

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

`app.use`触发了`router.use`方法
再来看看`this._router.use`的实现，其实也就是路由级中间件的use方法, 
我们使用路由级中间件`const router = express.Router();` 就是创建了一个router方法，之后再供`app.use(path, router)`来调用

```
proto.use = function use(fn) {
  var offset = 0;
  var path = '/';

  // default path to '/'
  // disambiguate router.use([fn])
  if (typeof fn !== 'function') {
    var arg = fn;

    while (Array.isArray(arg) && arg.length !== 0) {
      arg = arg[0];
    }

    // first arg is the path
    if (typeof arg !== 'function') {
      offset = 1;
      path = fn;
    }
  }

  var callbacks = flatten(slice.call(arguments, offset));

  if (callbacks.length === 0) {
    throw new TypeError('Router.use() requires a middleware function')
  }

  for (var i = 0; i < callbacks.length; i++) {
    var fn = callbacks[i];

    if (typeof fn !== 'function') {
      throw new TypeError('Router.use() requires a middleware function but got a ' + gettype(fn))
    }

    // add the middleware
    debug('use %o %s', path, fn.name || '<anonymous>')

    var layer = new Layer(path, {
      sensitive: this.caseSensitive,
      strict: false,
      end: false
    }, fn);

    layer.route = undefined;

    this.stack.push(layer);
  }

  return this;
};
```

其实跟app.use实现差不多，主要里面创建了`layer实例`, layer的handle就是我们的回调函数,并把layer层存储在router的stack数组中

所以大概是 app._router里面有个stack数组，stack里面存储Layer实例层，layer的hanlde属性也储存着我们的回调方法

因此我们使用`app.use`的时候，就是一直往根部router里面的stack添加layer层。这差不多就是中间件的方式。

下篇再进一步讲下express的next的实现，这是中间件的运行的关键。
理解通了前两篇文章，下一篇才能更好的理解
 