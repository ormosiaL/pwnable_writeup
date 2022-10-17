## Toddler's Bottle
### random

```
#include <stdio.h>

int main(){
        unsigned int random;
        random = rand();        // random value!

        unsigned int key=0;
        scanf("%d", &key);

        if( (key ^ random) == 0xdeadbeef ){
                printf("Good!\n");
                system("/bin/cat flag");
                return 0;
        }

        printf("Wrong, maybe you should try 2^32 cases.\n");
        return 0;
}
```

`rand()` 函数的种子默认为1，所以程序每次运行产生的随机数`key`其实是一样的。写一个类似的程序打印出random的值：<p>

```
#include <stdio.h>

int main(){
        unsigned int random;
        random = rand();

        printf("random is %x\n",random);
        return 0;
}
```

运行得到 `random is 6b8b4567`，由于进行两次异或操作得到的就是原始的值，所以将 0x6b8b4567 和 0xdeadbeef 异或得到 0xb526fb88，注意要转成10进制3039230856 (因为scanf("%d")。<p>

此外：
输入 logout 断开链接, 不要每次直接把终端关掉...<p>
LOWBYTE 取16bit数的最低byte(8bit)，对于十进制数字0-256，即取它本身。

