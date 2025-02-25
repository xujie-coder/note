# 典型回答


@Async中关于线程池的使用部分在AsyncExecutionInterceptor中，在这个类中有一个getDefaultExecutor方法， 当我们没有做过自定义线程池的时候，就会用SimpleAsyncTaskExecutor这个线程池。



```plain
@Override
protected Executor getDefaultExecutor(BeanFactory beanFactory) {
    Executor defaultExecutor = super.getDefaultExecutor(beanFactory);
    return (defaultExecutor != null ? defaultExecutor : new SimpleAsyncTaskExecutor());
}
```



> 但是这里需要注意的是，他并不是无脑的的直接创建一个新的，这部分在扩展知识中讲
>



SimpleAsyncTaskExecutor这玩意坑很大，其实他并不是真的线程池，它是不会重用线程的，每次调用都会创建一个新的线程，也没有最大线程数设置。并发大的时候会产生严重的性能问题。



他的doExecute核心逻辑如下：  


```c

	/**
	 * Template method for the actual execution of a task.
	 * <p>The default implementation creates a new Thread and starts it.
	 * @param task the Runnable to execute
	 * @see #setThreadFactory
	 * @see #createThread
	 * @see java.lang.Thread#start()
	 */
	protected void doExecute(Runnable task) {
		Thread thread = (this.threadFactory != null ? this.threadFactory.newThread(task) : createThread(task));
		thread.start();
	}

```



所以，我们应该自定义线程池来配合@Async使用，而不是直接就用默认的。



# 扩展知识


## 自定义线程池


我们可以通过如下方式，自定义一个线程池：



例子用的是这篇的改造的：

[✅在Spring中如何使用Spring Event做事件驱动](https://www.yuque.com/hollis666/qyhor6/lgs78ulq6l3cg1qk)



```c
@Configuration
@EnableAsync
public class AsyncExecutorConfig {
    @Bean("registerSuccessExecutor")
    public Executor caseStartFinishExecutor() {

        ThreadFactory namedThreadFactory = new ThreadFactoryBuilder()
                .setNameFormat("registerSuccessExecutor-%d").build();

        ExecutorService executorService = new ThreadPoolExecutor(100, 200,
                0L, TimeUnit.MILLISECONDS,
                new LinkedBlockingQueue<Runnable>(1024), namedThreadFactory, new ThreadPoolExecutor.AbortPolicy());

        return executorService;
    }

}

```



并且把他声明为@Configuration，然后也可以把Application.java中的 @EnableAsync放到这里。



接下来在使用@Async的时候，指定一下即可：



```c
/**
 * 案件中心内部事件监听器
 *
 * @author Hollis
 */
@Component
public class RegisterEventListener {

    @EventListener(RegisterSuccessEvent.class)
     @Async("registerSuccessExecutor")
    public void onApplicationEvent(RegisterSuccessEvent event) {
        RegisterInfo registerInfo = (RegisterInfo) event.getSource();

        //执行发送欢迎短信的逻辑
        //这里注意要控制好并发，这个例子就不细说了，可以参考：✅基于Redis的分布式锁，解决短信验证码重复发放等问题
    }
}

```



在@Async中指定registerSuccessExecutor即可。这样在后续执行时，就会用到我们自定义的线程池了。





## 不是无脑创建SimpleAsyncTaskExecutor


在getDefaultExecutor的实现中，并不是一上来就直接new SimpleAsyncTaskExecutor()的，而是先尝试着获取默认的执行器。



```java
protected Executor getDefaultExecutor(@Nullable BeanFactory beanFactory) {
    Executor defaultExecutor = super.getDefaultExecutor(beanFactory);
    return (defaultExecutor != null ? defaultExecutor : new SimpleAsyncTaskExecutor());
}

```



看一下这个`super.getDefaultExecutor(beanFactory);`代码：



```java
protected Executor getDefaultExecutor(@Nullable BeanFactory beanFactory) {
    if (beanFactory != null) {
        try {
            // Search for TaskExecutor bean... not plain Executor since that would
            // match with ScheduledExecutorService as well, which is unusable for
            // our purposes here. TaskExecutor is more clearly designed for it.
            return beanFactory.getBean(TaskExecutor.class);
        }
        catch (NoUniqueBeanDefinitionException ex) {
            logger.debug("Could not find unique TaskExecutor bean. " +
                    "Continuing search for an Executor bean named 'taskExecutor'", ex);
            try {
                return beanFactory.getBean(DEFAULT_TASK_EXECUTOR_BEAN_NAME, Executor.class);
            }
            catch (NoSuchBeanDefinitionException ex2) {
                if (logger.isInfoEnabled()) {
                    logger.info("More than one TaskExecutor bean found within the context, and none is named " +
                            "'taskExecutor'. Mark one of them as primary or name it 'taskExecutor' (possibly " +
                            "as an alias) in order to use it for async processing: " + ex.getBeanNamesFound());
                }
            }
        }
        catch (NoSuchBeanDefinitionException ex) {
            logger.debug("Could not find default TaskExecutor bean. " +
                    "Continuing search for an Executor bean named 'taskExecutor'", ex);
            try {
                return beanFactory.getBean(DEFAULT_TASK_EXECUTOR_BEAN_NAME, Executor.class);
            }
            catch (NoSuchBeanDefinitionException ex2) {
                logger.info("No task executor bean found for async processing: " +
                        "no bean of type TaskExecutor and no bean named 'taskExecutor' either");
            }
            // Giving up -> either using local default executor or none at all...
        }
    }
    return null;
}
```



简单总结一下，就是会先尝试获取TaskExecutor的实现类，这里如果能且仅能找到唯一一个，那么就用这个，如果找不到，或者找到了多个，那么就会走到`catch (NoUniqueBeanDefinitionException ex) `和`catch (NoSuchBeanDefinitionException ex)`的分支中，这里就是获取beanName为taskExecutor的Bean



以上查询如果还是没查到，那么再创建SimpleAsyncTaskExecutor。



**所以，也就是说Spring给了我们一个机会，他也知道SimpleAsyncTaskExecutor这玩意性能不行，所以他会尝试获取一个我们自定义的线程池，如果我们定义了一个，那么他就会用我们定义的这个，如果我们定义了多个，那么他就不用了。**

****

**所以，这也是为什么有的时候我们自定义了一个线程池，但是没有在@Async中指定，他也能用到他的原因，但是这种方式并不靠谱，万一后面又新定义了一个，那就凉凉了。或者我们可以把一个线程池的BeanName设置成taskExecutor也行。**

