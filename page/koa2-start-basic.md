## 快速创建一个服务器

```
npm install koa -S
```

```javascript
const Koa = require('koa');

let { Port } = require('./config');

let app = new Koa();

// response
app.use(ctx => {
  ctx.body = 'Hello Koa';
});

// 监听服务器启动端口
app.listen(Port, () => {
  console.log(`服务器启动在${ Port }端口`);
});
```

![image-20200314204035344](D:\Desktop\koa2-start-basic.assets\image-20200314204035344.png)

![image-20200314204104289](D:\Desktop\koa2-start-basic.assets\image-20200314204104289.png)

## MVC三层设计



## 路由中间件

```
npm install koa-router -S
```

### 模块子路由设计

```javascript
const Router = require('koa-router');
// 导入控制层
const usersController = require('../../controllers/usersController');

let usersRouter = new Router();

usersRouter
  .post('/users/login', usersController.Login)

module.exports = usersRouter;
```

### 模块子路由汇总

```javascript
const Router = require('koa-router');

let Routers = new Router();

const usersRouter = require('./router/usersRouter');

Routers.use(usersRouter.routes());

module.exports = Routers;
```

### 使用路由中间件

```javascript
// 使用路由中间件
const Routers = require('./routers');
app.use(Routers.routes()).use(Routers.allowedMethods());
```

 localhost:5000/users/login 

![image-20200314213958603](D:\Desktop\koa2-start-basic.assets\image-20200314213958603.png)



## 数据库连接

```
npm install mysql -S
```

在config.js添加如下代码

```javascript
// 数据库连接设置
dbConfig: {
  connectionLimit: 10,
  host: 'localhost',
  user: 'root',
  password: '',
  database: 'storeDB'
}
```

创建"./src/models/db.js"

```javascript
var mysql = require('mysql');
const { dbConfig } = require('../config.js');
var pool = mysql.createPool(dbConfig);

var db = {};

db.query = function (sql, params) {

  return new Promise((resolve, reject) => {
    // 取出链接
    pool.getConnection(function (err, connection) {

      if (err) {
        reject(err);
        return;
      }

      connection.query(sql, params, function (error, results, fields) {
        console.log(`${ sql }=>${ params }`);
        // 释放连接
        connection.release();
        if (error) {
          reject(error);
          return;
        }
        resolve(results);
      });

    });
  });
}
// 导出对象
module.exports = db;
```

## 控制层



## 数据持久层

数据模型层 执行数据操作



## 请求体数据处理

```
npm install koa-body -S
```

在config.js添加如下代码

```javascript
uploadDir: path.join(__dirname, path.resolve('../public/')), // 上传文件路径
```

app.js
```javascript
const KoaBody = require('koa-body');
let { uploadDir } = require('./config');
```

```javascript
// 处理请求体数据
app.use(KoaBody({
  multipart: true,
  // parsedMethods默认是['POST', 'PUT', 'PATCH']
  parsedMethods: ['POST', 'PUT', 'PATCH', 'GET', 'HEAD', 'DELETE'],
  formidable: {
    uploadDir: uploadDir, // 设置文件上传目录
    keepExtensions: true, // 保持文件的后缀
    maxFieldsSize: 2 * 1024 * 1024, // 文件上传大小限制
    onFileBegin: (name, file) => { // 文件上传前的设置
      // console.log(`name: ${name}`);
      // console.log(file);
    }
  }
}));
```

## 异常处理

在app.js添加如下代码

```javascript
// 异常处理中间件
app.use(async (ctx, next) => {
  try {
    await next();
  } catch (error) {
    console.log(error);
    ctx.body = {
      code: '500',
      msg: '服务器未知错误'
    }
  }
});
```

## 静态资源请求处理

```
npm install koa-router -S
```

在config.js添加如下代码

```javascript
staticDir: path.resolve('../public'), // 静态资源路径
```

app.js

```javascript
const KoaStatic = require('koa-static');
let { staticDir } = require('./config');
```

```javascript
// 为静态资源请求重写url
app.use(async (ctx, next) => {
  if (ctx.url.startsWith('/public')) {
    ctx.url = ctx.url.replace('/public', '');
  }
  await next();
});
// 使用koa-static处理静态资源
app.use(KoaStatic(staticDir));
```

![image-20200315014917150](D:\Desktop\koa2-start-basic.assets\image-20200315014917150.png)

## session

```
npm install koa-session -S
```

创建"./src/middleware/session.js"

```javascript
let store = {
  storage: {},
  set (key, session) {
    this.storage[key] = session;
  },
  get (key) {
    return this.storage[key];
  },
  destroy (key) {
    delete this.storage[key];
  }
}
let CONFIG = {
  key: 'koa:session',
  maxAge: 86400000,
  autoCommit: true,
  overwrite: true,
  httpOnly: true,
  signed: true,
  rolling: false,
  renew: false,
  sameSite: null,
  store
}

module.exports = CONFIG;
```

app.js

```javascript
const Session = require('koa-session');
// session
const CONFIG = require('./middleware/session');
app.keys = ['session app keys'];
app.use(Session(CONFIG, app));
```

## 登录拦截器

创建"./src/middleware/isLogin.js"

```javascript
module.exports = async (ctx, next) => {
  if (ctx.url.startsWith('/user/')) {
    if (!ctx.session.user) {
      ctx.body = {
        code: '401',
        msg: '用户没有登录，请登录后再操作'
      }
      return;
    }
  }
  await next();
};
```

app.js

```javascript
// 判断是否登录
const isLogin = require('./middleware/isLogin');
app.use(isLogin);
```





