## Toddler's Bottle
### collision

与前一题fd类似, 通过执行col程序得到root可读的flag<br>![ls](https://github.com/ormosiaL/pwnable_writeup/blob/main/img/collision_1.jpg) <br>![sourcecode](https://github.com/ormosiaL/pwnable_writeup/blob/main/img/collision_2.jpg) <br>

**main函数需要输入一个长20bytes的字符串，经过 check_password 后等于 0x21DD09EC** <br> 

对于`check_password 函数`，由于C语言中int是4个字节，char是1个字节, 所以输入的20bytes会被转换成5个int，相加后得到返回值res。<br> 又因为0x21DD09EC十进制是568134124不能被5整除，加1可以被整除为5个0x6C5CEC9，再将其中一个减1 (或者用其他方式凑出来)。即0x21DD09EC = 0x6C5CEC9 + 0x6C5CEC9 + 0x6C5CEC9 + 0x6C5CEC9 + 0x6C5CEC8 <br>
注意：
- 1 bytes = 8 bits 所以 1byte 在二进制中从0000 0000 到 1111 1111， 在十进制中从0到255，在十六进制中从 0x00 到 0xFF。<br>
- 又因为我们使用的Intel CPU和C语言的编译器都是小端序，小端序数据的低字节保存在内存的低地址中，即与人类的阅读习惯相反。所以输入时要以4个bytes为单位倒序输入。<br> 

即 payload ： ` \xc9\xce\xc5\x06\xc9\xce\xc5\x06\xc9\xce\xc5\x06\xc9\xce\xc5\x06\xc8\xce\xc5\x06 `<br>
此时会发现0x06是不可打印字符，无法通过命令行输入。使用此命令 `` `python -c print "\xAB"` ``可以在命令行生成直接操作内存的十六进制字节<br>
其中 python -c 用来在命令行中执行 python 代码, print 输出格式化后的字符串，用 \` 符号包裹，表示显示该命令返回值 <br> 
最终payload : `` ./col `python -c 'print "\xc9\xce\xc5\x06\xc9\xce\xc5\x06\xc9\xce\xc5\x06\xc9\xce\xc5\x06\xc8\xce\xc5\x06"'` ``
<br>![payload](https://github.com/ormosiaL/pwnable_writeup/blob/main/img/collision_3.jpg) 
