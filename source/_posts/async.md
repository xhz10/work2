---
title: 简单聊一下异步编程这件事
---
<a name="bqooO"></a>
## 前言
我之前闲来无事发现知乎上部分群体非常推崇rust，再加上我本质上并不是一个Java开发工程师而是一个Spring开发工程师，所以面对C、C++、C#、Go等群体，总觉得自己低人一等，于是在某个伸手不见六指的白天，我突然萌生了试着学一下rust的想法。于是有了以下的心路历程：<br />第一天：不错的包管理工具<br />第二天：舒服的编译器以及变量绑定的概念<br />第三天：优雅的语法以及有深度的所有权<br />第四天：吃瘪<br />第五天：吃瘪<br />第七天：去NM的老子不干了，什么SB东西(指的是我自己)，不学了不学了。<br />当然，学习一门编程语言就像是谈一场恋爱，很明显，我的爱情一开始就陷入了被动，最后无奈跟不上女朋友的思路最终成为了前男友。。。。<br />而我说这种感觉之所以像是谈恋爱是因为：虽然分手成为前任了，但是依旧在某个悠闲的午后或是寂寞的夜晚，抚摸起身边的键盘，总是忍不住点开那个教程网页(朋友圈)，幻想着它是否有新的人去爱，是否在没有自己的夜晚躲在被窝里蜷缩成一团，用力地抱紧玩偶在哭泣，哭泣我当初的轻言放弃、哭泣我不敢和它一起走入职业的下一个阶段。<br />这只是个开始，当你不经意间在B站刷到它的消息，刷到它的编译器、语法糖、性能对比的时候，依然控制不住自己的双手，点赞、收藏一气呵成。仿佛只要这么做了rust就永远陪在我的身边。到最后才发现，走不出“失恋”阴影的原来是我自己。<br />某一天的午后，我在rust的朋友圈发现了这样一句话“关于async和线程池的区别”。于是本篇文章的正文从现在才真正开始。
<a name="WDR2d"></a>
## 开始前的思考
<a name="QjbqY"></a>
### Javaer的思考
作为一名Spring开发者，我似乎对比其他语言的程序员天然缺少很多基础知识的磨砺，于是最初看见这句话我瞬间想到了SpringBoot中的Async注解。
```java
@Async
```
	如果我的记忆没错乱的话我记得这个注解的本质是没有任何配置的情况下SpringBoot会维护一个默认的会不断创建新线程的线程池，也就是说用它修饰的方法会**每次都开一个线程去执行异步程序**。<br />那么Async的概念和线程池有什么区别？不都是线程池嘛？  以上是我的一瞬间的想法。
<a name="gIE85"></a>
### Ruster的思考
当然随着我阅读的深入我发现rust的Async是一个异步编程的概念，**异步编程可以看做是一个并发编程模型，它的好处是允许我们同时并发运行大量的任务，但是只需要一个或者几个很少的线程或CPU的核心**。<br />怎么样，是不是觉得隐隐约约有些眼熟，这个概念怎么和“**协程**”这么类似啊。<br />而rust是利用async和await这样的语法实现的上述功能，看起来和Javascript比较相似。
<a name="DJUf4"></a>
### JavaScripter的思考
rust我不懂，但是Javascript我可以问同事啊！<br />于是我采访了身边的前端大佬，经过一番攀谈交心以及**不可以明说**的特殊交易后，我得出了以下结论：

1. Javascript执行引擎是**单线程**的
2. Javascript通过维护一个异步任务队列来做所谓的“**异步**”的处理
<a name="mh41g"></a>
#### 异步执行流程
我们来看一下下面的流程<br />![jsel](http://img.xuhongzhuo.com/speaive/image/show/ab110ffe512b453c832e24a1015f1a3d)<br />当然有懂JS的可能会说：“我们JS随着版本和浏览器的变化，执行流程不一定是你这种形式的”！还有的大佬会说：“JS的异步任务有很多种分为**宏任务**、**微任务**等，不同的任务流程调度的优先级也不一样的”！<br />嗯嗯，知道了，大佬们息怒。目前我通过一个简单的流程图介绍了一下在JavaScript中所谓异步的流程，因为网友都说我前女友Rust和JS在这方面比较像，我就想着先尝试攻略一下JS再接着探索rust这个前女友。<br />回到前女友rust这，我们已经通过攻略别人的女朋友JS了解到前女友rust的执行思想了，下一步则是稍微的深入看一下rust是怎么做的。
<a name="JbkZI"></a>
## 终于到正文了
我现女友是Java，本着好男友的角度来看我不应该和rust有太多瓜葛，所以即使是分析原理我们也只是谈论思想，具体的实现我们还是尽量用现女友Java去做的，这点大家放心。
<a name="HCaJm"></a>
### 简单的原理
上面我们已经明确，所谓的JS的异步不过是将我们的“任务”放入“队列”，然后在主线程程序执行完后再去处理队列里面的任务。我们再将这段流程抽象来看，本质上就是**将异步的任务放入队列，然后保证主线程当前做的事不受影响，并且找一个时间点去处理异步的队列**。<br />所以最简单的设计模型已经可以实现了，请看下面的流程（感觉有点抽象）![rustel.png]([./images/async/rustel.png](http://img.xuhongzhuo.com/speaive/image/show/79523ed80e33491ebc0cb3355d02b8c5))<br />实际上就是去单独开一个线程去从队列中处理任务。此时这个想法看似简单，实际上心里难免有些疑问。

1. 任务的调度是**怎么调度**的（为什么单线程上会支持并发）
2. 如何知道任务的状态从而对异步任务之间进行**编排** (await的原理)
3. 难道真的只用了一个单线程嘛？那凭什么说支持大量并发

这些问题我们一个一个的看
<a name="gMxhV"></a>
#### 任务是怎么调度的
其实如果你让我说出全部细节我大抵是说不出来的。。。。不过可以聊一下思想>=<<br />我们都知道线程是CPU调度的基本单位，意思是CPU会通过一些调度算法给自己的线程进行不同的时间片划分，通过极快的速度辗转与各个线程之间来实现所谓的“**并发**”，而rust的异步编程之的思想本质上就是重新在单个线程上维护一个**类似于CPU调度线程**的功能，例如通过给自己的任务分配时间片的方式去调度每一个任务。<br />当然学过操作系统的都知道，这种渣男行为需要做许多事情，比如维护线程的上下文，那么在这里就是维护任务的上下文；明确线程的状态是start还是block等，在这里就是给每个任务也分配状态。利用一个单一的线程去维护这样的并发调度功能。<br />没错，协程的想法大抵也是这样的。<br />我们的现女友Java的线程模型是**一对一**的，即一个Java线程对应一个OS线程。并发场景下我们一般使用一个线程去做一个任务的处理，通过复用线程的方式来节省开销、销毁线程的成本。但是这种异步编程模型通过提高单个线程的复杂度来实现单个线程同时运行多个任务的功能，极大的提高了单个线程的CPU的利用率，这种就是经典的**多对多**模型。
<a name="tI92C"></a>
#### 如何编排异步任务
我们现女友Java在java8之后有一个大杀器叫**Completablefuture**，该工具可以很好的对异步任务进行**编排**，避免了出现**回调地狱**的情况，实际上就是JDK研发的大佬帮我们屏蔽了回调地狱的复杂度，让我们可以很好的编排我们的并发任务。那么rust的异步编程模型是如何进行编排的呢？<br />本质上我们刚才已经说过了，我们对每个任务都会维护任务状态的概念，诸如**就绪态、挂起态、错误态**等，我们通过await关键字，当遇见该关键字时，调度程序会检查被await的任务的状态，根据其运行状态决定是否继续，一旦被await的任务状态是已完成，那么则继续执行，否则挂起当前任务。当然这只是粗浅的编排流程。
<a name="nxjFF"></a>
#### 真的是单线程做事嘛
这种问题问出来显得很蠢（我自己问的），如果只用一个线程去做并发的优化，你就是把飞天火箭的CPU拿过来它的吞吐量也终究是有限的啊，实际上rust的这种并发模型是支持线程池配置的，例如一个5个线程的线程池里面的每一个线程都利用调度程序去并发的执行多个任务。好家伙越来越像go的协程了。
<a name="rsqlr"></a>
### 适用场景
这种异步编程模型非常适合**IO密集型**的操作，因为传统的单线程在处理IO操作的时候会一直accept网络另一端的结果，于是我们一般采用多线程的方式提高并发量，为了控制资源从而利用线程池去统一管理资源。而这种异步编程模型在遇见IO任务时选择挂起直接处理其他的任务，保证了单个线程中CPU是一点也没闲着啊。这样极大地提升了吞吐量。于是IO密集型就选它。
<a name="isZgC"></a>
#### 还有个小问题
rust是凭什么知道我的IO任务在accept了，然后他就去调度别的程序了呢？<br />rust借用了操作系统的异步机制例如**多路复用epoll**等同时监听多个套接字的读写事件，保证了即使响应任务的网络状态。
<a name="Qjkvv"></a>
### Java小小模拟了一下
```java
@Data
public class AsyncTask {

    // 一个function 也就是一个方法
    private Function<AsyncParam, AsyncResult> function;

    // 一个异步状态
    private String status;

    private AsyncParam asyncParam;
}
@Data
public class AsyncResult {
}
@Data
public class AsyncParam {
}
@Data
public class TestStringParam extends AsyncParam {
    private String param;
    public TestStringParam(String param) {
        this.param = param;
    }
    public TestStringParam() {
    }
}
@Data
public class TestStringResult extends AsyncResult {

    private String result;

    public TestStringResult(String result) {
        this.result = result;
    }

    public TestStringResult() {
    }
}
```
```java
public class EventLoopQueue {

    private Queue<AsyncTask> queue;

    public EventLoopQueue(Queue<AsyncTask> queue) {
        this.queue = queue;
    }

    public EventLoopQueue() {
        queue = new LinkedList<>();
    }

    public AsyncTask poll() {
        return queue.poll();
    }

    public void addTask(AsyncTask asyncTask) {
        if (asyncTask == null) {
            throw new NullPointerException("asyncTask is null");
        }
        asyncTask.setStatus("ready");
        queue.add(asyncTask);
    }
}
```
```java
public class EventLoopSchedule {

    // 单个线程的线程池
    private static ExecutorService executor = Executors.newSingleThreadExecutor();


    private EventLoopQueue eventLoopQueue = new EventLoopQueue();

    private static class InstanceHolder {
        private static final EventLoopSchedule INSTANCE = new EventLoopSchedule();

    }

    private EventLoopSchedule() {
        // 直接开始新线程的异步轮训
        executor.submit(() -> {
            try {
                run();
            } catch (InterruptedException e) {
                log.error("你妹的，谁中断的我！！！", e);
            }
        });
    }


    private void run() throws InterruptedException {
        while (true) {
            // 取出队列
            AsyncTask task = eventLoopQueue.poll();
            System.out.println(Thread.currentThread().getName() + ": 取出了任务");
            if (task == null || task.getStatus().equals("done")) {
                System.out.println(Thread.currentThread().getName() + ": 没任务啊休息1秒");
                Thread.sleep(1000);
                continue;
            }
            AsyncResult result = task.getFunction().apply(task.getAsyncParam());
            if (result != null) {
                System.out.println(Thread.currentThread().getName() + ": " + result.doResult());
            }
            Thread.sleep(1000);
            System.out.println(Thread.currentThread().getName() + ": 做完任务任务休息的1秒");
        }
    }

    public void addTask(Function<AsyncParam,AsyncResult> function,AsyncParam param) {
        AsyncTask task = new AsyncTask();
        task.setFunction(function);
        task.setAsyncParam(param);
        task.setStatus("init");
        eventLoopQueue.addTask(task);
    }

    public static EventLoopSchedule getInstance() {
        return InstanceHolder.INSTANCE;
    }
}

```
```java
public class Main {
    public static EventLoopSchedule loopSchedule;

    public static void main(String[] args) throws InterruptedException {
        // 初始化异步任务调度器
        init();
        System.out.println(Thread.currentThread().getName() + ": 我是同步输出1");
        // 添加任务到异步队列
        System.out.println(Thread.currentThread().getName() + ": 添加任务到异步队列");
        loopSchedule.addTask(Main::asyncFunction, new TestStringParam("我是异步任务的结果"));
        System.out.println(Thread.currentThread().getName() + ": 我是同步输出2");
        Thread.sleep(1000);
        System.out.println(Thread.currentThread().getName() + ": 我是同步输出3");
    }

    /**
     * 异步任务
     */
    private static AsyncResult asyncFunction(AsyncParam param) {
        if (param instanceof TestStringParam) {
            TestStringParam testStringParam = (TestStringParam) param;
            System.out.println(testStringParam.getParam());
            return new TestStringResult("success: " + testStringParam.getParam());
        }
        return null;
    }


    public static void init() {
        // 初始化事件驱动队列
        loopSchedule = EventLoopSchedule.getInstance();
    }
}
```
<a name="KYcik"></a>
#### 运行结果
![imjavaelage.png](http://img.xuhongzhuo.com/speaive/image/show/c1a09e41ef07471982ada8c7316b94e7)
