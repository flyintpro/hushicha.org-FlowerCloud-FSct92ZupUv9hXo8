
汇编中，加法指令很重要，因为它是执行其他很多指令的基础。


同时，加法指令也会影响`NZCV`标志。有关`NZCV`的介绍，可以参看《一文搞懂 ARM 64 系列: ADC》。


`ARM64`汇编中，`ADD`指令有`3`种形式，这里介绍第一种形式，也就是与`立即数`相加。


# 1 指令语法



```


|  | ADD , , #imm{, shift} |
| --- | --- |


```

`{}`里的内容表示是可选的。


`shift`表示`LSL(逻辑左移)`的位数，有`2`个取值，一个是`0`，一个是`12`。`0`是其默认值。


所谓`LSL(逻辑左移)`，是指将数值整体向左移动，低位补`0`。如果高位被移出去，直接丢弃。


![image](https://img2024.cnblogs.com/blog/489427/202411/489427-20241115013057917-257886671.png)


# 2 指令语义


整个指令就是将源寄存器，与立即数`imm`(如果有必要，需要进行`LSL`)相加，将结果写入目的寄存器。


**注意**，这条指令不影响`NZCV`标志。



```


|  | (, _) =  + imm << shift |
| --- | --- |


```

# 3 NZCV 如何受影响


虽然这条指令最终不影响`NZCV`标志，但是搞清楚`NZCV`如何受影响，还是很有必要的。


`1` 将源寄存器的值和`imm << shift`都当成`无符号整型数`，两数相加，得到一个`无符号整型数`的结果，记作`u_result`。此时计算时不考虑溢出:



```


|  | = 0xffffffffffffffff // 64 bit 全 1 |
| --- | --- |
|  | (imm << shift) = 1 |
|  | u_result = 0xffffffffffffffff + 1 = 0x10000000000000000 // 2^64，而不是 0 |


```

`2`将源寄存器的值和`imm << shift`都当成`有符号整型数`，两数相加，得到一个`有符号整型数`的结果，记作`s_result`。此时计算时不考虑溢出:



```


|  | = 0xffffffffffffffff // 64 bit 全 1，此时当成 -1 看待 |
| --- | --- |
|  | (imm << shift) = 0x8000000000000000 // 64 bit 最小负整数 -9223372036854775808 |
|  | s_result = -1 + (-9223372036854775808) = -9223372036854775809 // 而不是 0x7fffffffffffffff |


```

`3` 从`u_result`中取`(63~0)`共`64bit`，记作`result`;


`4` 如果`result`的最高位是`1`，那么`N = 1`;


`5` 如果`result = 0`，那么`Z = 1`;


`6` 如果把`result`当成`无符号整型数`，它的值等于`u_result`，那么`C = 0`；如果不等于，那么`C = 1`，也就是在进行加法运算时，发生了进位。


`7` 如果把`result`当成`有符号整型数`，它的值等于`s_result`，那么`V = 0`；如果不等于，那么`V = 1`，也就是说在进行加法运算，发生了溢出。


 本博客参考[悠兔机场](https://xinnongbo.com)。转载请注明出处！
