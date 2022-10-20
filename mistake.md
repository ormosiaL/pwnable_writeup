## Toddler's Bottle
### mistake

```
#include <stdio.h>
#include <fcntl.h>

#define PW_LEN 10
#define XORKEY 1

void xor(char* s, int len){
        int i;
        for(i=0; i<len; i++){
                s[i] ^= XORKEY;
        }
}

int main(int argc, char* argv[]){

        int fd;
        if(fd=open("/home/mistake/password",O_RDONLY,0400) < 0){
                printf("can't open password %d\n", fd);
                return 0;
        }

        printf("do not bruteforce...\n");
        sleep(time(0)%20);

        char pw_buf[PW_LEN+1];
        int len;
        if(!(len=read(fd,pw_buf,PW_LEN) > 0)){
                printf("read error\n");
                close(fd);
                return 0;
        }

        char pw_buf2[PW_LEN+1];
        printf("input password : ");
        scanf("%10s", pw_buf2);

        // xor your input
        xor(pw_buf2, 10);

        if(!strncmp(pw_buf, pw_buf2, PW_LEN)){
                printf("Password OK\n");
                system("/bin/cat flag\n");
        }
        else{
                printf("Wrong Password\n");
        }

        close(fd);
        return 0;
}
```

based on the hint it given : operator priority, the mistake is in this part:
```
       if(fd=open("/home/mistake/password",O_RDONLY,0400) < 0){
                printf("can't open password %d\n", fd);
                return 0;
        }

```
The first line will do ` read(fd,pw_buf,PW_LEN) ` first, The return value of open() is a file descriptor, a small,nonnegative integer(the lowest-numbered file descriptor not currently open for the process). Then it become ` fd = 1 < 0`,but ` > ` has higher priority than ` = `, it will turn into `fd = 0` since `1 < 0` return false which is 0. In the end `fd = 0` means stdin or read from screen.<p>
The password will be what we type in instead of the what in the password file. Then we need to input for pw_buf2. SInce the pw_buf2 will do xor before be compared with the password, we just need to input a 10 length string then input its xor.<p>
payload : `1111111111` and `0000000000`