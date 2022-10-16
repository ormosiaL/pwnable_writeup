## Toddler's Bottle
### passcode

` $ ssh passcode@pwnable.kr -p2222`
连接到服务器后，如果直接运行程序，输入passcode1时会遇到报错：<p> ![seg](https://github.com/ormosiaL/pwnable_writeup/blob/main/img/passcode_segfault.jpg)<p> 

查看passcode.c 的源码：
```
#include <stdio.h>
#include <stdlib.h>

void login(){
        int passcode1;
        int passcode2;

        printf("enter passcode1 : ");
        scanf("%d", passcode1);
        fflush(stdin);

        // ha! mommy told me that 32bit is vulnerable to bruteforcing :)
        printf("enter passcode2 : ");
        scanf("%d", passcode2);

        printf("checking...\n");
        if(passcode1==338150 && passcode2==13371337){
                printf("Login OK!\n");
                system("/bin/cat flag");
        }
        else{
                printf("Login Failed!\n");
                exit(0);
        }
}

void welcome(){
        char name[100];
        printf("enter you name : ");
        scanf("%100s", name);
        printf("Welcome %s!\n", name);
}

int main(){
        printf("Toddler's Secure Login System 1.0 beta.\n");

        welcome();
        login();

        // something after login...
        printf("Now I can safely trust you that you have credential :)\n");
        return 0;
}
```
发现问题出现在 `scanf("%d", passcode1);`<p>
`int scanf(const char *format, ...)`： 从标准输入(stdin) 读取格式化输入(Scan Formatted String)。例如：`scanf（"%d",&a）;` 意思是把输入读取到a的地址处，其中 `&` 是取址符，用来获得变量a的内存地址。<p>
`segmentation fault` 意思是程序在访问不合法的内存地址。在这里是因为不正确的 scanf() 造成的，第二个参数前没有 `&` ，使得程序读取了变量值而不是它的地址。 <p> 
而`scanf("%100s", name);`读取字符串时不需要加取址符。因为C语言的字符串实际上是字符数组，一个数组的标识符代表的就是它的起始地址。<p>
![seg2](https://github.com/ormosiaL/pwnable_writeup/blob/main/img/passcode_segfault2.jpg)<p> 


---

查看程序的保护机制,发现存在栈溢出保护但是是partial relro，所以可以通过GOT覆写进行漏洞利用：<p>
![python](https://github.com/ormosiaL/pwnable_writeup/blob/main/img/passcode_python.jpg)<p> 
有堆栈地址随机化 RELRO，有partial relro (GOT 可写）和 full relro (GOT 不可写), 栈溢出保护 Canary 和 数据执行保护 NX(No execute) <p> 
GOT表/全局偏移表（Global offset Table) 覆写：<p>
PLT表：过程连接表(Procedure Linkage Table)，一个PLT条目对应一个GOT条目 <p> 
查看elf文件的GOT表可以使用：` readelf -r target_file` 或 ` objdump -R target_file ` <p>![got](https://github.com/ormosiaL/pwnable_writeup/blob/main/img/passcode_got.jpg)<p> 

---
通过gdb可以看到：name的地址在ebp-0x70, 而passcode1的地址在ebp-0x10，passcode2的地址在ebp-0xc （地址是负偏移量的是变量）。因为0x70-0x10 == 0x60 == 96；char name[100]，所以name的最后四个字节就是passcode1的值 <p>
因为welcome和login这两个函数是连续调用的，导致他们拥有相同的栈底。<p>

![gdb](https://github.com/ormosiaL/pwnable_writeup/blob/main/img/passcode_gdb.jpg) <p> 
![gdb2](https://github.com/ormosiaL/pwnable_writeup/blob/main/img/passcode_gdb2.jpg)<p> 

---
所以，我们可以利用name的最后4个字节把 passcode1 的值改为一个地址值 a ，这样在`scanf("%d", passcode1)`时，可以把命令b 在GOT表中地址值 写入 passcode1 存的地址a中，相当于`scanf("%d",fflush@plt());` 这样就可以在程序运行到地址a处时跳转到地址b处的执行。


此外：<P>
使用该命令可复制文件到本地进行调试：`scp -P 2222 passcode@pwnable.kr:~/passcode.c ./` <p>
`scp (secure copy)`是Linux下基于 ssh 登陆进行安全的远程文件拷贝命令， `-P` 指定端口号。 `./` 复制到的位置 <p>
看了其它的wp才发现服务器中有很多工具可以直接用，比如python，gdb，objdump之类的，不用复制到本地。<p>

退出gdb 不能用`ctrl + c`退出，而是`ctrl + d` 或者输入 `q` <p>

栈内初始化为0CCCCCCCCh，也就是常见的“烫”<p>

x86汇编语言常见的两种语法：AT&T和Intel

