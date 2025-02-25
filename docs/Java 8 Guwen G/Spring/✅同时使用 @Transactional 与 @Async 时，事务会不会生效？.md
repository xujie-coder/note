# 典型回答


这个问题其实和下面这个是类似的问题，要考察的内容是一样的。但是我们的这个问题要更加复杂一些。



[✅Spring的事务在多线程下生效吗？为什么？](https://www.yuque.com/hollis666/qyhor6/qi1vgi3yg8l663yy)



因为要回答这个问题，需要分情况讨论的。网上很多文章说会失效，但是其实不对的。我们分情况讨论：



**1、如果@Transactional 与 @Async加在同一个方法上，那么事务会生效的。**

****

```plain
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

/**
 * 用户服务
 *
 * @author hollis
 */
@Service
public class UserService {

    @Transactional
    @Async
    public void register(String telephone, String inviteCode) {
        userMapper.save(user);
        userStreamMapper.insertStream(user, UserOperateTypeEnum.REGISTER);
        throw new UnsupportedOperationException("");
    }
}

```



以上代码，在执行后，事务会因为捕获到UnsupportedOperationException er 导致回滚，即 user 表和 userStream 表都没有变更。

****

**2、如果@Transactional方法A调用 @Async方法B，那么A 抛异常了，A 自己会回滚，但是B 不会回滚；B 抛异常了 ， B 自己会回滚，但是A 也不会回滚。**

****

```plain
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

/**
 * 用户服务
 *
 * @author hollis
 */
@Service
public class UserService {

    @Autowired
    UserStreamService userStreamService;

    @Transactional
    public void register(String telephone, String inviteCode) {
        userMapper.save(user);
        userStreamService.insertStream(user, UserOperateTypeEnum.REGISTER);
        
        throw new UnsupportedOperationException("");
    }
}


@Service
public class UserStreamService {

    @Async
    public void insertStream(User user) {
        userStreamMapper.insertStream(user, UserOperateTypeEnum.REGISTER);
    }
}
```



以上代码，在执行后，事务会因为捕获到UnsupportedOperationExceptioner 导致回滚，但是，只有主线程回滚，子线程并不会跟着一起回滚。即 user 表没有变更，但是userStream会插入成功一条记录。



同理，如果在子线程中抛出异常，主线程也不会回滚，如：



```plain
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

/**
 * 用户服务
 *
 * @author hollis
 */
@Service
public class UserService {

    @Autowired
    UserStreamService userStreamService;

    @Transactional
    public void register(String telephone, String inviteCode) {
        userMapper.save(user);
        userStreamService.insertStream(user, UserOperateTypeEnum.REGISTER);
    }
}


@Service
public class UserStreamService {

    @Async
    public void insertStream(User user) {
        userStreamMapper.insertStream(user, UserOperateTypeEnum.REGISTER);
        throw new UnsupportedOperationException("");

    }
}
```

****

以上代码，在执行后， userStream 表没有变更，但是user表会插入成功一条记录。

****

**3、如果@Async方法A调用 @Transactional 方法B，那么A 抛异常了，B 不会回滚（如果 A 没有事务，自己也不会回滚的）；但是 B 抛异常了，B 自己会回滚。**



```plain
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

/**
 * 用户服务
 *
 * @author hollis
 */
@Service
public class UserService {

    @Autowired
    UserStreamService userStreamService;

    @Async
    public void register(String telephone, String inviteCode) {
        userMapper.save(user);
        userStreamService.insertStream(user, UserOperateTypeEnum.REGISTER);
    }
}


@Service
public class UserStreamService {

    @Transactional
    public void insertStream(User user) {
        userStreamMapper.insertStream(user, UserOperateTypeEnum.REGISTER);
        throw new UnsupportedOperationException("");

    }
}
```



这个情况和第一种非常像只不过比那个更加简单一些，所以就不展开说了。



通过以上三个 case 你会发现，**如果在事务中，开启了新的线程，那么就会导致多个线程之间不在同一个事务中，会导致事务隔离。**

****

这是因为，@Transactional 的事务管理使用的是 ThreadLocal 机制来存储事务上下文，而 ThreadLocal 变量是线程隔离的，即每个线程都有自己的事务上下文副本。因此，在多线程环境下，Spring 的声明式事务会“失效”，即新线程中的操作不会被包含在原有的事务中。

****

**但是，如果在一个新的线程开启后，又开始了一个事务，那么在这个线程执行过程中，当前线程的所有操作都是在同一个事务中的。**

