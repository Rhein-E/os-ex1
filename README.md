# Report

## 综述

在内核代码的 ```kernel/sys.c``` 中增加系统调用的实现，
在 ```kernel/system_call.s``` 中更改 ```nr_system_calls``` 为 ```92```，
在 ```include/unistd.h``` 中添加系统调用号，
在 ```include/linux/sys.h``` 中把系统调用加入 ```sys_call_table```。

重新编译内核并启动系统，编译并执行测试程序。

## sys_getdents 实现

### 总体思路

一个目录文件的内容即为该目录下的子目录项，因此可以通过读取当前目录文件的内容来得到当前目录下的子目录项。

### 数据交换

在内核态下只能通过 ```sys_read``` 读取文件，该函数接受的缓冲区地址为用户地址，内部通过 ```get_fs_byte``` 等函数借助段寄存器 ```fs``` 访问用户地址空间。

所以要把文件内容读入内核缓冲区，需要先调用 ```set_fs``` 临时修改段寄存器 ```fs``` 为内核数据段，完成文件读取操作后再次调用 ```set_fs``` 将其恢复。

同理，因为 ```sys_getdents``` 的参数 ```buf``` 也是用户地址，在将数据写入其中时也需要使用 ```put_fs_byte``` 等函数完成。

### 数据处理

每次从目录文件中读取一个目录项(16 bytes)，前 2 个字节为索引节点号，即 ```d_ino```；随后的 14 个字节为文件名，即 ```d_name```；```d_off``` 为文件在目录中的偏移量，可以通过前一个目录项的偏移量 + 16 得到；```d_reclen``` 为一个 ```struct linux_dirent``` 结构体的大小。

参考资料：

- [Linux0.11-内核和用户空间的数据传输](https://www.elecfans.com/emb/20190402899442.html)

## sys_getcwd 实现

已知索引节点号的情况下，可通过计算获得索引节点在硬盘中的地址，并用 ```block_read``` 读取其内容（同样需要修改段寄存器 ```fs```），进一步可知文件在硬盘中的地址，同样用 ```block_read``` 读取其内容。

通过获取 ```..``` 目录（父目录）的索引节点号，并在父目录文件中查找与子目录索引节点号相同的项，可以知道子目录的文件名。

不断向父目录追溯，直到 ```..``` 目录指向子目录本身，即到达根目录。

参考资料：

- [linux0.11源码分析-目录查找](https://juejin.cn/post/69148310594777579661)
- [block_read和block_write函数 设备块号](https://blog.csdn.net/yu704645129/article/details/50462808)

## sys_execve2 实现

参考资料：

- [Linux0.11系统调用之execve流程解析](https://blog.csdn.net/m0_37981610/article/details/117281419)
- [linux0.11内核源码剖析:第一篇 内存管理、memory.c](https://www.cnblogs.com/v-July-v/archive/2011/01/06/1983695.html)