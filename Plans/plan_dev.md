# RV32I 单周期 CPU 开发计划（修订版）

## 项目概述

本项目以学习 FPGA 与数字逻辑设计为目的，从零实现一个可运行
`riscv32-unknown-elf-gcc` 编译产物的 RV32I 单周期 CPU。

项目定位：

- 不做“课程作业式”的空壳 CPU；
- 不追求高性能与复杂微架构；
- 优先保证指令正确、可仿真、可综合、可上板运行；
- 在成功运行真实编译产物后，再回头做精简、校验与优化。

---

## 设计基线

| 项目 | 决策 |
|---|---|
| ISA | RV32I |
| 微架构 | 单周期 |
| 目标器件 | Xilinx Artix-7 XC7A35T |
| 指令存储器 | 暂不单独实现 ROM，使用 LUTRAM 并初始化 `.hex` |
| 数据存储器 | LUTRAM，异步读，支持字节使能 |
| 寄存器堆 | 不复位，x0 硬连线为 0 |
| 非法指令校验 | 当前阶段不做，后续补充 |
| 冗余逻辑清理 | 当前阶段不做，跑通后统一重构 |

---

## 目标与里程碑

### M1：基础数据通路跑通

- 完成 ALU、RegFile、Decoder、ImmGen、Controller、PC。
- 完成简单指令的逐模块仿真。

### M2：完整 CPU 可运行

- 完成 `cpu.sv` 顶层例化。
- 完成 LUTRAM 指令/数据存储器。
- 可运行由 `riscv32-unknown-elf-gcc` 编译的小程序。
- 覆盖 RV32I 基本指令：
  - R-type
  - I-type ALU
  - Load / Store
  - Branch
  - JAL / JALR
  - LUI / AUIPC

### M3：设计与代码整理

- 清理冗余数据通路与控制逻辑。
- 增加非法指令、保留编码等防御性处理。
- 完善 testbench 与回归测试。

---

## 当前进度

### 已完成

- [x] `cpu_pkg.sv`：全局类型与枚举定义
- [x] `alu.sv`：组合 ALU
- [x] `regfile.sv`：2R1W 寄存器堆，x0 只读为 0
- [x] `decoder.sv`：指令字段提取与 opcode 分类
- [x] `controller.sv`：控制信号生成
- [x] `pc.sv`：PC 寄存器
- [x] `imm_gen.sv`：I/S/B/U/J 立即数生成
- [x] `datapath.sv`：主要数据通路（待顶层联调验证）

### 未完成

- [ ] `rom.sv`：已决定暂不单独实现，改用 LUTRAM 初始化代替
- [ ] `ram.sv`：数据存储器 LUTRAM 实现
- [ ] `cpu.sv`：顶层例化与连线
- [ ] `controller_tb.sv`：Controller 定向测试
- [ ] `pc_tb.sv`
- [ ] `ram_tb.sv`
- [ ] `cpu_tb.sv`
- [ ] 运行第一个 gcc 编译产物

---

## 设计决策记录

### D1：单周期 + LUTRAM

为尽快看到可运行 CPU，采用单周期架构。

XC7A35T 的 LUTRAM 官方上限约为 **400 Kb（≈ 50 KB）**。
实际使用中需为 CPU 逻辑保留资源，因此建议初始容量：

```text
指令 LUTRAM：8 KB（约 2048 条指令）
数据 LUTRAM：8 KB
合计：16 KB（约 128 Kb）
