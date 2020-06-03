# node-博客（用户）

## 用户退出功能

1.找到管理员页面右上角，模板是`views/admin/header.art`

- `<li><a href="/admin/logout">退出登录</a></li>`

```html
<div class="profile dropdown fr">
    <!-- 显示用户名字，及下拉菜单 -->
    <span class="btn dropdown-toggle" data-toggle="dropdown">
        {{userInfo && userInfo.username}}
        <span class="caret"></span>
    </span>
    <!-- 下拉菜单中，退出登录功能定位服务器路由 -->
    <ul class="dropdown-menu">
        <li><a href="user-edit.html">个人资料</a></li>
        <li><a href="/admin/logout">退出登录</a></li>
    </ul>
</div>
```

2.在 `route/admin.js` 中添加退出功能

```js
// 实现退出功能
admin.get('/logout', require('./admin/logout'));
```

3.在 `route/admin/logout.js` 中实现退出重定向功能

- connect.sid 是浏览器默认的cookie名字

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

4.在 `app.js` 中设置退出后的cookie的过期时间，过期后则原保存的登录信息自动失效

```js
// 配置session
app.use(session({
    secret: 'secret key',
    saveUninitialized: false,
    cookie: {
        maxAge: 24 * 60 * 60 * 1000
    },
    resave: true,
}));
```

## 新增用户功能

### 1.为用户列表页面的新增用户按钮添加链接

- 在 `admin/use` 页面点击 新增用户 按钮后跳转到  ``  新增用户页面
- 使 `views/admin/user.art` 中新增链接定向
  - 定位`<a href="/admin/user-edit" class="btn btn-primary new">新增用户</a>`

views/admin/user.art

```html
<!-- 分类标题 -->
<div class="title">
    <h4>用户</h4>
    <span>找到1个用户</span>
    <a href="/admin/user-edit" class="btn btn-primary new">新增用户</a>
</div>
```

### 2.添加路由

route/admin.js

```js
// 创建用户编辑页面路由
admin.get('/user-edit', require('./admin/user-edit'));
```

### 3.为新增用户表单指定请求地址、请求方式、为表单项添加name属性

- 表单post，指定请求地址{{link}}---->即在js中修改
  - `<form class="form-container" action="{{link}}" method="post">`
- 为表单项添加name属性
  - name="username"
  - name="email"
  - name="password"
  - name="role"
  - name="state"


views/admin/user-edit.art

```html
<div class="main">
    <!-- 分类标题 -->
    <div class="title">
        <h4 style="display: {{button == '修改' ? 'block' : 'none'}}">{{@user && user._id}}</h4>
        <p class="tips">{{message}}</p>
    </div>
    <!-- /分类标题 -->
    <form class="form-container" action="{{link}}" method="post">
        <div class="form-group">
            <label>用户名</label>
            <input name="username" type="text" class="form-control" placeholder="请输入用户名" value="{{user && user.username}}">
        </div>
        <div class="form-group">
            <label>邮箱</label>
            <input type="email" class="form-control" placeholder="请输入邮箱地址" name="email" value="{{user && user.email}}">
        </div>
        <div class="form-group">
            <label>密码</label>
            <input type="password" class="form-control" placeholder="请输入密码" name="password">
        </div>
        <div class="form-group">
            <label>角色</label>
            <select class="form-control" name="role">
                <option value="normal" {{user && user.role == 'normal' ? 'selected' : ''}}>普通用户</option>
                <option value="admin" {{user && user.role == 'admin' ? 'selected' : ''}}>超级管理员</option>
            </select>
        </div>
        <div class="form-group">
            <label>状态</label>
            <select class="form-control" name="state">
                <option value="0" {{user && user.state == '0' ? 'selected' : ''}}>启用</option>
                <option value="1" {{user && user.state == '1' ? 'selected' : ''}}>禁用</option>
            </select>
        </div>
        <div class="buttons">
            <input type="submit" class="btn btn-primary" value="{{button}}">
        </div>
    </form>
</div>
```

### 4.数据库的user.js先写验证用户的规则，随后暴露

- Joi模块验证表单中填写的信息【name属性】
- 暴露用户集合变量`User`，暴露用户验证规则变量`validateUser`

model/user.js 数据库的user.js

```js
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

// 将用户集合做为模块成员进行导出
module.exports = {
	User,
	validateUser
}
```

### 5.写好用户编辑的总控制函数

$\color{red}注意next方法，只接受一个参数，并且必须是字符串格式，所以用JSON.stringify(..)$

- 先接收validateUser的验证变量
- 再接收浏览器中传过来req.body表单信息
  - `validateUser(req.body)`
- 最后用验证规则来验证
  - 把验证结果先从对象格式转换成字符串格式
  - `JSON.stringify(..)`
  - 再用path属性，用特定的验证函数js去处理route/admin/user-edit.js来**处理出错信息**（去掉route）【模块化细分】
  - 重定向地址：`path: '/admin/user-edit'`
- message是catch中的错误信息，由Joi自定义 e.message
  - const { message, id } = req.query;

route/admin/user-edit-fn.js

```js
// 引入用户集合的构造函数
const { User, validateUser } = require('../../model/user');

try {
    await validateUser(req.body)
}catch (e) {
    return next(JSON.stringify({path: '/admin/user-edit', message: e.message}))
}
```

### 6.验证不通过时，用特定的验证函数js来处理新增用户数据错误信息渲染到页面

渲染目标  
views/admin/user-edit.art

```html
<p class="tips">{{message}}</p>
```

### 7.验证通过时，再判断邮箱是否存在

- 返回到用户总控制函数user-edit-fn.js中继续编写
- 根据邮箱地址查询用户是否存在
  - 如果用户已经存在
    - 重定向到 `/admin/user-edit` 地址，处理同上
  - 如果用户不存在，则可以对密码进行加密，用 `bcrypt` 模块

- 加密操作
  - 先生成随机字符串，再把明文和随机字符串混合加密
    - `const salt = await bcrypt.genSalt(10);`
    - `const password = await bcrypt.hash(req.body.password, salt);`
  - 再用密文密码替换原用户输入的明文密码
    - `req.body.password = password;`
  - 最后添加到数据库中
    - `await User.create(req.body);`

- 最后结束后，重定向回用户列表页面
  - `res.redirect('/admin/user');`

route/admin/user-edit-fn.js

```js
// 根据邮箱地址查询用户是否存在
    let user = await User.findOne({email: req.body.email});
    // 如果用户已经存在,则邮箱地址已经被别人占用
    if (user) {
        // 重定向回用户添加页面
        // return res.redirect(`/admin/user-edit?message=邮箱地址已经被占用`);
        return next(JSON.stringify({path: '/admin/user-edit', message: '邮箱地址已经被占用'}))
    }
    // 对密码进行加密处理
    // 生成随机字符串
    const salt = await bcrypt.genSalt(10);
    // 加密
    const password = await bcrypt.hash(req.body.password, salt);
    // 替换密码
    req.body.password = password;
    // 将用户信息添加到数据库中
    await User.create(req.body);
    // 将页面重定向到用户列表页面
    res.redirect('/admin/user');
}
```

### 8.在app.js中做个拦截器【优化】

$\color{red}前面的next方法已经将对象转换为字符串，现在再做拦截器是，要将错误信息从字符串转换为对象类型，用JSON.parse()$

app.js

```js
app.use((err, req, res, next) => {
    const result = JSON.parse(err);
    // {path: '/admin/user-edit', message: '密码比对失败,不能进行用户信息的修改', id: id}
    let params = [];
    for (let attr in result) {
        if (attr != 'path') {
            params.push(attr + '=' + result[attr]);
        }
    }
    res.redirect(`${result.path}?${params.join('&')}`);
})
```

## 用户数据分页功能【页码】

- 先导入用户集合变量
  - `const { User } = require('../../model/user');`
- 再将**所有**用户信息{}从数据库中查询出来
  - `let users = await User.find({}).limit(pagesize).skip(start)`
- 1.当前页，用户通过点击上一页或者下一页或者页码产生
  - 客户端通过 `get` 参数方式传递到服务器端
- 2.总页数，根据总页数判断当前页是否为最后一页，
  - 总页数：Math.ceil（总数据条数 / 每页显示数据条数）
  - limit(pagesize) ---> limit 限制查询数量。传入每页显示的数据数量
  - skip(start) ---> skip 跳过多少条数据。传入显示数据的开始位置
- 3.数据开始查询位置=（当前页-1）* 每页显示的数据条数
- 最后将分页数据渲染到模板
  - res.render('admin/user', {})

```js
// 导入用户集合构造函数
const { User } = require('../../model/user');

module.exports = async (req, res) => {
    // 标识 标识当前访问的是用户管理页面
    req.app.locals.currentLink = 'user';

    // 接收客户端传递过来的当前页参数
    let page = req.query.page || 1;
    // 每一页显示的数据条数
    let pagesize = 10;
    // 查询数据库中用户数据的总数
    let count = await User.countDocuments({});
    // 总页数【结果向上取整】
    let total = Math.ceil(count / pagesize);

    // 页码对应的数据查询开始位置
    let start = (page - 1) * pagesize;

    // 将用户信息从数据库中查询出来
    let users = await User.find({}).limit(pagesize).skip(start)
    // 渲染用户列表模块
    res.render('admin/user', {
        count: count,
        users: users,
        page: page,
        total: total
    });
}
```

在 `views/admin/user.art` 模板中显示页面

- <% xxx %> 原生模板语法
- 先判断有无数据，根据结果显示样式，如果上一页无数据或下一页无数据，则把`《`或`》`隐藏
  - `<li style="display: <%=page-1 < 1 ? 'none' : 'inline' %>">`
- 通过a标签跳转

views/admin/user.art

```html
<!-- 分页 -->
<ul class="pagination">
    <!-- 这里是后退一页功能 -->
    <li style="display: <%=page-1 < 1 ? 'none' : 'inline' %>">
        <a href="/admin/user?page=<%=page-1%>"><span>&laquo;</span></a>
    </li>
    <!-- 这里是直接跳转链接 -->
    <% for (var i = 1; i <= total; i++) { %>
    <li><a href="/admin/user?page=<%=i %>">{{i}}</a></li>
    <% } %>
    <!-- 这里是前进一页功能 -->
    <li style="display: <%= page-0+1 > total ? 'none' : 'inline' %>">
        <a href="/admin/user?page=<%=page-0+1%>"><span>&raquo;</span></a>
    </li>
</ul>
<!-- /分页 -->
```

## 用户信息修改和删除功能

第一步：将要修改的用户ID传递到服务器端

- 在 `views/admin/user.art` 中操作一栏下，点击链接后
- **a 是修改链接**
  - 从`/admin/user-edit?id={{@$value._id}}` 页面获取用户的id【这是浏览器的get方式传数据到服务器端，？链接参数】,并跳转到/admin/user-edit路由
- **i 是删除弹框**
  - data-id="{{@$value._id}}

views/admin/user.art

```html
<td>
    <a href="/admin/user-edit?id={{@$value._id}}" class="glyphicon glyphicon-edit"></a>
    <i class="glyphicon glyphicon-remove delete" data-toggle="modal" data-target=".confirm-modal" data-id="{{@$value._id}}"></i>
</td>
```

第二步：建立用户信息修改功能对应的路由  
第三步：接收客户端表单传递过来的请求参数  

- 服务器得到传递过来message, id参数
  - `const { message, id } = req.query;`
- 如果数据库中存在id
  - 从数据库中提取数据，再渲染，按钮值为修改
  - `let user = await User.findOne({_id: id});`【查找】
- 如果数据库中不存在id
  - 直接渲染，按钮值为添加
- 渲染用户编辑页面(修改)
  - `res.render('admin/user-edit', {})`
- 把？后的id参数【get请求】传给服务器端，由另一个js函数继续执行修改功能【下面的更新功能】
  - `link: '/admin/user-modify?id=' + id,`

route/admin/user-edit.js

```js
const { User } = require('../../model/user');
module.exports = async (req, res) => {
    // 标识 标识当前访问的是用户管理页面
    req.app.locals.currentLink = 'user';
    // 获取到地址栏中的message，id参数
    const { message, id } = req.query;
    // 如果当前传递了id参数
    if (id) {
        // 修改操作
        let user = await User.findOne({_id: id});
        // 渲染用户编辑页面(修改)
        res.render('admin/user-edit', {
            message: message,
            user: user,
            link: '/admin/user-modify?id=' + id,
            button: '修改'
        });
    }else {
        // 添加操作
        res.render('admin/user-edit', {
            message: message,
            link: '/admin/user-edit',
            button: '添加'
        });
    }
}
```

第四步：对应的渲染页面的模板，进行显示数据【value属性】

- 模板获取从服务器端传过来的数据，先确保user用户存在
  - `value="{{user && user.username}}`
  - `value="{{user && user.email}}`
  - 密码显示另作处理
- 对于role身份，和state在线状态用selected

```html
<option value="normal" {{user && user.role == 'normal' ? 'selected' : ''}}>普通用户</option>
<option value="admin" {{user && user.role == 'admin' ? 'selected' : ''}}>超级管理员</option>
···
<option value="0" {{user && user.state == '0' ? 'selected' : ''}}>启用</option>
<option value="1" {{user && user.state == '1' ? 'selected' : ''}}>禁用</option>
```

- form表单的 `action="{{link}}"`
- 提交按钮显示值,显示修改或添加
  - `<input type="submit" class="btn btn-primary" value="{{button}}">`
- 顶部的标题显示，根据底部的提交按钮的改变而改变,并且显示用户的mongoDB的_id【注意@是原文输出，去掉引号】
  - `<h4 style="display: {{button == '修改' ? 'block' : 'none'}}">{{@user && user._id}}</h4>`

views/admin/user-edit.art

```html
<!-- 分类标题 -->
<div class="title">
    <h4 style="display: {{button == '修改' ? 'block' : 'none'}}">{{@user && user._id}}</h4>
    <p class="tips">{{message}}</p>
</div>
<!-- /分类标题 -->
<form class="form-container" action="{{link}}" method="post">
    <div class="form-group">
        <label>用户名</label>
        <input name="username" type="text" class="form-control" placeholder="请输入用户名" value="{{user && user.username}}">
    </div>
    <div class="form-group">
        <label>邮箱</label>
        <input type="email" class="form-control" placeholder="请输入邮箱地址" name="email" value="{{user && user.email}}">
    </div>
    <div class="form-group">
        <label>密码</label>
        <input type="password" class="form-control" placeholder="请输入密码" name="password">
    </div>
    <div class="form-group">
        <label>角色</label>
        <select class="form-control" name="role">
            <option value="normal" {{user && user.role == 'normal' ? 'selected' : ''}}>普通用户</option>
            <option value="admin" {{user && user.role == 'admin' ? 'selected' : ''}}>超级管理员</option>
        </select>
    </div>
    <div class="form-group">
        <label>状态</label>
        <select class="form-control" name="state">
            <option value="0" {{user && user.state == '0' ? 'selected' : ''}}>启用</option>
            <option value="1" {{user && user.state == '1' ? 'selected' : ''}}>禁用</option>
        </select>
    </div>
    <div class="buttons">
        <input type="submit" class="btn btn-primary" value="{{button}}">
    </div>
</form>
```

第五步：把修改好的信息更新到数据库

先在route/admin.js中创建修改路由

route/admin.js

```js
// 用户修改功能路由
admin.post('/user-modify', require('./admin/user-modify'));
```

- req.body拿到是post请求参数
  - `const { username, email, role, state, password } = req.body;`
- req.query拿到的是get请求的参数
  - id 就是get请求通过？发送的
  - `const id = req.query.id;`
- 根据id在数据库中查询用户信息【需要导入user.js才能查询】
  - `let user = await User.findOne({_id: id});`
- 将客户端用户修改好的并传递过来的密码和数据库中的密码进行比对【用到bcrypt模块】
  - `bcrypt.compare();` 密码比对函数
  - `const isValid = await bcrypt.compare(password, user.password);`
- 比对失败，对客户端做出响应
  - 触发之前`app.js`定义的**错误处理中间件**
  - 调用next方法，传递字符串类型的数据
  - `next(JSON.stringify(obj));`【json对象转字符串】
- 对比成功，将用户信息更新到数据库中
  - 数据库更新操作即可
  - `await User.updateOne({_id: id}, {...});`
  - 在重定向到用户列表页面
  - `res.redirect('/admin/user');`

route/admin/user-modify.js

```js
const { User } = require('../../model/user');
const bcrypt = require('bcrypt');

module.exports = async (req, res, next) => {
  const { username, email, role, state, password } = req.body;
  const id = req.query.id;
  let user = await User.findOne({_id: id});
  const isValid = await bcrypt.compare(password, user.password);
  // 密码比对成功
  if (isValid) {
    // res.send('密码比对成功');
    // 将用户信息更新到数据库中
    await User.updateOne({_id: id}, {
      username: username,
      email: email,
      role: role,
      state: state
    });
    // 将页面重定向到用户列表页面
    res.redirect('/admin/user');
  }else {
    // 密码比对失败
    let obj = {
      path: '/admin/user-edit',
      message: '密码比对失败,不能进行用户信息的修改',
      id: id
      }
    next(JSON.stringify(obj));
  }
  // 密码比对
  // res.send(user);
}
```

app.js

```js
app.use((err, req, res, next) => {
  // 将字符串对象转换为对象类型
  // JSON.parse()
  const result = JSON.parse(err);
  //这里是/user-modify.js中的
  // {path: '/admin/user-edit', message: '密码比对失败,不能进行用户信息的修改', id: id}
  let params = [];
  for (let attr in result) {
    if (attr != 'path') {
      params.push(attr + '=' + result[attr]);
    }
  }
  res.redirect(`${result.path}?${params.join('&')}`);
})
```

## 用户信息删除

1.为删除表单添加提交地址以及提交方式

- `action="/admin/delete" method="get"`

2.在确认删除框中添加隐藏域，`id="deleteUserId"`

- `<input type="hidden" name="id" id="deleteUserId">`

3.删除按钮

- **取类名** delete
  - `class="glyphicon glyphicon-remove delete"`
- **自定义属性**，存储要删除用户的ID值
  - `data-id="{{@$value._id}}"`
- **通过类名点击事件**，获取自定义属性中存储的ID值并将ID值存储在表单的隐藏域中
  - {{block 'script'}}中的内容
  - `$(this).attr('data-id')`;    获取用户id值
  - `$('#deleteUserId').val(id);`  ID值存储在表单的隐藏域中

views/admin/user.art

```html
<!-- i是删除按钮 -->
<td>
    <a href="/admin/user-edit?id={{@$value._id}}" class="glyphicon glyphicon-edit"></a>
    <i class="glyphicon glyphicon-remove delete" data-toggle="modal" data-target=".confirm-modal" data-id="{{@$value._id}}"></i>
</td>
···
<!-- 删除确认弹出框 -->
<div class="modal fade confirm-modal">
    <div class="modal-dialog modal-lg">
        <form class="modal-content" action="/admin/delete" method="get">
            <!-- 弹出框头部 -->
            <div class="modal-header">
                <button type="button" class="close" data-dismiss="modal"><span>&times;</span></button>
                <h4 class="modal-title">请确认</h4>
            </div>
            <!-- 弹出框主体 -->
            <div class="modal-body">
                <p>您确定要删除这个用户吗?</p>
                <input type="hidden" name="id" id="deleteUserId">
            </div>
            <!-- 弹出框底部 -->
            <div class="modal-footer">
                <button type="button" class="btn btn-default" data-dismiss="modal">取消</button>
                <input type="submit" class="btn btn-primary">
            </div>
        </form>
    </div>
</div>
···
{{block 'script'}}
    <script type="text/javascript">
        $('.delete').on('click', function () {
            // 获取用户id
            var id = $(this).attr('data-id');
            // 将要删除的用户id存储在隐藏域中
            $('#deleteUserId').val(id);
        })
    </script>
{{/block}}
```

4.在服务器端建立删除功能路由

route/admin.js

```js
// 用户删除功能路由
admin.get('/delete', require('./admin/user-delete'));
```

5.接收客户端传递过来的id参数

- 引入User
- 获取get请求传递过来的id参数
  - `_id: req.query.id`
- 根据id在数据库中删除用户信息
  - await User.findOneAndDelete({_id: req.query.id});
- 将页面重定向到用户列表页面
  - res.redirect('/admin/user');

route/admin/user-delete.js

```js
const { User } = require('../../model/user');
module.exports = async (req, res) => {
  await User.findOneAndDelete({_id: req.query.id});
  res.redirect('/admin/user');
}
```
