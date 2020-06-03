# node-博客（文章前台）

## 开发环境和生产环境，第三方模块morgan打印信息

通过系统环境变量区分【在电脑属性中】

开发环境

- 变量名
  - `NODE_ENV`
- 变量值
  - `development`
  
生产环境

- 变量名
  - `NODE_ENV`
- 变量值
  - `production`

- **只在开发环境中**，将客户端发送到服务器端的请求信息打印到控制台中
  - 需要导入 `morgan` 模块,是中间件函数
  - 通过 `app.use()` 中间件函数来调用

app.js

```js
// 导入morgan这个第三方模块
const morgan = require('morgan');

// 获取系统环境变量 返回值是对象
if (process.env.NODE_ENV == 'development') {
    // 当前是开发环境
    console.log('当前是开发环境')
    app.use(morgan('dev'))
} else {
    // 当前是生产环境
    console.log('当前是生产环境')
}
```

## 第三方模块config配置信息

1、在app.js导入config模块
2、建立config文件夹
3、在config文件夹下新建【根据环境的不同找到对应的json，如果都没找到，就是用dafault.json】

- `dafault.json`
- `development.json`
- `production.json`

***

- `config.get('xxx')`
  - 得到json中的对应属性的属性值

app.js

```js
// 导入config模块
const config = require('config');

//测试
console.log(config.get('title'))
```

dafault.json

```json
{
    "title": "博客管理系统1"
}
```

development.json

```json
{
}
```

production.json

```json
{
    "title": "博客管理系统2"
}
```

- 4、假设现在是开发环境development
  - 控制台输出 `dafault.json` 中的 **博客管理系统1**

- 5、将config用于敏感的密码保存，保存在电脑属性中的环境变量下

- 环境变量下新建系统环境变量
  - 变量名
    - `APP_PASSWORD`
  - 变量值
    - `xxxx`【这里是敏感密码】

custom-environment-variables.json

```json
{
    "db": {
        "pwd": "APP_PASSWORD"
    }
}
```

- 6、将config用于数据库连接

model/connect.js

```js
const config = require('config');
console.log(config.get('db.host'))
//连接数据库
mongoose.connect(`mongodb://${config.get('db.user')}:${config.get('db.pwd')}@${config.get('db.host')}:${config.get('db.port')}/${config.get('db.name')}`, {useNewUrlParser: true })
    .then(() => console.log('数据库连接成功'))
    .catch(() => console.log('数据库连接失败'))
```

development.json

```json
{
    "db": {
        "user": "blog",
        "pwd":"123456",
        "host": "localhost",
        "port": "27017",
        "name": "blog"
    }
}
```

## 文章前台展示--文章首页和文章详情

home.js【专门用于文章展示详情的页面路由】

```js
// 引用expess框架
const express = require('express');
// 创建博客展示页面路由
const home = express.Router();

// 博客前台首页的展示页面
home.get('/', require('./home/index'));

// 博客前台文章详情展示页面
home.get('/article', require('./home/article'));

// 创建评论功能路由
home.post('/comment', require('./home/comment'));

// 将路由对象做为模块成员进行导出
module.exports = home;
```

### 1、文章首页展示页面路由

- 1.1、将文章集合函数导入
  - `const { Article } = require('../../model/article');`
- 1.2、在数据库中异步查找数据 `find()`
- 1.3、**多集合联合查询,联合作者的集合**
  - `let result = await Article.find().populate('author')`
  - `res.send(result)` 在浏览器中查看有无数据
- 1.4、用到数据库的分页模块
  - `const pagination = require('mongoose-sex-page');`
  - `pagination(Article);` 【参数是文章集合的构造函数】
  - `page(page)`
    - 当前第page页
  - `size(2)`
    - 每页显示数据2条
  - `display(3)`
    - 客户端一次性显示的页面数量3页
  - `exec()`
    - 向数据库发出查询请求
  - 只有records中是自己写的，其他都是模块自带

```json
{
    "page":1,//当前页
    "size":2,//每页显示条数
    "total":8,//总共的数据数据条数
    "records":[
        {
            这里是Article集合中查询出来的数据对象
        }
    ],
    "page":4,//总共的页数
    "display":[1,2,3]//客户端一次页面中存放显示的页码
}
```

- 1.5、渲染文章首页页面模板并传递数据
  - `res.render('home/default.art', { xxx });`

route/home/index.js

```js
const { Article } = require('../../model/article');
// 导入分页模块
const pagination = require('mongoose-sex-page');

module.exports = async (req, res) => {
  // 获取页码值
  const page = req.query.page;

  // 从数据库中查询数据
  let result = await pagination(Article).page(page).size(4).display(5).find().populate('author').exec();

  // 渲染模板并传递数据
  res.render('home/default.art', {
    result: result
  });
}
```

### 2、文章详情页面路由

- 2.1、渲染文章详情页面模板并传递数据
  - `res.render('home/article.art', { xxx });`

route/home/article.js

```js
// 导入文章集合构造函数
const { Article } = require('../../model/article');
// 导入评论集合构造函数
const { Comment } = require('../../model/comment');

module.exports = async (req, res) => {
  // 接收客户端传递过来的文章id值
  const id = req.query.id;
  // 根据id查询文章详细信息
  let article = await Article.findOne({_id: id}).populate('author');
  // 查询当前文章所对应的评论信息
  let comments = await Comment.find({aid: id}).populate('uid')
  res.render('home/article.art', {
    article, comments
  });
}
```

### 3、抽离出相同的公共模板

公共模板抽离到common文件夹下

- 3.1、整体框架
  - 引用用继承`{{extend './common/layout.art'}}`

views/home/common/layout.art

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>首页</title>
  <link rel="stylesheet" href="/home/css/base.css">
  {{block 'link'}}{{/block}}
</head>
<body>
  {{block 'main'}}{{/block}}
</body>
</html>
```

- 3.2、头部框架
  - 引用用包含`{{include './common/header.art'}}`

views/home/common/header.art

```html
<!-- 头部框架开始 -->
<div class="header">
  <div class="w1100">
    <!-- 网站logo开始 -->
    <h1 class="logo fl">
      <a href="default.html"><img src="images/logo.png" alt="黑马程序员"></a>
    </h1>
    <!-- 网站logo结束 -->
    <!-- 网站导航开始 -->
    <ul class="navigation fr">
      <li>
        <a class="active" href="index.html">首页</a>
      </li>
      <li>
        <a href="#">登录</a>
      </li>
    </ul>
    <!-- 网站导航结束 -->
  </div>
</div>
<!-- 头部框架结束 -->
```

### 4、文章首页模板

- 4.1、必须修改链接的资源为绝对路径
  - 因为模板中引用的资源，如果是相对路径的话，是相对于地址栏中的
  - `href="/home/css/index.css"`【此时是在public文件夹下】

views/home/default.art

```html
{{block 'link'}}
  <link rel="stylesheet" href="/home/css/index.css">
{{/block}}
```

- 4.2、根据文章的循环是的索引判断文章出现在左边还是右边
  - 先循环`{{each result.records}}xxx{{/each}}`
  - 再索引判断【索引是偶数时，在左边显示样式；索引是奇数时，在右边显示样式】
    - `<li class="{{$index%2 == 0 ? 'fl' : 'fr'}}">xxx</li>`
- 4.3、显示数据
  - `{{$value.cover}}`【文章封面】
  - `{{$value.title}}`【文章标题】
  - `{{$value.author.username}}`【文章作者】
  - `{{dateFormat($value.publishDate, 'yyyy-mm-dd')}}`【日期，在app.js中引用到第三方模块】

- 4.4、文章内容简介，截取数据
  - `{{$value.content}}`【未截取】
  - `{{@$value.content.replace(/<[^>]+>/g, '').substr(0, 90) + '...'}}`【截取】
    - `replace(/<[^>]+>/g, '')`【把所有的html标签替换为空】
    - `substr(0, 90)`【对文章内容从下标0位置开始截取，最多显示90个字符】
    - `'...'`【剩余内容用省略号表示】

views/home/default.art

```html
<!-- 文章列表开始 -->
<ul class="list w1100">
  {{each result.records}}
  <li class="{{$index%2 == 0 ? 'fl' : 'fr'}}">
    <a href="/home/article?id={{@$value._id}}" class="thumbnail">
      <img src="{{$value.cover}}">
    </a>
    <div class="content">
      <a class="article-title" href="/home/article?id={{@$value._id}}">{{$value.title}}</a>
      <div class="article-info">
        <span class="author">{{$value.author.username}}</span>
        <span>{{dateFormat($value.publishDate, 'yyyy-mm-dd')}}</span>
      </div>
      <div class="brief">
        {{@$value.content.replace(/<[^>]+>/g, '').substr(0, 90) + '...'}}
      </div>
    </div>
  </li>
  {{/each}}
</ul>
<!-- 文章列表结束 -->
```

- 4.5、文章首页的分页显示
  - 页码存储在json数据中的display属性中
  - `{{each result.display}}xxx{{/each}}`
  - 文章页码的《
  - `{{if result.page > 1}}xxx{{/if}}`
  - 文章页码的》
  - `{{if result.page < result.pages}}xxx{{/if}}`

- 4.6、在客户端`views/home/default.art`
  - `<a href="/admin/article?page={{$value}}">`
  - 把当前页的值通过地址栏get请求，传递给服务器
  - `<a href="/home/?page={{result.page-1}}">`【上一页】
  - `<a href="/home/?page={{result.page - 0 + 1}}">`【下一页】

- 4.7、在服务器端`route/home/index.js`接收客户端传递过来的页码
  - `const page = req.query.page;`
  - `let articles = await pagination(Article).find().page(page)...`

- 4.8、做当前页页码的样式判断
  - `class="{{$value == result.page ? 'active' : ''}}"`

views/home/default.art

```html
<!-- 分页开始 -->
<div class="page w1100">
  {{if result.page > 1}}
  <a href="/home/?page={{result.page-1}}">上一页</a>
  {{/if}}
  {{each result.display}}
  <a href="/home/?page={{$value}}" class="{{$value == result.page ? 'active' : ''}}">{{$value}}</a>
  {{/each}}
  {{if result.page < result.pages}}
  <a href="/home/?page={{result.page - 0 + 1}}">下一页</a>
  {{/if}}
</div>
<!-- 分页结束 -->
```

route/home/index.js

```js
const { Article } = require('../../model/article');
// 导入分页模块
const pagination = require('mongoose-sex-page');

module.exports = async (req, res) => {
  // 获取页码值
  const page = req.query.page;
···
}
```

### 5、文章详情模板

- 5.1、必须修改链接的资源为绝对路径
  - 因为模板中引用的资源，如果是相对路径的话，是相对于地址栏中的
  - `href="/home/css/article.css"`【此时是在public文件夹下】

views/home/article.art

```html
{{block 'link'}}
  <link rel="stylesheet" href="/home/css/article.css">
{{/block}}
```

- 5.2、在文章首页点击**文章标题a标签**或者**点击图片**，到文章详情页的跳转
  - 通过get参数，通过地址栏传递id={{@$value._id}}【@表示id的原文输出】
  - `<a href="/home/article?id={{@$value._id}}" class="thumbnail"><img src="{{$value.cover}}"></a>`
  - `<a class="article-title" href="/home/article?id={{@$value._id}}">{{$value.title}}</a>`

views/home/default.art

```html
<a href="/home/article?id={{@$value._id}}" class="thumbnail">
  <img src="{{$value.cover}}">
</a>
<div class="content">
  <a class="article-title" href="/home/article?id={{@$value._id}}">{{$value.title}}</a>
  <div class="article-info">
    <span class="author">{{$value.author.username}}</span>
    <span>{{dateFormat($value.publishDate, 'yyyy-mm-dd')}}</span>
  </div>
  <div class="brief">
    {{@$value.content.replace(/<[^>]+>/g, '').substr(0, 90) + '...'}}
  </div>
</div>
```

- 5.3、服务器端获取到id，根据id查询文章数据
  - `const id = req.query.id;`
- 导入文章集合构造函数
  - `const { Article } = require('../../model/article');`
- 查询条件是id，同时多集合联合查询
  - `let article = await Article.findOne({_id: id}).populate('author');`
- 把查询出来的数据响应给客户端【第二个参数是发送数据】
  - `res.render('home/article.art', { article, comments });`

route/home/article.js

```js
// 导入文章集合构造函数
const { Article } = require('../../model/article');
// 导入评论集合构造函数
const { Comment } = require('../../model/comment');

module.exports = async (req, res) => {
  // 接收客户端传递过来的文章id值
  const id = req.query.id;
  // 根据id查询文章详细信息
  let article = await Article.findOne({_id: id}).populate('author');
  // 查询当前文章所对应的评论信息
  let comments = await Comment.find({aid: id}).populate('uid')

  // res.send('欢迎来到博客文章详情页面')
  res.render('home/article.art', { article, comments });
}
```

- 5.4、文章详情页的浏览器拿到服务器端发送的数据，将数据显示在页面上
  - `{{article.title}}`【文章标题】
  - `{{article.author.username}}`【文章作者】
  - `{{dateFormat(article.publishDate, 'yyyy-mm-dd')}}`【日期，，在app.js中调用第三方模块】
  - `{{@article.content}}`【文章内容】

views/home/default.art

```html
<!-- 文章头部开始 -->
<div class="article-header">
  <h3 class="article-title">{{article.title}}</h3>
  <div class="article-info">
    <span class="author">{{article.author.username}}</span>
    <span>{{dateFormat(article.publishDate, 'yyyy-mm-dd')}}</span>
  </div>
</div>
<!-- 文章头部结束 -->
<!-- 文章内容开始 -->
<div class="article-content">
  {{@article.content}}
</div>
<!-- 文章内容结束 -->
```

### 6、文章评论

- 6.1、定义文章评论集合规则，并创建评论集合
  - `ref: 'Article'`【关联文章集合】
  - `ref: 'User'`【关联评论人用户集合】

model/comment.js

```js
// 引入mongoose模块
const mongoose = require('mongoose');

// 创建评论集合规则
const commentSchema = new mongoose.Schema({
  // 文章id
  aid: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Article'
  },
  // 评论人用户id
  uid: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  },
  // 评论时间
  time: {
    type: Date
  },
  // 评论内容
  content: {
    type: String
  }
});

// 创建评论集合
const Comment = mongoose.model('Comment', commentSchema);

// 将评论集合构造函数作为模块成员进行导出
module.exports = {
  Comment
}
```

- 6.2、设置用户文章的登录拦截
  - 在用户已经登录的状态下。
  - 如果是普通用户，将用户重定向到文章首页
    - `if (req.session.role == 'normal') {return res.redirect('/home/')}`
  - 如果是普通用户，将用户重定向到后台管理页面
    - `next();`
    - 【因为这个js函数是直接在app.js中`app.use('/admin', require('./middleware/loginGuard'));`中拦截，所以跳转的是/admin后台管理】

middleware/loginGuard.js

```js
const guard = (req, res, next) => {
  // 如果用户是登录的 将请求放行
  // 如果用户不是登录的 将请求重定向到登录页面
  if (req.url != '/login' && !req.session.username) {
    res.redirect('/admin/login');
  } else {
    // 如果用户是登录状态 并且是一个普通用户
    if (req.session.role == 'normal') {
      // 让它跳转到博客首页 阻止程序向下执行
      return res.redirect('/home/')
    }
    // 用户是登录状态 将请求放行
    next();
  }
}

module.exports = guard;
```

- 6.3、根据用户登录状态，判断是否给出评论登录的提示【评论的显示和隐藏】
  - 因为根据`/route/admin/login.js`中的全局变量，可以直接拿到userInfo的信息
  - `req.app.locals.userInfo = user;`
- {{if userInfo}}xxx{{/if}}【这样模板可以直接判断是否登录】
  - $\color{red}但是当用户退出时，只是删除服务器端的session和客户端的cookie,并没有清空全局下的userInfo的用户数据$
  - 这样会造成在用户退出时，仍然可以评论
- 解决办法：新建一个logout.js，在用户退出时清空这些用户数据
  - `req.app.locals.userInfo = null;`

route/admin/login.js

```js
if ( isValid ) {
    // 登录成功
    // 将用户名存储在请求对象中
    req.session.username = user.username;
    // 将用户角色存储在session对象中
    req.session.role = user.role;
    req.app.locals.userInfo = user;
    // 对用户的角色进行判断
    if (user.role == 'admin') {
      // 重定向到用户列表页面
      res.redirect('/admin/user');
    } else {
      // 重定向到博客首页
      res.redirect('/home/');
    }
}else{···}
```

views/home/article.art

```html
{{if userInfo}}
      ···【这里是登录后显示的内容】
{{else}}
    <div><h2>先进行登录，再对文章进行评论</h2></div>
{{/if}}
```

route/admin/logout.js

```js
module.exports = (req, res) => {
  // 删除session
  req.session.destroy(function () {
      // 删除cookie
      res.clearCookie('connect.sid');
      // 重定向到用户登录页面
      res.redirect('/admin/login');
      // 清除登录页面中模板中的用户信息
      req.app.locals.userInfo = null;
  });
}
```

- 6.4、评论的添加
- form表单设置post请求,action属性跳转到/home/comment
  - `<form class="comment-form" action="/home/comment" method="post">`  
- 为评论的文本框添加name属性，用于发送到服务器端
  - `name="content"`
- $\color{red}必须添加隐藏域$，并添加 **name属性存储向服务器端发布评论的用户id和文章id** ，同时用 **value属性存储从服务器端获取的用户id和文章id，注意@原文输出**
  - `<input type="hidden" name="uid" value="{{@userInfo._id}}">`
  - `<input type="hidden" name="aid" value="{{@article._id}}">`
- 不用添加日期的name属性，可以直接在模板中写，并从服务器端直接获取即可

views/home/article.art

```html
<!-- 文章评论开始 -->
<div class="article-comment">
  {{if userInfo}}
      <h4>评论</h4>
      <form class="comment-form" action="/home/comment" method="post">
        <textarea class="comment" name="content"></textarea>
        <input type="hidden" name="uid" value="{{@userInfo._id}}">
        <input type="hidden" name="aid" value="{{@article._id}}">
        <div class="items">
          <input type="submit" value="提交">
        </div>
      </form>
  {{else}}
    <div><h2>先进行登录，再对文章进行评论</h2></div>
  {{/if}}

   ···

</div>
<!-- 文章评论结束 -->
```

- 6.5、创建评论功能提交路由comment.js
- 先导入评论集合构造函数
  - `const { Comment } = require('../../model/comment');`
- 接收客户端传递过来的信息
  - `const { content, uid, aid } = req.body;`
- 存储数据到评论数据库中
  - `await Comment.create({```});`
- 因为表单提交后会自动跳转到提交地址，但是正常提交应该仍然是原页面，所以是重定向会文章详情页面
- 在跳转的同时需要传递文章的id【不然不知道重定向到那篇文章】
  - `res.redirect('/home/article?id='+ aid);`

app.js

```js
// 创建评论功能路由
home.post('/comment', require('./home/comment'));
```

route/home/comment.js

```js
// 将评论集合构造函数进行导入
const { Comment } = require('../../model/comment');

module.exports = async (req, res) => {
  // 接收客户端传递过来的请求参数
  const { content, uid, aid } = req.body;

  // 将评论信息存储到评论集合中
  await Comment.create({
    content: content,
    uid: uid,
    aid: aid,
    time: new Date()
  });

  // 将页面重定向回文章详情页面
  res.redirect('/home/article?id='+ aid);
}
```

- 6.6、评论展示功能【数据库查询】
- 从保存到评论数据库中对应当前文章的评论信息中，重新查询数据对客户端进行页面展示
  - `let comments = await Comment.find({aid: id}).populate('uid')`
- 重定向页面并导出commits评论数据
  - `res.render('home/article.art', { ··· , comments });`

route/home/article.js

```js
// 导入文章集合构造函数
const { Article } = require('../../model/article');
// 导入评论集合构造函数
const { Comment } = require('../../model/comment');

module.exports = async (req, res) => {
  // 接收客户端传递过来的文章id值
  const id = req.query.id;
  ···
  // 查询当前文章所对应的评论信息
  let comments = await Comment.find({aid: id}).populate('uid')

  res.render('home/article.art', { ··· , comments });
}
```

- 6.7、评论展示功能【页面渲染】
- 循环commits评论数据
  - `{{each comments}}xxx{{/each}}`
- 渲染数据
  - `{{$value.uid.username}}`
  - `{{$value.uid.email}}`
  - `{{$value.content}}`
- 当初没有日期提交，就是现在从服务器端导出的评论时间进行展示
  - `{{dateFormat($value.time, 'yyyy-mm-dd')}}`

views/home/article.art

```html
···
  <div class="comment-list">
    {{each comments}}
    <div class="mb10">
      <div class="article-info">
        <span class="author">{{$value.uid.username}}</span>
        <span>{{dateFormat($value.time, 'yyyy-mm-dd')}}</span>
        <span>{{$value.uid.email}}</span>
      </div>
      <div class="comment-content">
        {{$value.content}}
      </div>
    </div>
    {{/each}}
  </div>
···
```
