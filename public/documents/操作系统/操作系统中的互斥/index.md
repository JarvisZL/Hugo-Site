# 操作系统中的互斥


## 操作系统上的互斥(多处理器多线程)
```C
void func(void* arg){
    while(1){
        lock(&biglock);
        printf("Thread-%s on CPU #%d acuquired the lock\n",arg,_cpu());
        unlock(&biglock);
    }
}
```
- 上述代码在多处理器上多线程运行时会发现一段时间后，总是一个线程在一个处理器上获得锁。safety和liveness都没问题而,fairness出现了问题。
- 使用状态机来理解发现：对于持有了锁的线程，其中断大概率出现在printf，而此时其他CPU上的其他线程都处于自旋状态，结果绕一圈回到自己继续执行。
- 这说明：获得自选锁的线程(处理器)不应该被中断，否则会因为其他线程的自旋浪费时间，所以可以在lock函数中加入关中断，在unlock中加入开中断
```C
void spin_lock(spinlock_t &lk){
    cli();
    while(atomic_xchg(&lk->locked,1));
}

void spin_unlock(spinlock_t &lk){
    atomic_xchg(&lk->locked,0);
    sti();
}
```
- 而上述代码又面临一个问题，如果一个线程多次获得不同的锁，之后又依次释放锁，则结果第一次释放锁就导致开中断，因此需要保存第一次获得锁的时候线程(CPU)中断状态，在最后一次释放锁的时候还原状态。
- 所以需要在CPU的结构中加上记录当前CPU关中断计数以及首次关中断时 %eflags & FL\_IF的值

## 互斥锁
- 自旋锁由于可能导致浪费很多资源，所以其只适用于很短并且希望能立即执行的临界区。而有的时候，只有互斥的需求但不介意被打断或者自身等待。
- 比如在读写磁盘时，此时如果使用自旋锁，会导致浪费很多时间，并且在这段时间内可能有很多中断，但由于关中断的原因无法到达CPU.
- 互斥锁的需求分析
  - 持有锁的线程
    - 允许处理器相应中断。
    - 允许切换到其他线程。
  - 等待锁的线程
    - 不要占用处理器资源自旋。
- 一个很简单的想法就是当我发现自身无法得到锁，需要自旋时就让另一个线程执行。
```C
void mutex_lock(mutexlock_t &lk){
    while(atomic_xchg(&lk->locked, 1)){
        let_another_thread_to_execute();
    }
}
```
- 上述方法值得优化的点在于：可能等待锁的线程有很多，前面需要空转很多线程(每个线程都会atomic\_xchg)才能让真正可以执行的线程执行。
- 所以可以加上一个调度者，每当线程无法获得锁时就直接放弃对CPU的占有，陷入等待，等待调度者的调度。
- 添加一个等待队列这样的并发数据结构，由调度者控制
```c
void mutex_lock(mutexlock_t &lk){
    int acquired = 0;
    spin_lock(&lk->spinlock);//保护等待队列
    if(lk->locked != 0){
        enqueue(lk->wait_list, current);
        current->status = BLOCKED;
    }
    else{
        lk->locked = 1;
        acquired = 1;
    }
    spin_unlock(&lk->spinlock);
    if(!acquired) yield();//主动切换。
}

void mutex_unlock(mutexlock_t &lk){
    spin_lock(&lk->spinlock);
    if(!empty(lk->wait_list)){
        //不为空
       struct task_t* task =  dequeque(lk->wait_list);
       task->status = RUNNABLE;
    }
    else
        lk->locked = 0;
    spin_unlock(&lk->spinlock);
}

```
