# Lab5 实验报告-阶段一

myl7 ***REMOVED***

yuanyiwei ***REMOVED***

## 实验要求

理解 `Mem2Reg` 与 `LoopSearch` 两种 Pass 优化，回答思考题

## 思考题

### LoopSearch

1. 循环的入口如何确定？循环的入口的数量可能超过 1 嘛？

根据有向图强连通分量，可以找出强连通的 sccs 和每张图中的 base node。

循环的入口就是条件语句，不会超过 1。

2. 简述一下算法怎么解决循环嵌套的情况

每次找到一个 loop 后，我们删去 loop base，确保此 loop 的环被破开，再在剩余的 BB 中搜索 loop，就能找到内部的 loop。

### Mem2reg

1. 请简述支配边界的概念

支配边界的概念由一个变量拥有，表示这个变量能支配的赋值语句的集合，在此集合内的赋值语句中以此变量作为左值，从而改变了此变量。
这些赋值语句的效果将会在这个变量被使用时体现出来，因而其效果借由此变量体现，支配边界就是这些语句的范围。

2. 请简述 `phi` 节点的概念，与其存在的意义

phi 节点用于辨认 SSA 模型中多个前驱所定的最终值。

由于 SSA 中，一个变量只能赋值一次，所以源码中的一个变量可能会对应 IR 中多个变量，需要使用 `Phi` 指令合并变量。

3. 请描述 `Mem2Reg Pass` 执行前后的 IR 的变化, 简述一下

在 `Mem2Reg Pass` 操作之后，IR 减少了「为了使用参数，创建新空间，把参数复制到新空间中」、「反复 `load` 某个指针」的操作，
多了 `Phi` 指令来辨认变量该使用来自哪个基本块的流。

4. 在放置 phi 节点的时候，算法是如何利用支配树的信息的

在 `Mem2Reg Pass` 中，算法先产生了一个支配树。

`Mem2Reg::generate_phi()` 中取出支配树产生的支配边界（也就是 `bb_dominance_frontier_bb`），在其中每一块插入 `Phi` 指令；
`Mem2Reg::re_name()` 中会用到支配树的 `dom_tree_succ_blocks_`，依次 `rename()`。

5. 算法是如何选择 `value`(变量最新的值)来替换 `load` 指令的（描述数据结构与维护方法）

`var_val_stack[val]` 栈存放一些左值所可能使用的值。

遇到 load 指令，`instr->replace_all_use_with(var_val_stack[l_val].back()); wait_delete.push_back(instr);` 代码块把 load 指令的输出用 `var_val_stack[l_val]` 的栈顶值替换（在 Value.cpp 中看到所有的使用都被 new_val Value 替代），同时把该 load 指令加入要删除的队列中。

遇到 store 指令，`var_val_stack[l_val].push_back(r_val); wait_delete.push_back(instr);` 代码块把 store 指令的值作为一种新的可能加入到 `var_val_stack[l_val]` 栈中，同时把该 store 指令加入要删除的队列。

最后，根据产生的 phi 指令，从前面的 var_val_stack 栈中取出每一块 value 和基本块组成 phi 语句。

### 代码阅读总结

在此阶段下对于代码优化有了一些基本认识，希望能在后续的实现中运用起来

### 实验反馈 （可选 不会评分）

代码建议增加注释。

### 组间交流 （可选）

无
