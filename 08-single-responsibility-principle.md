# Single Responsibility Principle

## SOLID Design Principles
* The Signle Responsibility Principle 单一职责原则
* The Open Closed Principle 开放封闭原则
* The Liskov Substitution Principle 里氏替换原则
* The Interface Segregation Principle 接口隔离原则
* The Dependency Inversion Principle 依赖反转原则

## In Action
* 单一职责原则规定 `一个类有且仅有一个理由使其改变`
* 一个类的功能边界和职责应当是十分狭窄且集中的
* 无知是福，一个类应当做它该做的事情
* 能被它的依赖的任何变化所印象
* 单一职责原则关键不仅仅是让函数变短
* 单一职责原则职责写出更精确更高内聚的类
* 确保类里面的方法都属于该类的职责之下
* 一个小巧、清晰且职责明确的类库，使得代码更加解耦，更容易与测试和更改。

### 案例
* `getRecentOrderCount` 拥有了数据库查看的权限
* 如果 验证规则 和 存储方式改变 则需要修改 `OrderProcessor`

```
class OrderProcessor
{
    public function __construct(BillerInterface $biller)
    {
         $this->biller = $biller;
    }
    public function process(Order $order)
    {
        $recent = $this->getRecentOrderCount($order);
        if($recent > 0) {
            throw new Exception('Duplicate order likely.');
        }
        $this->biller->bill($order->account->id, $order->amount);
        DB::table('orders')->insert([
            'account' => $order->account->id,
            'amount' => $order->amount,
            'create_at' => Carbon::now(),
        ]);
    }
    protected function getRecentOrderCount(Order $order)
    {
        $timestamp = Carbon::now()->subMinutes(5);
        return DB::table('orders')->where('account', $order->account->id)
            ->where('create_at', '>=', $timestamps);
    }
}
```

### 修改
* 提取了收集订单数据的责任
* 当读取和写入订单的方式改变
* 不需要再修改 `OrderProcessor` 这个类
* 提供了一个更干净、更有表现力的代码
* 更容易维护的代码

```
class OrderRepository {
    public function getRecentOrderCount(Account $account)
    {
        $timestamp = Carbon::now()->subMinutes(5);
        return DB::table('orders')->where('account', $account->id)
            ->where('created_at', '>=', $timestamp)
            ->count();
    }
    public function logOrder(Order $order)
    {
        DB::table('orders')->insert([
            'account' => $order->account->id,
            'amount' => $order->amount,
            'created_at' => Carbon::now(),
        ]);
    }
}
```

```
class OrderProcessor
{
    public function __construct(BillerInterface $biller, OrderRepository $orders)
    {
        $this->biller = $biller;
        $this->orders = $orders;
    }
    public function process(Order $order)
    {
        $recent = $this->orders->getRecentOrderCount($order->account);
        if($recent > 0)
        {
            throw new Exception('Duplicate order likely.');
        }
        $this->biller->bill($order->account->id, $order->amount);
        $this->orders->logOrder($order);
    }
}
```
