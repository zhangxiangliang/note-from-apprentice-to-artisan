# Open Closed Principle

## Introduction
* 一个应用的生命周期，大部分时间都是在增加功能，而非一直从零开始写新功能
* 修改代码的时候，可能引入新的程序错误，或者破坏原来的代码
* 使用 `开放封闭原则` 可以让我们像写新代码一样
* 开放封闭原则规定对扩展是开放，对修改是封闭

## In Action

### 案例
* 使用了依赖注入，代码可读性高，易于测试
* 如果判断规则改变了？
* 如果随着业务发展，要增加一大堆新规则呢？
* 导致这段代码需要随着业务逻辑的改变而改变
* 违反了开放封闭原则，我们希望对扩展开放，而不是修改

```
$recent = $this->orders->getRecentOrderCount($order->account);
if($recent > 0) {
    throw new Exception('Duplicate order likely.');
}
```

### 修改
* 定义订单验证接口

```
interface OrderValidatorInterface
{
    public function validate(Order $order);
}
```

* 实现接口来预防重复订单

```
class RecentOrderValidator implements OrderValidatorInterface
{
    public function __construct(OrderRepository $orders)
    {
        $this->orders = $orders;
    }
    public function validate(Order $order)
    {
        $recent = $this->orders->getRecentOrderCount($order->account);
        if($recent > 0) {
            throw new Exception('Duplicate order likely.');
        }
    }
}
```

* 实现接口来判断是否停用

```
class SuspendedAccountValidator implements OrderValidatorInterface
{
    public function validate(Order $order)
    {
        if($order->account->isSuspended()) {
            throw new Exception("Suspended accounts may not order.");
        }
    }
}
```

* 在 `OrderProcessor` 中注入验证器数组

```
class OrderProcessor
{
    public function __construct(BillerInterface $biller, OrderRepository $orders, array $validators = array())
    {
        $this->biller = $biller;
        $this->orders = $orders;
        $this->validators = $validators;
    }
    public function process(Order $order)
    {
        foreach($this->validators as $validator)
        {
            $validator->validate($order);
        }
    }
}
```

* IoC 容器里注册 OrderProcessor 类
```
App::bind('OrderProcessor', function()
{
    return new OrderProcessor(
        App::make('BillerInterface'),
        App::make('OrderRepository'),
        array(
            App::make('RecentOrderValidator'),
            App::make('SuspendedAccountValidator')
        )
    );
});
```

## 总结
### 抽象的漏洞
* 小心那些缺少实现细节的依赖
* 不应该要求它的调用者做任何修改
* 当需要调用者进行修改时，这就意味着该依赖遗漏了一些实现的细节
* 当抽象有漏洞的话，开放封闭原则就不管用了
* 每一块代码都应该是“热插拔”式不是必须的
* 不要盲目的应用设计原则，可能会导致过度设计
* 也不要不去使用和思考，当作懒惰的借口


