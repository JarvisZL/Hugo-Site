# 动态链接


## 静态链接
- 为了一个程序不同部分的解耦，可以将一个程序分成若干个文件是十分必要的，那么不同的文件最终怎么变成一个可执行文件加载到操作系统上执行的。

```c
//a.c
int foo(int a, int b){
    return a + b;
}

//b.c
int x = 100, y = 200;

//main.c
extern int x,y;
int foo(int a,int b);
int main(){
    printf("%d + %d = %d\n",x,y,foo(x,y));
}
```
- 不同的文件通过编译形成不同的$.o$文件，查看$main.o$可以发现，其对$x,y,foo$的使用，都使用$0(\%rip)$代替($rip$寻址，但偏移量不知)。
- 在静态链接的之后，就会根据ELF文件头信息，正确地填入偏移量，ELF文件中都是符号地址-4，是因为此时rip指向的是下一条指令，其相对于所需填入的地方增加了4个字节($0(\%rip)$中的0占4字节)。

- 静态ELF加载器：
  - 在加载静态链接得到的可执行文件时，直接根据ELF程序头表中信息使用mmap将文件中对应部分映射到地址空间中即可。

## 动态链接
- 如果都是用静态链接太耗费资源。

### 需求1
- 所需动态链接的库只含有代码。
- 实现：直接使用mmap将这部分映射到对应程序的地址空间，这样也保证了在物理内存中只有一个该库函数的备份。
- 实现前提：代码需要是位置无关代码，引用跳转使用(PC相对寻址)。

### 需求2
- 所需动态链接的库含有本库使用的代码和数据。
- 实现：同样只需要mmap，因为数据也可以根据数据和PC的Offset来相对寻址。

### 需求3
- <font color=red size=3>在一个动态链接库中可以使用其他动态链接库的符号。</font>
- 实现：使用表来记录相应的函数的地址。比如在foo中调用bar，动态链接器先加载bar所在的库，然后加载foo时，就将bar所在的地址填入表格中。

### Linux中的动态链接-GOT和PLT
- linux中动态链接同样使用了需求3类似的理念，用表格来记录地址，一个程序有一个GOT表，其结构如下：
  - $GOT[0] = .dynamic$ (所在的节的首地址)
  - $GOT[1] = link~map$  (用于遍历依赖的动态链接库)
  - $GOT[2] = \_dl\_runtime\_resolve$地址，该函数用来解析符号(即确定符号的地址)。
  - 剩下的表项就是一个个所需的动态符号地址。

- 那么如果一开始就把所有符号都解析一遍，可能最后程序使用的只是极少部分，显然不合理。
- 一个trival解决办法
```c
// 将调用printf函数翻译成指令 call printf_internal

int printf_internal(const char* fmt, ...){
    if(!GOT[PRINTF]){
        GOT[PRINTF] = call_dl_runtime_resolve("printf");
    }
    return GOT[PRINTF](...);
}
```
- 但问题在于每一次调用prinf都要执行一次判断。
- 引入plt的巧妙设计使得不需要这一次额外的判断，具体代码如下
```
<.plt>
    pushq <GOT+0x8>
    jmpq <GOT+0x10>
    nopl 0x0(%rax)

<printf@plt>
    jmpq printf@GOT
    pushq 0x0
    jmpq <.plt>
```

- 初始的时候 $printf@GOT$的地址就是其下一条指令$pushq$ 0x0的地址。
  - 首次调用$printf$:
    - 先$push$ 0x0(这里0表示$printf$)
    - 跳转到$<.plt>$执行
    - 将$GOT[1]$压栈
    - 跳转$GOT[2]$执行(即去解析$printf$的地址)
  - 首次调用后，会将$printf$的地址填入$printf@GOT$处，则之后调用，直接通过该处地址直接跳转执行。
