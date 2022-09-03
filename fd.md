## Toddler's Bottle
### fd
题目的`fd`和题目描述中的` file descriptor in Linux`提示此题的考查点为 file descriptor <br>
连接到题目环境后，使用`ls -la` 列出目录下的所有文件(包括以 . 开头的隐含文件)`ls -a`和文件的属性`ls -a`<br> ![ls](https://github.com/ormosiaL/pwnable_writeup/blob/main/img/fd_1.jpg) <br>
Linux 中第一个字符代表这个文件的类型：d 表示是文件夹， - 表示是文件;接下来以三个字符为一组，连续三组表示文件的权限<br>
`rwx`分别表示读、写、执行；特殊权限`t`（Sticky），一般用在目录上，使得用户可以写入文档，但不能删除这个目录下他人的文档；特殊权限`s`（SUID,Set UID），即使得文件在执行阶段具有文件所有者的权限，可以利用它得到root权限。<br>![file descriptor](https://github.com/ormosiaL/pwnable_writeup/blob/main/img/fd.png) <br>
**观察题目中的文件发现flag需要root权限，所以需要利用fd文件的执行得到root权限，而且题目又提供了 fd.c 便于我们查看源码来利用fd文件**<br>![sourcecode](https://github.com/ormosiaL/pwnable_writeup/blob/main/img/fd_2.jpg) <br>
`main函数`中，argc 指传入参数的个数，argv[] 是一个指针数组，指向传递给程序的每个参数，envp 是系统的环境变量。argv[0] 存储程序的名称，argv[1]指第一个命令行参数<br> `atoi函数`将字符串转换为一个整数<br>`read函数`中，三个参数依次为要读取的文件，保存读取内容的缓冲区及读取文件的长度 <br>`strcmp函数`比较两个字符串，相等时返回0<br>
**C/C++中0为false，非零值为true，则我们需要使得 buff == LETMEWIN 即可执行 /bin/cat flag**<br>令文件df == 0，使得我们可以自由控制输入进buff的值，又0x1234 转换为十进制为4660。payload: `./fd 4660 `<br>![payload](https://github.com/ormosiaL/pwnable_writeup/blob/main/img/fd_3.jpg) <br>



