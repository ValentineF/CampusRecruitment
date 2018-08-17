# Java多线程基础
## Java中多线程的实现方式
- 继承Thread类
     ```
    public class MyThread extends Thread {
        public MyThread(){
        }

        @Override
        public void run() {
            System.out.println(this.getName());
        }
    }

    public class Client {
        public static void main(String[] args) throws ExecutionException, InterruptedException {
            //1.继承Thread类
            MyThread myThread = new MyThread();
            myThread.start();
        }
    }
    ```
    1. 新建MyThread类，继承Thread父类，并重写父类的Run()方法
    2. 主程序中创建MyThread类的对象
        - 注，初始化的时候会调用父类构造函数，调用其中的init方法，进行初始化，比如设置优先级，是否守护进程之类的
    3. 使用Start方法启动该线程
        - Start方法源码简介（主要就是调用了Start0这个本地方法）
        - 其中start0为本地方法，其实现和平台有关，调用了pthread_create，POSIX中的API创建一个线程，然后调用run方法
        - ![调用图](https://www.linuxidc.com/upload/2016_03/160308082391524.jpg)
    4. （如果直接调用run方法吗，那就和普通的方法调用一样）
    5. Thread是实现了Runnable接口的
- 实现Runnable接口
    ```
    public class MyRunnable implements Runnable {
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName());
        }
    }

    public class Client {
        public static void main(String[] args) throws ExecutionException, InterruptedException {
            //2.实现Runnable接口
            Thread myRunnable = new Thread(new MyRunnable());
            myRunnable.start();
        }
    }

    ```
    1. 实现接口，并实现其run()方法（注，Runnable接口源码里其实只有一个abstract run）
    ```
        @FunctionalInterface
        public interface Runnable {
            /**
            * When an object implementing interface <code>Runnable</code> is used
            * to create a thread, starting the thread causes the object's
            * <code>run</code> method to be called in that separately executing
            * thread.
            * <p>
            * The general contract of the method <code>run</code> is that it may
            * take any action whatsoever.
            *
            * @see     java.lang.Thread#run()
            */
            public abstract void run();
        }

    ```
    2. 创建Thread对象，并将实现Runnable接口的类作为参数传入，并调用start方法
- 上面都是无返回值，下面将介绍两个有返回值的

- 使用Callable接口和FutureTask接口的组合（FutureTask是Future实现）
    ```
    public class MyCallable implements Callable<String> {
        @Override
        public String call() throws Exception {
            return "MyCallable_Test";
        }
    }
    public class Client {
        public static void main(String[] args) throws ExecutionException, InterruptedException {
            //3.1 Callable 和 FutureTask组合
            MyCallable myCallable = new MyCallable();
            FutureTask<String> futureTask = new FutureTask<>(myCallable);
            //注意这里的初始化函数，他是作为runnable为参数
            new Thread(futureTask).start();
            System.out.println(futureTask.get()+"_FutureTask");
        }
    }
    ```
    1. 实现Callable接口中的call方法
    2. 创建FutureTask对象，将Callable对象作为值传入构造函数中
    3. 启动线程
    4. 使用FutureTask的get方法获取返回值
    5. ![Future](https://images2015.cnblogs.com/blog/1025005/201610/1025005-20161030180319421-1644150953.png)
    
- 4.使用Callable接口和Future接口的组合，借助线程池实现
    ```
    public class MyCallable implements Callable<String> {
        @Override
        public String call() throws Exception {
            return "MyCallable_Test";
        }
    }
    public class Client {
        public static void main(String[] args) throws ExecutionException, InterruptedException {
            //3.2 Callable 和 Future组合，借助线程池
            ExecutorService threadPool = Executors.newCachedThreadPool();
            MyCallable callableWithFuture = new MyCallable();
            //提交当前任务进入线程池
            Future<String> future = threadPool.submit(callableWithFuture);
            threadPool.shutdown();
            System.out.println(future.get()+"_Future");
        }
    }
    ```
    1. 实现Callable接口中的call方法
    2. 创建线程池对象
    3. 将Callable实现类放入线程池中,并赋给Future对象
    4. 关闭线程池
    5. Future的get取结果
## Java的线程同步机制
- 使用synchronized关键字（主要有三种）
    1. 修饰实例方法，作用于当前实例加锁，进入同步代码前要获得当前实例的锁
    ```
    public synchronized void increase(){
        i++;
    }
    ```
    2. 修饰静态方法，作用于当前类对象加锁，进入同步代码前要获得当前类对象的锁
    ```
    public static synchronized void increase(){
        i++;
    }
    ```
    3. 修饰代码块，指定加锁对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁。
    ```
    //当前对象
    synchronized(this){
        for(int j=0;j<1000000;j++){
            i++;
        }
    }
    //类对象
    synchronized(AccountingSync.class){
        for(int j=0;j<1000000;j++){
            i++;
        }
    }
    ```
    4. 注：当有多个synchronized方法，当一个线程进入synchronizedA方法时，另外一个线程想进入synchronizedB方法会阻塞，因为第一个线程已经获取到了对象锁，当前其他非同步方法不受影响
    5. 使用synchronized的不足
        - 读写操作无法分离（影响效率）
        - 阻塞的时候会不断尝试获取锁，资源消耗大
        - 粒度大
        - 容易产生死锁（一个线程一直持有锁）
- Concurrent包中的Lock框架
![lock框架](https://github.com/ValentineF/NoteBook/blob/master/Picture/lock%E6%A1%86%E6%9E%B6.png?raw=true)
    ```
    public interface Lock {
        //加锁，如果锁已被其他线程获取，则进行等待
        void lock();
        //让在等待的锁可以相应中断
        void lockInterruptibly() throws InterruptedException;
        //试图去获取锁（立即返回结果）
        boolean tryLock();
        //增加时间限制（即未拿到锁的时候会进行等待，直到时间结束或者拿到锁，返回结果）
        boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
        //释放锁
        void unlock();
        Condition newCondition();
    }
    ```
    1. Lock(),一定要手动释放锁，故在finally块中加释放锁操作是比较正确的
        ```
        Lock lock = ...;
        lock.lock();
        try{
            //处理任务
        }catch(Exception ex){
            
        }finally{
            lock.unlock();   //释放锁
        }
        ```
    2. Trylock()
        ```
        Lock lock = ...;
        if(lock.tryLock()) {
            try{
                //处理任务
            }catch(Exception ex){
                
            }finally{
                lock.unlock();   //释放锁
            } 
        }else {
            //如果不能获取锁，则直接做其他事情
        }
        ```
    3. ReentrantLock(Lock接口的唯一实现类)-可重入锁
        ```
        public void insert(Thread thread) {
            Lock lock = new ReentrantLock();    //注意这个地方，有个坑（局部变量？）
            lock.lock();
            try {
                System.out.println(thread.getName() + "得到了锁");
                for (int i = 0; i < 5; i++) {
                    arrayList.add(i);
                }
            } catch (Exception e) {
                // TODO: handle exception
            } finally {
                System.out.println(thread.getName() + "释放了锁");
                lock.unlock();
            }
        }
        ```
    4. ReadWriteLock接口
        ```
        public interface ReadWriteLock {
            //获取读锁     
            Lock readLock();
            //获取写锁
            Lock writeLock();
        }
        ```
    5. ReentrantReadWriteLock类，实现ReadWriteLock接口
        ```
        private ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
        ...
        //单独加读锁
        public void get2(Thread thread) {
            rwl.readLock().lock();
            try {
                long start = System.currentTimeMillis();

                while(System.currentTimeMillis() - start <= 1) {
                    System.out.println(thread.getName()+"正在进行读操作");
                }
                System.out.println(thread.getName()+"读操作完毕");
            } finally {
                rwl.readLock().unlock();
            }
        }
        ```
## 线程状态及转换
![线程状态及转换图](https://github.com/ValentineF/NoteBook/blob/master/Picture/%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81.jpg?raw=true)

    1. Object.wait()
        让当前线程处于WAITING(放弃锁),前提是当前线程必须拥有此对象的monitor（即锁）,
    2. Object.notify()
        唤醒一个WAITING中的线程（如果有多个就随机唤醒一个）
    3. Object.notifyAll()
        唤醒所有WAITING中的线程，（线程被唤醒后不会立即进入runnable状态，重新获得锁以后才会进入）
    4. Thread.sleep()
        让当前线程处于TIMED_WAITING ，让出CPU，但是继续持有锁
    5. Thread.join()
        当前线程让出CPU，让调用join()的对象先执行，可以加入时间参数
    6. Thread.yeild()
        表明当前线程乐于暂停执行
## 
## 参考资料
- [Java并发编程：Callable、Future和FutureTask](http://www.cnblogs.com/dolphin0520/p/3949310.html)
- [Java多线程干货系列（1）：Java多线程基础](http://www.importnew.com/21136.html)
- [Java多线程干货系列—（二）synchronized](http://tengj.top/2016/05/03/threadsynchronized2/)
- [Java并发编程：Lock](http://www.cnblogs.com/dolphin0520/p/3923167.html)
- [java 线程方法join的简单总结](https://www.cnblogs.com/lcplcpjava/p/6896904.html)