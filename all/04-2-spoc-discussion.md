#lec9 虚存置换算法spoc练习

## 小组思考题目

----
(1)（spoc）请证明为何LRU算法不会出现belady现象

```
假设采用栈来实现LRU算法，设S为物理页面数量为n的LRU算法维护的栈，Sk为物理页面数量为n+k（k为正整数）的LRU算法维护的栈。只需证明对同一访存序列、在任意时刻，S包含于Sk，且S中任意元素在S中的位置小于等于在Sk中的位置，即可证明Sk的缺页率不高于S的缺页率。

起初S与Sk都为空，满足条件。
假设在t时刻满足条件，则在t+1时刻，假设此时访问页面a，则有以下几种可能的情况：
1、a在S中，a在Sk中，则S和Sk没有发生任何改变，仍然满足条件；
2、a不在S中，a不在Sk中，则S和Sk均需将栈底元素替换掉，并将a放入栈顶。设S栈底为x，Sk栈底为y，则弹出后的状态分别为S-x+a和Sk-y+a，由于弹出前满足条件，易知除非Sk弹出的y仍然在S中存在，否则仍然满足条件。假设y仍在S中存在，由于S弹出了x，可知x在y之前访问到；由于两者访问序列相同，因此在Sk中x也在y之前访问到，因此在弹出y之前x已被弹出。而归纳假设有S包含于Sk，因此x也在Sk中，矛盾，故这种情况仍然满足条件；
3、a不在S中，a在Sk中，则S替换栈底并将a放入栈顶，由于a本来已在Sk中，且Sk中元素位置不变，S1中元素整体前移一位，因此仍然满足条件；
4、a在S中，a不在Sk中，显然与归纳假设矛盾。

由数学归纳法可知，Sk的缺页率不高于S的缺页率，LRU算法不会出现belady现象。
```

(2)（spoc）根据你的`学号 mod 4`的结果值，确定选择四种替换算法（0：LRU置换算法，1:改进的clock 页置换算法，2：工作集页置换算法，3：缺页率置换算法）中的一种来设计一个应用程序（可基于python, ruby, C, C++，LISP等）模拟实现，并给出测试。请参考附件代码或独自实现。

代码在[这里](https://github.com/december/os_data/blob/master/w5l1.cpp)查看

## 扩展思考题
（1）了解LIRS页置换算法的设计思路，尝试用高级语言实现其基本思路。此算法是江松博士（导师：张晓东博士）设计完成的，非常不错！

参考信息：

 - [LIRS conf paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang02_LIRS.pdf)
 - [LIRS journal paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang05_LIRS.pdf)
 - [LIRS-replacement ppt1](http://dragonstar.ict.ac.cn/course_09/XD_Zhang/(6)-LIRS-replacement.pdf)
 - [LIRS-replacement ppt2](http://www.ece.eng.wayne.edu/~sjiang/Projects/LIRS/sig02.ppt)
