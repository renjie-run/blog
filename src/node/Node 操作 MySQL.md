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

