# free()函数是如何知晓内存块大小的？

想要理解这个问题，必须先要了解一下malloc函数的实现。首先在程序运行时，系统会为它分配4G的虚拟内存，其中就包括堆（heap），堆是可在运行时（为变量）动态进行内存分配的一块区域，堆顶端称作program break。malloc分配的内存允许随意释放，它们被维护于一张空闲内存列表中，在后续内存分配调用时循环使用。malloc()的实现很简单。它首先会扫描之前由 free()所释放的空闲内存块列表，以求找到尺寸大于或等于要求的一块空闲内存。（取决于具体实现，采用的扫描策略会有所不同。例如，first-fit 或 best-fito。）如果这一内存块的尺寸正好与要求相当，就把它直接返回给调用者。如果是一块较大的内存，那么将对其进行分割，在将一块大小相当的内存返回给调用者的同时，把较小的那块空闲内存块保留在空闲列表中。如果在空闲内存列表中根本找不到足够大的空闲内存块，那么 malloc()会调用 sbrk()以分配更多的内存。为减少对 sbrk()的调用次数，malloc()并未只是严格按所需字节数来分配内存，而是以更大幅度（以虚拟内存页大小的数倍）来增加 program break，并将超出部分置于空闲内存列表。当 malloc()分配内存块时，会额外分配几个字节来存放记录这块内存大小的整数值。该整数位于内存块的起始处，而实际返回给调用者的内存地址恰好位于这一长度记录字节之后。所以 free()将内存块置于空闲列表之上时，通过访问内存起始处的保存的长度便可以知道内存的大小。

![在这里插入图片描述](https://img-blog.csdnimg.cn/202004021526124.png#pic_center)

当将内存块置于空闲内存列表（双向链表）时，free()会使用内存块本身的空间来存放链表指针，将自身添加到列表中。


![malloc返回的内存块](https://img-blog.csdnimg.cn/20200402152529991.png#pic_center)

C语言允许程序创建指向堆中任意位置的指针，并修改其指向的数据，包括由 free()和 malloc()函数维护的内存块长度、指向前一空闲块和后一空闲块的指针。所以指针错误使用往往造成不可预料的结果。