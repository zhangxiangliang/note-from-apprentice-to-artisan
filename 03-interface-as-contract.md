# Interface As Contract

## Strong Typing & Water Fowl
* 强类型
    * 编译器通常能提供编译时错误检查的功能
    * 方法的输入和输出也更加明确
    * 但是会使得程序僵化
* 弱类型
    * 传递任何类型的对象都可以，只要这个对象能响应
    * 更加灵活的调用和组织代码

## A Contract
* 接口就是约定
* 接口不真正做任何事情
* 接口不包含任何代码实现
* 定义了一个对象应该实现的一系列方法
* 有约定保证了特定方法的实现标准
* 通过多态也能使类型安全的语言变得更灵活

## Interface & Team Development

### 接口就是大纲
* 在设计组件时，使用接口进行设计和讨论
* 实现代码前先用接口讨论好一套好的API

### 例子
* 当你的团队在开发大型应用时，不同的部分有着不同的开发速度
* 前后端开发速度不一致
* 可以使用接口来保证实现
* 前端可以实现一个假的数据接口来使用
* 后端可以在接口实现完成后对其进行替换
