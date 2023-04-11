- Web框架三大核心知识：路由、控制器与模型
- 验证器、读取器、缓存与全局异常处理
- ORM：模型与关联模型

### 控制器多级目录实现方式
v1 版本控制器目录：/app/controller/v1/Index.php
对应路由：Route::get('api/v1/banner/:id', 'v1.banner/getBannerById');
控制器对应命名空间：namespace app\controller\v1;

v2 版本控制器目录：/app/controller/v2/Index.php
对应路由：Route::get('api/v1/banner/:id', 'v2.banner/getBannerById');
控制器对应命名空间：namespace app\controller\v2;

## 自定义全局异常处理

### 指定全局异常处理类
在 app/provider.php 中指定全局异常处理的文件。

```php
use app\exception\ExceptionHandle;
...

return [
    ...
    'think\exception\Handle' => ExceptionHandle::class, // 指定为自定义的全局异常处理类
];
```

### 自定错误码、错误信息等

**定义基类**

```php
class BaseException
{

    public $code = 400;                   // HTTP 状态码
    public $msg = 'invalid parameter(s)'; // 错误信息
    public $err_code = '10000';           // 自定义错误码

}
```

**具体应用**

继承基类，并覆盖错误码等信息。
```php
class BannerMissingException extends BaseException
{

    public $code = 404;
    public $msg = 'banner to find is not exist';
    public $err_code = 40000;
}
```
