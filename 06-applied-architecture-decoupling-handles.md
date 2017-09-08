# Applied Architecture: Decoupling Handles

## Introduction
* 解耦 各种处理函数
* 解耦 队列处理函数
* 解耦 事件处理函数
* 解耦甚至其他“事件型”的结构 如路由过滤器

### Don't Clog Your Transport Layer
* 大部分的 `处理函数` 可以被当做传输层组件
* 队列触发器、被触发的事件、或者外部发来的请求等都可能调用处理函数
* 处理函数理解为控制器，避免在里面堆积太多具体业务逻辑实现

## Decoupling Handles

### 耦合的例子
```
class SendSMS
{
    public function fire($job, $data)
    {
        // 直接调用了 Twilio_SMS 类，不方便测试
        // 不能进行模拟短信测试，只能发送一条真实的短信
        $twilio = new Twilio_SMS($apiKey);
        $twilio->sendTextMessage([
            'to' => $data['user']['phone_number'],
            'message' => $data['message'],
        ]);
        // 直接使用了 Eloquent，导致测试时可能会影响数据库
        $user = User::find($data['user']['id']);
        $user->messages()->create([
            'to' => $data['user']['phone_number'],
            'message' => $data['message'],
        ]);
        // 无法在队列外使用短信发送
        $job->delete();
    }
}
```

### 解耦合的例子
* 不需要测试队列
* 需要测试短信发送
* 需要测试用户存储

短信发送逻辑抽出到User模型
```
class User extends Eloquent
{
    public function sendSmsMessage(SmsCourierInterface $courier, $message)
    {
        $courier->sendMessage($this->phone, $message);
        return $this->sms()->create([
            'to'=> $this->phone_number,
            'message'=> $message,
        ]);
    }
}
```

修改后的类
```
class SendSMS
{
    public function __construct(UserRepository $users, SmsCourierInterface $courier)
    {
        $this->users = $users;
        $this->courier = $courier;
    }
    public function fire($job, $data)
    {
        $user = $this->users->find($data['user']['id']);
        $user->sendSmsMessage($this->courier, $data['message']);
        $job->delete();
    }
}
```

测试用例
```
public function testUserCanBeSentSmsMessage()
{
    $user = Mockery::mock('User[sms]');
    $relation = Mockery::mock('StdClass');
    $courier = Mockery::mock('SmsCourierInterface');
    $relaction->shouldRecevie('create')->once()->with([
        'to' => '555-555-5555',
        'message' => 'Test'
    ]);
    $user->shouldReceive('sms')->once()->andReturn($relaction);
    $courier->shouldReceive('sendMessage')->once()->with('555-555-5555', 'Test');
    $user->phone = '555-555-5555';
    $user->sendSmsMessage($courier, 'Test');
}
```

## Other Handlers
* 使用类似的方式，改进和解耦很多其他类型的“处理函数”
* 将处理函数限制在转换层的状态
* 可以将庞大的业务逻辑和框架解耦
* 保持整洁的代码结构

### 例子
* 暴露了 plan 变量在传输层
* 如果将来要修改判断，需要修改路由控制器

```
Route::filter('premium', function()
{
    return Auth::user() && Auth::user()->plan == 'premium';
});
```

### 修改后
* 去除了路由中的判断
* 路由不需要管如何判断用户

```
Route::filter('premium', function()
{
    return Auth::user() && Auth::user()->isPremium();
});
```

### Who is Responsible?
* 始终保持一个类应该有什么样的责任
* 避免在处理函数这种传输层直接编写太多你应用的业务逻辑
