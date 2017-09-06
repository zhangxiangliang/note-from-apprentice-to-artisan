## 控制反转容器
* 继承自 Container 类
* 更容易管理依赖注入

### 绑定接口
使用 IoC Container，切换接口实现只需要一行代码。
```
App::bind('BillingNotifierInterface', function () {
    return new EmailBillingNotifier;
});
App::bind('BillerInterface', function () {
    return new StripeBiller(App::make('BillingNotifierInterface'));
});
```

调用接口
```
class UserController extends BaseController
{
    public function __construct(BillerInterface $biller)
    {
        $this->biller = $biller;
    }
}
```

切换接口
```
App::bind('BillingNotifierInterface', function () {
    return new SmsBillingNotifier;
});
```

### 绑定单例
容器只生成一次提示器对象，接下来使用只会使用这个对象。
```
App::singleton('BillingNotifierInterface', function()
{
    return new SmsBillingNotifier;
});
```

instance 和 singleton 类似，区别是绑定一个已经存在的对象。
```
$notifier = new SmsBillingNotifier;
App::instance('BillingNotifierInterface', $notifier);
```

## 反射解决方案
* 依靠反射来处理类和接口
* 用反射来自动处理依赖是Laravel容器的一个最强大的特性
* 反射是一种运行时探测类和方法的能力

### Laravel 依赖注入流程
* 是否已经有一个 `StripBiller` 的绑定(Resolver)
* 没有绑定？那使用反射来探测一下 `StripBiller` 需要什么依赖
* 递归处理解决 `StripBiller` 需要的所有依赖
* 使用 `ReflectionClass->newInstanceArgs()` 来实例化 `StripBiller`
