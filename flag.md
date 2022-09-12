## Toddler's Bottle
### flag

用ida打开后发现只有两个函数，也没有main函数。可以发现有写用了 UPX 加壳。 (题目图片是一个壳，大概能算提示吧...)<p> ![UPX](https://github.com/ormosiaL/pwnable_writeup/blob/main/img/flag_upx.jpg)<p> 用 `upx -d` 脱壳：<p> ![UPX](https://github.com/ormosiaL/pwnable_writeup/blob/main/img/flag_unpack.jpg)<p> 
再用ida打开就是很正常的样子了。伪代码意思是先分配100字节的空间，再把flag拷贝进去：<p> ![pseudo](https://github.com/ormosiaL/pwnable_writeup/blob/main/img/flag_pseudo.jpg) <p> 

`malloc() 函数`： void *malloc(size_t size) 分配所需的内存空间，并返回一个指向它的指针。<p>
点进flag就可以看到答案了。pwnable 的flag的标志是  `:)` <p>

还可以用 `Strings window` (主菜单View -> Open subviews -> strings) 直接看到带 `:)` 的字符串：<p> ![window](https://github.com/ormosiaL/pwnable_writeup/blob/main/img/flag_stringswindow.jpg) <p> 

