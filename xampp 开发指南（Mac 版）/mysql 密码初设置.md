一般安装完 xampp 后，mysql 密码为空，可以直接进入，然后可以设置一下密码。

### 相关问题1：设置完密码后无法进入 phpmyadmin 页面

具体报错为：mysqli_real_connect(): (HY000/1045): Access denied for user ‘root‘@‘localhost‘ (using password: YES

解决方案1：在 phpmyadmin 的目录中找到 config.inc.php 类似的配置文件
```bash
# $cfg['Servers'][$i]['auth_type'] = 'config'; # 注释掉该行
```
该行的作用是，使用配置中的账号密码自动进行登录，注释后，可进入登录页面，使用账号密码进行登录。

解决方案2：依旧在上述的配置文件中
```bash
$cfg['Servers'][$i]['controluser'] = 'root'; // 默认账户
$cfg['Servers'][$i]['controlpass'] = 'root-password'; // 默认账户对应的密码
```
配置完这两行后，会根据该账户和密码进行登录。
