# 个人博客

## 一、准备

### 1.结构

```txt
blog/
|
+- .vscode/
|  |
|  +- launch.json           <-- VSCode 配置文件
|
+- package.json             <-- 项目描述文件
|
+- node_modules/            <-- npm安装的所有依赖包
|
+- public/                  <-- 静态资源文件夹
   |
   +-uploads/
   |
   +-admin/
        |
        +- images/          <--
        |
        +- css/             <--
            |
            +- base.css/    <--
        |
        +- js/              <--
            |
            +- common.js/   <--
        |
        +- lib/             <--
            |
            +- bookstrap/   <--
            |
            +- ckeditor5/   <--
            |
            +- echarts/     <--
            |
            +- jquery/      <--
    |
   +-home/
        |
        +- images/          <--
        |
        +- css/             <--
            |
            +- base.css/    <--
            |
            +- article.css/ <--
            |
            +- index.css/   <--
        |
        +- js/              <--
        |
        +- lib/             <--
            |
            +- sweetalert/  <--
        |
        +- article.html     <--
        |
        +- default.html     <--
    |
    +- article.html         <--
    |
    +- article-edit.html    <--
    |
    +- user.html            <--
    |
    +- user-edit.html       <--
    |
    +- login.html           <--
|
+- route/                   <-- 路由文件夹
    |
    +-admin/
        |
        +- login.js         <--
        |
        +- loginPage.js     <--
        |
        +- layout.js        <--
        |
        +- user-add.js      <--
        |
        +- user-edit.js     <--
        |
        +- user-edit-fn.js  <--
        |
        +- userPage.js      <--
        |
        +- article.js       <--
        |
        +- article-add.js   <--
        |
        +- article-edit.js  <--
    |
    +-home/
        |
        +- article.js       <--
        |
        +- commen.js        <--
        |
        +- index.js         <--
    |
    +- admin.js             <--blog的管理页面路由
    |
    +- home.js              <--blog的展示页面路由
|
+- model/                   <--数据库文件夹
    |
    +- article.js           <--
    |
    +- comment.js           <--
    |
    +- connect.js           <--
    |
    +- user.js              <--
|
+- middleware/              <--
    |
    +- loginGuard.js        <--
|
+- views/                   <-- 模板文件夹
   |
   +- admin/                <--
        |
        +- common/          <--
            |
            +- xxx.art      <--
        |
        +- xxx.art          <--
   |
   +- home/                 <--
        |
        +- common/          <--
            |
            +- xxx.art      <--
        |
        +- xxx.art          <--
|
+- config/                  <--
   |
   +- custom-environment.json   <--
   |
   +- default.json          <--
   |
   +- development.json      <--
   |
   +- production.json       <--
|
+- app.js                   <-- 主js
|
+- hash.js                  <--
|
+- joi.js                   <--
```

### 2.引入

```node
npm init -y
npm i express
npm i express-session
npm i mongoose
npm i mongoose-sex-page
npm i art-template
npm i express-art-template
npm i formidable
npm i path
npm i body-parser
npm i dateformat
npm i morgan
npm i config
npm i bcrypt
npm i joi
npm i nodemon
```

### 3.流程

- 1.初始化
  - app.js创建网站服务器
  - route文件夹admin.js、home.js构建模块化路由
  - route文件夹admin.js、home.js构建博客管理页面模板

- 2.登录
  - 创建用户集合
  - 连接数据库
  - 创建用户集合
  - 初始化用户

- 3.为登录表单项设置请求地址、请求方式以及表单项name属性

- 4.当用户点击登录按钮时，客户端验证用户是否填写了登录表单
  - 如果其中一项没有输入，阻止表单提交

- 5.服务器端接收请求参数，验证用户是否填写了登录表单
  - 如果其中一项没有输入，为客户端做出响应，阻止程序向下执行

- 6.根据邮箱地址查询用户信息
  - 如果用户不存在，为客户端做出响应，阻止程序向下执行
  - 如果用户存在，将用户名和密码进行比对
    - 比对成功，用户登录成功
    - 比对失败，用户登录失败

- 7.保存登录状态

### 4.route文件夹和app.js

注意app.js用80端口监听

1.在admin.js和home.js中引入express模块，创建路由

> `const admin或home = express.Router();`

2.在admin.js和home.js中暴露

- admin.js
  - blog的管理页面路由，用`module.exports = admin;`暴露
- home.js
  - blog的展示页面路由，用`module.exports = home;`暴露

3.在app.js引入路由模块

> `const home = require('./route/home');`  
 `const admin = require('./route/admin');`

4.在app.js为路由匹配请求路径
> `app.use('/home', home);`  
 `app.use('/admin', admin);`

5.$\color{red}注意路由继承概念，一个请求地址下，继续接第二个请求地址$

- admin.js和home.js中
  - `/admin` 和 `/home` 是第一级的请求地址
- admin文件夹中的js文件
  - 如:`/admin/login`、`/admin/user`是第二级的请求地址
- home文件夹中的js文件
  - 如:`/home/article`、`/admin/comment`是第二级的请求地址

admin.js部分

```js
// 渲染登录页面
admin.get('/login', require('./admin/loginPage'));

// 实现登录功能
admin.post('/login', require('./admin/login'));

// 创建用户列表路由
admin.get('/user', require('./admin/userPage'));

// 实现退出功能
admin.get('/logout', require('./admin/logout'));

// 创建用户编辑页面路由
admin.get('/user-edit', require('./admin/user-edit'));

// 创建实现用户添加功能路由
admin.post('/user-edit', require('./admin/user-edit-fn'));

// 用户修改功能路由
admin.post('/user-modify', require('./admin/user-modify'));

// 用户删除功能路由
admin.get('/delete', require('./admin/user-delete'));

// 文章列表页面路由
admin.get('/article', require('./admin/article'));

// 文章编辑页面路由
admin.get('/article-edit', require('./admin/article-edit'));

// 实现文章添加功能的路由
admin.post('/article-add', require('./admin/article-add'));
```

home.js部分

```js
// 博客前台首页的展示页面
home.get('/', require('./home/index'));

// 博客前台文章详情展示页面
home.get('/article', require('./home/article'));

// 创建评论功能路由
home.post('/comment', require('./home/comment'));
```

6.public静态资源文件

在app.js中开放静态资源文件
> `app.use(express.static(path.join(__dirname, 'public')));`

## 二、views模板

### 1.模板在app.js的配置

```js
// 告诉express框架模板所在的位置
app.set('views', path.join(__dirname, 'views'));
// 告诉express框架模板的默认后缀是art
app.set('view engine', 'art');
// 当渲染后缀为art的模板时 所使用的模板引擎是express-art-template
app.engine('art', require('express-art-template'));
// 向模板内部导入dateFormate变量
template.defaults.imports.dateFormat = dateFormat;
```

### 2.在views文件夹要访问静态目录的文件，用$\color{red}绝对路径$

如`login.art`

- `<link rel="stylesheet" href="/admin/lib/bootstrap/css/bootstrap.min.css">`
- `<link rel="stylesheet" href="/admin/css/base.css">`

### 3.$\color{red}父模板extend继承$，html格式的骨架（head标签、script标签等）

- views/admin下的common文件夹中有
  - layout.art

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <title>Blog - Content Manager</title>
    <link rel="stylesheet" href="/admin/lib/bootstrap/css/bootstrap.min.css">
    <link rel="stylesheet" href="/admin/css/base.css">
    {{block 'link'}}{{/block}}
</head>
<body>
    {{block 'main'}} {{/block}}
    <script src="/admin/lib/jquery/dist/jquery.min.js"></script>
    <script src="/admin/lib/bootstrap/js/bootstrap.min.js"></script>
    <script src="/admin/js/common.js"></script>
    {{block 'script'}} {{/block}}
</body>

</html>
```

- views/home下的common文件夹中有
  - layout.art

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

### 4.把头部和侧边栏两个$\color{red}在多个文件中重复的部分$抽取出来放在common子文件夹下（$\color{red}模板include包含$）

- views/admin下的common文件夹中有
  - header.art头部、aside.art侧边栏
- views/home下的common文件夹中有
  - header.art头部

### 5.在views/admin、views/home中的其他文件，如果有用到重复文件只要$\color{red}相对路径$

```html
{{extend './common/layout.art'}}

{{block 'main'}}
    {{include './common/header.art'}}
    ···
        {{include './common/aside.art'}}
        ···
{{/block}}

{{block 'script'}}
这里写script代码
{{/block}}
```

## 三、model数据库

### 1.连接数据库【model/connect.js】

```js
// 引入mongoose第三方模块
const mongoose = require('mongoose');
// 导入config模块
const config = require('config');
console.log(config.get('db.host'))
// 连接数据库
mongoose.connect(`mongodb://${config.get('db.user')}:${config.get('db.pwd')}@${config.get('db.host')}:${config.get('db.port')}/${config.get('db.name')}`, {useNewUrlParser: true })
  .then(() => console.log('数据库连接成功'))
  .catch(() => console.log('数据库连接失败'))
```

- 在app.js中引入

> `require('./model/connect');`

### 2.创建用户Scheme集合【model/user.js】

- 创建用户集合规则
  - `userSchema = new mongoose.Schema({...})`
- 创建用户集合
  - `const User = mongoose.model('User', userSchema);`
- 创建一个用户(先创建一个admin超级管理员的初始化用户)

```js
// 先创建一个admin超级管理员用户
User.create({
  username:'Lewisqi',
  email:'Lewosqi@qq.com',
  password:'123456',
  role:'admin',
  state:'0'
}).then(()=>{
  console.log('用户创建成功')；
}).catch(()=>{
  console.log('用户创建失败');
})
```

- 在app.js中引入，最后需要去掉

> `require('./model/user');`

model/user.js  综合

```js
// 创建用户集合
const mongoose = require('mongoose');
// 导入bcrypt
const bcrypt = require('bcrypt');
// 引入joi模块
const Joi = require('joi');
// 创建用户集合规则
const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: true,  //验证username这个字段
    minlength: 2,
    maxlength: 20
  },
  email: {
    type: String,
    unique: true,  // 保证邮箱地址在插入数据库时不重复
    required: true
  },
  password: {
    type: String,
    required: true
  },
  // admin 超级管理员
  // normal 普通用户
  role: {
    type: String,    //admin、normal
    required: true
  },
  // 0 启用状态
  // 1 禁用状态
  state: {
    type: Number,
    default: 0      //默认0，启用状态
  }
});

// 创建集合
const User = mongoose.model('User', userSchema);

//异步创建
async function createUser () {
  const salt = await bcrypt.genSalt(10);
  const pass = await bcrypt.hash('123456', salt);
  const user = await User.create({
    username: 'iteheima',
    email: 'itheima@itcast.cn',
    password: pass,
    role: 'admin',
    state: 0
  });
}

// 创建一个用户
// User.create({
//   username:'Lewisqi',
//   email:'Lewosqi@qq.com',
//   password:'123456',
//   role:'admin',
//   state:'0'
// }).then(()=>{
//   console.log('用户创建成功')；
// }).catch(()=>{
//   console.log('用户创建失败');
// })

// 验证用户信息
const validateUser = user => {
  // 定义对象的验证规则
  const schema = {
    username: Joi.string().min(2).max(12).required().error(new Error('用户名不符合验证规则')),
    email: Joi.string().email().required().error(new Error('邮箱格式不符合要求')),
    password: Joi.string().regex(/^[a-zA-Z0-9]{3,30}$/).required().error(new Error('密码格式不符合要求')),
    role: Joi.string().valid('normal', 'admin').required().error(new Error('角色值非法')),
    state: Joi.number().valid(0, 1).required().error(new Error('状态值非法'))
  };
  // 实施验证
  return Joi.validate(user, schema);
}

// 将用户集合做为模块成员进行导出（对象形式）,
module.exports = {
  User,             //开放用户属性
  validateUser      //开放
}
```

### 3.为登录表单项设置请求地址、请求方式以及表单项name属性【view/admin/login.art】

- 登录页面模板
  - 设置 `action="/admin/login"` 提交跳转页面

view/admin/login.art

```html
<form action="/admin/login" method="post" id="loginForm">
    <div class="form-group">
        <label>邮件</label>
        <input name="email" type="email" class="form-control" placeholder="请输入邮件地址">
    </div>
    <div class="form-group">
        <label>密码</label>
        <input name="password" type="password" class="form-control" placeholder="请输入密码">
    </div>
    <button type="submit" class="btn btn-primary">登录</button>
</form>
```

- 编写表单提交事件
- 重写用jquery的 `serializeToJson` 方法获取表单的输入内容
  - 获取数组，`var result = serializeToJson($(this))`
  - 即`[{name:"email"，value:"这里是用户输入的内容"}]`
  - 即`[{name:"password"，value:"这里是用户输入的内容"}]`
- `return false;` 阻止表单默认提交

- `public/admin` 目录下的 `js` 目录下
  - 把 `serializeToJson` 作为公共方法，单独提取出来，放在public目录下，即 `public/admin/js/common.js`

public/admin/js/common.js 【获取数组信息】

```js
function serializeToJson(form) {
    var result = {};
    // [{name: 'email', value: '用户输入的内容'}]
    var f =  form.serializeArray();  
    f.forEach(function (item) {
        // result.email
        result[item.name] = item.value;
    });
    return result;
}
```

- `public/admin` 目录下的 `common` 目录下
  - 有layout.art 是父模板，可统一继承，可以引入

public/admin/common/layout.art

```html
<body>
  {{block 'main'}} {{/block}}
    ···
    <script src="/admin/js/common.js"></script>
  {{block 'script'}} {{/block}}
</body>
```

- 当用户点击登录按钮时，客户端验证用户是否填写了登录表单
  - 如果其中一项没有输入，阻止表单提交

view/admin/login.art

```js
<script type="text/javascript">
  $('#loginForm').on('submit', function () {

       // 获取到表单中用户输入的内容
      var result = serializeToJson($(this))

      // 如果用户没有输入邮件地址的话
      if (result.email.trim().length == 0) {
          alert('请输入邮件地址');
          // 阻止程序向下执行
          return false;
      }
      // 如果用户没有输入密码
      if (result.password.trim().length == 0) {
          alert('请输入密码')
          // 阻止程序向下执行
          return false;
      }
  });
</script>
```

### 4.登录服务端路由编写

- $\color{red}a。$服务器端接收请求参数，验证用户是否填写了登录表单
  - 如果其中一项没有输入，为客户端做出响应，阻止程序向下执行


- app.js 中
  - 引入post请求参数
  - 处理post请求参数，`extended: false` 用系统模块
  - `app.js` 链接至 `route` 目录下的 `admin.js`

```js
// 引入body-parser模块 用来处理post请求参数
const bodyParser = require('body-parser');
// 处理post请求参数
app.use(bodyParser.urlencoded({extended: false}));
// 引入路由模块
const admin = require('./route/admin');
// 为路由匹配请求路径
app.use('/admin', admin);
```

- `route` 目录下的 `admin.js` 【管理admin地址的主路由】
  - 用来构建管理页面的路由
  - `/login` 链接至 `./admin/login`

```js
// 引用express框架
const express = require('express');
// 创建博客展示页面路由
const admin = express.Router();
// 实现登录功能
admin.post('/login', require('./admin/login'));
```

- `route/admin` 目录下的 `login.js`
  - 该js是为了用户在客户端禁用js代码时，可以用服务端代码返回响应结果

  - 导入用户集合构造函数,即上一个链接中的js暴露
  - 再直接用 `module.exports` 暴露登录函数的异步代码即可

- 针对 `return res.status(400).render('admin/error', {msg: '邮件地址或者密码错误'});`代码
- 可以通过 `render('admin/error',..)` 来链接至 `views/admin/error.art` 下
  - `views/admin/error.art` 中定位 `location.href = '/admin/login';`
  - 添加响应结果的样式，并接收信息 {{msg}}
  - 定时器在3秒钟之后，重新跳转到登录页面

views/admin/error.art

```html
{{extend './common/layout.art'}}

{{block 'main'}}
  <p class="bg-danger error">{{msg}}</p>
{{/block}}

{{block 'script'}}
  <script type="text/javascript">
    setTimeout(function () {
      location.href = '/admin/login';
    }, 3000)
  </script>
{{/block}}
```

- $\color{red}b。$根据邮箱地址和密码查询用户信息
- `route/admin/login.js` 需要获取用户集合的构造函数
  - `const { User } = require('../../model/user');`
- 再根据邮箱地址查询用户信息
  - 如果查询用户 user 变量，值是对象类型，对象中存储的是用户信息
  - 如果查询用户 user 变量，值为空，则没有查询到

- $\color{red}c。$熟悉密码加密处理 $\color{red}bcrypt$ 模块
  - 在根目录新建一个`hash.js`
  - 执行`node hash.js`

- `genSalt();` 方法生成随机字符串
  - 参数:数值越大 生成的随机字符串复杂度越高
  - 默认值是 10
- `hash('123456', salt);` 方法加密
  - 第一个参数：要进行加密的明文
  - 第二个参数：生成的随机字符串

hash.js

```js
const bcrypt = require('bcrypt');
async function run () {
  const salt = await bcrypt.genSalt(10);
  const result = await bcrypt.hash('123456', salt);
  console.log('生成的随机字符串：' + salt);
  console.log('加密后的密码：'+ result);
}
run();
```

- $\color{red}d。$把加密函数应用添加到 `model/user.js` 用户集合中
  - 异步创建用户函数createUser()

model/user.js

```js
// 导入bcrypt
const bcrypt = require('bcrypt');
//加密函数创建用户
async function createUser () {
  const salt = await bcrypt.genSalt(10);
  const pass = await bcrypt.hash('123456', salt);
  const user = await User.create({
    username: 'iteheima',
    email: 'itheima@itcast.cn',
    password: pass,
    role: 'admin',
    state: 0
    });
}
```

- $\color{red}e。$加密函数在 `route/admin/login.js` 中比对密码
- 将客户端传递过来的密码和用户信息中的密码进行比对
  - 比对成功 `isValid=true` ，用户登录成功,可以验证 `res.send('登录成功');`
  - 比对失败 `isValid=false` ，用户登录失败
- compare(xxx, xxx); 方法比对
  - 第一个参数：明文密码
  - 第二个参数：加密密码

> `let isValid = await bcrypt.compare(password, user.password);`

- $\color{red}f。$cookie和session

- 在 `app.js` 中导入express-session模块,并配置session
- 用中间件 `app.use（）` 处理先拦截请求，再交给session处理
  - session() 生成session对象
  - **secret参数**：存储秘钥，对数据加密

app.js

```js
// 导入express-session模块
const session = require('express-session');
// 配置session
app.use(session({
  secret: 'secret key',
  saveUninitialized: false,
  cookie: {
    maxAge: 24 * 60 * 60 * 1000
  }
}));
```

- 在 `route/admin/login.js` 中使用session
- 第一步：将用户信息存储到请求对象中
- 第二步：将变量设置到app.locals对象下面
  - `req.app.locals.userInfo = user;`
  - userInfo 自定义为用户信息,**这个数据在所有的模板中都可以获取到**。
  - 最后在公共模板 `views/admin/common/header.art` 中，
  - 在页面头部的右上角显示拿到的数据: `{{userInfo && userInfo.username}}`，如果userInfo存在，再显示userInfo.username

views/admin/common/header.art

```html
<!-- 用户信息 -->
<div class="info">
  <div class="profile dropdown fr">
    <span class="btn dropdown-toggle" data-toggle="dropdown">
        {{userInfo && userInfo.username}}
        <span class="caret"></span>
    </span>
    <ul class="dropdown-menu">
        <li><a href="user-edit.html">个人资料</a></li>
        <li><a href="/admin/logout">退出登录</a></li>
    </ul>
  </div>
</div>
<!-- /用户信息 -->
```

- 第三步：对用户角色判断
  - `user.role == 'admin'`（超级管理员）跳转到用户列表页面
    - `res.redirect('/admin/user');`
  - `user.role == 'normal'`（普通用户）跳转到博客首页
    - `res.redirect('/home/');`

route/admin/login.js

```js
// 如果密码比对成功
if ( isValid ) {
    // 登录成功
    // 将用户名存储在请求对象中
    req.session.username = user.username;
    // 将用户角色存储在session对象中
    req.session.role = user.role;
    // 测试代码：res.send('登录成功');

    //userInfo 自定义为用户信息,直接拿到用户信息
    req.app.locals.userInfo = user;
    // 对用户的角色进行判断
    if (user.role == 'admin') {
        // 重定向到用户列表页面
        res.redirect('/admin/user');
    } else {
        // 重定向到博客首页
        res.redirect('/home/');
    }
}
```

route/admin/login.js 综合

```js
// 导入用户集合构造函数
const { User } = require('../../model/user');
const bcrypt = require('bcrypt');

module.exports = async (req, res) => {
  // 接收请求参数
  const {email, password} = req.body;

  // 如果用户没有输入邮件地址
  if (email.trim().length == 0 || password.trim().length == 0)
  return res.status(400).render('admin/error', {msg: '邮件地址或者密码错误'});

  // 根据邮箱地址查询用户信息
  let user = await User.findOne({email});

  // 查询到了用户
  if (user) {
    // 将客户端传递过来的密码和用户信息中的密码进行比对。true 比对成功，false 对比失败
    let isValid = await bcrypt.compare(password, user.password);

    // 如果密码比对成功
    if ( isValid ) {
        // 登录成功,将用户名存储在请求对象中
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
    } else {
        // 没有查询到用户
        res.status(400).render('admin/error', {msg: '邮箱地址或者密码错误'})
    }
  } else {
      // 没有查询到用户
      res.status(400).render('admin/error', {msg: '邮箱地址或者密码错误'})
  }
}
```

- $\color{red}g。$登录拦截功能【普通用户除了博客正文，其他后台页面不能看见】

- `app.js` 中拦截登录请求，【/admin是匹配以admin开头的请求】
  - `app.use('/admin', require('./middleware/loginGuard'));`
  - 必须写在 $\color{red}路由匹配请求$ 的上面，否则拦截无效
  - 这里拦截函数链接到 `middleware/loginGuard.js`

app.js

```js
// 拦截请求 判断用户登录状态
app.use('/admin', require('./middleware/loginGuard'));

// 为路由匹配请求路径
app.use('/home', home);
app.use('/admin', admin);
```

【注意：middleware文件夹专门存放app.use的中间件，登录拦截】

- `middleware/loginGuard.js` 通过判断当前页面是不是 `/login` 登录页面 以及 通过存储的sessionId判断当前用户是否是登录状态
  - 如果用户不是登录状态 将请求**重定向到登录页面**
  - 如果用户是登录的，
    - 如果是 `normal` 普通用户，**重定向到博客首页**
    - 如果是 `admin` 管理员用户，继续向下执行 `next()`

middleware/loginGuard.js

```js
// 判断用户的登录状态
const guard = (req, res, next) => {
  if (req.url != '/login' && !req.session.username) {
      res.redirect('/admin/login');
  } else {
      // 如果用户是登录状态 并且是一个普通用户
      if (req.session.role == 'normal') {
      // 让它跳转到博客首页 阻止程序向下执行
      return res.redirect('/home/')
      }
      // 继续向下执行
      next();
  }
}
module.exports = guard;
```
