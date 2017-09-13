# Dependency Inversion Principle

## Introduction
* 高等级的代码不应该依赖低等级的代码
* 高等级的代码应该依赖着抽象层
* 抽象定义不应该依赖着具体实现
* 具体实现应该依赖着抽象定义

## In Action
### 实例
* Authenticator 与 数据库紧密联系在一起
* 高级代码 `Authenticator` 依赖 低级代码 `DatabaseConnection`

```
class Authenticator
{
    public function __construct(DatabaseConnection $db)
    {
        $this->db = $db;
    }
    public function findUser($id)
    {
        return $this->db->exec('select * from users where id = ?', array($id));
    }
    public function authenticate($credentials) {}
}
```

### 修改
* Authenticator现在依赖于两个高级抽象：UserProviderInterface和HasherInterface
* 低级别代码实现了UserProviderInterface接口，该低级代码依赖着这个接口
* 传统的依赖关系链已经被反转了，代码变得更灵活，更加无惧变化
* 在反转Authenticator的依赖之前，它除了使用数据库存储系统别无选择。如果我们改变了存储系统，Authenticator也需要被修改，这就违背了开放封闭原则。这些设计原则通常一荣俱荣一损俱损。

```
class Authenticator {
    public function __construct(UserProviderInterface $users, HasherInterface $hash)
    {
        $this->hash = $hash;
        $this->users = $users;
    }
    public function findUser($id)
    {
        return $this->users->find($id);
    }
    public function authenticate($credentials)
    {
        $user = $this->users->findByUsername($credentials['username']);
        return $this->hash->make($credentials['password']) == $user->password;
    }
}
```

```
interface UserProviderInterface
{
    public function find($id);
    public function findByUsername($username);
}
```

```
class RedisUserProvider implements UserProviderInterface
{
    public function __construct(RedisConnection $redis)
    {
        $this->redis = $redis;
    }
    public function find($id)
    {
        $this->redis->get('users:'.$id);
    }
    public function findByUsername($username)
    {
        $id = $this->redis->get('user:id:'.$username);
        return $this->find($id);
    }
}
```

## 总结
* 低级代码用于实现基本的操作，比如从磁盘读文件，操作数据库等
* 高级代码用于封装复杂的逻辑，依靠低级代码来达到功能目的，不能直接和低级代码耦合在一起
* 高级代码应该依赖着低级代码的顶层抽象，比如接口
* 贯彻这一原则会反转好多开发者设计应用的方式。不再将高级代码直接和低级代码以“自上而下”的方式耦合在一起，这个原则提出无论高级还是低级代码都要依赖于一个高层次的抽象
