# lab1 SPOC思考题

## 个人思考题

NOTICE
- 有"w2l2"标记的题是助教要提交到学堂在线上的。
- 有"w2l2"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。
---

请描述ucore OS配置和驱动外设时钟的准备工作包括哪些步骤？ (w2l2)
```
  + 采分点：说明了ucore OS在让外设时钟正常工作的主要准备工作
  - 答案没有涉及如下3点；（0分）
  - 描述了对IDT的初始化，包了针时钟中断的中断描述符的设置（1分）
  - 除第二点外，进一步描述了对8259中断控制器的初始过程（2分）
  - 除上述两点外，进一步描述了对8253时钟外设的初始化，或描述了对EFLAG操作使能中断（3分）
 ```
- 阅读init.c中的代码，以下一段即为主要准备工作：

```
kern_init(void){
    extern char edata[], end[];
    memset(edata, 0, end - edata);

    cons_init();                // init the console

    const char *message = "(THU.CST) os is loading ...";
    cprintf("%s\n\n", message);

    print_kerninfo();

    grade_backtrace();

    pmm_init();                 // init physical memory management

    pic_init();                 // init interrupt controller
    idt_init();                 // init interrupt descriptor table

    clock_init();               // init clock interrupt
    intr_enable();              // enable irq interrupt

    //LAB1: CAHLLENGE 1 If you try to do it, uncomment lab1_switch_test()
    // user/kernel mode switch test
    lab1_switch_test();

    /* do nothing */
    while (1);
}）
 ```
- 注意到，准备工作主要包括初始化内存管理（pmm_init()）、8259中断控制器（pic_init()）、中断描述符表（idt_init()）、8253时钟外设（clock_init()）。
- 对于IDT（中断描述符表），阅读idt_init()函数，首先引用一个在vectors.S中定义的_vector，里面存放所有中断描述符对应的跳转地址。中断描述符表的每一项用gatedesc结构表示，用SETGATE函数将各个参数填入结构中再作为一项加入到IDT中（for循环中为内核态，for循环外为用户态），最后用lidt命令通知系统已经就绪。
- 对于8259中断控制器，阅读pic_init()函数，主要通过多个内联汇编outb命令设定各种中断源的优先级，并能够利用mask对中断源进行屏蔽。
- 对于8253时钟外设，阅读clock_init()函数，同样通过内联汇编outb命令初始化8253芯片，之后初始化时间计数器tick，打印相关信息后打开时钟中断请求使能。


>  

lab1中完成了对哪些外设的访问？ (w2l2)
 ```
  + 采分点：说明了ucore OS访问的外设
  - 答案没有涉及如下3点；（0分）
  - 说明了时钟（1分）
  - 除第二点外，进一步说明了串口（2分）
  - 除上述两点外，进一步说明了并口，或说明了CGA，或说明了键盘（3分）
 ```
- 完成了对时钟、串口、并口、CGA、键盘等外设的访问。 
- 以时钟、串口、键盘为例，阅读lab1中的trap.c和trap.h代码，注意到以下部分：
 ```
 //from trap.h
#define IRQ_TIMER                0
#define IRQ_KBD                    1
#define IRQ_COM1                4

//from trap.c
trap_dispatch(struct trapframe *tf) {
    char c;

    switch (tf->tf_trapno) {
    case IRQ_OFFSET + IRQ_TIMER:
        /* LAB1 YOUR CODE : STEP 3 */
        /* handle the timer interrupt */
        /* (1) After a timer interrupt, you should record this event using a global variable (increase it), such as ticks in kern/driver/clock.c
         * (2) Every TICK_NUM cycle, you can print some info using a funciton, such as print_ticks().
         * (3) Too Simple? Yes, I think so!
         */
        ticks ++;
        if (ticks % TICK_NUM == 0) {
            print_ticks();
        }
        break;
        break;
    case IRQ_OFFSET + IRQ_COM1:
        c = cons_getc();
        cprintf("serial [%03d] %c\n", c, c);
        break;
    case IRQ_OFFSET + IRQ_KBD:
        c = cons_getc();
        cprintf("kbd [%03d] %c\n", c, c);
        break;
    //LAB1 CHALLENGE 1 : YOUR CODE you should modify below codes.
    case T_SWITCH_TOU:
    case T_SWITCH_TOK:
        panic("T_SWITCH_** ??\n");
        break;
    case IRQ_OFFSET + IRQ_IDE1:
    case IRQ_OFFSET + IRQ_IDE2:
        /* do nothing */
        break;
    default:
        // in kernel, it must be a mistake
        if ((tf->tf_cs & 3) == 0) {
            print_trapframe(tf);
            panic("unexpected trap in kernel.\n");
        }
    }
}
 ```
- 由以上代码可见，通过中断访问外设，IRQ_TIMER、IRO_KBD、IRQ_COM1分别代表时钟、键盘、串口的编号，通过switch来访问具体的外设。对于时钟，每次收到信号时tick加一，到100时触发中断（TICK_NUM = 100）；对于串口和键盘，都是调用cons_getc()函数，该函数是从类似总线的数据通路上读取数据，键盘或串口将数据输入到该通路上，从而触发中断并输出信息。
- 对代码进行make qemu操作，在qemu中可以看到tick＝100时屏幕上会打印出时钟中断的信息，按下键盘上按键时屏幕也会打印键盘中断的信息。

>  

lab1中的cprintf函数最终通过哪些外设完成了对字符串的输出？ (w2l2)
 ```
  + 采分点：说明了cprintf函数用到的3个外设
  - 答案没有涉及如下3点；（0分）
  - 说明了串口（1分）
  - 除第二点外，进一步说明了并口（2分）
  - 除上述两点外，进一步说明了CGA（3分）
 ```
- 通过串口、并口、CGA完成了对字符串的输出。  

>  

---

## 小组思考题

---

lab1中printfmt函数用到了可变参，请参考写一个小的linux应用程序，完成实现定义和调用一个可变参数的函数。(spoc)
- [x]  



如果让你来一个阶段一个阶段地从零开始完整实现lab1（不是现在的填空考方式），你的实现步骤是什么？（比如先实现一个可显示字符串的bootloader（描述一下要实现的关键步骤和需要注意的事项），再实现一个可加载ELF格式文件的bootloader（再描述一下进一步要实现的关键步骤和需要注意的事项）...） (spoc)
- 先实现一个可显示字符串的bootloader，并能切换进入保护模式。需要清理环境，将flag标志和段寄存器置0，开启A20，初始化GDT表，通过将cr0寄存器PE位置1便开启了保护模式，用ljmp更新CS的基地址，设置段寄存器并建立堆栈。
- 再实现一个可加载ELF格式文件的bootloader，要能够访问硬盘，从设备的指定扇区读取数据到指定位置，能够通过读取ELF文件头来将ELF文件中的数据载入内存。
- 然后实现ucore中的函数调用栈，包括规定栈结构内存空间，完成相应空间的初始化，实现参数压栈的约定，需要注意函数调用栈必须能够体现函数的调用关系，能够完成现场保存与恢复工作，保证在嵌套调用时不出错。
- 然后实现ucore中的中断控制与处理，需要初始化中断控制器、中断描述符表、时钟以及其它外设等，收到中断时，能够根据中断优先级和中断号进行响应的处理。
- 最后完成其它细节，例如内核态与用户态的切换等。

> 


如何能获取一个系统调用的调用次数信息？如何可以获取所有系统调用的调用次数信息？请简要说明可能的思路。(spoc)
- 许多操作系统都支持一些命令或调试工具，能够跟踪一个进程的系统调用或信号产生的情况，有的可以跟踪进程调用库函数的情况。例如上次作业中用到的linux下的strace命令，就能跟踪进程执行时的系统调用和所接收的信号，能够获取该进程所有系统调用的调用次数信息，又能得到其中每个系统调用的返回值、次数、执行消耗的时间等参数。类似的还有很多unix系统上的truss命令、Itrace命令等。
- 就实验用的ucore系统来说，由于我们有操作系统的代码，可以在内核代码上进行修改，以获取一个系统调用的调用次数和所有系统调用的调用次数。例如在syscall.c中添加代码，对每个系统调用维护一个计数器，统计其调用次数；并维护一个全局计数器，统计所有系统调用的调用次数；在调用系统调用时打印当前调用次数，或是在trap.c等文件中统一添加打印次数的函数等，都是可行的实现思路。

> 

如何修改lab1, 实现一个可显示字符串"THU LAB1"且依然能够正确加载ucore OS的bootloader？如果不能完成实现，请说明理由。
- [x]  

> 

对于ucore_lab中的labcodes/lab1，我们知道如果在qemu中执行，可能会出现各种稀奇古怪的问题，比如reboot，死机，黑屏等等。请通过qemu的分析功能来动态分析并回答lab1是如何执行并最终为什么会出现这种情况？
- [x]  

> 

对于ucore_lab中的labcodes/lab1,如果出现了reboot，死机，黑屏等现象，请思考设计有效的调试方法来分析常在现背后的原因。
- [x]  

> 

---

## 开放思考题

---

如何修改lab1, 实现在出现除零错误异常时显示一个字符串的异常服务例程的lab1？
- [x]  

> 


在lab1/bin目录下，通过`objcopy -O binary kernel kernel.bin`可以把elf格式的ucore kernel转变成体积更小巧的binary格式的ucore kernel。为此，需要如何修改lab1的bootloader, 能够实现正确加载binary格式的ucore OS？ (hard)
- [x]  

>

GRUB是一个通用的bootloader，被用于加载多种操作系统。如果放弃lab1的bootloader，采用GRUB来加载ucore OS，请问需要如何修改lab1, 能够实现此需求？ (hard)
- [x]  

>


如果没有中断，操作系统设计会有哪些问题或困难？在这种情况下，能否完成对外设驱动和对进程的切换等操作系统核心功能？
- [x]  

>  

---

## 各知识点的小练习

### 4.1 启动顺序
---
读入ucore内核的代码？

- [x]  

> 

跳转到ucore内核的代码？

- [x]  

> 

全局描述符表的初始化代码？

- [x]  

> 

GDT内容的设置格式？初始映射的基址和长度？特权级的设置位置？

- [x]  

> 

可执行文件格式elf的各个段的数据结构？

- [x]  

> 

如果ucore内核的elf是否要求连续存放？为什么？

- [x]  

> 
---

### 4.2 C函数调用的实现
---

函数调用的stackframe结构？函数调用的参数传递方法有哪几种？
- [x]  

> 

系统调用的stackframe结构？系统调用的参数传递方法有哪几种？

- [x]  

> 
---

### 4.3 GCC内联汇编
---

使用内联汇编的原因？

- [x]  

> 特权指令、性能优化

对ucore中的一段内联汇编进行完整的解释？

- [x]  

> 
---

### 4.4 x86中断处理过程
---

4.4 x86中断处理过程

中断描述符表IDT的结构？

- [x]  

> 

中断描述表到中断服务例程的地址计算过程？

- [x]  

> 

中断处理中硬件压栈内容？用户态中断和内核态中断的硬件压栈有什么不同？

- [x]  

> 

中断处理中硬件保存了哪些寄存器？

- [x]  

> 
---

### 4.5 练习一 ucore编译过程
---

gcc编译、ld链接和dd生成两个映像对应的makefile脚本行？

- [x]  

> 
---

### 4.6 练习二 qemu和gdb的使用
---

qemu的命令行参数含义解释？

- [x]  

> 

gdb命令格式？反汇编、运行、断点设置

- [x]  

> 
---

### 练习三 加载程序
---

A20的使能代码分析？

- [x]  

> 

生成主引导扇区的过程分析？

- [x]  

> 

保护模式的切换代码？

- [x]  

> 

如何识别elf格式？对应代码分析？

- [x]  

> 

跳转到elf的代码？

- [x]  

> 
函数调用栈获取？

- [x]  

> 
---

### 4.8 练习四和五 ucore内核映像加载和函数调用栈分析
---

如何识别elf格式？对应代码分析？

- [x]  

> 

跳转到elf的代码？

- [x]  

> 

函数调用栈获取？
- [x]  

> 
---


### 4.9 练习六 完善中断初始化和处理
---

各种设备的中断初始化？
- [x]  

> 
中断描述符表IDT的排列顺序？
- [x]  

> 中断号
CPU加电初始化后中断是使能的吗？为什么？

- [x]  

> 
中断服务例程的入口地址在什么地方设置的？

- [x]  

> 
alltrap的中断号是在哪写入到trapframe结构中的？

- [x]  

> 
trapframe结构？
- [x]  

> 
---
