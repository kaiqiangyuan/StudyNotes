[toc]

# 一、Condition

![2](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602093534.png)

```java
package com.ykq.juc;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class App {
	public static void main(String[] args) {
		Tick tick = new Tick();
		new Thread(() -> {
			for (int i = 0; i < 10; i++) {
				tick.printA();
			}
		}, "A").start();
		new Thread(() -> {
			for (int i = 0; i < 10; i++) {
				tick.printB();
			}
		}, "B").start();
		new Thread(() -> {
			for (int i = 0; i < 10; i++) {
				tick.printC();
			}
		}, "C").start();
	}
}

class Tick {
	
	private Lock lock = new ReentrantLock();
	private Condition condition1 = lock.newCondition();
	private Condition condition2 = lock.newCondition();
	private Condition condition3 = lock.newCondition();
	private int number = 1;
	
	public void printA() {
		lock.lock();
		try {
			//业务代码 ，判断-》执行-》通知
			while(number!=1) {
				condition1.await();
			}
			System.out.println(Thread.currentThread().getName()+"->AA");
			number = 2;
			condition2.signal();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}
	
	public void printB(){
		lock.lock();
		try {
			//业务代码 ，判断-》执行-》通知
			while(number!=2) {
				condition2.await();
			}
			System.out.println(Thread.currentThread().getName()+"->BB");
			number = 3;
			condition3.signal();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}
	public void printC() {
		lock.lock();
		try {
			//业务代码 ，判断-》执行-》通知
			while(number!=3) {
				condition3.await();
			}
			System.out.println(Thread.currentThread().getName()+"->CC");
			number = 1;
			condition1.signal();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}
}
```

**并发下ArrayList不安全解决方案**

```java
public static void main( String[] args ) {
	/**
	 * 并发下ArrayList不安全
	 * 解决方案：
	 * 1、List<String> list = new Vector<String>();
	 * 2、List<String> list = Collections.synchronizedList(new ArrayList<>());
	 * 3、List<String> list = new CopyOnWriteArrayList<>();
	 */
	List<String> list = new CopyOnWriteArrayList<>();
	for (int i = 0; i < 10; i++) {
        new Thread(()->{
        list.add("a");
        System.out.println(list);
        },String.valueOf(i)).start();
   }
}
```

# 二、CountDownLatch（减法计数器）

==countDownLatch.countDown();== //数量减1

==countDownLatch.await();==  //等待计数器归零，然后再向下执行

每次有线程调用countDownLatch()数量-1，假设计数器变为0，countDownLatch.await();就会被唤醒，继续执行。

```java
CountDownLatch countDownLatch = new CountDownLatch(6);

for (int i = 0; i < 6; i++) {
    new Thread(()->{
    System.out.println(Thread.currentThread().getName()+"Go out");
    countDownLatch.countDown(); //数量减1
    },String.valueOf(i)).start();
}

countDownLatch.await();  //等待计数器归零，然后再向下执行

System.out.println("Close Door");
```

# 三、CyclicBarrier(加法计数器)

```java
	public static void main(String[] args){
		CyclicBarrier cyclicBarrier = new CyclicBarrier(7, ()->{
			System.out.println("hello");
		});
		for (int i = 1; i <= 7; i++) {
			int temp = i;
			new Thread(()->{
				System.out.println(Thread.currentThread().getName()+temp);
				try {
					cyclicBarrier.await();
				} catch (InterruptedException | BrokenBarrierException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}).start();	
		}
	}
```

# 四、Semaphore(限流)

==semaphore.acquire(==);获得，假设如果已经满了，等待，等待被释放位置。

==semaphore.release();==释放，会将当前的信号释放，然后唤醒等待的线程。

作用：**多个共享资源互斥的使用，并发限流，控制最大的线程数**。

```java
public static void main(String[] args) {
		//线程数量  限流操作
		Semaphore semaphore = new Semaphore(2);
		
		for (int i = 1; i <=6 ; i++) {
			new Thread(()->{
				try {
					semaphore.acquire();   //得到
					System.out.println(Thread.currentThread().getName()+"执行");
					TimeUnit.SECONDS.sleep(2);    //模拟执行的时间
					System.out.println(Thread.currentThread().getName()+"结束");
				} catch (InterruptedException e) {
					e.printStackTrace();
				} finally {
					semaphore.release();   //释放
				}
			},String.valueOf(i)).start();
		}
	}
```

# 五、ReadWriteLock（读写锁）

```java
/**
 * 独占锁（写锁） 一次只能被一个线程占有
 * 共享锁（读锁） 多个线程可以同时占有
 */
public class ReadWriteLock {
	public static void main(String[] args) {
		MyLock myLock = new MyLock();
		//写入
		for (int i = 1; i <=5; i++) {
			final int temp = i;
			new Thread(()->{
				myLock.put(temp+"", temp+"");
			},String.valueOf(i)).start();
		}
		//读取
		for (int i = 1; i <=5; i++) {
			final int temp = i;
			new Thread(()->{
				myLock.get(temp+"");
			},String.valueOf(i)).start();
		}
	}
}

class MyLock{
	//被volatile修饰的变量能够保证每个线程能够获取该变量的最新值，从而避免出现数据脏读的现象。
	private volatile Map<String,Object> map = new HashMap<>();
	//读写锁，更加细粒度的控制
	private ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();
	
	//存，写入的时候，只希望同时只有一个线程写
	public void put(String key,Object value) {
		readWriteLock.writeLock().lock();
		try {
			System.out.println(Thread.currentThread().getName()+"写入"+key);
			map.put(key, value);
			System.out.println(Thread.currentThread().getName()+"写入ok");
		} catch(Exception e) {
			e.printStackTrace();
		} finally {
			readWriteLock.writeLock().unlock();
		}
	}
	
	//取，读的时候，所有人都可以读
	public void get(String key) {
		readWriteLock.writeLock().lock();
		try {
			System.out.println(Thread.currentThread().getName()+"读取"+key);
			Object object = map.get(key);
			System.out.println(Thread.currentThread().getName()+"读取ok");
		} catch(Exception e) {
			e.printStackTrace();
		} finally {
			readWriteLock.writeLock().unlock();
		}
	}
}
```

# 六、BlockingQueue（队列）四组API

![QQ20200502-094624@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602093936.jpg)

|   **方式**   | **抛出异常** | **有返回值，不抛出异常** | **阻塞，等待** | **超时等待** |
| :----------: | :----------: | :----------------------: | :------------: | :----------: |
|     添加     |    add()     |         offer()          |     put()      |  offer(,,)   |
|     移除     |   remove()   |          poll()          |     take()     |   poll(,)    |
| 检测队首元素 |  element()   |          peek()          |       -        |      -       |

```java
	public static void test1() throws InterruptedException {
		ArrayBlockingQueue<Object> blockingQueue = new ArrayBlockingQueue<>(3);
		
		blockingQueue.offer("a");
		blockingQueue.offer("b");
		blockingQueue.offer("c");
		//blockingQueue.offer("d",3,TimeUnit.SECONDS);  //等待3秒
		System.out.println(blockingQueue.poll());
		System.out.println(blockingQueue.poll());
		System.out.println(blockingQueue.poll());
		System.out.println(blockingQueue.poll(3, TimeUnit.SECONDS));
	}
```

# 七、SynchronousQueue（同步队列）

```java
/**
 * 同步队列
 * 和其它的BlockingQueue不一样，SynchronousQueue不存储元素
 * put了一个元素，必须从里面先take取出来，否则不能在put进去值。
 */
public class SynchronousQueueDemo {
	public static void main(String[] args) {
		
		BlockingQueue<String> blockingQueue = new SynchronousQueue<>();
		
		new Thread(()-> {
			try {
				System.out.println(Thread.currentThread().getName()+"->1");
				blockingQueue.put("1");
				System.out.println(Thread.currentThread().getName()+"->2");
				blockingQueue.put("2");
				System.out.println(Thread.currentThread().getName()+"->3");
				blockingQueue.put("3");
			
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		},"存").start();
		new Thread(()-> {
			try {
				TimeUnit.SECONDS.sleep(3);
				System.out.println(Thread.currentThread().getName()+"->"+blockingQueue.take());
				TimeUnit.SECONDS.sleep(3);
				System.out.println(Thread.currentThread().getName()+"->"+blockingQueue.take());
					TimeUnit.SECONDS.sleep(3);
				System.out.println(Thread.currentThread().getName()+"->"+blockingQueue.take());
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		},"取").start();
		
	}
}
```

# 八、线程池（4大方法，7大参数，4大拒绝策略）

## 8.1 四大方法（==不安全，需要手动创建线程池==）

- **newCachedThreadPool**
  创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。
- **newFixedThreadPool**
  创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。
- newScheduledThreadPool
  创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。
- newSingleThreadExecutor
  创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。

```java
//Executors 工具类：3大方法
public class Demo1 {
	public static void main(String[] args) {
		//单个线程
//		ExecutorService threadPool = Executors.newSingleThreadExecutor();
		//创建一个固定的线程池大小
//		ExecutorService threadPool = Executors.newFixedThreadPool(5);
		//动态的大小
		ExecutorService threadPool = Executors.newCachedThreadPool();
		try {
			for (int i = 0; i < 10; i++) {
				//通过线程池来创建线程
				threadPool.execute(()->{
					System.out.println(Thread.currentThread().getName()+"ok");
				});
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			//线程池用完，程序结束，关闭线程池
			threadPool.shutdown();
		}
		
	}
}
```

**submit和execute区别**

| 方法名  | 返回值 |      任务接口      |           向外层调用者抛出异常           |
| :-----: | :----: | :----------------: | :--------------------------------------: |
| execute |  void  |      Runnable      |               无法抛出异常               |
| submit  | Future | Callable和Runnable | 能抛出异常，通过Future.get捕获抛出的异常 |

**==阿里云开发手册中说明使用Executors容易导致OOM，建议手动创建线程池，*ThreadPoolExecutor*==**

![QQ20200502-125342@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602094536.jpg)

## 8.2 七大参数：==ThreadPoolExecutor==

```java
public static void main(String[] args) {
		//自定义线程池
		ExecutorService threadPool = new ThreadPoolExecutor(
				2,   //核心线程池大小
				5,   //最大核心线程池大小
				3,    //超时了没有人调用就会释放
				TimeUnit.SECONDS,  //超时单位
				new LinkedBlockingDeque<>(3),  //阻塞队列
				Executors.defaultThreadFactory(),  //线程工厂，创建线程,一般不需要动
				new ThreadPoolExecutor.AbortPolicy());     //拒绝策略
		/**
		 * 1、new ThreadPoolExecutor.AbortPolicy()  丢弃任务并抛出RejectedExecutionException异常(默认)。
		 * 2、new ThreadPoolExecutor.CallerRunsPolicy()  由调用线程处理该任务。哪来去哪里处理执行
		 * 3、new ThreadPoolExecutor.DiscardPolicy()    丢弃任务，但是不抛出异常。
		 * 4、new ThreadPoolExecutor.DiscardOldestPolicy()  丢弃队列最前面的任务，然后重新尝试执行任务。   丢列满了，尝试和最早当竞争，当线程很多时可以会有效，不会抛出异常
		 */		
		try {
			//最大承载等于队列new LinkedBlockingDeque<>(3)+max的值5
			for (int i = 1; i <= 9; i++) {
				//通过线程池来创建线程
				threadPool.execute(()->{
					System.out.println(Thread.currentThread().getName()+" ok");
				});
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			//线程池用完，程序结束，关闭线程池
			threadPool.shutdown();
		}
		
}
```

手动创建线程池

![QQ20200502-135919@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602094719.jpg)

- **@Configuration：**Spring 容器在启动时，会加载带有 @Configuration 注解的类，对其中带有 @Bean 注解的方法进行处理。
- **@Bean：**是一个方法级别上的注解，主要用在 @Configuration 注解的类里，也可以用在 @Component 注解的类里。添加的 bean 的 id 为方法名。
- **@PropertySource：**加载指定的配置文件。value 值为要加载的配置文件，ignoreResourceNotFound 意思是如果加载的文件找不到，程序是否忽略它。默认为 false 。如果为 true ，则代表加载的配置文件不存在，程序不报错。在实际项目开发中，最好设置为 false 。**如果 application.properties 文件中的属性与自定义配置文件中的属性重复，则自定义配置文件中的属性值被覆盖，加载的是 application.properties 文件中的配置属性。**
- **@Slf4j：**lombok 的日志输出工具，加上此注解后，可直接调用 log 输出各个级别的日志。
- **@Value：**调用配置文件中的属性并给属性赋予值。

ExecutorConfig

```java
@Configuration
@EnableAsyncpublic
class ExecutorConfig {
    @Bean(name = "asyncTaskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(200);
        executor.setKeepAliveSeconds(60);
        executor.setThreadNamePrefix("taskExecutor-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());

        return executor;
    }
}
```

通常 **ThreadPoolTaskExecutor 是和 @Async 一起使用**。在一个方法上添加 @Async 注解，表明是异步调用方法函数。@Async 后面加上线程池的方法名或 bean 名称，表明异步线程会加载线程池的配置，**一定要在启动类上添加 @EnableAsync 注解，这样 @Async 注解才会生效**，或者在ExecutorConfig中增加**@EnableAsync**

```java
@Componentpublic
class ThreadTest {
    /**     * 每10秒循环一次，一个线程共循环10次。     */ @Async("asyncTaskExecutor")
    public void ceshi3() {
        for (int i = 0; i <= 10; i++) {
            log.info("ceshi3: " + i);

            try {
                Thread.sleep(2000 * 5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

***或者通过AsyncConfigurer自定义创建线程池***

> **四种拒绝策略**

![QQ20200502-131347@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602094855.jpg)

- ThreadPoolExecutor.AbortPolicy 丢弃任务并抛出RejectedExecutionException异常(默认)。
- ThreadPoolExecutor.DiscardPolic 丢弃任务，但是不抛出异常。
- ThreadPoolExecutor.DiscardOldestPolicy 丢弃队列最前面的任务，然后重新尝试执行任务。
- ThreadPoolExecutor.CallerRunsPolic 由调用线程处理该任务。

> **最大线程如何定义：（调优）**

1. CPU 密集型，几核就是几，可以保持CPU的效率最高

```java
System.out.println(Runtime.getRuntime().availableProcessors());   //查看CPU核数
```

2. IO  密集度    判断程序中十分耗IO的线程 。 可以设置成两倍

## 8.3 ThreadPoolExecutor和ForkJoinPool

- ForkJoinPool适用于许多依赖于任务的，生成的，短暂的，几乎没有阻塞(即计算密集型)的任务
- ThreadPoolExecutor用于很少的，独立的，外部生成的，较长的，有时是阻塞的任务
- **当需要处理==递归分治算法==时，考虑使用ForkJoinPool。**
- 仔细设置不再进行任务划分的阈值，这个阈值对性能有影响。
- Java 8中的一些特性会使用到ForkJoinPool中的通用线程池。在某些场合下，需要调整该线程池的默认的线程数量。

ThreadPool(TP)和ForkJoinPool(FJ)针对不同的用例.主要区别在于不同执行者使用的队列数量决定了哪种类型的问题更适合任一执行者。

FJ执行器具有**n个(又是并行性级别)单独的并发队列(双端队列)**，而TP执行器只有**一个并发队列**(这些队列/双端队列可能是不遵循JDK Collections API的自定义实现).结果，在您生成大量(通常**运行时间相对较短**)任务的情况下，**FJ执行程序的性能会更好**，因为独立队列将最大程度地减少并发操作，而很少的窃取将有助于负载平衡.在TP中，由于只有一个队列，所以每次将工作出队时都会有并发操作，这将成为一个相对的瓶颈并限制性能.

相反，如果长期运行的任务相对较少，则TP中的单个队列不再是性能的瓶颈。但是，**n无关的队列和相对频繁的偷窃尝试现在将成为FJ的瓶颈，因为可能会有很多徒劳的偷窃尝试，这会增加开销**。

>四大函数式接口

**function（函数型接口）**

```java
public static void main(String[] args) {
    //  funciton 函数接口，有一个输入参数，有一个输出		
    Function<String, String> function = new Function<String, String>() {						
        @Override			
        public String apply(String t) {				
            // TODO Auto-generated method stub				
            return t;			
        }		
    };		
    System.out.println(function.apply("123"));	
}
```

只要是函数式接口 可以用lambda表达式简化

```java
public static void main(String[] args) {		        
    Function function = (str)->{return str;};        
    System.out.println(function.apply("123"));
}
```

**Predicate（断定型接口）**

```java
public static void main(String[] args) {		
    //   断定型接口：有一个输入参数，返回值只能是 布尔值！		
    //判读字符串是否为空		
    Predicate<String> predicate = new Predicate<String>() {						
        @Override			
        public boolean test(String str) {				
            return str.isEmpty();			
        }		
    };		
    System.out.println(predicate.test("123"));	
}
```

用lambda表达式简化

```java
public static void main(String[] args) {		
    //   断定型接口：有一个输入参数，返回值只能是 布尔值！		
    //判读字符串是否为空		
    Predicate<String> predicate = (str)->{
        return str.isEmpty();                    
    };			
    System.out.println(predicate.test("123"));	
}
```

**Consumer（消费型接口）**

```java
public static void main(String[] args) {		
    //Consumer 消费性接口 只有输入 没有返回值		
    Consumer<String> consumer = new Consumer<String>() {						
        @Override			
        public void accept(String t) {				
            System.out.println(t);			
        }		
    };		
    consumer.accept("123");	
}
```

用lambda表达式简化

```java
public static void main(String[] args) {		
    Consumer<String> consumer = (str)->{
        System.out.println(str);
    };		
    consumer.accept("123");	
}
```

**Supplier（供给型接口）**

```java
public static void main(String[] args) {		
    //没有参数，只有返回值		
    Supplier<String> supplier = new Supplier<String>() {						
        @Override			
        public String get() {				
            System.out.println("123");				
            return "abc";			
        }		
    };		
    System.out.println(supplier.get());	
}
```

用lambda表达式简化

```java
public static void main(String[] args) {		
    //没有参数，只有返回值		
    Supplier<String> supplier = ()->{
        return "abc";
    };		
    System.out.println(supplier.get());	
}
```

# 九、lambda表达式、链式编程、函数式接口、stream流式计算

```java
@Data@NoArgsConstructor@AllArgsConstructorpublic class User {		
    private int id;	
    private String name;	
    private int age;
}
```

```java
/**
* 1、ID  必须是偶数
* 2、年龄必须大于23岁
* 3、用户名转为大写字母
* 4、用户名字母倒着排序
* 5、只输出一个用户 
*/
public class Test {	
    public static void main(String[] args) {		
        User u1 = new User(1, "a", 21);		
        User u2 = new User(2, "b", 22);		
        User u3 = new User(3, "c", 23);		
        User u4 = new User(4, "d", 24);		
        User u5 = new User(6, "e", 25);		
        List<User> list = Arrays.asList(u1,u2,u3,u4,u5);		
        // lambda表达式、链式编程、函数式接口、stream流式计算		
        list.stream()			
            .filter((u)->{return u.getId()%2==0;})			
            .filter((u)->{return u.getAge()>23;})			
            .map((u)->{return u.getName().toUpperCase();})			
            .sorted((uu1,uu2)->{return uu2.compareTo(uu1);})			
            .limit(1)			
            .forEach(System.out::println);	
    }
}
```

# 十、ForkJoin

大数据量时使用

![QQ20200503-122411@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602095448.jpg)

```java
/** 
* 求和计算的任务 
*   如何使用forkjoin 
*   1、通过  forkjoinpool 执行 
*   2、计算任务forkjoinpool.execute​(ForkJoinTask<?> task) 
*   3、计算类要继承   RecursiveTask 
*/
public class ForkJoinDemo extends RecursiveTask<Long>{	
    private Long start;  //  开始	
    private Long end;   //结束		//临界值	
    private Long temp = 10000L;	
    public ForkJoinDemo(Long start, Long end) {		
        this.start = start;		
        this.end = end;	
    }	
    //计算方法	
    @Override	
    protected Long compute() {		
        if((end-start)>temp) {			
            Long sum = 0L;			
            for (Long i = start; i <= end; i++) {				
                sum = sum + i;			
            }			
            return sum;		
        }else {    
            //   forkjoin			
            long middle = (start+end)/2;   //中间值			
            ForkJoinDemo task1 = new ForkJoinDemo(start, middle);			t
                ask1.fork();			
            ForkJoinDemo task2 = new ForkJoinDemo(middle+1, end);			
            task2.fork();			
            return task1.join()+task2.join();		
        }	
    }
}
```

```java
public class Test {	
    public static void main(String[] args) throws InterruptedException, ExecutionException {		
        test3();	
    }	
    //   sum500000000500000000时间3055	
    public static void test1() {		
        long sum = 0L;		
        long start = System.currentTimeMillis();		
        for (Long i = 1L;  i <= 10_0000_0000 ; i++) {			
            sum = sum + i;		
        }		
        long end = System.currentTimeMillis();		
        System.out.println("sum"+sum+"时间"+(end-start));	
    }		
    //   forkjoin  sum500000000500000000时间5874	
    public static void test2() throws InterruptedException, ExecutionException {		
        long start = System.currentTimeMillis();				
        ForkJoinPool forkJoinPool = new ForkJoinPool();		
        ForkJoinTask<Long> task = new ForkJoinDemo(0L, 10_0000_0000L);		
        ForkJoinTask<Long> submit = forkJoinPool.submit(task);		
        Long sum = submit.get();				
        long end = System.currentTimeMillis();		
        System.out.println("sum"+sum+"时间"+(end-start));	}	
    //  sum500000000500000000时间192	
    public static void test3() {		
        //stream并行流		
        long start = System.currentTimeMillis();		
        Long sum = LongStream.rangeClosed(0L, 10_0000_0000L).parallel().reduce(0, Long::sum);		
        long end = System.currentTimeMillis();		
        System.out.println("sum"+sum+"时间"+(end-start));	
    }
}
```

# 十一、Future

**使用场景**

1. 计算密集场景
2. 处理大数据量
3. 远程方法调用等

> 5个方法

​		Future接口中有下表所示方法，可以获取当前正在执行的任务相关信息。

|                方法                 |                        说明                        |
| :---------------------------------: | :------------------------------------------------: |
| boolean cancel(boolean interruptIf) |                   取消任务的执行                   |
|        boolean isCancelled()        | 任务是否已取消，任务正常完成前将其取消，返回 true  |
|          boolean isDone()           | 任务是否已完成，任务正常终止、异常或取消，返回true |
|               V get()               |         等待任务结束，然后获取V类型的结果          |
| V get(long timeout, TimeUnit unit)  |               获取结果，设置超时时间               |

> 测试Future并行处理

**测试Future与普通处理的区别**，下面为测试目录结构

<img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210719091237.png" alt="image-20210719091237829" style="zoom: 50%;" />

AsyncConfig

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    // 线程池名前缀
    private static final String threadNamePrefix = "Async-Service-";

    @Override
    @Bean(name = "taskExecutor")
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        // 核心线程数
        taskExecutor.setCorePoolSize(8);
        // 最大线程数
        taskExecutor.setMaxPoolSize(16);
        // 队列大小
        taskExecutor.setQueueCapacity(100);
        // 线程池名前缀
        taskExecutor.setThreadNamePrefix(threadNamePrefix);
        // 线程的空闲时间（单位秒）
        taskExecutor.setKeepAliveSeconds(3);
        // 完成任务自动关闭 , 默认为false
        taskExecutor.setWaitForTasksToCompleteOnShutdown(true);
        // 核心线程超时退出，默认为false
        taskExecutor.setAllowCoreThreadTimeOut(true);
        // 线程池对拒绝任务的处理策略
        // CallerRunsPolicy：由调用线程（提交任务的线程）处理该任务
        // 线程池对拒绝任务（无线程可用）的处理策略，目前只支持AbortPolicy、CallerRunsPolicy； 默认为后者
        taskExecutor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        taskExecutor.initialize();
        return taskExecutor;
    }
    
    // 异常处理
    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return new MyAsyncUncaughtExceptionHandler();
    }
    
    class MyAsyncUncaughtExceptionHandler implements AsyncUncaughtExceptionHandler {
        @Override
        public void handleUncaughtException(Throwable ex, Method method, Object... params) {
            System.out.println("class#method: " + method.getDeclaringClass().getName() + "#" + method.getName());
            System.out.println("type        : " + ex.getClass().getName());
            System.out.println("exception   : " + ex.getMessage());
    }
}
```

FutureService

```java
@Service
public class FutureService {
 
    /**
     * @Title: futureTest
     * @Description: 异步处理
     * @Author yuankaiqiang
     * @DateTime 2021-07-19 08:42:07
     * @return
     * @throws InterruptedException
     */
    @Async("taskExecutor")
    public Future<String> futureTest() throws InterruptedException {
        System.out.println("任务执行开始,需要：1000ms");
        for (int i = 0; i < 10; i++) {
            Thread.sleep(100);
            System.out.println("do:" + i);
        }
        System.out.println("完成任务");
        return new AsyncResult<>(Thread.currentThread().getName());
    }
    
    /**
     * @Title: ordinaryTest
     * @Description: 普通处理
     * @Author yuankaiqiang
     * @DateTime 2021-07-19 08:41:53
     * @throws InterruptedException
     */
    public void ordinaryTest(CountDownLatch latch) throws InterruptedException {
        System.out.println("任务执行开始,需要：1000ms");
        for (int i = 0; i < 10; i++) {
            Thread.sleep(100);
            System.out.println("do:" + i);
        }
        System.out.println("完成任务");
        latch.countDown();
    }
}
```

@SpringBootTest

```java
    @Resource
    private FutureService futureService;

    @Test
    void asyncTest() throws InterruptedException, ExecutionException {
        long start = System.currentTimeMillis();
        System.out.println("开始");
        // 耗时任务
        Future<String> future = futureService.futureTest();
        // 另外一个耗时任务
        Thread.sleep(500);
        System.out.println("另外一个耗时任务，需要500ms");

        String s = future.get();
        System.out.println("计算结果输出:" + s);
        System.out.println("共耗时：" + (System.currentTimeMillis() - start));
    }

    @Test
    void ordinaryTest() throws InterruptedException, ExecutionException {
        // 为等待其它线程执行完毕后才结束主线程
        final CountDownLatch latch = new CountDownLatch(1);

        long start = System.currentTimeMillis();
        System.out.println("开始");

        new Thread(()-> {
            try {
                futureService.ordinaryTest(latch);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
        
        latch.await();
        
        // 另外一个耗时任务
        Thread.sleep(500);
        System.out.println("另外一个耗时任务，需要500ms");

        System.out.println("计算结果输出:" + Thread.currentThread().getName());
        System.out.println("共耗时：" + (System.currentTimeMillis() - start));
    }
```

运行测试

```java
// 总耗时1039
开始
任务执行开始,需要：1000ms
do:0
do:1
do:2
do:3
另外一个耗时任务，需要500ms
do:4
do:5
do:6
do:7
do:8
do:9
完成任务
计算结果输出:Async-Service-1
共耗时：1039
// 下面为普通测试的结果，总耗时 1532
开始
任务执行开始,需要：1000ms
do:0
do:1
do:2
do:3
do:4
do:5
do:6
do:7
do:8
do:9
完成任务
另外一个耗时任务，需要500ms
计算结果输出:main
共耗时：1532
```

可以看到 **Future 处理时耗时小于1500ms**，而**普通处理时间大于1500ms**，因为Future在执行耗时任务1的同时也在执行耗时任务2，两个任务并行执行，这就是future模式的好处，在等待时间内去执行其它任务，能够充分利用时间

**注意**

- 异步方法使用static修饰

- 异步类没有使用@Component注解（或其他注解）导致spring无法扫描到异步类


- 异步方法不能与被调用的异步方法在同一个类中


- 类中需要使用@Autowired或@Resource等注解自动注入，不能自己手动new对象


- 如果使用SpringBoot框架必须在启动类中增加@EnableAsync注解

> Future的类图结构

​		Future接口定义了主要的5个接口方法，有**RunnableFuture**和**SchedualFuture**继承这个接口，以及**CompleteFuture**和**ForkJoinTask**实现这个接口。

![20180606202542500](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210717220521.png)

**RunnableFuture**

​		这个接口同时继承Future接口和Runnable接口，在成功执行run（）方法后，可以通过Future访问执行结果。这个接口都实现类是FutureTask,一个可取消的异步计算，这个类提供了Future的基本实现，后面我们的demo也是用这个类实现，它实现了启动和取消一个计算，查询这个计算是否已完成，恢复计算结果。计算的结果只能在计算已经完成的情况下恢复。如果计算没有完成，get方法会阻塞，一旦计算完成，这个计算将不能被重启和取消，除非调用runAndReset方法。

FutureTask能用来包装一个Callable或Runnable对象，因为它实现了Runnable接口，而且它能被传递到Executor进行执行。为了提供单例类，这个类在创建自定义的工作类时提供了protected构造函数。

**SchedualFuture**

​		这个接口表示一个延时的行为可以被取消。通常一个安排好的future是定时任务SchedualedExecutorService的结果。

```java
@Componentpublic class SchedulerService {    
    @Bean    
    public ThreadPoolTaskScheduler threadPoolTaskScheduler() {        
        return new ThreadPoolTaskScheduler();    
    }
}
```

```java
@RestController@RequestMapping("schedual")public class SchedualFutureRest {        
    @Autowired    
    private ThreadPoolTaskScheduler threadPoolTaskScheduler;        
    /**     
    * 在ScheduledFuture中有一个cancel可以停止定时任务。     
    */    
    private ScheduledFuture<?> future;        
    /**     
    * @Title: startCron     
    * @Description: 启动定时任务     
    * @Author yuankaiqiang     
    * @DateTime 2021-07-18 17:19:08     
    */    
    @RequestMapping("startCron")    
    public void startCron() {        
        future = threadPoolTaskScheduler.schedule(new Runnable() {           
            @Override            
            public void run() {                
                System.out.println("hello world！！！");            
            }            
            // 每隔一秒钟发送一次数据        
        }, new CronTrigger("0/1 * * * * *"));    
    }    
    /**     
    * @Title: getCronStatus     
    * @Description: 获取定时任务     
    * @Author yuankaiqiang     
    * @DateTime 2021-07-18 17:27:17     
    */    
    @RequestMapping("getCronStatus")    
    public boolean getCronStatus() {        
        return future.isDone();    
    }        
    /**     
    * @Title: stopCron     
    * @Description: 停止定时任务     
    * @Author yuankaiqiang     
    * @DateTime 2021-07-18 17:27:40     
    */    
    @RequestMapping("stopCron")    
    public boolean stopCron() {        
        return future.cancel(true);    
    }
}
```

**CompleteFuture**

​		一个Future类是显示的完成，而且能被用作一个完成等级，通过它的完成触发支持的依赖函数和行为。当两个或多个线程要执行完成或取消操作时，只有一个能够成功。

- 在Java8中，CompletableFuture提供了非常强大的Future的扩展功能，可以帮助我们简化异步编程的复杂性，并且提供了函数式编程的能力，可以通过回调的方式处理计算结果，也提供了转换和组合 CompletableFuture 的方法。
- 它可能代表一个明确完成的Future，也有可能代表一个完成阶段（ CompletionStage ），它支持在计算完成以后触发一些函数或执行某些动作。
- 它实现了Future和CompletionStage接口

**ForkJoinTask**

​		基于任务的抽象类，可以通过ForkJoinPool来执行。一个ForkJoinTask是类似于线程实体，但是相对于线程实体是轻量级的。大量的任务和子任务会被ForkJoinPool池中的真实线程挂起来，以某些使用限制为代价。

# 十二、JMM

[java内存模型](https://www.cnblogs.com/null-qige/p/9481900.html) [^2]

> 内存划分

​		JMM规定了内存主要划分为主内存和工作内存两种。此处的主内存和工作内存跟JVM内存划分（堆、栈、方法区）是在不同的层次上进行的，如果非要对应起来，主内存对应的是Java堆中的对象实例部分，工作内存对应的是栈中的部分区域，从更底层的来说，主内存对应的是硬件的物理内存，工作内存对应的是寄存器和高速缓存。

<img src="https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210902202541.png" alt="1102674-20180815143324915-2024156794"  />

**内存交互操作有8种，虚拟机实现必须保证每一个操作都是原子的，不可在分的（对于double和long类型的变量来说，load、store、read和write操作在某些平台上允许例外）**

- - lock   （锁定）：作用于主内存的变量，把一个变量标识为线程独占状态
  - unlock （解锁）：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定
  - read  （读取）：作用于主内存变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用
  - load   （载入）：作用于工作内存的变量，它把read操作从主存中变量放入工作内存中
  - use   （使用）：作用于工作内存中的变量，它把工作内存中的变量传输给执行引擎，每当虚拟机遇到一个需要使用到变量的值，就会使用到这个指令
  - assign （赋值）：作用于工作内存中的变量，它把一个从执行引擎中接受到的值放入工作内存的变量副本中
  - store  （存储）：作用于主内存中的变量，它把一个从工作内存中一个变量的值传送到主内存中，以便后续的write使用
  - write 　（写入）：作用于主内存中的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中**JMM对这八种指令的使用，制定了如下规则：**

- - 不允许read和load、store和write操作之一单独出现。即使用了read必须load，使用了store必须write
  - 不允许线程丢弃他最近的assign操作，即工作变量的数据改变了之后，必须告知主存
  - 不允许一个线程将没有assign的数据从工作内存同步回主内存
  - 一个新的变量必须在主内存中诞生，不允许工作内存直接使用一个未被初始化的变量。就是怼变量实施use、store操作之前，必须经过assign和load操作
  - 一个变量同一时间只有一个线程能对其进行lock。多次lock后，必须执行相同次数的unlock才能解锁
  - 如果对一个变量进行lock操作，会清空所有工作内存中此变量的值，在执行引擎使用这个变量前，必须重新load或assign操作初始化变量的值
  - 如果一个变量没有被lock，就不能对其进行unlock操作。也不能unlock一个被其他线程锁住的变量
  - 对一个变量进行unlock操作之前，必须把此变量同步回主内存

**JMM对这八种操作规则和对**[**volatile的一些特殊规则**](https://www.cnblogs.com/null-qige/p/8569131.html)**就能确定哪里操作是线程安全，哪些操作是线程不安全的了。但是这些规则实在复杂，很难在实践中直接分析。所以一般我们也不会通过上述规则进行分析。更多的时候，使用java的happen-before规则来进行分析。**

```java
public class JMMDemo {
	//加上volatile可以保证可见性
     private volatile static int num = 0;
//	private static int num = 0;
	
	public static void main(String[] args) {
		//A线程不知道值已经被修改过了，所以一直在执行
		new Thread(()-> {
			while(num==0) {     
			}
		},"A").start();
		
		try {
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
		num = 1;
		System.out.println(num);
	}
}
```

问题：程序不知道主内存中的值已经被修改过了

![QQ20200503-151203@2x](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210602095712.jpg)

# 十三、volatile

> 如何保证可见性

volatile保证了修饰的共享变量在**转换为汇编语言**时，会加上一个以`lock`为前缀的指令，当CPU发现这个指令时，立即会做两件事情：

1. 将当前内核中线程工作内存中该共享变量刷新到主存；
2. 通知其他内核里缓存的该共享变量内存地址无效；

> 如何保证有序性（禁止指令重排）

```java
r1=a;
r2=r1.x;
r3=r1.x;
//编译器则可能会进行优化，将r3=r1.x这条指令替换成r3=r2，这就是指令的重排
```

![22284259-e9c4f31fcb75c4fd](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210902202322.webp)

普通变量: 仅保证在该方法执行过程中所有依赖赋值结果的地方都能得到正确的结果，而不能保证变量赋值操作的顺序和代码中的顺序一致。

被volatile修饰的变量，会加一个`lock`**前缀的汇编指令**。若变量被修改后，**会立刻将变量由工作内存回写到主存中。那么意味了之前的操作已经执行完毕**: **Memory Barrier(内存屏障)**。

总结: 其实就是汇编之后得到的指令是会加一个`lock`**前缀的汇编指令**从而出现内存屏障.

1. 保证可见性

```java
//加上volatile可以保证可见性
private volatile static int num = 0;
```

2. 不保证原子性

```java
public class JMMDemo {
	//voatile不保证原子性
	private volatile static int num = 0;
	//synchronized可以保证
	public static void add() {
		num++;
	}
	public static void main(String[] args) {
		// 理论上num的结果应该为20000
		for (int i = 1; i <= 20; i++) {
			new Thread(()->{
				for (int j = 0; j < 1000; j++) {
					add();
				}
			}).start();
		}
		while(Thread.activeCount()>2) {  //  main  gcc
			Thread.yield();
		}
		System.out.println(Thread.currentThread().getName()+" "+num);
	}
}
```

**解决方案：**

1. 加上==lock==和==synchronized==可以保证可以保证原子性
2. 使用原子类，在java.util.concurrent.atomic下   这些类的底层都直接和操作系统挂钩，在内存中修改值。Unsafe类是一个很特殊的存在。

```java
public class JMMDemo {	
    //voatile不保证原子性	
    private volatile static AtomicInteger num = new AtomicInteger();	
    //synchronized可以保证	
    public static void add() {		
        num.getAndIncrement();	
    }	
    public static void main(String[] args) {		
        // 理论上num的结果应该为20000		
        for (int i = 1; i <= 20; i++) {			
            new Thread(()->{				
                for (int j = 0; j < 1000; j++) {					
                    add();				
                }			
            }).start();		
        }		
        while(Thread.activeCount()>2) {  
            //  main  gcc			
            Thread.yield();		
        }		
        System.out.println(Thread.currentThread().getName()+" "+num);	
    }
}
```

3. 禁止指令重排

   由于内存屏障，可以避免指令重拍的现象产生。

# 十四、锁（synchronized，Lock）

两个都是**可重入锁**（某个线程已经获得某个锁，可以再次获取锁而不会出现死锁）

|   区别   |                         synchronized                         |                     Lock                     |
| :------: | :----------------------------------------------------------: | :------------------------------------------: |
| 存在层次 |                          java关键字                          |                     接口                     |
| 锁的状态 |                           无法判断                           |                   可以判断                   |
|  释放锁  | 1、执行完同步代码，会释放锁<br>2、线程执行发生异常，会让线程释放锁 | 在 finallyt中必须释放锁,不然容易造成线程死锁 |
|   性能   |                          少量的同步                          |                   大量同步                   |
|   类型   | 悲观锁<br>可重入 不可中断 非公平<br>在JDK1.5之前是一个重量级锁，JDK1.6后对它进行优化，引入了偏向锁，轻量级锁，自旋锁 |  乐观锁<br>可重入 可判断 可公平（两者皆可）  |
| 底层实现 | synchronznized映射成字节码指令就是增加两个指令：==monitorenter、monitorexit==，<br/><br/>当一条线程执行时遇到monitorenter指令时，它会尝试去获得锁，如果获得锁，那么所计数器+1（为什么要加1，因为它是可重入锁，可根据这个琐计数器判断锁状态），如果没有获得锁，那么阻塞，<br/><br/>当它遇到一个monitoerexit时，琐计数器会-1，当计数器为0时，就释放锁<br/><br/>（tips：节码中出现的两个monitoerexit指令的原因是：一个正常执行-1，令一个异常时执行，这两个用goto的方式只执行一个）<br/> |          底层基于volatile和cas实现           |

**synchronized使用场景**

1. 修饰一个方法：被修饰的方法称为同步方法，其作用的范围是整个方法，**作用的对象是调用这个方法的对象**；

   ```java
   public synchronized void method(){   
       // todo
   }
   ```

2. 修饰代码块：被修饰的代码块称为同步语句块，其作用范围是大括号{}括起来的代码块，**作用的对象是调用这个代码块的对象**；

   ```java
   public  void method(){   
       synchronized(this)   
           synchronized(XX.class)
   }
   //synchronized(this)锁的是当前对象，当前有几个对象那么这个this就是有多份,这里的this只能锁同一个对象。
   //synchronized(XX.class)只要是这个类型的class这把锁就都有用
   ```

3. 修饰静态方法：其作用的范围是整个方法，**作用的对象是这个类的所有对象**；

   ```java
   public synchronized static void method() {   // todo}
   ```

4. 修饰一个类：其作用的范围是synchronized后面括号括起来的部分，**作用的对象是这个类的所有对象**；

   ```java
   class ClassName {    
       public void method() {       
           synchronized(ClassName.class) {          
               // todo       
           }    
       } 
   }
   ```

**Lock使用场景**

==lock()、tryLock()、tryLock(long time, TimeUnit unit)== 和 lockInterruptibly()都是用来获取锁的。

(1)lock()方法是平常使用得最多的一个方法，就是用来获取锁。如果锁已被其他线程获取，则进行等待。

(2)tryLock()方法是有返回值的，它表示用来尝试获取锁，如果获取成功，则返回true，如果获取失败(即锁已被其他线程获取)，则返回false，也就说这个方法无论如何都会立即返回。在拿不到锁时不会一直在那等待。

(3)tryLock(long time, TimeUnit unit)方法和tryLock()方法是类似的，只不过区别在于这个方法在拿不到锁时会等待一定的时间，在时间期限之内如果还拿不到锁，就返回false。如果如果一开始拿到锁或者在等待期间内拿到了锁，则返回true。

(4)lockInterruptibly()方法比较特殊，当通过这个方法去获取锁时，如果线程正在等待获取锁，则这个线程能够响应中断，即中断线程的等待状态。也就使说，当两个线程同时通过lock.lockInterruptibly()想获取某个锁时，假若此时线程A获取到了锁，而线程B只有在等待，那么对线程B调用threadB.interrupt()方法能够中断线程B的等待过程。

```java
public class Demo2 {	
    public static void main(String[] args) {		
        Phone2 phone = new Phone2();		
        new Thread(()->{			
            phone.sms();		
        },"A").start();		
        new Thread(()->{			
            phone.sms();		
        },"B").start();	
    }}
class Phone2 {		
    Lock lock = new ReentrantLock();				
    public void sms() {		
        //  lock锁必须配对  不然会死锁		
        lock.lock();		
        try {			
            System.out.println(Thread.currentThread().getName()+"sms");			
            call();		
        } catch(Exception e) {			
            e.getMessage();		
        } finally {			
            lock.unlock();		
        }	
    }	
    public void call() {		
        lock.lock();		
        try {			
            System.out.println(Thread.currentThread().getName()+"call");		
        } catch(Exception e) {			
            e.getMessage();		
        } finally {			
            lock.unlock();		
        }	
    }
}
```

# 十五、死锁

> 什么是死锁？

​		多个进程同时占有对方需要的资源而同时请求对方的资源，而它们在得到请求之前不会释放所占有的资源，那么就会导致死锁的发生，也就是进程不能实现同步。

> 产生死锁的原因

​		线程将自己锁住。为了保证线程之间的同步和互斥，我们往往需要给其加锁，有时候，线程申请了锁资源，还没有等待释放，又一次申请这把锁，结果就是挂起等待这把锁的释放，但是这把锁是被自己拿着，所以就会永远挂起等待，就造成了死锁。

​		竞争资源。当系统中供多个进程共享的资源如打印机、公用队列等，其数目不足以满足进程的需要时，会引起诸进程的竞争而产生死锁。

​		进程间推进顺序非法。进程在运行过程中，请求和释放资源的顺序不当，也同样会导致产生进程死锁。

> 死锁条件

1. 互斥条件（Mutual exclusion）：某种资源一次只允许一个进程访问，即该资源一旦分配给某个进程，其他进程就不能再访问，直到该进程访问结束。

2. 占有且等待（Hold and wait）：一个进程本身占有资源（一种或多种），同时还有资源未得到满足，正在等待其他进程释放该资源。

3. 不可抢占（No pre-emption）：别人已经占有了某项资源，你不能因为自己也需要该资源，就去把别人的资源抢过来。

4. 循环等待（Circular wait）： 存在一个进程链，使得每个进程都占有下一个进程所需的至少一种资源。

> 预防死锁

1. 采用资源静态分配策略，破坏"部分分配"条件；
2. 允许进程剥夺使用其他进程占有的资源，从而破坏"不可剥夺"条件；
3. 采用资源有序分配法，破坏"环路"条件。

> 查看死锁

第一步：使用jps -l定位进程号

第二步：jstack 进程号找到死锁问题

# 十六、三种常用的阻塞方式

参考：[^3] [^4] [^5] [^6]

https://blog.csdn.net/kangkanglou/article/details/82221301

https://www.cnblogs.com/suixing123/p/13861657.html

https://blog.csdn.net/u013332124/article/details/84647915

https://blog.csdn.net/asdasdasd123123123/article/details/107814280

三种线程阻塞方法

> LockSupport

==**可以指定线程，唤醒指定的线程**==

1. void park()：阻塞当前线程，如果调用unpark方法或者当前线程被中断，从能从park()方法中返回

2. void park(Object blocker)：功能同方法1，入参增加一个Object对象，用来记录导致线程阻塞的阻塞对象，方便进行问题排查；

3. void parkNanos(long nanos)：阻塞当前线程，最长不超过nanos纳秒，增加了超时返回的特性；

4. void parkNanos(Object blocker, long nanos)：功能同方法3，入参增加一个Object对象，用来记录导致线程阻塞的阻塞对象，方便进行问题排查；

5. void parkUntil(long deadline)：阻塞当前线程，知道deadline；

6. void parkUntil(Object blocker, long deadline)：功能同方法5，入参增加一个Object对象，用来记录导致线程阻塞的阻塞对象，方便进行问题排查；
7. void unpark(Thread thread): ==唤醒==处于阻塞状态的指定线程

|            |                        Object.wait()                         |                  Thread.sleep()                   |       LockSupport.park()       |
| :--------: | :----------------------------------------------------------: | :-----------------------------------------------: | :----------------------------: |
|    同步    | 只能在同步上下文中调用wait方法，否则或抛出IllegalMonitorStateException异常 |          不需要在同步方法或同步块中调用           | 不需要在同步方法或同步块中调用 |
|  作用对象  |           wait方法定义在Object类中，作用于对象本身           | sleep方法定义在java.lang.Thread中，作用于当前线程 |         作用于当前线程         |
| 释放锁资源 |                              是                              |                        否                         |               否               |
|  唤醒条件  |        其他线程调用对象的notify()或者notifyAll()方法         |           超时或者调用interrupt()方法体           |             unpark             |
|  方法属性  |                        wait是实例方法                        |                  sleep是静态方法                  |          park静态方法          |



![5bff9535e4b04dd2799a6ae8](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210609152320.png)

![1](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210609154247.png)

```java
public static void main(String[] args) throws InterruptedException {		
    Object lock = new Object();		
    synchronized (lock) {			
        try {				
            System.out.println("线程运行");				
            lock.wait(5000);				
            System.out.println("线程继续运行");			
        } catch (Exception ex) {			
        }		
    }		
    Thread thread1 = new Thread(new Runnable() {			
        @Override			
        public void run() {				
            System.out.println("线程1运行");				
            // 等待获取许可				
            LockSupport.park();				
            // 输出thread over.true				
            System.out.println("线程1继续运行");			
        }		
    });		
    Thread thread2 = new Thread(new Runnable() {			
        @Override			
        public void run() {				
            System.out.println("线程2运行");				
            // 5秒钟后获取许可				
            LockSupport.parkNanos(5_000_000_000L);				
            // 输出thread over.true				
            System.out.println("线程2继续运行");			
        }		
    });		
    thread1.start();		
    thread2.start();		
    Thread.sleep(2000);		
    LockSupport.unpark(thread1);	
}
```

# 十七、高并发三大特性

**原子性**：即一个操作或者多个操作，要么全部执行并且不被打断，要么就都不执行。

**可见性**：当一个线程修改了共享变量的值，其他线程会马上知道这个修改。当其他线程要读取这个变量的时候，最终会去**内存**中读取，而不是从缓存中读取。

**有序性**：虚拟机在进行代码编译时，对于那些改变顺序之后不会对最终结果造成影响的代码，虚拟机不一定会按照我们写的代码的顺序来执行，有可能将他们重排序。实际上，

# 十八、线程五种状态

**新建**

当用new操作符创建一个线程时。此时程序还没有开始运行线程中的代码。

**就绪**

一个新创建的线程并不自动开始运行，要执行线程，必须调用线程的start()方法。当线程对象调用start()方法即启动了线程，start()方法创建线程运行的系统资源，并调度线程运行run()方法。当start()方法返回后，线程就处于就绪状态。

处于就绪状态的线程并不一定立即运行run()方法，线程还必须同其他线程竞争CPU时间，只有获得CPU时间才可以运行线程。因为在单CPU的计算机系统中，不可能同时运行多个线程，一个时刻仅有一个线程处于运行状态。因此此时可能有多个线程处于就绪状态。对多个处于就绪状态的线程是由[**Java**](http://lib.csdn.net/base/java)运行时系统的线程调度程序来调度的。

**运行**

当线程获得CPU时间后，它才进入运行状态，真正开始执行run()方法。

**阻塞**

线程运行过程中，可能由于各种原因进入阻塞状态：

①线程通过调用sleep方法进入睡眠状态；

②线程调用一个在I/O上被阻塞的操作，即该操作在输入输出操作完成之前不会返回到它的调用者；

③线程试图得到一个锁，而该锁正被其他线程持有；

④线程在等待某个触发条件；

所谓阻塞状态是正在运行的线程没有运行结束，暂时让出CPU，这时其他处于就绪状态的线程就可以获得CPU时间，进入运行状态。

------

堵塞状态是前述四种状态中最有趣的，值得我们作进一步的探讨。线程被堵塞可能是由下述五方面的原因造成的：

(1) 调用sleep(毫秒数)，使线程进入"睡眠"状态。在规定的时间内，这个线程是不会运行的。

(2) 用suspend()暂停了线程的执行。除非线程收到resume()消息，否则不会返回"可运行"状态。

(3) 用wait()暂停了线程的执行。除非线程收到nofify()或者notifyAll()消息，否则不会变成"可运行"（是的，这看起来同原因2非常相象，但有一个明显的区别是我们马上要揭示的）。

(4) 线程正在等候一些IO（输入输出）操作完成。

(5) 线程试图调用另一个对象的"同步"方法，但那个对象处于锁定状态，暂时无法使用。

**死亡**

有两个原因会导致线程死亡：

①run方法正常退出而自然死亡；

②一个未捕获的异常终止了run方法而使线程猝死；

为了确定线程在当前是否存活着（就是要么是可运行的，要么是被阻塞了），需要使用isAlive方法，如果是可运行或被阻塞，这个方法返回true；如果线程仍旧是new状态且不是可运行的，或者线程死亡了，则返回false。

# 十九、ThreadLocal

​		ThreadLocal叫做线程变量，意思是ThreadLocal中填充的变量属于当前线程，该变量对其他线程而言是隔离的，也就是说该变量是当前线程独有的变量。ThreadLocal为变量在每个线程中都创建了一个副本，那么每个线程可以访问自己内部的副本变量。

![20201217201331591](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/202109271911862.png)

***使用场景***

- **每个线程需要有自己单独的实例**
- **实例需要在多个方法中共享，但不希望被多线程共享**

|   区别   |                         ThreadLocal                          |                       Synchonized                        |
| :------: | :----------------------------------------------------------: | :------------------------------------------------------: |
|    -     |                       线程间的数据隔离                       |                     线程间的数据共享                     |
|    -     | 为每一个线程都提供了变量的副本，使得每个线程在某一时间访问到的并不是同一个对象 | 利用锁的机制，使变量或代码块在某一时该只能被一个线程访问 |
| 使用场景 |         存储用户Session、数据库连接，处理数据库事务          |                    并发情况下同步问题                    |

# 二十、其它

## 1. 什么是线程安全

**多个线程同一时刻对同一个全局变量(同一份资源)做写操作(读操作不会涉及线程安全)时**，如果跟我们预期的结果一样，我们就称之为线程安全，反之，线程不安全。

## 2. 实际开发中使程序休眠使用TimeUnit而不是sleep，TimeUnit将使整个程序直接休眠

## 3. 线程创建四种方式[^1]

- 通过继承Thread类实现，多个线程之间无法共享该线程类的实例变量。
- 实现Runnable接口，较继承Thread类，避免继承的局限性，适合资源共享。
- 使用Callable和Future创建线程，方法中可以有返回值，并且抛出异常。
- 创建线程池实现，线程池提供了一个线程队列，队列中保存所有等待状态的线程，避免创建与销毁额外开销，提高了响应速度。

|      |                继承Thread                |                       实现Runnable接口                       |
| ---- | :--------------------------------------: | :----------------------------------------------------------: |
| 好处 | 可以直接使用Thread类中的方法，代码简单。 | 即使自己定义的线程类有了父类也没关系，因为有了父类也可以实现接口，而且接口是可以多实现的。 |
| 弊端 |   如果已经有了父类，就不能用这种方法。   | 不能直接使用Thread中的方法需要先获取到线程对象后，才能得到Thread的方法，代码复杂。 |

## 4. 进程与线程的区别

| **区别** |                           **进程**                           |                           **线程**                           |
| :------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| 根本区别 |                    作为资源分配的最小单位                    |                 作为资源调度和执行的最小单位                 |
|  开 销   | 每个进程都有独立的代码和数据空间(进程上下文)， 进程间的切换会有较大的开销。 | 线程可以看成时轻量级的进程， 同一类线程共享代码和数据空间， 每个线程有独立的运行栈和程序计数器(PC)， 线程切换的开销小。 |
| 所处环境 |             在操作系统中能同时运行多个任务(程序)             |             在同一应用程序中有多个顺序流同时执行             |
| 分配内存 |        系统在运行的时候会为每个进程分配不同的内存区域        | 除了CPU之外， 不会为线程分配内存（线程所使用的资源是它所属的进程的资源） ， 线程组只能共享资源 |
| 包含关系 | 没有线程的进程是可以被看作单线程的， 如果一个进程内拥有多个线程， 则执行过程不是一条线的， 而是多条线（线程） 共同完成的。 | 线程是进程的一部分， 所以线程有的时候被称为是轻权进程或者轻量级进程。 |

## 5. 什么是乐观锁，什么是悲观锁？

|   区别   |                            乐观锁                            |                            悲观锁                            |
| :------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|   概念   | 乐观锁在操作数据时非常乐观，认为别人不会同时修改数据。因此乐观锁不会上锁，只是在执行更新的时候判断一下在此期间别人是否修改了数据：如果别人修改了数据则放弃操作，否则执行操作。 | 悲观锁在操作数据时比较悲观，认为别人会同时修改数据。因此操作数据时直接把数据锁住，直到操作完成后才会释放锁；上锁期间其他人不能修改数据。 |
| 实现方式 |                 **CAS机制**和**版本号机制**                  | **加锁**，加锁既可以是对代码块加锁（如Java的synchronized关键字），也可以是对数据加锁（如MySQL中的排它锁）。 |

## 6. 同步与异步，阻塞与非阻塞

同步和异步关注的是**消息通信机制**

阻塞和非阻塞关注的是**程序在等待调用结果（消息，返回值）时的状态.**

- **同步**：指调用者发送一个请求，需要等待返回，然后才能够发送下一个请求，有个等待过程；
- **异步**：指调用者发送一个请求，不需要等待返回，随时可以再发送下一个请求，即不需要等待（可以放入消息队列中）。 
  - **区别**：一个需要等待，一个不需要等待，在部分情况下，我们的项目开发中都会优先选择不需要等待的异步交互方式。
- **阻塞**：阻塞调用是指调用结果返回之前，当前线程会被挂起。调用线程只有在得到结果之后才会返回。
- **非阻塞**：非阻塞调用指在不能立刻得到结果之前，该调用不会阻塞当前线程。
- **阻塞与死锁的区别**：
  - 阻塞是由于资源不足引起的排队等待现象。
  - 死锁是由于两个对象在拥有一份资源的情况下申请另一份资源，而另一份资源恰好又是这两对象正持有的，导致两对象无法完成操作，且所持资源无法释放。

## 7. 公平锁/非公平锁、可重入锁、独享锁/共享锁、互斥锁/读写锁、乐观锁/悲观锁、分段锁、偏向锁/轻量级锁/重量级锁、自旋锁

> 公平锁/非公平锁

- 公平锁是指多个线程按照申请锁的顺序来获取锁。
- 非公平锁是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。有可能，会造成优先级反转或者饥饿现象。
- 对于ReentrantLock而言，通过构造函数指定该锁是否是公平锁，默认是非公平锁。非公平锁的优点在于吞吐量比公平锁大。
- 对于Synchronized而言，也是一种非公平锁。由于其并不像ReentrantLock是通过AQS的来实现线程调度，所以并没有任何办法使其变成公平锁。

> 可重入锁

- 可重入锁又名递归锁，是指在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁。
- 说的有点抽象，下面会有一个代码的示例。
- 对于Java ReentrantLock而言, 他的名字就可以看出是一个可重入锁，其名字是Re entrant Lock重新进入锁。
- 对于Synchronized而言,也是一个可重入锁。可重入锁的一个好处是可一定程度避免死锁。

```java
synchronized void setA() throws Exception{    Thread.sleep(1000);    setB();}synchronized void setB() throws Exception{    Thread.sleep(1000);}
```

> 独享锁/共享锁

- 独享锁是指该锁一次只能被一个线程所持有。
- 共享锁是指该锁可被多个线程所持有。
- 对于Java ReentrantLock而言，其是独享锁。但是对于Lock的另一个实现类ReadWriteLock，其读锁是共享锁，其写锁是独享锁。
- 读锁的共享锁可保证并发读是非常高效的，读写，写读 ，写写的过程是互斥的。
- 独享锁与共享锁也是通过AQS来实现的，通过实现不同的方法，来实现独享或者共享。
- 对于Synchronized而言，当然是独享锁。

>互斥锁/读写锁

- 上面讲的独享锁/共享锁就是一种广义的说法，互斥锁/读写锁就是具体的实现。
- 互斥锁在Java中的具体实现就是ReentrantLock
- 读写锁在Java中的具体实现就是ReadWriteLock

>乐观锁/悲观锁

- 乐观锁与悲观锁不是指具体的什么类型的锁，而是指看待并发同步的角度。
- 悲观锁认为对于同一个数据的并发操作，一定是会发生修改的，哪怕没有修改，也会认为修改。因此对于同一个数据的并发操作，悲观锁采取加锁的形式。悲观的认为，不加锁的并发操作一定会出问题。
- 乐观锁则认为对于同一个数据的并发操作，是不会发生修改的。在更新数据的时候，会采用尝试更新，不断重新的方式更新数据。乐观的认为，不加锁的并发操作是没有事情的。
- 从上面的描述我们可以看出，悲观锁适合写操作非常多的场景，乐观锁适合读操作非常多的场景，不加锁会带来大量的性能提升。
- 悲观锁在Java中的使用，就是利用各种锁。
- 乐观锁在Java中的使用，是无锁编程，常常采用的是CAS算法，典型的例子就是原子类，通过CAS自旋实现原子操作的更新。

>分段锁

- 分段锁其实是一种锁的设计，并不是具体的一种锁，对于ConcurrentHashMap而言，其并发的实现就是通过分段锁的形式来实现高效的并发操作。
- 我们以ConcurrentHashMap来说一下分段锁的含义以及设计思想，ConcurrentHashMap中的分段锁称为Segment，它即类似于HashMap(JDK7与JDK8中HashMap的实现)的结构，即内部拥有一个Entry数组，数组中的每个元素又是一个链表;同时又是一个ReentrantLock(Segment继承了ReentrantLock)。
- 当需要put元素的时候，并不是对整个hashmap进行加锁，而是先通过hashcode来知道他要放在那一个分段中，然后对这个分段进行加锁，所以当多线程put的时候，只要不是放在一个分段中，就实现了真正的并行的插入。
- 但是，在统计size的时候，可就是获取hashmap全局信息的时候，就需要获取所有的分段锁才能统计。
- 分段锁的设计目的是细化锁的粒度，当操作不需要更新整个数组的时候，就仅仅针对数组中的一项进行加锁操作。

>偏向锁/轻量级锁/重量级锁

- 这三种锁是指锁的状态，并且是针对Synchronized。在Java 5通过引入锁升级的机制来实现高效Synchronized。这三种锁的状态是通过对象监视器在对象头中的字段来表明的。
- 偏向锁是指一段同步代码一直被一个线程所访问，那么该线程会自动获取锁。降低获取锁的代价。
- 轻量级锁是指当锁是偏向锁的时候，被另一个线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获取锁，不会阻塞，提高性能。
- 重量级锁是指当锁为轻量级锁的时候，另一个线程虽然是自旋，但自旋不会一直持续下去，当自旋一定次数的时候，还没有获取到锁，就会进入阻塞，该锁膨胀为重量级锁。重量级锁会让其他申请的线程进入阻塞，性能降低。

>自旋锁

- 在Java中，自旋锁是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU。
- 线程自旋和适应性自旋  我们知道，java线程其实是映射在内核之上的，线程的挂起和恢复会极大的影响开销。 并且jdk官方人员发现，很多线程在等待锁的时候，在很短的一段时间就获得了锁，所以它们在线程等待的时候，并不需要把线程挂起，而是让他无目的的循环，一般设置10次。 这样就避免了线程切换的开销，极大的提升了性能。  而适应性自旋，是赋予了自旋一种学习能力，它并不固定自旋10次一下。 他可以根据它前面线程的自旋情况，从而调整它的自旋，甚至是不经过自旋而直接挂起。

## 8. 三个线程按顺序打印A、B、C打印

Synchronized锁、使用线程通信wait()、notifyAll()

调用wait方法的线程，当前持有锁的该线程等待，直至该对象的另一个持锁线程调用notify/notifyAll操作

```java
public class ABCThread  extends Thread{
    String [] ABC=new String[]{"A","B","C"};
    //当前线程编号
    private Integer index;
    //线程间通信对象
    private Opt opt;
    //统计次数
    int number=0;

    public ABCThread(Integer index, Opt opt) {
        this.index = index;
        this.opt = opt;
    }

    @Override
    public void run() {
        while (true) {
            //通信锁的对象
            synchronized (opt) {
                //循环执行 判断是否是当前线程执行（当前线程编号和opt记录的线程编号一致则是要执行的线程）
                while (opt.getNextIndex() != index) {
                    try {
                        //不是当前线程继续等待
                        opt.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.print(ABC[index] + " ");
                //统计次数加一
                number++;
                //设置下一个执行的线程编号A（0）、B（1）、C（2）；
                opt.setNextIndex((index+1)%3);
                //通知其他两个线程
                opt.notifyAll();
                //当前执行到10次，执行结束
                if (number>9) break;

            }
        }
    }

    public static void main(String[] args) {
        Opt opt=new Opt();
        //设置的第一个执行线程编号
        opt.setNextIndex(0);
        new ABCThread(0,opt).start();//A
        new ABCThread(1,opt).start();;//B
        new ABCThread(2,opt).start();//C
    }
}
```

## 9. CAS机制

CAS机制：（Compare and set）比较和替换

　　**使用一个==期望值==来和==当前变量的值==进行比较，如果当前的变量值与我们期望的值相等，就用一个==新的值==来更新当前变量的值。**
		CAS有**三个操作数**：**内存值V、旧的预期值A、要修改的值B**，当且仅当预期值A和内存值V相同时（条件），将内存值修改为B并返回true，否则条件不符合返回false。条件不符合说明该变量已经被其它线程更新了。

1. 在内存地址V当中，存储着值为10的变量

2. 此时线程1想要把变量的值增加1。对线程1来说，旧的预期值A=10，要修改的新值B=11。

3. 在线程1要提交更新之前，另一个线程2抢先一步，把内存地址V中的变量值率先更新成了11。

4. 线程1开始提交更新，首先进行A和地址V的实际值比较（Compare），发现A不等于V的实际值，提交失败。

5. 线程1重新获取内存地址V的当前值，并重新计算想要修改的新值。此时对线程1来说，A=11，B=12。这个重新尝试的过程被称为自旋。

6. 这一次比较幸运，没有其他线程改变地址V的值。线程1进行Compare，发现A和地址V的实际值是相等的。

7. 线程1进行SWAP，把地址V的值替换为B，也就是12。

Synchronized属于悲观锁，悲观地认为程序中的并发情况严重，所以严防死守。CAS属于乐观锁，乐观地认为程序中的并发情况不那么严重，所以让线程不断去尝试更新。

|               优点               |                             缺点                             |
| :------------------------------: | :----------------------------------------------------------: |
| 乐观锁避免了悲观锁独占对象的现象 | CPU可能开销较大：在并发量比较高的情况下，如果许多线程反复尝试更新某一个变量，却又一直更新不成功，循环往复，会给CPU带来很大的压力 |
|          提高了并发性能          | 不能保证代码块的原子性：CAS机制所保证的只是一个变量的原子性操作，而不能保证整个代码块的原子性。比如需要保证3个变量共同进行原子性的更新，就不得不使用悲观锁了。 |
|                                  | **ABA问题**：假如内存值原来是A，后来被一条线程改为B，最后又被改成了A，则CAS认为此内存值并没有发生改变，但实际上是有被其他线程改过的。<br>**解决方式**：给值加一个修改版本号，每次值变化，都会修改它版本号，CAS操作时都对比此版本号。（**AtomicStampedReference**/**AtomicMarkableReference**） |

**AtomicStampedReference** 本质是有一个int 值作为版本号，每次更改前先取到这个int值的版本号，等到修改的时候，比较当前版本号与当前线程持有的版本号是否一致，如果一致，则进行修改，并将版本号+1（当然加多少或减多少都是可以自己定义的），在zookeeper中保持数据的一致性也是用的这种方式；

**AtomicMarkableReference**则是将一个boolean值作是否有更改的标记，本质就是它的版本号只有两个，true和false，修改的时候在这两个版本号之间来回切换，这样做并不能解决ABA的问题，只是会降低ABA问题发生的几率而已；

## 10. Java中如何正确而优雅的终止运行中的线程

线程终止有两种情况：

1、线程的任务执行完成

2、线程在执行任务过程中发生异常

这两者属于线程自行终止，如何让线程 A 把线程 B 终止呢？

**1、使用stop()方法，已被弃用。**

原因是：stop()是立即终止，会导致一些数据被到处理一部分就会被终止，而用户并不知道哪些数据被处理，哪些没有被处理，产生了不完整的“残疾”数据，不符合完整性，所以被废弃。So, forget it!

**2、使用volatile标志位**

首先，实现一个Runnable接口，在其中定义volatile标志位，在run()方法中使用标志位控制程序运行。

```java
public class MyRunnable implements Runnable {
 
	//定义退出标志，true会一直执行，false会退出循环
	//使用volatile目的是保证可见性，一处修改了标志，处处都要去主存读取新的值，而不是使用缓存
	public volatile boolean flag = true;
 
	public void run() {
		System.out.println("第" + Thread.currentThread().getName() + "个线程创建");
		
		try {
			Thread.sleep(1000L);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		//退出标志生效位置
		while (flag) {
		}
		System.out.println("第" + Thread.currentThread().getName() + "个线程终止");
	}
}
```

然后，在main()方法中创建线程，在合适的时候，修改标志位，终止运行中的线程。

```java
public class TreadTest {
	public static void main(String[] arg) throws InterruptedException {
		MyRunnable runnable = new MyRunnable();
		
		//创建3个线程
		for (int i = 1; i <= 3; i++) {
			Thread thread = new Thread(runnable, i + "");
			thread.start();
		}
		//线程休眠
		Thread.sleep(2000L);
		System.out.println("——————————————————————————");
		//修改退出标志，使线程终止
		runnable.flag = false;	
	}
}
```

## 11. 使用interrupt()中断的方式

​		注意使用interrupt()方法中断正在运行中的线程只会修改中断状态位，可以通过isInterrupted()判断。如果使用interrupt()方法中断阻塞中的线程，那么就会抛出InterruptedException异常，可以通过catch捕获异常，然后进行处理后终止线程。有些情况，我们不能判断线程的状态，所以使用interrupt()方法时一定要慎重考虑。

```java
public class MyRunnable2 implements Runnable{
	
	public void run(){
		System.out.println(Thread.currentThread().getName() + " 线程创建");
		try {
			Thread.sleep(1000L);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		//运行时的线程被中断后，只会修改中断标记，不会抛出异常
		while(Thread.currentThread().isInterrupted()){
			
		}
		System.out.println(Thread.currentThread().getName() + " 线程被中断");
	}
 
}
```

```java
public class TreadTest {
	public static void main(String[] arg) throws InterruptedException {
		Runnable runnable = new MyRunnable2();
		
		//创建3个线程
		Thread thread1 = new Thread(runnable, "1");
		Thread thread2 = new Thread(runnable, "2");
		Thread thread3 = new Thread(runnable, "3");
		thread1.start();
		thread2.start();
		thread3.start();
		
		//线程休眠
		Thread.sleep(2000L);
		
		//修改退出标志，使线程终止
		thread1.interrupt();
		thread2.interrupt();
		thread3.interrupt();
	}
}
```



# 参考网址

[^1]: https://www.cnblogs.com/wxw7blog/p/7727510.html
[^2]: https://www.cnblogs.com/null-qige/p/9481900.html
[^3]: https://blog.csdn.net/kangkanglou/article/details/82221301
[^4]: https://www.cnblogs.com/suixing123/p/13861657.html
[^5]: https://blog.csdn.net/u013332124/article/details/84647915
[^6]: https://blog.csdn.net/asdasdasd123123123/article/details/107814280

函数式接口







