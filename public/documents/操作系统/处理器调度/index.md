# 处理器调度


## 单处理器分时调度

### Round-Robin调度方法
- 将时间划分成一个个时间片，每个进程轮流获取一个时间片的CPU。 
- 但是不同进程对cpu的需求是不一样的，一个极端的例子就是10个进程，有9个是死循环，还有一个vim进程，如果此时使用Round-Robin算法，则会导致vim进程获取输入缓慢，所以我们需要给不同进程设置优先级。

### 优先级
- Unix nicess
  - nice属性的值的范围在-20~19，其中-20意味着极坏，19意味着极好，越坏的进程越容易获得cpu。
- 如果使用Round-Robin的形式，如何划分优先级
  - 在圆上根据nice值，按照一定比例划分进程对应的圆弧，然后一个指针在圆上旋转，指针指到哪个进程部分，就执行哪个进程。

### 多级反馈调度
- 在系统中存在多个Round-Robin调度器，每个调度器对应一个优先级，优先级高的先调度，只有优先级高的都暂时不需要cpu时，才可以调度低优先级的。


### Linux调度器-CFS(完全公平调度)
- 每个进程都有一个vruntime属性记录该进程到目前为止在cpu上的虚拟运行时间，每次需要切换进程时都选择补偿虚拟运行时间最短的那个。
- 如果一个进程创建子进程，则其子进程会继承父进程的vruntime

#### 一些小问题
- 问题1：如果一个进程沉睡了较长时间，那么被唤醒后一段长时间都是该进程占据cpu
- 解决办法：唤醒进程时给进程的$vruntime = max(vruntime, min(vruntime_{others}))$,这样就能保证其既能很快获得调度又不会长时间占据cpu。
- 问题2：优先级如何表示
- 解决办法：进程的vruntime实际上就是根据优先级来使用不同权重计算的，对于坏的进程，可能其一个vruntime就能获得多个时间片，而对于好的进程，可能一个vruntime只能获得1个或少量的时间片。
- 问题3：用什么数据结构实现vruntime
- 解决办法：linux中复用了红黑树。
- 问题四：用64位无符号整数表示vruntime,其运行500多年会发生溢出问题，如何解决
- 解决办法：Linux内部使用的是两个vruntime之间的$\Delta$来确定两个进程谁应该被调度。
  - 其假设不会有进程沉睡很长很长时间，所以系统中的所有进程的vruntime相差不会很大(假设不会超过u64的一半)。
    ```c
    s64 delta = (s64)(a->vruntime - b->vruntime);
    //如果delta < 0, 则说明应该调度a.
    //否则应该调度b.

    /* 如果a溢出了那么在数值上a->vruntime远小于
    *  b->vruntime，但是两者的差转换为带符号64位整
    *  数后就会变为正数。
    */
    ```

### 优先级翻转
- 实际上当考虑到互斥同步的情况后，会给某些调度方法带来一些问题：
  - 如果3个进程
    ```c
    thread_1 {
      P(empty);
      printf("(");
      V(fill);
    } 
    thread_2{
      P(fill);
      printf(")");
      V(empty);
    }
    thread_3{
      while(1);
    }
    ```
    - 那么如果使用Round-Robin调度方法，则会发现thread\_1和thread\_2之间必须等待一个thread_3的时间片才能够执行，这显然浪费了时间
    - 但这个问题在CFS上就能够被较好解决。

  - 假设有3个进程
    ```c
    bad_guy{
      mutex_lock(&mutex);
      ...
      mutex_unlock(&mutex);
    }
    nice_guy{
      ...
    }
    very_nice_guy{
      mutex_lock(&mutex);
      ...
      mutex_unlock(&mutex);
    }
    ```
    - 当very\_nice\_guy执行并获取互斥锁mutex后，发生中断，进行进程切换。
    - 切换到bad\_guy执行，其无法获得互斥锁mutex,主动放弃cpu，进行进程切换。
    - 切换到nice\_guy执行。
    - 于是，<font color=blue size=3>nice\_guy在bad\_guy之前执行</font>，这就是<font color=red size=3>优先级翻转</font>

- 优先级翻转的部分解决(针对互斥锁)：进程可以继承申请互斥锁的进程的优先级
  - 当bad\_guy申请但无法获得mutex后，获取该互斥锁的very\_nice\_guy的优先级就会被boost到bad\_guy一样的优先级，然后被执行，释放互斥锁。
