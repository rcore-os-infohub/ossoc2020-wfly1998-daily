# Lab 5 实验记录

## VirtIO

### 思考：为什么物理地址到虚拟地址转换直接线性映射，而虚拟地址到物理地址却要查表

回答：我觉得在内核中，物理地址到虚拟地址都是线性映射的，不需要查表吧，用户态非线性的地址映射才需要查表

参考答案：

> 我们拿到地址的究极目的是访问地址上的内容，需要注意到在 0x80000000 到 0x88000000 的区间的物理页有可能对应着两个虚拟页，我们在启动或是新建内核线程的时候都包含诸如 0xffffffff80000000 到 0x80000000 这样的线性映射，这意味着，在内核线程里面，只要一个物理地址加上偏移得到的虚拟地址肯定是可以访问对应的物理地址的。所以，把物理地址转为虚拟地址加个偏移既可。
>
> 也需要注意到，内核线程虽然代码和数据都是线性映射的，但是内核栈是以 Frame 为单位分配的（除了 boot 线程是直接放在 .bss 中），而以 Frame 为单位分配意味着，虚拟地址可能从 0 开始，这个时候要转为物理地址，显然不是减去偏移量的线性映射，而必须查当前的表。
>
> 这个时候，你可能问了：为什么 RISC-V 处理器可以通过虚拟地址来访问，但是我还要手写查表来做这件事呢？这是因为 RISC-V 还真没有直接用 MMU 得到地址的指令，我们只能手写。

`println` 了一下，大部分地址映射都是形如：

> ffffffff80a6c668 -> 80a6c668
>
> ffffffff80263600 -> 80263600
>
> ffffffff80a6c678 -> 80a6c678

这样子的，但是也有部分为：

> 000000000107d660 -> 80bb5660
>
> 000000000107d670 -> 80bb5670

跑了几次，的确有从 `0` 开始的虚拟地址，所以必须要查表了

这里没太理解，本来想跟踪一下这些地址是从哪来的，但是很难跟踪，就先算了吧

## 杂

感觉这部分没什么好分析的，这部分的核心都被封装在大佬们写的库里面了，虽然很方便但是不太容易理解，有点怀念当年的 uCore 了
