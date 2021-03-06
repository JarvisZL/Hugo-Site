# 崩溃一致性


## 崩溃一致性
- 希望文件系统在任何时候都能保持一致性。
- 导致不一致的原因: 数据结构的维护需要多个数据块的写入
  - RAID: $write(V_0) \rightarrow write(A_0), write(B_0)$
  - ext2(文件追写一块): $write(inode),write(bitmap),write(data)$
- 磁盘收到多个I/O请求，磁盘是有权利调换一些顺序的，除非收到一个flush请求。
- 比如上面第二种情况
  - inode写入了：导致指向了没有分配的地方，以后可能被分配导致两个文件共享这部分，十分严重的问题.(corrupt fs)
  - bitmap写入了：导致了分配了一块，但是没有办法访问到也没有办法free，导致了(deadblock).
  - data写入了：相当于写了一些garbage到文件系统中
  - inode,bitmap写入了：数据未写入，相当于文件内容出错(corrupt file)
  - inode,data写入了：同样可能导致共享问题(corrupt fs)
  - bitmap,data写入了：deadblock  

## fsck(File System Checking)
- 根据磁盘中已有的信息，恢复出最可能的数据结构的状态，但有时候很难决定什么是对的结构。
- fsck也是一个程序，有可能在其运行过程中发生崩溃，这时文件系统可能进入彻底无法恢复的状态
- 所以需要一个更加"constructive"的办法实现崩溃一致性，所以提出了journaling.

## journaling 
- 实现append-only的block vector
  - 比如append(234)到1后面，要不然就全部写入要不然就认为全部没有写入的原子append
  - 写完234后使用一次flush，成功返回后写入一个OK标记再flush，只有第二次flush返回成功才认为前面已经都写入磁盘。 
- 实现数据结构有两种方法，第一种是常见的记录其结构，另一种是记录使用该数据结构的一些操作，而在实现了append-only的基础上，可以使用append-only记录这些操作。
  - 从而保证先将操作记录写入磁盘，然后再执行这些操作。
- journaling的优化
  - 合并多个flush标记Tx减少log大小
  - 不记录Txbegin,Txend而是记录checksum,Tx的长度
