# Liskov Substitution Principle

## Introduction
* 如果一个类使用了一个接口的一个实现类
* 该接口的任何其他实现类也可以被这里直接使用，不用做出任何修改
* 里氏原则规定对象可以被该对象子类的实例所替换，并且不会影响到程序的正确性

## In Action
### 案例
* 例子没有遵循里氏替换原则
* 如果不附加“启动”代码来调用connect方法，则这段代码就没法用

```
class DatabaseOrderRepository implements OrderRepositoryInterface
{
    protected $connection;
    public function connect($username, $password)
    {
        $this->connection = new DatabaseConnection($username, $password);
    }
    public function logOrder(Order $order)
    {
        $this->connection->run('insert into orders values (?, ?)', array(
            $order->id, $order->amount
        ));
    }
}
```

```
public function process(Order $order)
{
    if($this->repository instanceof DatabaseOrderRepository)
    {
        $this->repository->connect('root', 'password');
    }
    $this->repository->logOrder($order);
}
```

### 修改
* DatabaseOrderRepository 移除了启动代码
* 这样可以想用CsvOrderRepository也行，想用DatabaseOrderRepository也行
* 代码实现了里氏替换原则，不用改OrderProcessor一行代码

```
class DatabaseOrderRepository implements OrderRepositoryInterface {
    protected $connector;
    public function __construct(DatabaseConnector $connector)
    {
        $this->connector = $connector;
    }
    public function connect()
    {
        return $this->connector->bootConnection();
    }
    public function logOrder(Order $order)
    {
        $connection = $this->connect();
        $connection->run('insert into orders values (?, ?)', array(
            $order->id, $order->amount
        ));
    }
}
```

## 总结
* 知识就是一个类和它所具有的周边领域
* 用来帮助类完成任务的外围代码和依赖
* 制作一个容错性强大的应用架构
* 限制类的知识是一种常用且重要的手段
* 如果不遵守里氏替换原则，那后果可能会影响其他原则
* 不遵守里氏替换原则，那么开放封闭原则一定也会被打破
