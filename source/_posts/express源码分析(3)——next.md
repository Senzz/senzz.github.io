---
title: express源码分析(3)——next
date: 2018-02-11 15:33:31
tags: express
---

我们平时启动express主要有两种方式：
1. 第一个分析中所说的：
```
var app = require('express')();
var server = http.createServer(app);
var port = 3000;
server.listen(port);
```

2. 使用express的api
```
var port = 3000;
var server = app.listen(port, function () {
  var host = server.address().address;
  var port = server.address().port;

  console.log('Example app listening at http://%s:%s', host, port);
});
```

这两种方式其实差不多，看下`app.listen`源码

```
app.listen = function listen() {
  var server = http.createServer(this);
  return server.listen.apply(server, arguments);
};
```
里面的this就是app，是不是跟第一种方法一样。

现在开始将中间件怎么运行，主要也就是next。
当我们通过浏览器浏览该路由，或者通过postman、ajax等方式请求METHOD接口的时候。
就触发app的方法。
也就是我们刚刚所说的，nodejs的`http.createServer([requestListener])`其实差不多就是监听请求事件。
`requestListener` 是一个函数，会被自动添加到 [`'request'`](http://nodejs.cn/api/http.html#http_event_request) 事件

当我们监听到请求，就是触发app这个方法
```
// express.js
var app = function(req, res, next) {
  app.handle(req, res, next);
};
```
这里的`next`是没有东西的，是`undefined`

```
// application.js
app.handle = function handle(req, res, callback) {
  var router = this._router;

  // final handler
  // finalhandler作为最后一步调用，收集信息
  var done = callback || finalhandler(req, res, {
    env: this.get('env'),
    onerror: logerror.bind(this)
  });

  // no routes
  if (!router) {
    debug('no routes defined on app');
    done();
    return;
  }

  router.handle(req, res, done);
};
```

再看`router.handle`
```
// router/index.js
proto.handle = function handle(req, res, out) {
  var self = this;

  debug('dispatching %s %s', req.method, req.url);

  var idx = 0;
  var protohost = getProtohost(req.url) || ''
  var removed = '';
  var slashAdded = false; 
  var paramcalled = {};

  // store options for OPTIONS request
  // only used if OPTIONS request
  var options = [];

  // middleware and routes
  var stack = self.stack;

  // manage inter-router variables
  var parentParams = req.params;
  var parentUrl = req.baseUrl || '';
  var done = restore(out, req, 'baseUrl', 'next', 'params');

  // setup next layer
  req.next = next;

  // for options requests, respond with a default if nothing else responds
  if (req.method === 'OPTIONS') {
    done = wrap(done, function(old, err) {
      if (err || options.length === 0) return old(err);
      sendOptionsResponse(res, options, old);
    });
  }

  // setup basic req values
  req.baseUrl = parentUrl;
  req.originalUrl = req.originalUrl || req.url;

  // console.log({
  //   idx, protohost, removed, slashAdded, paramcalled,
  //   options, stack, parentParams, parentUrl, done
  // });

  next();

  function next(err) {
    var layerError = err === 'route'
      ? null
      : err;

    // remove added slash
    if (slashAdded) {
      req.url = req.url.substr(1);
      slashAdded = false;
    }

    // restore altered req.url
    if (removed.length !== 0) {
      req.baseUrl = parentUrl;
      req.url = protohost + removed + req.url.substr(protohost.length);
      removed = '';
    }

    // signal to exit router
    if (layerError === 'router') {
      setImmediate(done, null)
      return;
    }

    // no more matching layers
    if (idx >= stack.length) {
      setImmediate(done, layerError);
      return;
    }

    // get pathname of request
    var path = getPathname(req);

    if (path == null) {
      return done(layerError);
    }

    // find next matching layer
    var layer; 
    var match;
    var route;

    while (match !== true && idx < stack.length) {
      
      layer = stack[idx++];
      match = matchLayer(layer, path);
      route = layer.route;
      console.log({idx, stackLength: stack.length, match, layer, layerHandle: layer.handle.toString()})
      if (typeof match !== 'boolean') {
        // hold on to layerError
        layerError = layerError || match;
      }

      if (match !== true) {
        continue;
      }

      if (!route) {
        // process non-route handlers normally
        continue;
      }

      if (layerError) {
        // routes do not match with a pending error
        match = false;
        continue;
      }

      var method = req.method;
      var has_method = route._handles_method(method);

      // build up automatic options response
      if (!has_method && method === 'OPTIONS') {
        appendMethods(options, route._options());
      }

      // don't even bother matching route
      if (!has_method && method !== 'HEAD') {
        match = false;
        continue;
      }
    }
    console.log('while doned', match, idx)
    // no match
    if (match !== true) {
      return done(layerError);
    }

    // store route for dispatch on change
    if (route) {
      req.route = route;
    }

    // Capture one-time layer values
    req.params = self.mergeParams
      ? mergeParams(layer.params, parentParams)
      : layer.params;
    var layerPath = layer.path;

    // this should be done for the layer
    self.process_params(layer, paramcalled, req, res, function (err) {
      if (err) {
        return next(layerError || err);
      }

      if (route) {
        return layer.handle_request(req, res, next);
      }

      trim_prefix(layer, layerError, layerPath, path);
    });
  }

  function trim_prefix(layer, layerError, layerPath, path) {
    console.log({layer, layerError, layerPath, path})

    if (layerPath.length !== 0) {
      // Validate path breaks on a path separator
      var c = path[layerPath.length]
      if (c && c !== '/' && c !== '.') return next(layerError)

      // Trim off the part of the url that matches the route
      // middleware (.use stuff) needs to have the path stripped
      debug('trim prefix (%s) from url %s', layerPath, req.url);
      removed = layerPath;
      req.url = protohost + req.url.substr(protohost.length + removed.length);

      // Ensure leading slash
      if (!protohost && req.url[0] !== '/') {
        req.url = '/' + req.url;
        slashAdded = true;
      }

      // Setup base URL (no trailing slash)
      req.baseUrl = parentUrl + (removed[removed.length - 1] === '/'
        ? removed.substring(0, removed.length - 1)
        : removed);
    }

    debug('%s %s : %s', layer.name, layerPath, req.originalUrl);

    if (layerError) {
      layer.handle_error(layerError, req, res, next);
    } else {
      layer.handle_request(req, res, next);
    }
  }
};
```
关键部分就在这里了，这里定义了`next`函数。
并有`stack`储存的中间件，`var stack = self.stack;`。还有一些参数如`var idx = 0;`。
next能访问到这些变量，在传递`next`函数时就起到了非常重要的作用，也就是形成了闭包。
遍历stack数组，获取到里面的layer层，并判断该layer是否符合规则`matchLayer(layer, path)`，就比如我`get`请求`/login`，只会匹配到使用`app.use`的中间件(这是默认绑定在根部的）和符合规则的`get("/login")`的layer。
匹配到后，触发handle，也就是`layer.handle_request(req, res, next);`，这里就把next方法传递过去了，形成了闭包环境。

```
// layer.js
Layer.prototype.handle_request = function handle(req, res, next) {
  var fn = this.handle;
  console.log(fn.toString());
  if (fn.length > 3) {
    // not a standard request handler
    return next();
  }

  try {
    fn(req, res, next);
  } catch (err) {
    next(err);
  }
};
```
我们知道`var fn = this.handle;`就是我们定义的回调方法。然后`fn(req, res, next);`执行我们的回调函数，我们写的回调函数(`app.use(function(req, res, next)`)的`next`就是这个`next`方法,用于一直往下遍历`stack`（因为形成了闭包，所以idx会一直增加)，寻找符合规则的`layer`执行它的`handle`回调函数
所以我们书写项目的时候，终止函数就不要写`next()`了，会耗费不必要的性能。

总体就是这样吧，还有那些细节上的、匹配规则等等，有兴趣的可以去看看。




