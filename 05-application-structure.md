# Application Structure

## Where does this class belong?
* 查毒于 `MVC` 模型，大量开发人员都被灌输了很多概念
    * Controller 处理 HTTP 请求
    * Model 操作 Database
    * View 渲染 HTML
* Email 类写到哪 ???
* Validate 类写到哪 ???
* 调用外部 API 类写到哪 ???

## MVC is Killing You
* 一个应用不单单需要一个数据库访问类
* 一个应用还需要 数据验证、调用外部服务、发送电子邮件 等等
* Model 是用来将我们的应用划分成更小、更清晰的类，使得各部分代码有着明确的权责。

## Bye, Bye Models
* 不要惧怕建立目录来管理应用
* 要常常将你的应用切割成小组件，每一个组件都要有十分专注的职责
* 跳出“模型”的框框来思考。比如我们之前就说过，你可以创建个Repositories目录来存放你所有的数据访问类
* 优化应用的设计结构的关键就是责任划分，或者说是创建不同的责任层次
* 摆脱了models目录后，你通常就能克服心理障碍，实现好的设计
* 创建一个更合适的目录结构来为你的应用服务

### 验证数据
* 数据验证方法写进你的实体类里面 `User@validForCreation`, `User@hasValidDomain`
* 或者专门创建个验证器类UserValidator，放到Validation命名空间，将验证器类注入到你的repository类

### 核心思想是分层
* 优化应用的设计结构的关键就是 `责任划分`
* 或者创建不同的责任层次
* 控制器只负责接收和响应HTTP请求然后调用合适的业务逻辑层的类
* 业务逻辑/领域逻辑层才是真正的程序
* 程序包含了读取数据，验证数据，执行支付，发送电子邮件等等
* 事实领域逻辑层不需要知道任何关于“网络”的事情
* 网络仅仅是个访问你程序的传输机制
* 关于网络和HTTP请求的一切不应该超出路由和控制器层

### 判断类的作用范围
* 要时常检查一个类是否管得太宽
* 你要常常问自己“这个类需不需要关心XXX呢？”
* 如果不需要，那么把这块逻辑抽出来放到另一个类里面，然后用依赖注入的方式进行处理
* 方法： 检查你为什么要改这块儿代码
* 例子：
    * 当我们想调整通知逻辑的时候，我们需要修改Biller的实现代码么？
    * Biller的实现仅仅需要考虑支付，它与通知逻辑应当仅通过约定来进行交互
    * 所以可以将 Biller 抽出

### 例子
```
// app
    // QuickBill
        // Repositories
            -> UserRepositories.php
            -> PaymentRepository.php
        // Billing
            -> BillerInterface.php
            -> StripeBiller.php
        // Notifications
            -> BillingNotifierInterface
            -> SmsBillingNotifier.php
        // Validation
            -> UserValidation.php
            -> PaymentValidation.php
    -> User.php
    -> Payment.php
```

## Where To Put "Stuff"
* 经常迷惑于东西应该放哪：
    * 辅助函数 放哪？
    * 视图组件 放哪？
    * 事件监听器 放哪？
* 答案：想放哪都行，Laravel 并未作出任何文件系统的约定

## Helper Functions
* 辅助函数可以在 `autoload.php` 中引入
* 辅助函数可以在 `composer.php` 中引入
* 辅助函数可以在 `app/helper.php` 目录

## Event Listeners
* 监听器 可以放在 Service Provider 中
* 利用 Service Provider 可以管理监听器
* 利用 Service Provider 可以让代码整洁
* 应该放置于 `ServiceProvider@boot`


## Error Handlers
* 错误处理方法也最好放到服务提供者里面
* 让呆板的代码离你应用的业务逻辑越远越好
* 如果错误处理方法不多，可以简单的写在启动文件中

## The Rest
* 遵循 `PSR-4` 可以保持类的整洁
* 有其他“注册”性质的操作都可以放在服务提供者里面
* 可以根据自己的需求去搭建适合自己的应用结构

### 一个简单的目录结构例子
```
// app
    // QuickBill
        // Billing
        // Extensions
            //Pagination
                -> Environment.php
        // Providers
            -> EventPusherServiceProvider.php
        // Repositories
    User.php
    Payment.php
```
