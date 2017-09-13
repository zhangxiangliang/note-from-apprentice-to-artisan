# Interface Segregation Principle

## Introduction
* 接口隔离原则规定在实现接口的时候，不能强迫去实现没有用处的方法
* 一个接口的一个实现类，不应该去实现那些自己用不到的方法
* 如果需要，那就是接口设计有问题，违背了接口隔离原则

## In Action
* `Memcached` 实现 `SessionHandlerInterface`，open、close、gc 不需要实现
* 对于需要垃圾回收的时候，我们可以专门定义一个小巧的接口来回收垃圾

### 案例
* 对于 `PasswordReminder@remind`
* 不需要知道 Contact 那么多细节
* 没有必要依赖着一个特定的 ORM
* 切断对 ORM 的依赖，我们可以自由的改变我们后台的存储机制

```
class PasswordReminder {
    public function remind(Contact $contact, $view)
    {
        // Send password reminder e-mail...
    }
}
```

```
class Contact extends Eloquent
{
    public function getNameAttribute()
    {
        return $this->attributes['name'];
    }
    public function getEmailAttribute()
    {
        return $this->attribute['email'];
    }
}
```

### 修改
* 移除了密码找回组件里不必要的依赖
* 足够灵活能使用任何实现了RemindableInterface的类或ORM
* 这其实正是Laravel的密码找回组件如何保持与数据库ORM无关的秘诀

```
interface RemindableInterface {
    public function getReminderEmail();
}
```

```
class Contact extends Eloquent implements RemindableInterface {
    public function getReminderEmail()
    {
        return $this->email;
    }
}
```

```
class PasswordReminder
{
    public function remind(RemindableInterface $remindable, $view) {};
}
```

## 总结
* 我们再次发现了一个使类知道太多东西的陷阱
* 通过小心留意是否让一个类知道了太多
* 我们就可以遵守所有的“坚实”原则
