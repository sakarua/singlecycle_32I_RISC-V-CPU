# 单周期 32 位 RISC-V CPU

这是一个单周期 RISC-V CPU 实现，采用 Verilog 编写，包含完整的控制器与数据通路，并附带可直接运行的测试程序。

当前工程使用 RV32I 子集指令完成验证，测试通过后会在数据存储器地址 `100` 写入数值 `25`。

## 1. 项目结构

```
riscvsingle.v   # 顶层、CPU核心、控制器、数据通路、存储器、测试平台
riscvtest.s     # 汇编测试程序（便于阅读）
riscvtest.txt   # 指令存储器初始化文件（十六进制机器码）
```

## 2. CPU 组成

主要模块都在 `riscvsingle.v` 中：

- `testbench`：产生时钟/复位，检查仿真是否成功
- `top`：系统顶层，连接 CPU 核、指令存储器、数据存储器
- `riscvsingle`：CPU 顶层（`controller + datapath`）
- `controller`：主译码器 + ALU 译码器
- `datapath`：PC、寄存器堆、立即数扩展、ALU、结果回写选择
- `imem`：指令存储器（通过 `$readmemh` 读取 `riscvtest.txt`）
- `dmem`：数据存储器

## 3. 当前已验证指令

测试程序 `riscvtest.s` 覆盖并验证了以下指令：

- R-type：`add` `sub` `and` `or` `slt`
- I-type：`addi` `lw`
- S-type：`sw`
- B-type：`beq`
- J-type：`jal`

合计 10 条基础指令，已满足当前测试样例。

## 4. 仿真运行方式

下面给出基于 Icarus Verilog 的最小流程。

1. 编译：

```bash
iverilog -g2012 -o simv riscvsingle.v
```

2. 运行：

```bash
vvp simv
```

3. 查看波形（可选）：

```bash
gtkwave riscvsingle1.vcd
```

## 5. 仿真通过判据

`testbench` 在每个时钟下降沿检测写存储器行为：

- 若检测到 `MemWrite=1` 且 `DataAdr==100` 且 `WriteData==25`，输出：

```
Simulation succeeded
```

- 若出现非预期写地址（不是中间过程允许的地址），输出：

```
Simulation failed
```

## 6. 如何替换测试程序

1. 在 `riscvtest.s` 中修改/新增汇编程序。
2. 将汇编重新汇编为 32 位机器码（十六进制，每行一条指令）。
3. 覆盖 `riscvtest.txt`。
4. 重新编译并运行仿真。

