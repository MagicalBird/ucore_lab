# Lab2实验报告

2011011384 计34 郑玉昆

## 练习1

#### 1.1 请在实验报告中简要说明你的设计实现过程。

``default_init`` : 初始化free_list链表和nr_free空闲页个数。

``default_init_memmap`` : 对内存中空闲页进行初始化，将n个空闲页加入到空闲页的列表中，对于非保留页设置标志位表明该页有效，property的值表示以该页开头有多少个连续的页，除了base外都为0，先统一设置成0，将页按顺序加到链表的末尾，再按需修改首页（也就是base）的property和nr_free的值，用来表示此段连续的空闲内存空间。

``default_alloc_pages`` : 如果当前总的空闲页总数nr_free比需求n小的话，返回空，表示内存不足，分配失败，否则从环形链表free_list的第一个开始遍历空闲页链表，知道找到一个大于等于n的连续空闲块，若遍历一遍后仍未找到则返回空。假设找到了，那么将该内存块中的对于分配的项，设置标志表明它们已经被使用，在空闲页列表里删除当前页。如果当前空闲块的页数大于n，那么分配n个页后剩下的第一个页为新的块的形状，它的property比原来的小n。

``default_free_pages`` : 这个函数的作用是释放从base开始的n个连续页，并对空闲块做合并工作。按顺序寻找，找到第一个地址大于base的页的前面作为释放的块应插入的位置。对于要释放的每一个页，置引用个数为0，清除使用标志，并将它们重新加入的空闲链表当中。当前块是否能和后面的块进行合并，如果能，修改它们的property值，完成合并。

#### 1.2 你的first fit算法是否有进一步的改进空间？

在进行分配时，算法需要遍历每一个空闲页，但实际可以只遍历每个空闲块，判断是否符合大小要求即可，所以可以考虑改进存储方式，将空闲页按连续块形式进行存储，而每一块中则只需要记录第一页的起始地址和块中页的个数即可，这样在遍历的时候可以提高效率。

## 练习2

#### 2.1 请在实验报告中简要说明你的设计实现过程。

实现过程：实现函数``get_pte(pde_t *pgdir, uintptr_t la, bool create)``

首先要查询一级页表，根据线性地址la在一级页表pgdir中寻找二级页表的起始地址。如果二级页表不存在，那么要根据参数create来判断是否分配二级页表的内存空间，若不分配则返回空。

get_pte函数是通过PDT的基址pgdir和线性地址la来获取pte。PDX根据la获取其页目录的索引，根据此索引可以得到页目录项pde，由于可能对其进行修改，这里采用指向pde的指针pdep，而*pdep中保存的便是pde的真实内容。创建了pde后，需要返回的值是pte的指针，这里先将pde中的地址转化为程序可用的虚拟地址。将这个地址转化为pte数据类型的指针，然后根据la的值索引出对应的pte表项。

#### 2.2 请描述页目录项（Pag Director Entry）和页表（Page Table Entry）中每个组成部分的含义和以及对ucore而言的潜在用处。

PDE和PTE的组成（详见mmu.h文件）

  ```C
  #define PTE_P           0x001 // 当前项是否存在，用于判断缺页
  #define PTE_W           0x002 // 当前项是否可写，标志权限
  #define PTE_U           0x004 // 用户是否可获取，标志权限
  #define PTE_PWT         0x008 // 写直达缓存机制,硬件使用Write Through
  #define PTE_PCD         0x010 // 禁用缓存，硬件使用Cache-Disable
  #define PTE_A           0x020 // Accessed
  #define PTE_D           0x040 // 页是否被修改，硬件使用（dirty)
  #define PTE_PS          0x080 // 页大小
  #define PTE_MBZ         0x180 // 必须为0的位
  #define PTE_AVAIL       0xE00 // 软件使用的位，可任意设置
  ```

#### 2.3 如果ucore执行过程中访问内存，出现了页访问异常，请问硬件要做哪些事情？

出现了页访问异常的话，硬件需要根据设置的IDT找到对应异常的处理例程的入口，然后跳转到该处理例程处理该异常。

## 练习3

#### 3.1 请在实验报告中简要说明你的设计实现过程。

主要实现page_remove_pte函数：释放la地址所指向的页，并设置对应的pte的值.首先确保页存在,找到pte所在的页,把pte所在页的ref减一。如果该页ref为0，则释放pte所在的页，再清空TLB。


#### 3.2 数据结构Page的全局变量（其实是一个数组）的每一项与页表中的页目录项和页表项有无对应关系？如果有，其对应关系是啥？

有关系。Page的全局变量pages对应的是每一个页的虚拟地址，而页表项和页目录项都指向一个页，它们保存的是页的物理地址，通过pte2page、pde2page可以将pte、pte中保存的页映射到page中的虚拟地址对应的页。

#### 3.3 如果希望虚拟地址与物理地址相等，则需要如何修改lab2，完成此事？ 鼓励通过编程来具体完成这个问题

可能需要修改pages变量的地址，把虚拟地址映射到0处就实现了page的虚拟地址等于物理地址。

## 运行的结果

```
Kernel executable memory footprint: 99KB
ebp:0xc0116f38 eip:0xc01009d0 args:0x00010094 0x00000000 0xc0116f68 0xc01000bc 
    kern/debug/kdebug.c:310: print_stackframe+21
ebp:0xc0116f48 eip:0xc0100cbf args:0x00000000 0x00000000 0x00000000 0xc0116fb8 
    kern/debug/kmonitor.c:129: mon_backtrace+10
ebp:0xc0116f68 eip:0xc01000bc args:0x00000000 0xc0116f90 0xffff0000 0xc0116f94 
    kern/init/init.c:49: grade_backtrace2+33
ebp:0xc0116f88 eip:0xc01000e5 args:0x00000000 0xffff0000 0xc0116fb4 0x00000029 
    kern/init/init.c:54: grade_backtrace1+38
ebp:0xc0116fa8 eip:0xc0100103 args:0x00000000 0xc010002a 0xffff0000 0x0000001d 
    kern/init/init.c:59: grade_backtrace0+23
ebp:0xc0116fc8 eip:0xc0100128 args:0xc0105f5c 0xc0105f40 0x00000f32 0x00000000 
    kern/init/init.c:64: grade_backtrace+34
ebp:0xc0116ff8 eip:0xc010007f args:0x00000000 0x00000000 0x0000ffff 0x40cf9a00 
    kern/init/init.c:29: kern_init+84
memory management: default_pmm_manager
e820map:
  memory: 0009fc00, [00000000, 0009fbff], type = 1.
  memory: 00000400, [0009fc00, 0009ffff], type = 2.
  memory: 00010000, [000f0000, 000fffff], type = 2.
  memory: 07efe000, [00100000, 07ffdfff], type = 1.
  memory: 00002000, [07ffe000, 07ffffff], type = 2.
  memory: 00040000, [fffc0000, ffffffff], type = 2.
check_alloc_page() succeeded!
check_pgdir() succeeded!
check_boot_pgdir() succeeded!
-------------------- BEGIN --------------------
PDE(0e0) c0000000-f8000000 38000000 urw
  |-- PTE(38000) c0000000-f8000000 38000000 -rw
PDE(001) fac00000-fb000000 00400000 -rw
  |-- PTE(000e0) faf00000-fafe0000 000e0000 urw
  |-- PTE(00001) fafeb000-fafec000 00001000 -rw
--------------------- END ---------------------
++ setup timer interrupts
```

由于本人能力有限，代码是在借鉴参考答案的基础上完成的，主要工作量在看懂和理解了ucore以及答案中的实现方法，具体代码部分与标准答案并无太大区别。


