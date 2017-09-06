# 依赖注入
* Laravel 框架的基础是 IoC Container (Inversion Of Control)

## 问题
* 控制器 中我们经常操作实际的 数据对象模型库 这样会和控制器带来紧耦合
* 控制器 中只要知道如何访问数据，不需要知道数据从哪里来

### 关注分离 (Separation Of Concerns)
* 每个类都有自己单独的职责，并且该职责应完全被这个类封装
* 分离关注 使得 Web控制器 和 数据访问解耦，更方便存储迁移和测试

## 契约 (Build A Contract)

创建契约

```
interface UserRepositoryInterface
{
    public function all();
}

class DBUserRepository implements UserRepositoryInterface
{
    public function all()
    {
        return User::all()->toArray();
    }
}
```

接口注入容器

```
class UserController extends BaseController
{
    public function __construct(UserRepositoryInterface $users)
    {
        $this->users = $users;
    }
    public function index()
    {
        $users = $this->users->all();
        return View::make('users.index', compact('users'));
    }
}
```

### 严守边界 (Respect Boundaries)
* 保持清晰的职责边界
* HTTP <=> Middleware(Controller, Route) <=> Application
* 不让逻辑入侵 Controller, Route, etc


### 测试用例
```
public function testIndexActionBindsUsersFromRepository()
{
    // Arrange
    $repository = Mockery::mock('UserRepositoryInterface');
    $repository->shouldReceive('all')->once()->andReturn(['foo']));
    App::instance('UserRepositoryInterface', $repository);
    // Act
    $response = $this->action('GET', 'UserController@index');
    // Assert
    $this->assertResponseOk();
    $this->assertViewHas('users', ['foo']);
}
```

## 更进一步

例子: 提醒用户缴费
```
interface BillerInterface {
    public function bill(array $user, $amount);
}
interface BillingNotifierInterface {
    public function notify(array $user, $amount);
}
```

BillerInteface 实现，将不同的 notifier 注入到 biller 中，以后如果想使用短信通知只需要 `new StripeBiller(new SmsNotifier)`，代码更容易维护，明确了类的职责边界。
```
class StripeBiller implements BillerInterface
{
    public function __construct(BillingNotifierInterface $notifier)
    {
        $this->notifier = $notifier;
    }
    public function bill(array $user, $amount)
    {
        $this->notifier->notify($user, $amount);
    }
}
```

### 使用接口
* 加速开发速度
* 提升代码灵活性
* 提升可测试性
* `happy coding` 不喜欢写接口可以日后再精进
