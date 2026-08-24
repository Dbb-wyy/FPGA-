# Verilog HDL 基础学习文档

> 面向初学者，采用「规则 → 用例」的形式，先讲规则，再给可以直接上板/仿真验证的例子。
> 本文聚焦**数字逻辑 / 可综合**的基础语法，深入但不脱离基础，不涉及状态机等高级话题。

---

## 目录

1. [Verilog HDL 的基本单元——模块](#1-verilog-hdl-的基本单元模块)
2. [Verilog 基本语法](#2-verilog-基本语法)
3. [运算符与表达式](#3-运算符与表达式)
4. [过程语句](#4-过程语句)
5. [块语句](#5-块语句)
6. [赋值语句](#6-赋值语句)
7. [条件语句](#7-条件语句)
8. [循环语句](#8-循环语句)
9. [task 和 function 说明语句](#9-task-和-function-说明语句)
10. [编译向导（编译器指令）](#10-编译向导编译器指令)
11. [其他补充内容](#11-其他补充内容)

---

## 1. Verilog HDL 的基本单元——模块

### 1.1 规则

- **模块（module）**是 Verilog 最基本的组成单位，一个设计（design）由若干模块组合而成。
- 每个模块以 `module` 关键字开始，以 `endmodule` 结束。
- 模块的端口（port）用于与外界通信，分为三种方向：
  - `input`：输入端口
  - `output`：输出端口
  - `inout`：双向端口（例如三态总线）
- 模块内部可以声明 **内部信号（wire / reg）**，并描述逻辑功能。
- 一个工程里，每个模块名必须唯一；模块之间通过**模块实例化（instance）**连接。

### 1.2 基本结构

```verilog
module 模块名 ( 端口列表 );
    // 端口方向声明
    input  端口名;
    output 端口名;
    // 内部信号、逻辑、子模块实例化
    ...
endmodule
```

### 1.3 用例

一个最简单的二输入与门模块：

```verilog
module and_gate (
    input  wire a,   // 输入 a
    input  wire b,   // 输入 b
    output wire y    // 输出 y
);
    assign y = a & b;   // 组合逻辑：y 等于 a 与 b
endmodule
```

> 说明：`wire` 表示连线型信号（用 `assign` 连续赋值驱动）；`reg` 表示寄存器型信号（在过程块中用 `<=` / `=` 赋值）。

---

## 2. Verilog 基本语法

### 2.1 标识符

- **规则**：标识符（变量名、模块名等）只能由字母、数字、下划线 `_` 和美元符号 `$` 组成，且**首字符不能是数字**。标识符区分大小写。

```
规则：a_b, _flag, data8  → 合法
规则：8data, a-b, "x"    → 非法
```

### 2.2 关键字

- **规则**：Verilog 保留了一批关键字（如 `module`、`always`、`if`、`else` 等），**不能**用作标识符。

```
合法：module  my_circuit ...
非法：module  module ...     // module 是关键字
```

### 2.3 常量与数值表示

**规则：** 数字可以用以下形式书写：

```
<位宽>'<进制><数值>
```

- 位宽：二进制数的位数（十进制表示）。
- 进制：`b`（二进制）、`o`（八进制）、`d`（十进制）、`h`（十六进制）。
- 数值：对应该进制的数字。

```
4'b1010    // 4 位二进制数 1010
8'hFF      // 8 位十六进制数 FF（即 255）
16'd1024   // 16 位十进制数 1024
```

**特殊值：**

- `0`、`1`：逻辑 0、逻辑 1
- `z` / `Z`：高阻态（三态），用于未驱动的线
- `x` / `X`：不确定态（未知值）

```
4'b10zx    // 位分别为 1,0,z,x
8'bz       // 8 位全为高阻
```

### 2.4 数据类型

Verilog 常用两大类信号类型：

| 类型 | 关键字 | 含义 | 典型用法 |
|------|--------|------|----------|
| 连线型 | `wire` | 由驱动源驱动（`assign` 或模块输出） | 模块间连接、组合逻辑输出 |
| 寄存器型 | `reg` | 在过程块（`always` / `initial`）中赋值 | 时序逻辑、锁存逻辑、过程块变量 |

**规则：**
- `wire` 只能被 `assign` 连续赋值或外部驱动，**不能**在 `always` 中被赋值。
- `reg` 只能在过程块（`always`/`initial`）中被赋值，**不能**用 `assign` 驱动。
- 组合逻辑既可以用 `assign` + `wire` 实现，也可以用 `always` + `reg` 实现。

```
规则：wire 信号不能被 always 赋值；reg 信号不能被 assign 驱动。
```

### 2.5 参数（parameter）

**规则：** `parameter` 用来定义常量，可在实例化时被重新指定，常用于可配置模块。

```verilog
module counter #(
    parameter WIDTH = 8   // 默认位宽 8
) (
    input  wire                clk,
    input  wire                rst_n,
    output reg  [WIDTH-1:0] q
);
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n)
            q <= 0;
        else
            q <= q + 1;
    end
endmodule
```

---

## 3. 运算符与表达式

### 3.1 算术运算符

| 运算符 | 含义 | 示例 |
|--------|------|------|
| `+` | 加 | `a + b` |
| `-` | 减 | `a - b` |
| `*` | 乘 | `a * b` |
| `/` | 除 | `a / b` |
| `%` | 取模（求余） | `a % b` |

```
规则：除法、取模在可综合电路里开销大，尽量用移位（乘以/除以 2 的幂）。
```

### 3.2 位运算符（按位逻辑）

| 运算符 | 含义 |
|--------|------|
| `&` | 按位与 |
| `\|` | 按位或 |
| `^` | 按位异或 |
| `~` | 按位取反 |

```verilog
assign y = a & b;    // 按位与
assign y = a ^ b;    // 按位异或（常用于校验）
assign y = ~a;       // 按位取反
```

### 3.3 逻辑运算符

| 运算符 | 含义 |
|--------|------|
| `&&` | 逻辑与（结果为 0/1） |
| `\|\|` | 逻辑或（结果为 0/1） |
| `!` | 逻辑非 |

```
规则：位运算 `&` 是对每一位做与；逻辑运算 `&&` 把整个操作数当做一个布尔值判断真假。
区分：4'b1010 & 4'b1100 = 4'b1000；而 4'b1010 && 4'b1100 = 1（两数都非零，故为真）。
```

### 3.4 关系运算符

| 运算符 | 含义 |
|--------|------|
| `<` `>` | 小于 / 大于 |
| `<=` `>=` | 小于等于 / 大于等于 |
| `==` `!=` | 相等 / 不等（比较逻辑值） |
| `===` `!==` | 全等 / 不全等（比较时 x、z 也参与判断） |

```verilog
if (a == b) ...        // 一般相等比较
if (a === b) ...       // 全等，a、b 中 x/z 的位置也须一致才算相等
```

### 3.5 归约运算符（单目）

对单个操作数的**所有位**逐位归约为 1 位结果。

| 运算符 | 含义 |
|--------|------|
| `&a` | 归约与：所有位都为 1 时结果为 1 |
| `\|a` | 归约或：任一位置 1 结果为 1 |
| `^a` | 归约异或：奇偶校验位 |

```
规则：&a 即 "a 的所有位与运算"，常用来判断一个数是否全为 1。
```

### 3.6 移位运算符

| 运算符 | 含义 |
|--------|------|
| `<<` | 逻辑左移 |
| `>>` | 逻辑右移 |
| `<<<` | 算术左移（符号位不动） |
| `>>>` | 算术右移（补符号位） |

```verilog
assign y = a << 2;   // a 左移两位，相当于乘 4
assign y = a >> 1;   // a 右移一位，相当于除以 2
```

### 3.7 拼接与复制运算符

- **拼接** `{ }`：把若干信号按位拼接成新向量。
- **复制** `{n{信号}}`：把信号复制 n 次。

```verilog
wire [3:0] a;
wire [3:0] b;
wire [7:0] c;
assign c = {a, b};        // c 高 4 位 = a，低 4 位 = b
assign c = {2{a}};        // c = a 拼接 a，共 8 位
assign c = {4'd0, a};     // 高位补 4 个 0（位宽补齐常用写法）
```

### 3.8 条件运算符（三目）

```verilog
assign y = sel ? a : b;   // sel 为 1 时 y=a，否则 y=b
```

### 3.9 运算符优先级

从高到低大致为：`~` → `* / %` → `+ -` → `<< >>` → 关系 `< <= > >=` → `== !=` → `&` → `^` → `|` → `&&` → `||` → `?:`。

```
规则：当表达式复杂时，多使用括号 `()` 明确优先级，提高可读性、避免歧义。
```

### 3.10 用例：综合使用多种运算符

```verilog
module alu (
    input  wire [7:0] a, b,
    input  wire [1:0] op,
    output reg  [7:0] y
);
    always @(*) begin
        case (op)
            2'b00:   y = a + b;        // 加
            2'b01:   y = a - b;        // 减
            2'b10:   y = a & b;        // 按位与
            2'b11:   y = a ^ b;        // 按位异或
            default: y = 8'b0;
        endcase
    end
endmodule
```

---

## 4. 过程语句

### 4.1 规则

- 过程语句包括 **`initial`** 和 **`always`** 两种，均用于过程块（procedural block）。
- 过程块内部只能给 `reg` 类型变量赋值。
- `initial`：只在仿真开始时**执行一次**，通常用于仿真激励（testbench）初始化，**不可综合**。
- `always`：条件满足时反复执行，既可用于时序逻辑，也可用于组合逻辑。

### 4.2 initial 语句

```verilog
initial begin
    a = 0;
    b = 1;
    #10;            // 延时 10 个时间单位
    b = 0;
end
```

### 4.3 always 语句

`always` 后面跟**敏感列表**（触发条件）。

```
规则：
always @(敏感列表) begin ... end
敏感列表写法：
  @(a or b)          组合逻辑：a 或 b 任一变化时执行
  @(*)               组合逻辑：任一输入变化时执行（推荐）
  @(posedge clk)     时序逻辑：时钟上升沿触发
  @(negedge rst_n)   时序逻辑：复位下降沿触发
```

**用例 1：组合逻辑（always 实现）**

```verilog
module mux2 (
    input  wire a, b, sel,
    output reg  y
);
    always @(*) begin
        if (sel)
            y = a;
        else
            y = b;
    end
endmodule
```

**用例 2：时序逻辑（带同步/异步复位）**

```verilog
module counter8 (
    input  wire       clk,
    input  wire       rst_n,   // 低电平复位
    output reg  [7:0] cnt
);
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n)
            cnt <= 8'd0;
        else
            cnt <= cnt + 1;
    end
endmodule
```

```
规则：时序逻辑使用非阻塞赋值 <=；组合逻辑使用阻塞赋值 =。
```

---

## 5. 块语句

### 5.1 规则

块语句用于把多条语句组织在一起：

- **顺序块 `begin ... end`**：其中的语句**按书写顺序依次执行**。用于 `always`、`initial` 等过程块。
- **并行块 `fork ... join`**：其中的语句**同时（并行）执行**，主要用于仿真，**不可综合**。

### 5.2 顺序块用例

```verilog
initial begin
    #0  a = 0;   // 先执行
    #10 a = 1;   // 延时 10 后执行
    #10 a = 0;   // 再延时 10 后执行
end
```

### 5.3 并行块用例（仅仿真）

```verilog
initial fork
    #10 a = 1;   // 三个语句同时从 0 开始计时
    #20 b = 1;
    #30 c = 1;
join
```

> 上面的 fork 块中三条语句是并行的，都从时刻 0 开始，分别在 10、20、30 个时间单位后触发。

### 5.4 命名块

**规则：** 可以用 `begin : 名称 ... end` 给块命名，便于调试（仿真波形中定位）。

```verilog
always @(posedge clk) begin : counter_blk
    cnt <= cnt + 1;
end
```

---

## 6. 赋值语句

### 6.1 规则

Verilog 有两类赋值：

| 类型 | 关键字 | 使用场景 | 执行时机 |
|------|--------|----------|----------|
| 连续赋值 | `assign` | 组合逻辑，驱动 `wire` | 右端表达式变化时立即更新 |
| 过程赋值 | `=` / `<=` | 在 `always`/`initial` 中给 `reg` 赋值 | 过程块执行时 |

其中过程赋值又分：
- **阻塞赋值 `=`**：顺序执行，立即生效（组合逻辑常用）。
- **非阻塞赋值 `<=`**：并行执行，在当前过程块结束后才更新（时序逻辑常用）。

### 6.2 连续赋值用例

```verilog
module gates (
    input  wire a, b,
    output wire y_and, y_or, y_not
);
    assign y_and = a & b;
    assign y_or  = a | b;
    assign y_not = ~a;
endmodule
```

### 6.3 阻塞赋值 vs 非阻塞赋值

```
规则（关键）：
- 组合逻辑：用阻塞赋值 =
- 时序逻辑：用非阻塞赋值 <=
- 一个 always 块内不要混用两种赋值
```

**用例：区别演示**

```verilog
// 非阻塞（时序，寄存器更新是并行的）
always @(posedge clk) begin
    q1 <= d;   // q1 拿到旧 d
    q2 <= q1;  // q2 拿到旧的 q1（不是新 q1）
end

// 阻塞（顺序执行）
always @(posedge clk) begin
    q1 = d;
    q2 = q1;   // q2 拿到新的 q1
end
```

> 上面两种写法在综合出的硬件结果不同，务必区分。

---

## 7. 条件语句

### 7.1 规则

- 条件语句用于根据条件选择执行分支：`if ... else if ... else` 和 `case`。
- 条件语句只能用在**过程块**（`always`/`initial`）中。

### 7.2 if / else 语句

```verilog
if (条件1)
    语句1;
else if (条件2)
    语句2;
else
    语句3;
```

```
规则：if 条件不完整（缺少 else）在组合逻辑中会综合出锁存器（latch），
一般不希望产生 latch，所以组合逻辑里尽量把 else 写全，或用 default。
```

**用例：比较器**

```verilog
module compare (
    input  wire [3:0] a, b,
    output reg  eq, gt, lt
);
    always @(*) begin
        if (a > b)      begin gt = 1; eq = 0; lt = 0; end
        else if (a == b)begin gt = 0; eq = 1; lt = 0; end
        else            begin gt = 0; eq = 0; lt = 1; end
    end
endmodule
```

### 7.3 case 语句

**规则：**

```verilog
case (表达式)
    值1: 语句1;
    值2: 语句2;
    ...
    default: 默认语句;
endcase
```

- `case` 用于多分支选择，可读性比一串 `if` 好。
- `default` 分支用于处理未列出的情况，组合逻辑建议写 `default`。

**用例：7 段数码管译码（部分）**

```verilog
module seg7 (
    input  wire [3:0] bcd,
    output reg  [6:0] seg
);
    always @(*) begin
        case (bcd)
            4'd0: seg = 7'b0111111;
            4'd1: seg = 7'b0000110;
            4'd2: seg = 7'b1011011;
            4'd3: seg = 7'b1001111;
            4'd4: seg = 7'b1100110;
            4'd5: seg = 7'b1101101;
            default: seg = 7'b0000000;
        endcase
    end
endmodule
```

### 7.4 casez 与 casex

```
规则：casez 把 z（高阻）当作无关项匹配；casex 把 x 和 z 都当作无关项匹配。
一般可综合设计中很少用，主要用于仿真。
```

```verilog
casez (sel)
    4'b1???: out = a;   // 最高位为 1 即命中，其余位任意
    ...
endcase
```

---

## 8. 循环语句

### 8.1 规则

Verilog 提供四种循环语句：

| 语句 | 含义 |
|------|------|
| `for` | 指定初值、条件、步长的计数循环 |
| `while` | 条件为真时循环 |
| `repeat` | 固定重复次数 |
| `forever` | 无限循环 |

```
规则：
- for 在可综合逻辑中可用于在 always 里展开组合/时序逻辑（本质是循环展开）。
- while / repeat / forever 通常只用于仿真（testbench），一般不可综合。
```

### 8.2 for 循环用例（可综合）

**用例：把 8 位输入按位或输出到 y（归约功能的手动展开）**

```verilog
module or8 (
    input  wire [7:0] data,
    output reg        y
);
    integer i;
    always @(*) begin
        y = 1'b0;
        for (i = 0; i < 8; i = i + 1) begin
            y = y | data[i];
        end
    end
endmodule
```

> 说明：`for` 在综合时会被**展开**成 8 级逻辑，并非真实循环硬件。

### 8.3 仿真用循环用例

```verilog
// testbench 中循环初始化一组寄存器
initial begin
    for (i = 0; i < 16; i = i + 1)
        mem[i] = 8'd0;
end

// forever 产生时钟
initial begin
    clk = 0;
    forever #5 clk = ~clk;   // 每 5 个时间单位翻转一次，周期 10
end
```

---

## 9. task 和 function 说明语句

### 9.1 规则

- `function`：**返回一个值**的函数，只能有输入参数，不能包含延时，不能调用 task。用于简化组合逻辑运算。
- `task`：可执行多条语句、可有输入/输出/双向参数，可含延时。常用于仿真中重复的激励过程。

```
规则：
- function 至少有一个输入，且只能返回一个值，可用于可综合代码。
- task 可以没有返回值、可以有多个输出，常用于 testbench，一般用于仿真。
```

### 9.2 function 用例

```verilog
module parity (
    input  wire [7:0] data,
    output wire       even_parity
);
    function [0:0] compute_parity;
        input [7:0] d;
        integer i;
        begin
            compute_parity = 0;
            for (i = 0; i < 8; i = i + 1)
                compute_parity = compute_parity ^ d[i];
        end
    endfunction

    assign even_parity = compute_parity(data);
endmodule
```

### 9.3 task 用例（仿真）

```verilog
module tb;
    reg clk, rst_n;

    task reset_system;
        begin
            rst_n = 0;
            #20;
            rst_n = 1;
        end
    endtask

    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end

    initial begin
        reset_system;   // 调用 task
        #100 $finish;
    end
endmodule
```

---

## 10. 编译向导（编译器指令）

### 10.1 规则

- 编译向导以 **反引号 `` ` ``** 开头（不是单引号 `'`），作用于整个编译单元，不分大小写。
- 常用指令见下表。

| 指令 | 作用 |
|------|------|
| `` `define `` | 定义宏（文本替换） |
| `` `include `` | 包含其他文件 |
| `` `timescale `` | 设置时间单位与精度 |
| `` `ifdef `` / `` `ifndef `` / `` `else `` / `` `endif `` | 条件编译 |
| `` `resetall `` | 复位所有编译指令 |

### 10.2 宏定义用例

```verilog
`define WIDTH 8
`define TRUE  1'b1

wire [`WIDTH-1:0] data;   // 展开为 wire [7:0] data
```

### 10.3 timescale 用例

```verilog
`timescale 1ns / 1ps
// 时间单位 1ns，精度 1ps，仿真中 #10 表示 10ns
```

### 10.4 条件编译用例

```verilog
`define DEBUG

`ifdef DEBUG
    // 仅 DEBUG 时包含的代码
    initial $display("Debug mode on");
`else
    initial $display("Normal mode");
`endif
```

---

## 11. 其他补充内容

### 11.1 模块实例化与信号连接

**规则：** 一个模块可在另一个模块中被实例化，端口连接方式有两种：

- 按**顺序**连接（不推荐，易错）。
- 按**端口名**连接（推荐，更清晰）。

```verilog
module top;
    wire a, b, y;

    // 按端口名连接
    and_gate u_and (
        .a(a),
        .b(b),
        .y(y)
    );
endmodule
```

### 11.2 位选择与部分选择

```verilog
reg [7:0] data;
wire bit3 = data[3];     // 选择第 3 位
wire [3:0] nibble = data[7:4];   // 选择高 4 位
```

### 11.3 三态输出与 inout

```verilog
module tristate (
    input  wire        oe,     // 输出使能
    input  wire [7:0]  data,
    inout  wire [7:0]  bus
);
    assign bus = oe ? data : 8'bz;   // 无效时输出高阻
endmodule
```

### 11.4 存储器（reg 数组）

```verilog
reg [7:0] mem [0:255];   // 256 个 8 位寄存器组成的存储区

// 读
wire [7:0] rdata = mem[addr];
// 写
always @(posedge clk)
    if (we) mem[addr] <= wdata;
```

### 11.5 测试平台（testbench）最小框架

```verilog
`timescale 1ns/1ps
module tb_and_gate;
    reg  a, b;
    wire y;

    // 实例化被测模块 DUT
    and_gate u_dut (.a(a), .b(b), .y(y));

    initial begin
        $dumpfile("tb.vcd");
        $dumpvars(0, tb_and_gate);

        a = 0; b = 0; #10;
        a = 0; b = 1; #10;
        a = 1; b = 0; #10;
        a = 1; b = 1; #10;
        $finish;
    end
endmodule
```

### 11.6 常见代码风格建议

- 端口声明与内部信号区分清楚，命名要有含义（如 `data_in`、`clk`、`rst_n`）。
- 时序逻辑用 `<=`，组合逻辑用 `=`。
- 组合逻辑用 `always @(*)`，避免遗漏输入导致锁存器。
- 复位信号通常低电平有效，命名加 `_n` 后缀（如 `rst_n`）。
- 位宽、进制写明确，避免默认值歧义。
- 代码块正确缩进，提高可读性与可维护性。

---

## 附录：快速对照速查表

| 需要实现 | 推荐写法 |
|----------|----------|
| 组合逻辑（单输出） | `assign y = ...;` |
| 组合逻辑（多分支） | `always @(*) ... if/case ... =` |
| 时序逻辑（寄存器） | `always @(posedge clk or negedge rst_n) ... <=` |
| 多位选择 | `case` / 三目 `?:` |
| 数组/存储 | `reg [..] mem [..];` |
| 可配置位宽 | `parameter` + `#(...)` 实例化 |
| 仿真延时 | `#数值` |
| 仿真时钟 | `forever #5 clk = ~clk;` |

---

*本文档为 Verilog HDL 基础入门参考，覆盖模块、语法、运算符、过程/块/赋值/条件/循环语句、task/function、编译向导及常用补充知识。建议结合仿真工具（如 Icarus Verilog、Vivado、ModelSim）动手验证每个用例。*
