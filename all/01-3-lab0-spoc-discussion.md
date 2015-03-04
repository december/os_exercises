# lab0 SPOC思考题

## 个人思考题

---

能否读懂ucore中的AT&T格式的X86-32汇编语言？请列出你不理解的汇编语言。
- 不能完全读懂，不理解的部分主要是之前汇编和计原课没接触过的指令、用法，例如inb、lgdt等指令，圆点的用法等。  

>  http://www.imada.sdu.dk/Courses/DM18/Litteratur/IntelnATT.htm

虽然学过计算机原理和x86汇编（根据THU-CS的课程设置），但对ucore中涉及的哪些硬件设计或功能细节不够了解？
- 中断寄存器、段寄存器及其执行相应功能的具体实现形式。

>   


哪些困难（请分优先级）会阻碍你自主完成lab实验？
- 1、课程难度太大，对操作系统知识理解可能不够充分；2、对汇编代码可能不够熟悉，会影响对代码的阅读、理解与编写；3、其它可能占用时间的杂事。

>   

如何把一个在gdb中或执行过程中出现的物理/线性地址与你写的代码源码位置对应起来？
- 在gdb中可查看当前物理地址和源码行号，如有必要可将其一起打印到文件中。  

>   

了解函数调用栈对lab实验有何帮助？
- 了解函数调用栈，能够帮助我们理解函数的调用和程序的执行过程，函数调用栈在系统实现管理内存、提供中断、内核线程管理等方面都扮演着重要角色，而这些都在lab实验中有着直接或间接的体现。

>   

你希望从lab中学到什么知识？
- 希望能够通过阅读和理解代码理解操作系统的概念、知识和实现方式，能够在实验中了解到操作系统是如何实现内存管理、资源调配、进程控制等功能的，以及如何通过操作系统将外设、CPU、应用软件等联系在一起。  

>   

---

## 小组讨论题

---

搭建好实验环境，请描述碰到的困难和解决的过程。
- 我使用的是Mac OS X系统，因此直接下载了github上的VisualBox虚拟机。在搭建过程中没有遇到太多困难，主要是解压和配置过程中由于Mac上的工具缺失遇到些问题，经过查阅资料后解决。  

> 

熟悉基本的git命令行操作命令，从github上的[ucore git repo](http://www.github.com/chyyuu/ucore_lab)下载ucore lab实验
- 熟悉了基本的git命令行操作指令，并利用init、clone命令将ucore实验下载到本地，对pull、push、fetch等其它常用指令也都能够熟练运用。  

> 

尝试用qemu+gdb（or ECLIPSE-CDT）调试lab1
- 依照视频中所演示的方法尝试进行了调试。 

> 

对于如下的代码段，请说明”：“后面的数字是什么含义
```
/* Gate descriptors for interrupts and traps */
struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
};
```
- 表示数据所占的位数。  

> 

对于如下的代码段，
```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 0,1,2,3);
```
请问执行上述指令后， intr的值是多少？
- 运行程序计算，答案为65538。（[查看代码](https://github.com/december/os_data/blob/master/test.c)）  

> 

请分析 [list.h](https://github.com/chyyuu/ucore_lab/blob/master/labcodes/lab2/libs/list.h)内容中大致的含义，并能include这个文件，利用其结构和功能编写一个数据结构链表操作的小C程序
- list是一通用的双向链表，提供添加，删除，更改的操作。尝试建立了一种新的数据结构test，然后提供了其的添加，删除查找等操作，[具体代码在此查看](https://github.com/december/os_data/blob/master/ds_link.c)。  

> 

---

## 开放思考题

---

是否愿意挑战大实验（大实验内容来源于你的想法或老师列好的题目，需要与老师协商确定，需完成基本lab，但可不参加闭卷考试），如果有，可直接给老师email或课后面谈。
- [x]  

>  

---
