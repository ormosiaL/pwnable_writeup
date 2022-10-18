## Toddler's Bottle
### input

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>

int main(int argc, char* argv[], char* envp[]){
        printf("Welcome to pwnable.kr\n");
        printf("Let's see if you know how to give input to program\n");
        printf("Just give me correct inputs then you will get the flag :)\n");

        // argv
        if(argc != 100) return 0;
        if(strcmp(argv['A'],"\x00")) return 0;
        if(strcmp(argv['B'],"\x20\x0a\x0d")) return 0;
        printf("Stage 1 clear!\n");

        // stdio
        char buf[4];
        read(0, buf, 4);
        if(memcmp(buf, "\x00\x0a\x00\xff", 4)) return 0;
        read(2, buf, 4);
        if(memcmp(buf, "\x00\x0a\x02\xff", 4)) return 0;
        printf("Stage 2 clear!\n");

        // env
        if(strcmp("\xca\xfe\xba\xbe", getenv("\xde\xad\xbe\xef"))) return 0;
        printf("Stage 3 clear!\n");

        // file
        FILE* fp = fopen("\x0a", "r");
        if(!fp) return 0;
        if( fread(buf, 4, 1, fp)!=1 ) return 0;
        if( memcmp(buf, "\x00\x00\x00\x00", 4) ) return 0;
        fclose(fp);
        printf("Stage 4 clear!\n");

        // network
        int sd, cd;
        struct sockaddr_in saddr, caddr;
        sd = socket(AF_INET, SOCK_STREAM, 0);
        if(sd == -1){
                printf("socket error, tell admin\n");
                return 0;
        }
        saddr.sin_family = AF_INET;
        saddr.sin_addr.s_addr = INADDR_ANY;
        saddr.sin_port = htons( atoi(argv['C']) );
        if(bind(sd, (struct sockaddr*)&saddr, sizeof(saddr)) < 0){
                printf("bind error, use another port\n");
                return 1;
        }
        listen(sd, 1);
        int c = sizeof(struct sockaddr_in);
        cd = accept(sd, (struct sockaddr *)&caddr, (socklen_t*)&c);
        if(cd < 0){
                printf("accept error, tell admin\n");
                return 0;
        }
        if( recv(cd, buf, 4, 0) != 4 ) return 0;
        if(memcmp(buf, "\xde\xad\xbe\xef", 4)) return 0;
        printf("Stage 5 clear!\n");

        // here's your flag
        system("/bin/cat flag");
        return 0;
}
```

- Stage 1: <p>

` int strcmp(const char *str1, const char *str2)` , 自左向右逐个字符比较字符串 str1 和字符串 str2, 直到出现不同的字符或遇 结束符 "\0" 为止。<p>
`argv['A']` ,表示args\[65] (根据ascii表) (如果想表示argv\[10]应该不加单引号？还没试)<p>

- Stage 2: <p>
`int memcmp(const void *str1, const void *str2, size_t n)`, 比较内存块 str1 和内存块 str2 的前 n 个字节。因为Stage 2 中比较的字符串有`\x00`, 所以用的是 `memcmp`。 <p>
read 的第一个变量是 `文件描述符(fd)` : 标准输入 0 —> 键盘; 标准输出 1 —> 显示器; 标准错误 2 —> 显示器 <p>

- Stage 3:<p>
` char * getenv(const char *name)`, 获取参数name环境变量的内容。` int putenv(const char * string)`：改变或增加环境变量, string的格式为"name＝value",执行成功返回0，否则返回-1；<p>

- Stage 4:<p>
` size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream) `: ptr: 读取数据后储存的地址，size: 读取的每个元素的字节大小， nmemb:读取元素的个数。成功读取的元素总数会以 size_t 对象返回<p>

- Stage 5:<p>
`int socket(int af, int type, int protocol) `, af （Address Family）表示 IP 地址类型，常用的有 AF_INET （IPv4 地址）和 AF_INET6（IPv6 地址）；type 为数据传输方式/套接字类型，常用的有 SOCK_STREAM（流格式套接字/面向连接的套接字） 和 SOCK_DGRAM（数据报套接字/无连接的套接字）； protocol 表示传输协议。一般情况下有了 af 和 type 两个参数操作系统就会自动推演出协议类型创建套接字。本题的 `sd = socket(AF_INET, SOCK_STREAM, 0);` 是TCP 套接字。<p>
`  saddr.sin_port = htons( atoi(argv['C']) ); ` ,其中 atoi()表示ascii to integer，即“把字符串转换成有符号整数”；htons() 把无符号整数从小端序转换为大端序 ( function converts the unsigned short integer hostshort from host byte order to network byte order /big-endian.) 即开一个以 argv['C'] 作端口的socket，再传入规定的值。不得不说用python做这个网络连接比C容易得多（大概...  <p>

接下来就是写代码，似乎这题用C更合适，不过python更简单。<p>
写好代码后可以拷贝到服务器上运行（或者直接在服务器端写？：`scp -P2222 ./exp.py input2@pwnable.kr:/tmp/exp.py`
