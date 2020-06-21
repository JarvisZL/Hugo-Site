# 终端和shell 应用眼中的OS


## 终端
- 终端是操作系统中的一个<font color=red size=3>对象</font>,也是一个I/O设备。
- 在Unix看来，终端也是一个文件(everything is a file)

### ANSI Escape Codes
- 终端支持Escape Codes，通过这些Codes可以完成屏幕清除/滚动，设置颜色等。
- 如果用cat输出某些二进制文件之后，可能会发生一些神奇的事情，比如光标没了之类的，这是因为可能该二进制文件中正好有一个负责控制光标的Escape Code，然后其在终端中输入就执行了。
- 可以用reset来还原终端

### 终端的模式
- 默认是cooked mode, 也就是终端自带一个行编辑器，按下回车才去执行这一行的命令
- 还有一个raw mode，按键就返回。

### 向终端输出
- 使用ls --color等变体发现
  - ls --color : 显示颜色，输出在一行
  - ls --color | less : 不显示颜色(只显示ESC序列)并在多行输出。
- 那么发生了为什么？ 使用strace 和 lstrace 康康不同层面干了什么吧
- 查看ltrace可以发现，都调用了一个isatty()函数，用来判断标准输出是不是终端，然后自适应输出。
- 查看strace可以发现isatty()会调用一个系统调用ioctl().



## Shell
- 操作系统 = 对象 + API, 那么用户怎么调用API?
- shell是一门<font color=red size=3>编程语言</font>.
  - interactive shell: 收到一行执行一行
    - env | grep PS1
  - non-interactive shell: 就是普通的语言解释器
    - bash -c 'env' | grep PS1
- shell是<font color=red size=3>人机接口</font>,interactive shell将用户给I/O设备的指令翻译成了系统调用的序列, 也就是说shell是用户眼中的操作系统(I/O设备)和应用眼中的操作系统(syscall)的桥梁。
- shell正如其名，它是操作系统内核对外的一个外壳，外界将需要传达的shell，shell则翻译成一系列系统调用然后传达给内核。
- shell的分类：CLI/GUI

### shell的实现
- shell的实现中有两个最基本的功能：向操作系统的某个对象读和写，比如cat命令，通过strace可以发现调用了read和write两个系统调用，这两个系统调用都是向文件描述符里读或写，那么<font color=blue size=3>什么是文件描述符</font>
- **文件描述符：** 是一个指向操作系统中某个对象的指针，是进程(状态机)的一部分，所以在fork的时候也会被拷贝。

### shell中的重定向
```c
echo Hello World > a.txt
//上面的shell命令发生了什么

pid_t p = fork();
if(p == 0){
    if(redirect_stdout){
        fd = open(...);
        dup(fd, STDOUT_FILENO);
        close(fd);
    }
    if(redirect_...){
        ...
    }
    ...
    execve(filename, argv, envp);
}
else{
    //如果父进程没有在后台执行就等到子进程完成
    if(!background) wait(&status);
}
```
- 可以发现重定向的过程就是：1)打开目标文件。2)使用dup复制文件描述符。3)释放打开所使用的文件描述符。
- 所以可以解释为什么<font color=blue size=3>sudo echo hello > /etc/a.txt</font>还是permission denied. 
- 因为我们从上面发现，重定向发生在execve之前，所以在重定向时还没有sudo权限。
