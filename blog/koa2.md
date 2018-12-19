- 前言

2018年的愿望一个都没有完成，觉得除了自己写个完整的网站能弥补这种滔天大罪之外，也没有什么能够挽回的了，一直想写个自己的博客，但是上班的时候，时间总是莫名流失了,划水的时光总是如此短暂。

今年好像一直在学新的东西，而且全是底层，由此也深深感受到了关于计算机网络的一穷二白，像食堂的汤一样清澈见底。

由于在egg上吃亏太多，我决定放过它也放过我。选择koa作为我咸鱼翻身的铁铲。

- 开始前的准备工作 

 了解node;会ES6语法;
 
- go

 1. 新建koa项目
 一般是项目文件夹使用直接使用```npm install```,哪里需要加哪里，还有一种方式在package.json中，dependencies是描述工程依赖和版本号，
 其它的字段一般是用来描述关于项目的信息，
 在dependencies里写好要安装的依赖后，就可以对项目执行npm install 命令，这样dependencies里面的依赖就会自动安装。
 2. 项目启动命令设置。
 对于新建的文件项目，由于底层是node,要运行项目的时候，比如app.js,可以直接运行 node app.js来启动项目，还有一种方式就是把启动命令
 定义在package.json中，也就是scripts描述。例：
 ```
 "scripts":{
 "start":"node app.js"
 }
 ```
 3. koa middleware
  koa的执行逻辑的核心代码是 :
 ```
 app.use(async(ctx,next)=>{
  await next();
  ctx.response.type="text/html"
  ctx.response.body='<h1> hello word</h1>';
  })
  
 ```
 原理是koa把很多async函数组合成一条有序链，每个async的函数都是其中一环，它们各自执行自己的职责，而```await next()```则是用来调用下一个async函数。
 所以要注意顺序。下面这个例子就很完美的诠释了”app.use()的顺序决定了middleware的顺序“。
 ```
  const Koa=require('koa');
  const app=new Koa();
  app.use(async (ctx, next) => {
    console.log('第一1');
    await next(); // 调用下一个middleware
    console.log('第一2');
  });

  app.use(async (ctx, next) => {
    console.log('第二1');
    await next(); // 调用下一个middleware
    console.log('第二2');
  });

  app.use(async (ctx, next) => {
    console.log('第三1');
    await next();
    console.log('第三2');
  });
  app.listen(3000);
 ```
  如果一个middleware没有调用await next(),后面的就不再执行了，所以可以用来决定是否继续处理请求，还是直接返回错误码。
附上摘抄的示例。
```
app.use(async (ctx, next) => {
    if (await checkUserPermission(ctx)) {
        await next();
    } else {
        ctx.response.status = 403;
    }
});
```
4. 处理url （route）

首先是和上一步一样安装依赖，可以直接把```"koa-router": "7.0.0"```在package.json里。接下来就是使用koa-router来处理url。
在没用koa-router之前 ，我们一般这样来处理url:
```
app.use(async (ctx, next) => {
    if (ctx.request.path === '/') {
        ctx.response.body = 'index page';
    } else {
        await next();
    }
});
```
第一步是调用：
```
const koa =require("koa");
const router = require('koa-router)(); //这里是返回函数，相当于 const fn_router=require("koa-router");const router= fn_router();

```
接下来是通过router处理url
```$xslt
router.get('/hello/:name', async (ctx, next) => {
    var name = ctx.params.name;
    ctx.response.body = `<h1>Hello, ${name}!</h1>`;
});

app.use(router.routes()); // 把router加入middleware
```
5. koa-bodyparser

由于post请求经常是发送表单或者是json，由于node和koa提供的request对象，都不提供解析request的body的功能，
所以需要引入一个新的middleware用以解析原始的request请求，然后把解析后的参数，绑定到ctx.request.body.这就是koa-bodyparser的功能。
依然可以通过npm install 或 dependencies来安装，引入时需要注意要放在router之前。依然是一个摘抄的示例~
```$xslt
router.get('/', async (ctx, next) => {
    ctx.response.body = `<h1>Index</h1>
        <form action="/signin" method="post">
            <p>Name: <input name="name" value="koa"></p>
            <p>Password: <input name="password" type="password"></p>
            <p><input type="submit" value="Submit"></p>
        </form>`;
});

router.post('/signin', async (ctx, next) => {
    var
        name = ctx.request.body.name || '', //表单的name字段，如果该字段不存在，默认值设置为''
        password = ctx.request.body.password || ''; 
    console.log(`signin with name: ${name}, password: ${password}`);
    if (name === 'koa' && password === '12345') {
        ctx.response.body = `<h1>Welcome, ${name}!</h1>`;
    } else {
        ctx.response.body = `<h1>Login failed!</h1>
        <p><a href="/">Try again</a></p>`;
    }
});
```
如果对每个url都做此处理的话，整个文件就会十分臃肿，所以需要整合一下。具体的方法是，用函数封装对url的处理，并通过module.exports暴露给外界。
依然是坚强的摘抄。
```$xslt
var fn_index = async (ctx, next) => {
    ctx.response.body = `<h1>Index</h1>
        <form action="/signin" method="post">
            <p>Name: <input name="name" value="koa"></p>
            <p>Password: <input name="password" type="password"></p>
            <p><input type="submit" value="Submit"></p>
        </form>`;
};

var fn_signin = async (ctx, next) => {
    var
        name = ctx.request.body.name || '',
        password = ctx.request.body.password || '';
    console.log(`signin with name: ${name}, password: ${password}`);
    if (name === 'koa' && password === '12345') {
        ctx.response.body = `<h1>Welcome, ${name}!</h1>`;
    } else {
        ctx.response.body = `<h1>Login failed!</h1>
        <p><a href="/">Try again</a></p>`;
    }
};

module.exports = {
    'GET /': fn_index,
    'POST /signin': fn_signin
};
```
然后就是对入口文件的处理，使它自动扫描controller目录，找到js文件，并注册每个url；思路是先过滤出.js文件，再对每个文件进行处理。
封装进函数后再进行调用。更好的方法是把扫描和创建router的代码拿出来，作为一个middleware使用。see the demo copy from Dalao~~
```$xslt
var files = fs.readdirSync(__dirname + '/controllers');

// 过滤出.js文件:
var js_files = files.filter((f)=>{
    return f.endsWith('.js');
});
// 处理每个js文件:
for (var f of js_files) {
    console.log(`process controller: ${f}...`);
    // 导入js文件:
    let mapping = require(__dirname + '/controllers/' + f);
   
    
    }



//controller.js
const fs = require('fs');

function addMapping(router, mapping) {
    for (var url in mapping) {
            if (url.startsWith('GET ')) {
                var path = url.substring(4);
                router.get(path, mapping[url]);
                console.log(`register URL mapping: GET ${path}`);
            } else if (url.startsWith('POST ')) {
                var path = url.substring(5);
                router.post(path, mapping[url]);
                console.log(`register URL mapping: POST ${path}`);
            } else {
                console.log(`invalid URL: ${url}`);
            }
        }
}

function addControllers(router, dir) {
    var files = fs.readdirSync(__dirname + '/controllers');
       var js_files = files.filter((f) => {
           return f.endsWith('.js');
       });
   
       for (var f of js_files) {
           console.log(`process controller: ${f}...`);
           let mapping = require(__dirname + '/controllers/' + f);
           addMapping(router, mapping);
       }
}
addControllers(router);

module.exports = function (dir) {
    let
        controllers_dir = dir || 'controllers', // 如果不传参数，扫描目录默认为'controllers'
        router = require('koa-router')();
    addControllers(router, controllers_dir);
    return router.routes();
};
// app.js 入口文件


const controller = require('./controller');// 导入controller middleware:
...
app.use(controller());// 使用middleware:
```
所有处理URL的函数按功能组存放在controllers目录，今后我们也只需要不断往这个目录下加东西就可以了，app.js保持不变。

6. 接下来就是view模板。
有很多可以选择，但是由于大佬选的nunjucks,菜的如我就没有选择的余地了。天下的模板都一样，套上自带的神奇语法。nunjucks也不例外，
特别是它基本上是用javascript重新实现了jinjia2.【由于偶然机会，帮小伙伴处理了jinja2相关项目问题借此知道了还存在这么奇怪的名字的框架
故也不算是全抄】

过程是先用nunjucks写html模板，并且用实际数据来渲染模板并获得最终的html输出。
接下来是要写使用nunjucks的函数render.
在入口文件中抄入：
```$xslt
const nunjucks = require('nunjucks');

function createEnv(path, opts) {
    var
        autoescape = opts.autoescape === undefined ? true : opts.autoescape,
        noCache = opts.noCache || false,
        watch = opts.watch || false,
        throwOnUndefined = opts.throwOnUndefined || false,
        env = new nunjucks.Environment(
            new nunjucks.FileSystemLoader('views', {
                noCache: noCache,
                watch: watch,
            }), {
                autoescape: autoescape,
                throwOnUndefined: throwOnUndefined
            });
    if (opts.filters) {
        for (var f in opts.filters) {
            env.addFilter(f, opts.filters[f]);
        }
    }
    return env;
}

var env = createEnv('views', {
    watch: true,
    filters: {
        hex: function (n) {
            return '0x' + n.toString(16);
        }
    }
});
// 变量env就表示Nunjucks模板引擎对象，它有一个render(view, model)方法，正好传入view和model两个参数，并返回字符串。
   
//    创建env需要的参数可以查看文档获知。我们用autoescape = opts.autoescape && true这样的代码给每个参数加上默认值，
 //  最后使用new nunjucks.FileSystemLoader('views')创建一个文件系统加载器，从views目录读取模板。
```
nunjucks还提供了条件判断、循环等功能，对于输出文档更加方便。可以看文档了解模板继承，以及细小语法。

来自大佬的提醒 ：性能问题主要出现在从文件读取模板内容这一步。这是一个IO操作，在Node.js环境中，我们知道，单线程的JavaScript最不能忍受的就是同步IO
，但Nunjucks默认就使用同步IO读取模板文件。好消息是Nunjucks会缓存已读取的文件内容，也就是说，模板文件最多读取一次，就会放在内存中，
后面的请求是不会再次读取文件的，只要我们指定了noCache: false这个参数。在开发环境下，可以关闭cache，这样每次重新加载模板，便于实时修改模板。
在生产环境下，一定要打开cache，这样就不会有性能问题。Nunjucks也提供了异步读取的方式，但是这样写起来很麻烦，有简单的写法我们就不会考虑复杂的写法。
保持代码简单是可维护性的关键。

7. 使用MVC。

也就是将url和模板有机充分合理的结合起来，当用户请求url里，koa调用函数处理url并将渲染好的模板输出给浏览器。






















 




























-  附录

ctx对象有一些简写的方法，例如ctx.url相当于ctx.request.url，ctx.type相当于ctx.response.type。
 
 
 
------------------------------来源于 https://www.liaoxuefeng.com/ -------------------------------------
