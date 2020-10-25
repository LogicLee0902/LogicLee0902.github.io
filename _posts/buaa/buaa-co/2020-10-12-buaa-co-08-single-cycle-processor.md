---
layout: "post"
title: "「BUAA-CO」 08 单周期 CPU"
subtitle: "搭建单周期CPU"
author: "roife"
date: 2020-10-12
tags: ["C「BUAA - Computer Organization」", "B「Digital Design and Computer Architecture」", "BUAA", "计算机组成", "数字电路", "L「Verilog-HDL」"]
status: Completed

language: zh-CN
catalog: true
header-image: ""
header-style: text
mathjax: true
---

# 状态元件

CPU 的状态元件有四个:
- 程序计数器 (Program Counter, PC), 其输出指向当前的指令
- 指令存储器 (Instruction Memory, IM), 只有一个读端口, 输出地址对应的指令
- 寄存器文件 (Register File, RF) 表示 32\*32 的寄存器, WE3 控制 WD3 的写入功能
- 数据存储器 (Data Memory, DM) 表示存储器, 当 WE 端为 1 时可写入

![CPU 状态元件](/img/in-post/post-buaa-co/cpu-state-elements.png "cpu-state-elements"){:height="600px" width="600px"}

其中, PC/RF/DM的读取过程呈现出组合逻辑特征, 不需要时钟参与; 而写入过程仅在时钟上升沿发生.

# 数据路径

## 第一条指令: lw

`lw` 指令的格式为:

![lw 指令格式](/img/in-post/post-buaa-co/lw-format.png "lw-format"){:height="600px" width="600px"}

![lw - 计算地址](/img/in-post/post-buaa-co/single-lw-compute-memory-address.png "single-lw-compute-memory-address"){:height="700px" width="700px"}

首先从 PC 中取出指令地址, 并据此从 IM 中取出对应的指令. 根据指令格式, 用其 21~25 位从 RF 中读取对应的 `base` 寄存器.

同时, 对其 0~15 位的 `offset` 进行符号扩展, 送至 ALU 中算出 $R[base] + \mathrm{sign\\\_ext}(offset)$, 即对应的 DM 中数据的地址.

其中 ALU 的运算符用一个 3 位的信号 `ALUControl` 控制, 这个信号由 CU 根据指令的 `opcode` 发出.

![lw - 写入数据](/img/in-post/post-buaa-co/single-lw-write-to-rf.png "single-lw-write-to-rf"){:height="700px" width="700px"}

然后将从 DM 中读取的值重新写入 RF, 其地址由指令的 16~20 位决定.

![计算 NPC](/img/in-post/post-buaa-co/single-lw-npc.png "single-lw-npc"){:height="700px" width="700px"}

最后将 PC 后移 4 位, 即下一条指令的地址.

## sw/r-type/beq

使用类似的做法, 可以一步步添加 `sw`, `R-type`, `beq` 指令的数据路径.

![sw](/img/in-post/post-buaa-co/single-sw.png "single-sw"){:height="700px" width="700px"}

`sw` 指令和 `lw` 类似, 其中写入地址为 16~20 位.
- `MemWrite`: 控制写入使能.

![R-type](/img/in-post/post-buaa-co/single-r-type.png "single-r-type"){:height="700px" width="700px"}

R 类型指令相对比较复杂, 需要多个控制信号.
- `ALUSrc`: R 类型指令直接用 16~20 位对应的寄存器与 21~25 对应的寄存器相加, 而不是加 `offset`, 所以用 `ALUSrc` 信号来选择
- `ALUControl`: ALU 执行的运算类型由 `ALUControl` 决定
- `MemtoReg`: 相比于 `sw` 与 `lw`, R 类型指令不需要经过 DM, 因此通过一个 `MemtoReg` 信号将 ALU 的结果直接返回
- `RegDst`: 数据返回时, 对于 R 型指令直接放到地址为 11~15 位对应的寄存器, 而非16~20 位, 这里通过 `RegDst` 进行选择

![beq](/img/in-post/post-buaa-co/single-beq.png "single-beq"){:height="700px" width="700px"}

`beq` 指令当两个数字相等时跳转, 比较相等可以利用减法运算, 当相等时结果为 0, 此时 ALU 的 `zero` 端口为 1.
- `Branch`: `Branch` 表示当前为分支语句, 如果值为 1 且数字相等, 则 PC 不再使用 $PC = PC + 4$, 而是 $PC = SigImm * 4 + PC$.

# 单周期控制

所有控制信号都要使用 CU 根据指令 opcode 和 funct (R 型指令) 来发出不同的信号.

![完整 MIPS 处理器](/img/in-post/post-buaa-co/single-complete-processor.png "single-complete-processor"){:height="700px" width="700px"}

因为 R 型指令比较特殊, 所以将 CU 分为两部分进行设计, ALU 译码器根据 funct 字段控制 ALU (R 型指令), 主译码器根据不同的指令需要控制 ALU (其他指令).

![CU 内部结构](/img/in-post/post-buaa-co/single-cu-internal.png "single-cu-internal"){:height="250px" width="250px"}

主译码器通过不同的 `ALUop` 控制 ALU 译码器:
- 输出 `00` 时总是执行加法
- 输出 `01` 时总是执行减法
- 输出 `10` 时检测 funct 字段 (R 型指令)
- `11` 无定义

| `ALUop` | `Funct`      | `ALUControl` |
|---------|--------------|--------------|
| 00      | X            | 010 (add)    |
| X1      | X            | 110 (sub)    |
| 1X      | 100000 (add) | 010 (add)    |
| 1x      | 100010 (sub) | 110 (sub)    |
| 1x      | 100100 (and) | 110 (and)    |
| 1x      | 100101 (or)  | 110 (or)     |
| 1x      | 101010 (slt) | 110 (slt)    |

由于 funct 指令最高两位总是 `10`, 所以可以将其忽略.

| Instruction | Opcode | `RegWrite` | `RegDst` | `ALUSrc` | `Branch` | `MemWrite` | `MemtoReg` | `ALUOp` |
|-------------|--------|------------|----------|----------|----------|------------|------------|---------|
| R-type      | 000000 | 1          | 1        | 0        | 0        | 0          | 0          | 10      |
| lw          | 100011 | 1          | 0        | 1        | 0        | 0          | 1          | 00      |
| sw          | 101011 | 0          | X        | 1        | 0        | 1          | X          | 00      |
| beq         | 000100 | 0          | X        | 0        | 1        | 0          | X          | 01      |

# 更多指令

## addi

不需要增加数据通路, 只要在 CU 中加一条控制信号即可.

| Instruction | Opcode | `RegWrite` | `RegDst` | `ALUSrc` | `Branch` | `MemWrite` | `MemtoReg` | `ALUOp` |
|-------------|--------|------------|----------|----------|----------|------------|------------|---------|
| addi        | 001000 | 1          | 0        | 1        | 0        | 0          | 0          | 00      |

## j

对于 j 指令, 相当于直接设置 $PC = Instr * 4$

![j指令](/img/in-post/post-buaa-co/single-j.png "single-j"){:height="800px" width="800px"}

# 单周期性能分析

单周期 CPU 的时钟周期必须考虑最慢的指令.

考虑关键路径最长的 `lw` 指令.

$$T_c = t_{pcq\\\_PC} + t_{mem} + max[t_{RFread}, t_{sext}, t_{mux}] + t_{ALU} + t_{mem} + t_{mux} + t_{RFsetup}$$

由于 ALU, DM 和 RF 操作一般是最慢的, 所以可以简化为

$$T_c = t_{pcq\\\_PC} + 2t_{mem} + t_{RFread} + t_{ALU} + t_{mux} + t_{RFsetup}$$