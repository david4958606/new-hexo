---
title: Verilog 学习笔记 03 - Verilog 模块
tags:
  - Verilog
  - EE
description: Verilog 模块
date: 2024-08-06 17:16:18
---


在 Verilog 中，电路是作为模块出现的，每个模块对外暴露出数个输入和/或输出端口，许多个模块组合成更为复杂的电路。模块与模块间也形成了一定的层次结构。

通常而言，任何电路都有一个顶层模块 `top_module`，在顶层模块中将线连到子模块的输入输出端口来实例化子模块。

与 C 类似，在实例化模块时首先声明模块，并可以根据端口的位置或名称将线连接到端口。

比如，有一个名为 `mod_a` 的子模块：

```verilog
module mod_a ( input in1, input in2, output out );
    // Module body
endmodule
```

要实例化它，有两种写法：

```verilog
mod_a mod_a_1 (wa, wb, wc); // 按位置，或
mod_a mod_a_2 (.out(wc), .in1(wa), .in2(wb)); // 按名称
```

注意端口名称前面的句点。
