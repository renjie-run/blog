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
