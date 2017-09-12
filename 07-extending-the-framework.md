# Extending the Framework

## Introduction
* Laravel 提供了大量可以扩展的地方
* 可以完全替换掉所有组件
* 组件的扩展有两种方法：
    * 在 IoC 容器里绑定新的实现
    * 用 Mannger 类注册一个扩展，Mannger 类实现了工厂设计模式，负责组件的实例化

## Manager & Factories
* Laravel 有好多 Manager 类用来管理基于驱动的组件的生成过程
* 驱动组件包括：缓存、会话、身份认证、队列组件等
* 管理类负责根据应用程序的配置，来生成特定的驱动
* 每个 Manager 类都包含名为extend的方法，用于将新功能注入到管理类中

### Learn About Your Managers
* 阅读 Laravel 中的各个 Manager 类的代码 (例如 CacheManger 和 SessionManager)
* 对 Manager 类机制更加清楚透彻
* 所有的管理类都继承自 Illuminate\Support\Manager 基类

## Cache 缓存

### 扩展 MongoDB 缓存
* 使用 CacheManager 里的 extend 方法来绑定缓存驱动
* 驱动代码可以放在 `Packagist` 或者 `Extensions` 目录
* 放置在 `ServiceProvider` 中，来管理代码
* entend($name, $callback)
    * $name 对应配置文件中的 driver 项
    * $callback 返回一个 Illuminate\Cache\Repository 实例

实现接口
```
class MangoStore implements Illuminate\Cache\StoreInterface
{
    public function get($key) {}
    public function put($key, $value, $minutes) {}
    public function increment($key, $value = 1) {}
    public function decrement($key, $value = 1) {}
    public function forever($key, $value) {}
    public function forget($key) {}
    public function flush() {}
}
}
```

绑定类
```
// Return Illuminate\Cache\Repository instance
Cache::extend('mongo', function ($app) {
    return new Illuminate\Cache\Repository(new MongoStore);
});
```

## Session 会话
* 扩展 Laravel 的会话机制

实现接口
```
class MongoHandler implements SessionHandlerInterface {
    public function open($savePath, $sessionName) {}
    public function close() {}
    public function read($sessionId) {}
    public function write($sessionId, $data) {}
    public function destroy($sessionId) {}
    public function gc($lifetime) {}
}
```

绑定类
```
// Return implementation of SessionHandlerInterface
Session::extend('mongo', function($app)
{
    return new MongoHandler;
});
```

## Authentication
* 需要扩展 `Authentication@extend` return `Illuminate\Auth\UserProviderInterface`
* 实现 `UserProviderInterface`
* 实现 `UserInterface`
* 在 `ServiceProvider` 中注册
* 在 `config/auth.php` 中配置驱动

## IoC Based Extension
### Pagination
* 继承 并 重写 `\Illuminate\Pagination\Environment`
* 继承 `Illuminate\Pagination\PaginationServiceProvider` 并绑定 新的 paginator
* 修改 `config/app.php` 的配置文件
* 这是扩展容器核心类的一般方法，基本上每一个核心类都可以用这种方式绑定进容器

## Request
* Request 是框架最基础的部分
* 在请求流中很早就被实例化了
* 继承并重写 `Illuminate\Http\Request`
* 在 `bootstrap/app.php` 中 `Application::requestClass('New\Request')`
