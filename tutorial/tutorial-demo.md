# Router 路由 

Router 主要用来描述请求 URL 和具体承担执行动作的 Controller 的对应关系， 框架约定了 `./route.js` 文件用于统一所有路由规则。 


通过统一的配置，我们可以避免路由规则逻辑散落在多个地方，从而出现未知的冲突，集中在一起我们可以更方便的来查看全局的路由规则。 

<br>

## 如何定义 Router 

* `./route.js` 里面定义 URL 路由规则 

```js
// route.js
module.exports = [
  {
    match: "/",
    controller: "home.index"
  },
  {
    match: "/login",
    controller: "home.login",
    method: "post"
  }
]
```

<br>

* `controller/` 目录下面实现 Controller

```js
module.exports = {
  // 注释
  index: async function (scope) {
    Object.assign(scope,{
      title: "首页",
      content: "正常进入首页 Hello World"
    })
    console.log(this.say)
    await this.render("index")
  }
}
```

<br>

## 中间件的使用 
>如果我们想把用户某一类请求的参数都大写，可以通过中间件来实现。这里我们只是简单说明下如何使用中间件。 

<br>

### 我们先来通过编写一个简单的 gzip 中间件，来看看中间件的写法。 

```js
const isJSON = require('koa-is-json');
const zlib = require('zlib');
function* gzip(next) {
  yield next;
  // 后续中间件执行完成后将响应体转换成 gzip
  let body = this.body;
  if (!body) return;
  if (isJSON(body)) body = JSON.stringify(body);
  // 设置 gzip body，修正响应头
  const stream = zlib.createGzip();
  stream.end(body);
  this.body = stream;
  this.set('Content-Encoding', 'gzip');
}
``` 

可以看到，框架的中间件和 Koa 的中间件写法是一模一样的，所以任何 Koa 的中间件都可以直接被框架使用。

<br>  

### 在应用中使用中间件  
> 在应用中，我们可以完全通过配置来加载自定义的中间件，并决定它们的顺序。

```js
module.exports = {
  // 配置需要的中间件，数组顺序即为中间件的加载顺序
  middleware: [ 'gzip' ],
  // 配置 gzip 中间件的配置
  gzip: {
    threshold: 1024, // 小于 1k 的响应体不压缩
  },
};
```

