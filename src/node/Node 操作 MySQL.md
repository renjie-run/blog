# Node 操作 MySQL 的一些方式

- mysql2
- typeorm


## mysql2

是目前连接 MySQL 使用最多的 npm 包之一。顾名思义，是 mysql 的升级版，提供了更多的特性。


### 安装

```bash
npm install --save mysql2
```


### 创建连接

创建连接的主要参数
- host，主机
- port，端口
- user，用户名
- password，密码
- database，要使用的数据库

```js
const mysql = require('mysql2');

const connection = mysql.createConnection({
  host: 'localhost',
  port: 3306,
  user: 'your root',
  password: 'your password',
  database: 'your database',
});
```

### 查询

查询使用的是 connection.query(sql, callback)。其中，callback 分别返回的有 error（查询错误信息）、result（查询结果）、fields（查询到的表的字段）。

针对下面这张表进行查询

<img width="328" alt="image" src="https://github.com/user-attachments/assets/f77d0806-e199-41a9-a1bf-e3a5094d3e3a">

```js
connection.query('select * from customers', (err, res, fields) => {
  if (err) {
    console.log(err);
    return;
  }
  console.log(res);
  console.log(fields);
  console.log(fields.map(item => item.name));
});
```

<img width="678" alt="image" src="https://github.com/user-attachments/assets/eda103b0-38e6-45cb-92a1-f92c34f92ff9">


#### 指定占位符查询

```js
connection.query('select * from customers where name like ?', ['李%'], (err, res, fields) => {
  if (err) {
    console.log(err);
    return;
  }
  console.log(res); // [ { id: 2, name: '李明' }, { id: 15, name: '李娜' } ]
});
```
查询效果同直接使用一条 sql 语句：select * from customers where name like '李%'。


### 增加数据

通过 connection.execute(sql, callback) 来增加数据。回调返回同 connection.query。我们来试一下

```js
connection.execute(`insert into customers (name) values ('Jerry Ren')`, (err, res, fields) => {
  if (err) {
    console.log(err);
    return;
  }
  console.log(res)
  console.log(fields)
});
```

可以看到打印的结果中，新增数据的 id 等信息

<img width="407" alt="image" src="https://github.com/user-attachments/assets/93a4e755-fa4a-4518-9216-75288d8a598b">

然后看下这个表中的数据，的确成功新增了对应 id 的数据

<img width="276" alt="image" src="https://github.com/user-attachments/assets/615610af-2c10-4785-85ac-7014a8a66704">

说明：connection.execute 也支持占位符，写法与 connection.query 一致。


### 修改数据

修改数据使用的也是 connection.execute 方法。

```js
connection.execute(`update customers set name = 'Jerry' where id = 21`, (err, res, fields) => {
  if (err) {
    console.log(err);
    return;
  }
  console.log(res)
  console.log(fields)
});
```

<img width="413" alt="image" src="https://github.com/user-attachments/assets/c9519050-4430-49da-8460-14868fe757f0">

<img width="245" alt="image" src="https://github.com/user-attachments/assets/20149714-2e4d-4649-94a2-e2bbfdfe5d2f">

可以看到修改成功了！


### 删除数据

删除数据这里依旧使用的是 connection.execute 方法。

```js
connection.execute('delete from customers where id = 21', (err, res, fields) => {
  if (err) {
    console.log(err);
    return;
  }
  console.log(res)
  console.log(fields)
});
```

<img width="243" alt="image" src="https://github.com/user-attachments/assets/c7730913-8b3e-493e-8feb-16fae0aa4058">


### promise 版

```js
const mysql = require('mysql2/promise');

(async () => {
  const connection = await mysql.createConnection({
    host: 'localhost',
    port: 3306,
    user: 'your root',
    password: 'your password',
    database: 'your database',
  });

  const [results, fields] = await connection.query('select * from customers');

  console.log(results);
  console.log(fields.map(item => item.name));
})();
```

<img width="278" alt="image" src="https://github.com/user-attachments/assets/c141e8ac-c6cc-42a0-ac3a-0d94fb4d96eb">

可以看到查询结果与之前的一致。


### 使用连接池

前面实现的查询性能都不太好，因为数据库的连接还是很耗时的，而且一个连接也不够用。此时，就考虑使用连接池进行管理了。当然 mysql2 也是支持的。

连接池通过 mysql2.createPool 来创建的，除了基本的数据库连接配置外，还需要结合一些额外的连接池配置来使用。

- connectionLimit，指定最大连接数。如果连接超出这个数量需要排队等待
- maxIdle，最大空闲数。超出这个数量的空闲会被释放掉
- waitForConnections，超出最大连接时是否等待。设置为 false 时表示为不等待，那么超出最大连接数的连接会返回错误
- idleTimeout，空闲的连接多久会断开
- queueLimit，可以排队的请求数量。设置为 0 表示无上限
- enableKeepAlive，与 keepAliveInitialDelay 都是保持心跳用的，使用默认的就好
- keepAliveInitialDelay

```js
const mysql = require('mysql2/promise');

(async () => {
  const pool = await mysql.createPool({
    host: 'localhost',
    port: 3306,
    user: 'your root',
    password: 'your password',
    database: 'your database',
    connectionLimit: 10,
    maxIdle: 10,
    idleTimeout: 60000,
    waitForConnections: true,
    queueLimit: 0,
    enableKeepAlive: true,
    keepAliveInitialDelay: 0,
  });

  const [results, fields] = await pool.query('select * from customers');

  console.log(results); // 查询结果与之前的一致
  console.log(fields.map(item => item.name));
```


#### 手动获取连接

```js
const mysql = require('mysql2/promise');

(async () => {
  const pool = await mysql.createPool({
    ......
  });

  const connection = await pool.getConnection();
  const [results, fields] = await connection.query('select * from customers');

  console.log(results);
```


## TypeORM

通过上面的学习知道了如果通过 Node 操作 MySQL，但一般我们不会直接执行 sql，而是使用 ORM（Object Relational Mapping，对象关系映射）框架。也就是把关系型数据库的表映射成面向对象的 class，表的字段映射成对象的属性映射，表与表的关联映射成属性的关联。而 TypeORM 就是一个流行的 ORM 框架。

接下来，我们来试下这个 typeorm

### 安装

这里直接通过 npx 安装 typeorm 的最新版

```bash
npx typeorm@latest init --name typeorm-mysql --database mysql
```

这里通过 `--name` 来指定创建的项目名，通过 `--database` 来指定用的数据库引擎。

<img width="1632" alt="WeCom20241028-140102@2x" src="https://github.com/user-attachments/assets/929c1399-62d7-46e9-99ac-300364d939c1">

### 初始化配置

#### 安装 mysql2

这里是为了将 mysql 更换为 mysql2。

```bash
npm install mysql2 -S
```


#### 修改数据库连接配置

修改 src/data-source.ts 文件。

```bash
{
  ......
  connectorPackage: 'mysql2',
  extra: {
    authPlugin: 'sha256_password',
  },
  ......
}
```

这里主要是修改了 connectorPackage 指定驱动包为 mysql2，其余的配置只需要按照自己 MySQL 的实际配置情况进行修改即可。


### 运行

通过 `npm start` 命名启动程序。此时，我们会发现数据库中多了个 user 表，并插入了一条数据。

<img width="1347" alt="image" src="https://github.com/user-attachments/assets/18fb61dc-2a01-4075-a7b8-147b5a6c97b8">

<img width="806" alt="image" src="https://github.com/user-attachments/assets/c3898acd-3cb2-4aa1-8d56-f5aafa67eb1b">

我们来看下程序中都做了哪些操作。

<img width="1075" alt="image" src="https://github.com/user-attachments/assets/d0045bf5-7f01-4d9c-a5c6-fbf720481f31">

通过这里的代码可以看到这里通过一个 User 的 entity 创建了 user 表，并通过 save 方法往 user 表中插入了一条数据，然后再通过 find 方法查出了这条数据。
注：这里没有 user 表时会自动创建 user 表是因为开启了 synchronize 配置。

#### 如何确定创建表的结构？

这里主要是得益于 entity 的字段与实际表中的结构一一对应。可以通过装饰器来确定主键列和其他列。

<img width="644" alt="image" src="https://github.com/user-attachments/assets/05fcd362-61c2-4346-be0e-d507ad661dc3">

#### tpyeorm 对应到 mysql 中执行了什么操作？

我们将数据库连接配置中的 logging 改为 true，然后通过日志看一看具体都执行哪些 MySQL 操作。

这里删掉 user 表，重新启动下项目。

<img width="1340" alt="image" src="https://github.com/user-attachments/assets/e04d6e6d-f156-40fe-97db-4ccee66e841b">

这里通过日志可以看到具体执行 sql 语句。

```bash
query: SELECT VERSION() AS `version` # 先查询数据版本
query: START TRANSACTION # 开启事务
query: SELECT DATABASE() AS `db_name` # 查询当前使用的数据库
query: SELECT `TABLE_SCHEMA`, `TABLE_NAME`, `TABLE_COMMENT` FROM `INFORMATION_SCHEMA`.`TABLES` WHERE `TABLE_SCHEMA` = 'practice' AND `TABLE_NAME` = 'user'
query: SELECT * FROM `INFORMATION_SCHEMA`.`COLUMNS` WHERE `TABLE_SCHEMA` = 'practice' AND `TABLE_NAME` = 'typeorm_metadata'
query: CREATE TABLE `user` (`id` int NOT NULL AUTO_INCREMENT, `firstName` varchar(255) NOT NULL, `lastName` varchar(255) NOT NULL, `age` int NOT NULL, PRIMARY KEY (`id`)) ENGINE=InnoDB # 创建 user 表
query: COMMIT # 提交事务
Inserting a new user into the database...
query: START TRANSACTION # 开启事务
query: INSERT INTO `user`(`id`, `firstName`, `lastName`, `age`) VALUES (DEFAULT, ?, ?, ?) -- PARAMETERS: ["Timber","Saw",25] # 插入一条数据
query: COMMIT # 提交事务
Saved a new user with id: 1
Loading users from the database...
query: SELECT `User`.`id` AS `User_id`, `User`.`firstName` AS `User_firstName`, `User`.`lastName` AS `User_lastName`, `User`.`age` AS `User_age` FROM `user` `User` # 查询数据
Loaded users:  [ User { id: 1, firstName: 'Timber', lastName: 'Saw', age: 25 } ]
Here you can setup and run express / fastify / any other framework.
```

通过日志信息可以看到 typeorm 执行 sql 时还使用了事务机制。

更多 typeorm 特性及使用可探索官网：[https://typeorm.io/](https://typeorm.io/)
