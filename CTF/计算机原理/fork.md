# fork

fork()函数通过系统调用创建一个与原来进程几乎相同的进程，如fork过程中传入参数或变量不同，两个进程可以做不同的事。fork()后，系统会给新进程分配空间，然后把原来的进程的所有值复制到新进程的空间中，只有少数值与原来进程不同。

```C
#include <unistd.h>  
#include <stdio.h>   
int main ()   
{   
    pid_t fpid; //fpid表示fork函数返回的值  
    int count=0;  
    fpid=fork();   
    if (fpid < 0)   
        printf("error in fork!");   
    else if (fpid == 0) {  
        printf("i am the child process, my process id is %d/n",getpid());   
        printf("我是爹的儿子/n");//对某些人来说中文看着更直白。  
        count++;  
    }  
    else {  
        printf("i am the parent process, my process id is %d/n",getpid());   
        printf("我是孩子他爹/n");  
        count++;  
    }  
    printf("统计结果是: %d/n",count);  
    return 0;  
}  
```

运行结果是：
    i am the child process, my process id is 5574
    我是爹的儿子
    统计结果是: 1
    i am the parent process, my process id is 5573
    我是孩子他爹
    统计结果是: 1

在执行fork()前，只有一个进程，执行后，便有两个进程向下执行。

![](http://doze9097.top//1574007505967.png)

fork()被调用一次，返回两次，分别是对父子进程的返回：

​    1）在父进程中，fork返回新创建**子进程的进程ID**；
​    2）在子进程中，fork返回0；
​    3）如果出现错误，fork返回一个负值；

fork()出错的原因：

​	1）当前进程数达到系统上线，这时error值被设置为EAGAIN

​	2）系统内存不足，这时error值被设置为ENOMEN

新进程创建后，系统出现两个基本相同的进程，这两个进程没有固定的执行顺序，哪个进程先执行要看系统调度。



```C
#include <unistd.h>
#include <stdio.h>
int main(void)
{
   int i=0;
   printf("i son/pa ppid pid  fpid/n");
   //ppid指当前进程的父进程pid
   //pid指当前进程的pid,
   //fpid指fork返回给当前进程的值
   for(i=0;i<2;i++){
       pid_t fpid=fork();
       if(fpid==0)
    	   printf("%d child  %4d %4d %4d/n",i,getppid(),getpid(),fpid);
       else
    	   printf("%d parent %4d %4d %4d/n",i,getppid(),getpid(),fpid);
   }
   return 0;
}
```

执行结果：

​	 i son/pa ppid pid  fpid
​    0 parent 2043 3224 3225
​    0 child  3224 3225    0
​    1 parent 2043 3224 3226
​    1 parent 3224 3225 3227
​    1 child     **1** 3227    **0**
​    1 child     **1** 3226    **0**

由于3224、3225执行循环后已经退出，此时这两个进程已经死亡，所以3226、3227的父进程就被置为p1，p1是永不死亡的。

由于fork出3227，3226后，程序结束，所以这两个进程没有子进程，故fpid为0.

![](http://doze9097.top//1574052619772.png)

```C
#include <unistd.h>
#include <stdio.h>
int main() {
	pid_t fpid;//fpid表示fork函数返回的值
	//printf("fork!");
	printf("fork!/n");
	fpid = fork();
	if (fpid < 0)
		printf("error in fork!");
	else if (fpid == 0)
		printf("I am the child process, my process id is %d/n", getpid());
	else
		printf("I am the parent process, my process id is %d/n", getpid());
	return 0;
}
```

执行结果如下：
    fork!
    I am the parent process, my process id is 3361
    I am the child process, my process id is 3362
    如果把语句printf("fork!/n");注释掉，执行printf("fork!");
    则新的程序的执行结果是：
    fork!I am the parent process, my process id is 3298
    fork!I am the child process, my process id is 3299

原因：printf()函数仅仅是把数据写入stdout的缓存队列中，只有当队列刷新(\n时会刷新该缓冲区)时，才能看到内容被打印出来。没有\n的程序中，fork会把stdout缓冲区的数据复制给子进程，所以会被print两次。