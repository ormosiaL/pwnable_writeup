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

` int strcmp(const char *str1, const char *str2)` , ??????????????????????????????????????? str1 ???????????? str2, ????????????????????????????????? ????????? "\0" ?????????<p>
`argv['A']` ,??????args\[65] (??????ascii???) (???????????????argv\[10]?????????????????????????????????)<p>

- Stage 2: <p>
`int memcmp(const void *str1, const void *str2, size_t n)`, ??????????????? str1 ???????????? str2 ?????? n ??????????????????Stage 2 ????????????????????????`\x00`, ??????????????? `memcmp`??? <p>
read ????????????????????? `???????????????(fd)` : ???????????? 0 ???> ??????; ???????????? 1 ???> ?????????; ???????????? 2 ???> ????????? <p>

- Stage 3:<p>
` char * getenv(const char *name)`, ????????????name????????????????????????` int putenv(const char * string)`??????????????????????????????, string????????????"name???value",??????????????????0???????????????-1???<p>

- Stage 4:<p>
` size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream) `: ptr: ?????????????????????????????????size: ??????????????????????????????????????? nmemb:????????????????????????????????????????????????????????? size_t ????????????<p>

- Stage 5:<p>
`int socket(int af, int type, int protocol) `, af ???Address Family????????? IP ??????????????????????????? AF_INET ???IPv4 ???????????? AF_INET6???IPv6 ????????????type ?????????????????????/?????????????????????????????? SOCK_STREAM?????????????????????/??????????????????????????? ??? SOCK_DGRAM?????????????????????/??????????????????????????? protocol ?????????????????????????????????????????? af ??? type ???????????????????????????????????????????????????????????????????????????????????? `sd = socket(AF_INET, SOCK_STREAM, 0);` ???TCP ????????????<p>
`  saddr.sin_port = htons( atoi(argv['C']) ); ` ,?????? atoi()??????ascii to integer???????????????????????????????????????????????????htons() ???????????????????????????????????????????????? ( function converts the unsigned short integer hostshort from host byte order to network byte order /big-endian.) ??????????????? argv['C'] ????????????socket??????????????????????????????????????????python????????????????????????C?????????????????????...  <p>

??????????????????????????????????????????C??????????????????python????????????<p>
???????????????????????????????????????????????????????????????????????????????????????`scp -P2222 ./exp.py input2@pwnable.kr:/tmp/exp.py`
