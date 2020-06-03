# node-博客（文章后台）

## 对文章和用户的左侧边栏做标识，进行对应操作时呈选中状态

1、在服务器端建立文章功能路由

route/admin.js

```js
// 文章列表页面路由
admin.get('/article', require('./admin/article'));
// 文章编辑页面路由
admin.get('/article-edit', require('./admin/article-edit'));
```

2、admin的父模板中找到侧边栏管理

- 用户管理，点击后右侧跳转路由（上篇文章中）
  - `<a class="item  href="/admin/user">`
- 文章管理，点击后右侧跳转路由
  - `<a class="item  href="/admin/article">`
- 判断当前选中的左侧侧边栏链接【用样式区分】
  - `{{currentLink == 'user' ? 'active' : ''}}"`
  - `{{currentLink == 'article' ? 'active' : ''}}"`

views/admin/common/aside.art

```html
<!-- 侧边栏 -->
<div class="aside fl">
    <ul class="menu list-unstyled">
        <li>
            <a class="item {{currentLink == 'user' ? 'active' : ''}}" href="/admin/user">
                <span class="glyphicon glyphicon-user"></span>
                用户管理
            </a>
        </li>
        <li>
            <a class="item {{currentLink == 'article' ? 'active' : ''}}" href="/admin/article">
                <span class="glyphicon glyphicon-th-list"></span>
                文章管理
            </a>
        </li>
    </ul>
    <div class="cprt">
        Powered by <a href="#" target="_blank">Lewis-qi</a>
    </div>
</div>
<!-- 侧边栏 -->
```

3、在选中**用户**侧边栏中的js文件下添加标识，并把标识开放到模板中

route/admin/userPage.js

```js
// 标识 标识当前访问的是用户管理页面
req.app.locals.currentLink = 'user';
```

route/admin/user-edit.js

```js
// 标识 标识当前访问的是用户管理页面
req.app.locals.currentLink = 'user';
```

4、在选中**文章**侧边栏中的js文件下添加标识，并把标识开放到模板中

route/admin/article.js

```js
// 标识 标识当前访问的是文章管理页面
req.app.locals.currentLink = 'article';
```

route/admin/article-edit.js

```js
// 标识 标识当前访问的是文章管理页面
req.app.locals.currentLink = 'article';
```

## 创建文章的数据库规则

- 注意author作者字段存储
  - `type: mongoose.Schema.Types.ObjectId,`
- 与数据库存储User集合中的作者id进行关联
  - `ref: 'User',`
- `require` 是必选字段

route/admin/article.js

```js
// 1.引入mongoose模块
const mongoose = require('mongoose');

// 2.创建文章集合规则
const articleSchema = new mongoose.Schema({
    title: {
        type: String,
        maxlength: 20,
        minlength: 4,
        required: [true, '请填写文章标题']
    },
    author: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'User',
        required: [true, '请传递作者']
    },
    publishDate: {
        type: Date,
        default: Date.now
    },
    cover: {
        type: String,
        default: null
    },
    content: {
        type: String
    }
});

// 3.根据规则创建集合
const Article = mongoose.model('Article', articleSchema);

// 4.将集合做为模块成员进行导出
module.exports = {
    Article
}
```

## 文章添加功能

### 1、文章表单请求地址，请求方式

- action="/admin/article-add"
- method="post"

### 2、**对于文件上传，form属性必须用二进制传递到服务器端**

- 默认 `enctype=application/x-www-form-urlencoded`
  - 这是将数据变成如“name=zhangsan&age=20”的形式
- 二进制 `enctype="multipart/form-data"`
  - 这是将数据指定二进制

### 3、为每个表单项添加**name 属性**【方便服务器端接收客户端传递的参数】

- **name 属性值**与数据库的字段值保持一致
  - `name="title"` 标题
  - `name="author"` 作者
  - `name="publishDate"` 发布时间
  - `name="cover"` 文章封面
  - `name="content"` 内容

views/admin/article.art

```html
 <form class="form-container" action="/admin/article-add" method="post" enctype="multipart/form-data">
    <div class="form-group">
        <label>标题</label>
        <input type="text" class="form-control" placeholder="请输入文章标题" name="title">
    </div>
    <div class="form-group">
        <label>作者</label>
        <input name="author" type="text" class="form-control" readonly value="{{@userInfo._id}}">
    </div>
    <div class="form-group">
        <label>发布时间</label>
        <input name="publishDate" type="date" class="form-control">
    </div>
    <div class="form-group">
        <label for="exampleInputFile">文章封面</label>
        <!--multiple 允许用户一次性选择多个文件-->
        <input type="file" name="cover" id="file" >
        <div class="thumbnail-waper">
            <img class="img-thumbnail" src="" id="preview">
        </div>
    </div>
    <div class="form-group">
        <label>内容</label>
        <textarea name="content" class="form-control" id="editor"></textarea>
    </div>
    <div class="buttons">
        <input type="submit" class="btn btn-primary">
    </div>
</form>
```

### 4、再在 `app.js` 中添加一个文章添加功能的路由

app.js

```js
// 实现文章添加功能的路由
admin.post('/article-add', require('./admin/article-add'));
```

### 5、编写文章添加功能的函数

- 5.1、在 `app.js` 中已经导入了 `body-parser` 模块
  - 处理post请求的简单数据
- 5.2、需要在 `article-add.js` 继续导入 `formidable` 模块
  - 处理post传递过来的二进制数据
- 5.3、另外 需要导入 `article` 的数据库规则
  - `const { Article } = require('../../model/article');`

- 5.4、上传文件存储位置
  - dirname：当前路径为：根目录/route/admin/article-add.js
  - '../', '../', 'public', 'uploads' ：根目录/public/uploads/
  - 意思：上一级文件夹的上一级文件夹的public文件夹里的uploads文件夹中存放

- 5.5、在创建表单解析对象form后，有 `form.parse(req,res)` 方法,其中res变成异步回调函数，`async (err, fields, files) => {...}`
  - 第一个参数：err 错误对象类型
    - **如果表单解析失败，err对象里面存储错误信息**
    - **如果表单解析成功，err将会是null**
  - 第二个参数：fields 对象类型
    - **保存用户填写好的普通表单数据**
  - 第三个参数：files 对象类型
    - **保存和上传文件相关的数据**

- 5.6、$\color{red}文件读取FileReader$

实例：

```js
var reader = new FIleReader();
reader.readASDataURL('文件')；
reader.onload = function(){
    console.lpg(reader.result)
}
```

- 直接在art模板中添加js，
  - 用户选择的文件列表数组（可以选择多个，当前情况是一组图片中选择第一张图片）：
    - `console.log(this.files[0])`
  - **必须监听onload事件，代表文件读取完成后**，将文件读取的结果显示在页面中：【preview是img的id】
    - `preview.src = reader.result;`

views/admin/article.art

```js
<div class="form-group">
    <label for="exampleInputFile">文章封面</label>
    <!--在input中 multiple 允许用户一次性选择多个文件 -->
    <input type="file" name="cover" id="file" >
    <div class="thumbnail-waper">
        <img class="img-thumbnail" src="" id="preview">
    </div>
</div>


// 选择文件上传控件
var file = document.querySelector('#file');
var preview = document.querySelector('#preview');
// 当用户选择完文件以后
file.onchange = function () {
    // 1 创建文件读取对象
    var reader = new FileReader();
    // 2 读取文件，一次性可以读取多个文件
    reader.readAsDataURL(this.files[0]);
    // 3 监听onload事件
    reader.onload = function () {
        console.log(reader.result)
        // 将文件读取的结果显示在页面中
        preview.src = reader.result;
    }
}
```

- 5.7、在表单中的作者一栏，直接用保存过的数据显示
  - 早在route/admin/login.js中保存过【`req.app.locals.userInfo = user;`】
  - `value="{{@userInfo._id}}"`
  - $\color{red}@是原文输出$

```html
<div class="form-group">
    <label>作者</label>
    <input name="author" type="text" class="form-control" readonly value="{{@userInfo._id}}">
</div>
```

- 5.8、把获取到的文章的数据存到数据库中【await】
  - `await Article.create({...})`
- 5.9、解析表单
  - 服务器的路径，之前设置过静态资源目录，需截取public后的路径，并取图片数组中的第一张图片
  - `cover: files.cover.path.split('public')[1]`
- 5.10、存储数据好后，最后重定向到文章管理页面
  - `res.redirect('/admin/article');`

route/admin/article-add.js

```js
// 引入formidable第三方模块
const formidable = require('formidable');
const path = require('path');
const { Article } = require('../../model/article');

module.exports = (req, res) => {
    // 1.创建表单解析对象
    const form = new formidable.IncomingForm();
    // 2.配置上传文件的存放位置
    form.uploadDir = path.join(__dirname, '../', '../', 'public', 'uploads');
    // 3.保留上传文件的后缀
    form.keepExtensions = true;
    // 4.解析表单
    form.parse(req, async (err, fields, files) => {
        await Article.create({
            title: fields.title,
            author: fields.author,
            publishDate: fields.publishDate,
            cover: files.cover.path.split('public')[1],
            content: fields.content,
        });
        // 将页面重定向到文章列表页面
        res.redirect('/admin/article');
    })
    // res.send('ok');
}
```

## 文章添加后展示到文章管理页面

### 1、文章数据重数据库中导出

route/admin/article.js

```js
// 将文章集合的构造函数导入到当前文件中
const { Article } = require('../../model/article');
// 导入mongoose-sex-page模块
const pagination = require('mongoose-sex-page');

module.exports = async (req, res) => {
    // 接收客户端传递过来的页码
    const page = req.query.page;
    // 标识 标识当前访问的是文章管理页面
    req.app.locals.currentLink = 'article';
    //查询所有文章数据，同时增加分页功能，见下第3点
    let articles = await pagination(Article).find().page(page).size(2).display(3).populate('author').exec();

    // res.send(articles);

    // 渲染文章列表页面模板
    res.render('admin/article.art', {
        articles: articles
    });
}
```

### 2、把文章内容渲染在文章管理页面上

- 2.1、展示id值，@是原文输出，去掉引号
  - `{{@$value._id}}`

- 2.2、展示标题
  - `{{$value.title}}`

- 2.3、展示日期
  - 【注意在app.js中用到第三方dataformat模块】
  - 可以在全局中在模板内部导入 **dateFormat变量**，使任意的模板都可以使用该变量。
    - `template.defaults.imports.dateFormat = dateFormat;`
  - 最后在模板中可以直接使用**dateFormat函数及其publishDate属性**
  - `{{dateFormat($value.publishDate, 'yyyy-mm-dd')}}`

app.js

```js
// 导入express-session模块
const session = require('express-session');
// 导入art-template模板引擎
const template = require('art-template');
// 导入dateformat第三方模块
const dateFormat = require('dateformat');

···

// 告诉express框架模板所在的位置
app.set('views', path.join(__dirname, 'views'));
// 告诉express框架模板的默认后缀是什么
app.set('view engine', 'art');
// 当渲染后缀为art的模板时 所使用的模板引擎是什么
app.engine('art', require('express-art-template'));
// 向模板内部导入dateFormate变量
template.defaults.imports.dateFormat = dateFormat;
```

- 2.4、展示作者，根据id值查找作者名
  - 需要$\color{red}多表联合查询$【即多集合联合查询】
  - 在 `find()` 查询函数后，加上 `populate('author')` 函数
  - 没有分页情况下：
    - let articles = await pagination(Article).find().populate('author').exec();
  - 分页情况下：
    - let articles = await pagination(Article).find().page(page).size(2).display(3).populate('author').exec();

- 2.5、展示操作
  - a标签修改
  - i标签删除

views/admin/article.art

```html
<!-- 内容列表 -->
<table class="table table-striped table-bordered table-hover custom-table">
    <thead>
        <tr>
            <th>ID</th>
            <th>标题</th>
            <th>发布时间</th>
            <th>作者</th>
            <th>操作</th>
        </tr>
    </thead>
    <tbody>
        {{each articles.records}}
        <tr>
            <td>{{@$value._id}}</td>
            <td>{{$value.title}}</td>
            <td>{{dateFormat($value.publishDate, 'yyyy-mm-dd')}}</td>
            <td>{{$value.author.username}}</td>
            <td>
                <a href="article-edit.html" class="glyphicon glyphicon-edit"></a>
                <i class="glyphicon glyphicon-remove" data-toggle="modal" data-target=".confirm-modal"></i>
            </td>
        </tr>
        {{/each}}
    </tbody>
</table>
<!-- /内容列表 -->
```

### 3、分页

- 3.1、导入数据库分页模块
  - `const pagination = require('mongoose-sex-page');
route/admin/article.js`

- 3.2、pagination函数，可链式调用
  - `pagination(文章的集合构造函数)`
  - `page(page)`
    - 当前第page页
  - `size(2)`
    - 每页显示数据2条
  - `display(3)`
    - 客户端一次性显示的页面数量3页
  - `exec()`
    - 向数据库发出查询请求

```js
// 导入mongoose-sex-page模块
const pagination = require('mongoose-sex-page');
route/admin/article.js

···

// 查询所有文章数据
let articles = await pagination(Article).find().page(page).size(2).display(3).populate('author').exec();
```

- 3.3、mongoose-sex-page模块接口在exec()后查询出来的json格式
  - 只有records中是自己写的，其他都是模块自带

```json
{
    "page":1,//当前页
    "size":2,//每页显示条数
    "total":8,//总共的数据数据条数
    "records":[
        {
            "_id":"xxxxxxxxxxx",
            "title":"文章标题"
        }
    ],
    "page":4,//总共的页数
    "display":[1,2,3]//客户端一次页面中存放显示的页码
}
```

- 3.4、在管理文章页面渲染数据

- 渲染的数据循环必须根据mongoose-sex-page模块接口填写
  - articles.records是文章内容
  - `{{each articles.records}}xxx{{/each}}`
  - 文章页码的《
  - `{{if articles.page > 1}}xxx{{/if}}`
  - display是页码数组
  - `{{each articles.display}}xxx{{/each}}`
  - 文章页码的》
  - `{{if articles.page < articles.pages}}xxx{{/if}}`

- 在客户端`views/admin/article.art`
  - `<a href="/admin/article?page={{$value}}">`
  - 把当前页的值通过地址栏get请求，传递给服务器
- 在服务器端`route/admin/article.js`
  - `const page = req.query.page;`
  - `let articles = await pagination(Article).find().page(page)...`
  - 接收客户端传递过来的页码

views/admin/article.art

```html
{{each articles.records}}
    ···之前的文章数据
{{/each}}

<!-- 分页 -->
<ul class="pagination">
    {{if articles.page > 1}}
    <li>
        <a href="/admin/article?page={{articles.page - 1}}">
        <span>&laquo;</span>
        </a>
    </li>
    {{/if}}

    {{each articles.display}}
    <li><a href="/admin/article?page={{$value}}">{{$value}}</a></li>
    {{/each}}

    {{if articles.page < articles.pages}}
    <li>
        <a href="/admin/article?page={{articles.page - 0 + 1}}">
        <span>&raquo;</span>
        </a>
    </li>
    {{/if}}
</ul>
<!-- /分页 -->
```
