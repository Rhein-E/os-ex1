# os

1.	新的系统调用号定义在../include/unistd.h中。

2.	为了在Linux 0.11中顺利进行make，需要在bochs虚拟机中对硬盘作一些调整，如下：

	```sh
	cd /bin
	mv sh sh.old
	ln bash sh
	```

3.	编译
	编译所有文件：

	```sh
	make
	```

	编译某个文件，如：

  	```sh
	make execve2
	```

4.	测试

	```sh
	make test
	```

5.	清除中间文件

	```sh
	make clean
	```
