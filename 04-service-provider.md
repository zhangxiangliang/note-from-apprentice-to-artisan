# Service Provider

## As Bootstrapper
* `Laravel Service Provider` 就是用来进行 IoC 绑定的类
* 自带的 `Laravel Service Provider` 用来管理框架核心组件的容器绑定
* 可以在 `config/app.php` 中查看绑定的容器
* 使用 `ServiceProvider@register` 方法来绑定 IoC
* 当一个 HTTP Request 接收后，框架刚启动会调用 `ServiceProvider@register` 方法。
* !!! `ServiceProvider@register` 只能用来 IoC 绑定，其他的判断、交互要在 `ServiceProvider@boot` 中使用

### Deferred Providers
* 并不是所有的 Service Provider 都会被使用
* Laravel 会创建一个延迟加载 `服务清单`，列出所有 `Service Provider` 和 所绑定的名称
* 通过延迟加载，框架能延迟加载每个请求需要的服务，性能得到提高
* 当添加新的 `Service Provider` 后，Laravel 会在下一次请求时自动生成服务清单

## As Orgainzer
* 使用 `Service Provider` 来管理代码
* 使得 Laravel 结构优美合理
* 如果在一次请求周期中该类只需要有一个实例，就使用singleton，否则就使用bind
* 使用 `Service Provider` 来创建服务提供者，而不只是在发布软件包时才使用

## Booting Providers
* 在所有服务提供者都注册以后
* 会进行启动过程，触发服务提供者的 `boot` 方法
* `boot` 调用方法 (注册事件监听，引入路由文件，注册过滤器)
* `register` 注册 IoC
* 也可以把一系列方法写入一个 `ServiceProvider@boot`

## Providing The Core
* 理解 Laravel Core 的最好方法是去读 `Core Service Provider`
* 大部分服务是延迟加载
* 核心基础服务是每次都会加载，例如 `ExceptionServiceProvider`
* Laravel 通过 `Service Provider` 和 `IoC Container` 来将不同的部分联系起来，形成一个单一的、内聚的整体。
