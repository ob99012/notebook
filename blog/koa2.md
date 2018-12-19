- 前言

2018年的愿望一个都没有完成，觉得除了自己写个完整的网站能弥补这种滔天大罪之外，也没有什么能够挽回的了，一直想写个自己的博客，但是上班的时候，时间总是莫名流失了,划水的时光总是如此短暂。

今年好像一直在学新的东西，而且全是底层，由此也深深感受到了关于计算机网络的一穷二白，像食堂的汤一样清澈见底。

由于在egg上吃亏太多，我决定放过它也放过我。选择koa作为我咸鱼翻身的铁铲。

- 开始前的准备工作 

 了解node;会ES6语法;
 
- go

 1. 新建koa项目
 对项目文件夹使用```npm init```，在package.json中，dependencies是描述工程依赖和版本号，其它的字段一般是用来描述关于项目的信息，
 在dependencies里写好要安装的koa版本后，就可以对项目执行npm install 命令，这样dependencies里面的依赖就会自动安装。
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
 原理是koa把很多async函数组合成一条有序链，每个async的函数都是其中一环，它们各自执行自己的职责，而```await next()```则是用来调用下一个async函数。所以要注意顺序。下面这个例子就很完美的诠释了”app.use()的顺序决定了middleware的顺序“。
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

首先是和上一步一样安装依赖，可以直接加在
 




























-  附录

ctx对象有一些简写的方法，例如ctx.url相当于ctx.request.url，ctx.type相当于ctx.response.type。
 
 
 
