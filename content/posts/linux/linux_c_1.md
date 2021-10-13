---
title: "Linux C之进程"
date: 2022-01-30T21:33:51+08:00
categories: ["C","Linux"]
tags: ["C","Linux","线程","进程","管道"]
#description: "简单的介绍了在linux下c语言对于进程的操作"

# weight: 1

TocOpen: false
draft: false

hideSummary: false
hidemeta: false

disableHLJS: true # to disable highlightjs
disableShare: false

comments: true
canonicalURL: "https://www.niuwx.cn/"
cover:
    image: ""
---

## 初识进程和线程

### 认识进程

在Linux系统中，每一个进程都有自己的ID，就如同人的身份证一样。Linux中有一个数据类型pid_t，它定义了进程的ID。

<!--more-->

#### fork()

首先看下如何创建新的进程，这里需要用到`fork()`函数，其返回值类型为`pid_t`。

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/unistd.h>
void get_pid() {
    pid_t pid = fork();
    if (pid < 0) {
        printf("fork error\n");
        exit(1);
    } else if (pid > 0) {
        printf("parent: the pid is %d\n", pid);
        while (1)
            ;
    } else {
        printf("child: this pid is %d\n", pid);
        while (1)
            ;
    }
}
int main() {
    get_pid();
    return 0;
}
```

执行结果：

```bash
parent: the pid is 20580
child: this pid is 0
```

执行结果中有两条输出，分别是父进程和子进程。

通过命令行执行`htop`找到对应的进程，我们可以发现父子进程都有其特定的pid，那么为什么程序会输出0呢？

![htop.png](/htop.png)

实际上，在调用`fork()`函数之后，程序创建了一个子进程，程序本身成为了父进程。 

而`fork()`的返回值代表什么意义呢？

*   负数：创建子进程失败。
*   零：在子进程中，`fork()`返回0
*   正数：在父进程中，`fork()`返回子进程的pid

至此，以上的疑惑也就迎刃而解了，在这里介绍两个进程相关的函数：

*   `getpid()`：获取当前进程的pid
*   `getppid()`：获取当前进程的父进程的pid



接下来再看这段程序：

增加了一个全局变量和一个函数内部的局部变量。

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/unistd.h>
int global_var = 2;
void get_pid() {
    pid_t pid = fork();
    int var = 5;
    if (pid < 0) {
        printf("fork error\n");
        exit(1);
    } else if (pid > 0) {
        printf("parent: this pid is %d, global_var = %d, var = %d\n", pid, global_var, var);
        exit(0);
    } else {
        global_var--;
        var++;
        printf("child: this pid is %d, global_var = %d, var = %d\n", pid, global_var, var);

        exit(0);
    }
}

int main() {
    get_pid();
    return 0;
}
```

执行结果：

```bash
parent: this pid is 22982, global_var = 2, var = 5
child: this pid is 0, global_var = 1, var = 6
```

在子进程中，`global_var`与`var`分别执行的加减操作，但是在父进程中，这两个变量的值都未发生改变，这是为什么呢？

原因在于：执行`fork()`函数时，子进程复制了父进程的所有资源，包括内存等等。因此，父子进程属于不同的内存空间，那么子进程中变量发生改变时，父进程中的变量必然不会改变。

#### vfork()

除了`fork()`函数，还有一个`vfork()`函数，同样是系统调用函数，用来创建子进程。但是这二者是有区别的：`vfork()`函数在创建子进程时，父子进程共享地址空间。因此子进程中修改的全局变量在父进程中也会被修改。

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/unistd.h>
int global_var = 2;
void get_pid() {
    pid_t pid = vfork();
    int var = 5;
    printf("pid = %d\n", getpid());
    printf("global_var = %d, var = %d\n", global_var, var);
    if (pid < 0) {
        printf("fork error\n");
        exit(1);
    } else if (pid > 0) {
        printf("parent: this pid is %d, global_var = %d, var = %d\n", pid, global_var, var);
        exit(0);
    } else {
        global_var--;
        var++;
        printf("child: this pid is %d, global_var = %d, var = %d\n", pid, global_var, var);
        exit(0);
    }
}

int main() {
    get_pid();
    return 0;
}
```

执行结果：

```bash
pid = 25371
global_var = 2, var = 5
child: this pid is 0, global_var = 1, var = 6
pid = 25370
global_var = 1, var = 5
parent: this pid is 25371, global_var = 1, var = 5
```

#### execv()

在`fork()`与`vfork()`中，子进程与父进程都运行同样的代码。如果需要子进程执行不同的操作，就要用到`execv()`函数了

```c
//parent.c
#include <stdio.h>
#include <unistd.h>
int main(int argc, char* argv[]) {
    execv("child", argv);
    return 0;
}
```

```c
//child.c
#include <stdio.h>

int main(int argc, char* argv[]) {
    puts("welcome!");
    for (int i = 0; i < argc; i++) {
        puts(argv[i]);
    }
    return 0;
}
```

分别编译两个c文件，执行`parent.c`编译得到的可执行文件：`./parent 1 2 3`。

执行结果：

```bash
welcome!
./parent
1
2
3
```



### 进程等待

进程等待就是同步父子进程。可以通过调用`wait()`函数来实现。

`wait()`函数的工作原理是首先判断子进程是否存在， 如果创建失败，子进程不存在，那么就直接退出进程，并提示相关错误信息。如果创建成功，`wait()`函数将父进程挂起，知道子进程结束，并返回结束的状态和最后结束的子进程的pid。

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>
#include <wait.h>

void exit_s(int status) {
    if (WIFEXITED(status)) {
        printf("normal exist, status = %d\n", WEXITSTATUS(status));
    } else if (WIFSIGNALED(status)) {
        printf("signal exit! status = %d\n", WTERMSIG(status));
    }
}

void wait_test() {
    pid_t pid_a;
    int status;
    int ret;

    pid_a = fork();
    if (pid_a < 0) {
        printf("child process error\n");
        exit(0);
    } else if (pid_a > 0) {
        printf("the pid of parent is %d\n", getpid());
        printf("wait for child ...\n");
        int pid_child = wait(&status);
        if (pid_child > 0) {
            printf("i catch a child process with pid of %d\n", pid_child);
        }
        exit_s(status);
    } else {
        printf("the pid of child is %d\n", getpid());
        sleep(3);
        exit(2);
    }
}

int main() {
    wait_test();
    return 0;
}
```

执行结果：

```bash
the pid of parent is 46045
wait for child ...
the pid of child is 46046
i catch a child process with pid of 46046
normal exist, status = 2
```

在子进程中调用`sleep()`函数，睡眠3秒，只有子进程完成睡眠，才能正常退出并被父进程捕捉到。在此期间父进程会继续等待下去。在wait函数中会将子进程的状态保存到`status`中。

在处理`status`的`exit_s()`函数中，调用了几个宏：

*   `WIFEXITED`：当子进程正常退出时，返回真值
*   `WEXITSTATUS`：返回子进程正常退出时的状态
*   `WTERMSIG`：用于子进程被信号终止的情况

如果在程序执行过程中子进程异常退出，会是怎样的情况呢？

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>
#include <wait.h>

void exit_s(int status) {
    if (WIFEXITED(status)) {
        printf("normal exist, status = %d\n", WEXITSTATUS(status));
    } else if (WIFSIGNALED(status)) {
        printf("signal exit! status = %d\n", WTERMSIG(status));
    }
}

void wait_test() {
    pid_t pid_a;
    int status;
    int ret;

    pid_a = fork();
    if (pid_a < 0) {
        printf("child process error\n");
        exit(0);
    } else if (pid_a > 0) {
        printf("the pid of parent is %d\n", getpid());
        printf("wait child ...\n");
        int pid_child = wait(&status);
        if (pid_child > 0) {
            printf("i catch a child process with pid of %d\n", pid_child);
        }
        exit_s(status);
    } else {
        printf("the pid of child is %d\n", getpid());
        // sleep(3);
        pid_t pid = getpid();
        kill(pid, 9);
        exit(2);
    }
}

int main() {
    wait_test();
    return 0;
}
```

执行结果：

```bash
the pid of parent is 46749
wait child ...
the pid of child is 46750
i catch a child process with pid of 46750
signal exit! status = 9
```

### 线程

需要先了解一下进程和线程的区别：

*   进程是资源分配的最小单位，每个进程都占有独立内存空间。
*   线程是程序执行的最小单位，多个线程共享同一个内存空间。
*   一个进程由几个线程组成，线程与同属一个进程的其他线程共享当前进程拥有的全部资源。

那么，线程有哪些优势呢？

*   线程不需要额外的内存申请
*   线程共享进程内的数据，访问数据方便，而进程则需要通过通信的方式进行。

在linux下，进程使用`ps`命令查看，`线程`则通过`top`命令来查看。还可以通过`top -p pid`来查看某个进程内的线程。

先看一个创建进程的例子：

```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void *thread1(void) {
    int i;
    for (int i = 0; i < 5; i++) {
        printf("this is the first thread\n");
        sleep(1);
    }
}

void *thread2(void) {
    int i;
    for (int i = 0; i < 5; i++) {
        printf("this is the second thread\n");
        sleep(1);
    }
}

int main() {
    int ret = 0;
    pthread_t id1, id2;
    pthread_create(&id1, NULL, (void *)thread1, NULL);
    pthread_create(&id2, NULL, (void *)thread2, NULL);
    pthread_join(id1, NULL);
    pthread_join(id2, NULL);
    return 0;
}
```

在这里，`thread1`和`thread2`两个函数分别属于不同的线程。

`pthread_t`是线程的id。

`pthread_create()`是创建线程的函数，参数分别是：参数id，线程属性，线程运行函数的起始地址，运行函数的参数。

注意，由于`pthread`并非Linux系统的默认库，而是POSIX线程库，所以在编译的时候需要加上`-lpthread`来显式链接该库：`gcc -g main.c -lpthread`。

执行结果：

```bash
this is the first thread
this is the second thread
this is the first thread
this is the second thread
this is the first thread
this is the second thread
this is the first thread
this is the second thread
this is the first thread
this is the second thread
```



## 进程间通信

进程间的通信包括管道、共享内存、信号量通信、消息队列、套借口(socket)和全双工管道通信。

### 管道

管道顾名思义，就如同水管一样，当水从水管的一端流向另一端的时候，水流是单方向的。某一时刻只能从单方向传递数据，不能双向传递，这种就是半双工模式。半双工模式只能一端写数据，一端读数据，先来看一个半双工的例子：

1.   在父进程中通过`pipe()`函数创建一个管道，得到管道读端ppe[0]，写端ppe[1]。
2.   在父进程中调用`fork()`产生一个子进程
3.   在子进程中关闭ppe[0]，往ppe[1]中写入数据，在父进程中关闭ppe[1]，从ppe[0]中读取数据。

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <wait.h>

int pipe_test() {
    int ppe[2];

    if (pipe(ppe) == -1) {
        perror("not create a new process!\n");
        return 1;
    }

    int pid = fork();
    if (pid == 0) {
        close(ppe[0]);
        printf("child process send message\n");

        char *message = "happy new year!\n";
        write(ppe[1], message, strlen(message));
    } else {
        close(ppe[1]);
        sleep(2);
        printf("parent process receive message\n");
        char message[100];
        int line = read(ppe[0], message, 100);
        write(STDOUT_FILENO, message, line);
        wait(NULL);
        exit(0);
    }
    return 0;
}

int main() {
    pipe_test();
    return 0;
}
```

执行结果：

```bash
child process send message
parent process receive message
happy new year!
```

上面这个例子演示了单向通信，如果我们需要双向通信：父进程在读的同时也给子进程写。要实现这样的功能，就必须建立两个管道：一个管道从父进程流向子进程，一个管道从子进程流向父进程。

代码如下：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <wait.h>

int pipe_test() {
    int ppea[2], ppeb[2];

    if (pipe(ppea) == -1 || pipe(ppeb) == -1) {
        perror("not create a new process!\n");
        return 1;
    }

    int pid = fork();
    if (pid < 0) {
        perror("not create a new process !");
        return 1;
    } else if (pid == 0) {
        close(ppea[0]);
        printf("child process send message\n");
        char *message_send = "happy new year!\n";
        write(ppea[1], message_send, strlen(message_send));

        sleep(2);

        close(ppeb[1]);
        printf("child process receive message\n");
        char message_receive[100];
        int line = read(ppeb[0], message_receive, 100);
        write(STDOUT_FILENO, message_receive, line);
        exit(0);
    } else {
        close(ppea[1]);
        sleep(2);
        printf("parent process receive message\n");
        char message_receive[100];
        int line = read(ppea[0], message_receive, 100);
        write(STDOUT_FILENO, message_receive, line);

        close(ppeb[0]);
        printf("parent process send message\n");
        char *message_send = "happy new year my child!\n";
        write(ppeb[1], message_send, strlen(message_send));
        exit(0);
    }
    return 0;
}

int main() {
    pipe_test();
    return 0;
}
```

执行结果：

```bash
child process send message
parent process receive message
happy new year!
parent process send message
child process receive message
happy new year my child!
```



### 命令管道

上个部分介绍了管道，然而管道只能在有关联的进程中进行通信，也就是父子进程之间。那如果不相关的两个进程间也需要进行通信，这里就需要用到命令管道了，通常称为`FIFO`。通过这个名称可以知道命令管道遵循先进先出的原则，与数据结构中的队列类似。

创建一个命令管道有两种方法：

*   通过shell命令创建
*   通过函数创建命名管道

#### shell命令创建命名管道

1.   首先通过`mkfifo`创建一个管道文件test：`mkfifo test`
2.   通过`cat ./test`查看，此时，管道文件中没有任何数据
3.   打开另一个终端，向FIFO中写入数据：`echo "hello fifo" > ./test`
4.   再次通过`cat ./test`查看，将会得到之前写入的数据。

#### 函数创建命名管道

下面通过c来实现命令管道：

```c
//write.c
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

int mkfifo_write() {
    int ppe = open("./fifo", O_RDWR);
    printf("write the message:\n");

    char* message = "hello world\n";
    write(ppe, message, strlen(message));
    close(ppe);
}

int main() {
    mkfifo_write();
    return 0;
}
```

```c
//read.c
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int mkfifo_read() {
    char message[100];

    int ppe = open("./fifo", O_RDWR);
    printf("read the message:\n");
    int line = read(ppe, message, 100);
    close(ppe);
    write(STDOUT_FILENO, message, line);
}

int main(int argc, char* argv[]) {
    mkfifo_read();
    return 0;
}
```

编译后，开启两个终端，一个终端先执行`./read`，另一个终端再执行`./write`，第一个终端中就会显示发出的信息。
