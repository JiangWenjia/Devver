# GCD
Grand Central Dispatch

> dispatch 英 [dɪˈspætʃ] 派遣，调度;

## 任务
表示要添加到队列的操作，在GCD中为Block

执行（调度派发）任务的方式有两种： 同步执行（sync）和异步执行（async）

### 同步执行 sync
> sync 英[sɪŋk] n.	同时，同步;

会**阻塞**当前线程，并**等待**Block中的任务执行完成完毕后，当前的线程才会继续执行

对应的函数：

```objc
void dispatch_sync(dispatch_queue_t queue, dispatch_block_t block);
```
第一个参数为执行队列，第二个参数为执行的block

### 异步执行
> async	英[ə'zɪŋk] adj.异步的

**不会阻塞**当前线程，**不用等待**会直接往下执行

对应的函数：

```objc
void dispatch_async(dispatch_queue_t queue, dispatch_block_t block);
```

## 队列
用于存放任务，分为两种：**串行队列** 和 **并发队列**
对应GCD的 

```objc
dispatch_queue_t
```

### 串行队列
> 添加到串行的队列的任务，GCD会FIFO地按顺序取出，放到对应的队列中。

sync：当前的线程，顺序执行
asyn：其他的线程，顺序执行

### 并行队列
> 添加到并行队列的任务，GCD会多个同时，放到队列中执行

sync：当前线程，顺序执行
asyn：多个线程，并行执行

### 队列的创建
> concurrent: 英 [kənˈkʌrənt] adj.同时发生的;同时完成的;
> serial: 英 [ˈsɪəriəl] 连续的;顺序排列的;
> priority: 英 [praɪˈɒrəti] 优先，优先权;

- 主队列


```objc
dispatch_queue_t main_queue = dispatch_get_main_queue();
```

- 全局并行队列


```objc
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
```
对应的优先级

```objc
#define DISPATCH_QUEUE_PRIORITY_HIGH 2
#define DISPATCH_QUEUE_PRIORITY_DEFAULT 0
#define DISPATCH_QUEUE_PRIORITY_LOW (-2)
#define DISPATCH_QUEUE_PRIORITY_BACKGROUND INT16_MIN
```
- 用户队列

```objc
//对应的函数
//第一个参数表示为队列标示名称，可在Debug中区分队列，第二参数为队列的配置属性,例如并行还是串行
dispatch_queue_create(const char *_Nullable label,dispatch_queue_attr_t _Nullable attr);
		
// 使用
dispatch_queue_t queue = dispatch_queue_create("queue.name", DISPATCH_QUEUE_CONCURRENT);		
```

https://blog.csdn.net/liuyinghui523/article/details/50618092


