**概要：**
- Web框架三大核心知识：路由、控制器与模型
- 验证器、读取器、缓存与全局异常处理
- ORM：模型与关联模型

## 控制器多级目录实现方式
v1 版本控制器目录：/app/controller/v1/Index.php
对应路由：Route::get('api/v1/banner/:id', 'v1.banner/getBannerById');
控制器对应命名空间：namespace app\controller\v1;

v2 版本控制器目录：/app/controller/v2/Index.php
对应路由：Route::get('api/v1/banner/:id', 'v2.banner/getBannerById');
控制器对应命名空间：namespace app\controller\v2;

## 验证器

**定义基类**

```php
class BaseValid extends Validate
{
    /**
     * @throws ParameterException
     */
    public function goCheck()
    {
        // 获取参数
        $params = request()->param();
        // 验证参数
        $result = $this->batch()->check($params);
        if (!$result) {
            $e = new ParameterException([ // 抛出参数异常错误
                'msg' => $this->error,
            ]);
            throw $e;
        }
        return true;
    }
}
```

**参数验证示例**

```php
class IdMustBePositiveInt extends BaseValid
{
    protected $rule = [
        'id' => 'require|isPositiveInt',
    ];

    // 自定义验证规则
    protected function isPositiveInt($value, $rule, array $data, string $field = '')
    {
        if (is_numeric($value) && is_int($value - 0) && ($value - 0 ) > 0) {
            return true;
        }
        return $field.'必须是正整数';
    }
}
```

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

### 定义全局异常处理类

```php
<?php

namespace app\exception;

use think\exception\Handle;
use think\facade\Request;
use think\Response;
use Throwable;

class ExceptionHandle extends Handle
{
    // 不需要记录日志的异常处理类
    protected $ignoreReport = [
        BaseException::class,
    ];

    private $code;
    private $msg;
    private $err_code;

    // 改写 Exception 的 render 方法
    public function render($request, Throwable $e): Response
    {
        if ($e instanceof BaseException) {
            // 基本异常，一般由用户错误行为导致，返回相应错误信息即可
            $this->code = $e->code;
            $this->msg = $e->msg;
            $this->err_code = $e->err_code;
        } else {
            // 系统内部异常，一般由代码等导致的错误，会记录日志
            // 开发环境需要显示具体的错误信息：错误原因、所在行等，可交由框架默认异常处理机制来处理
            if (env('APP_DEBUG')) {
                return parent::render($request, $e);
            }
            $this->code = 500;
            $this->msg = 'internal server error';
            $this->err_code = 999;
        }
        $url = Request::url();
        $result = [
            'msg' => $this->msg,
            'err_code' => $this->err_code,
            'url' => $url,
        ];
        return json($result, $this->code);
    }

    public function report(Throwable $exception): void
    {
        parent::report($exception);
    }
}
```

### 自定错误码、错误信息等

**定义基类**

```php
class BaseException extends Exception
{

    public $code = 400;                   // HTTP 状态码
    public $msg = 'invalid parameter(s)'; // 错误信息
    public $err_code = 10000;           // 自定义错误码


    // 改写属性
    public function __construct($params = [])
    {
      if (!is_array($params)) {
        return;
      }

      if (key_exist('code')) {
        $this->code = $params['code'];
      }

      if (key_exist('msg')) {
        $this->msg = $params['msg'];
      }

      if (key_exist('err_code')) {
        $this->err_code = $params['err_code'];
      }
    }
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
