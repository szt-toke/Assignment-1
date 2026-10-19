## Assignments for Introduction to Static Program Analysis.

- After completing the code assignemts in the corresponding subdirectories, you can go to the root direction, i.e., `Assignments/`, and type:
  - `./build.sh`: to compile and build the code in all subdirectories; or
  - `./build.sh [list_of_directories]`: to compile and build the code in the subdirectories you specified.
- After running `./build.sh`, an executable is expected to be in each corresponding subdirectory.
- Then, you can use the test cases in `Assignment.../Test-Cases` to check whether you code is correct.

#### Note:

Every time you upload your assignments, please attach your **NAME** and **STUDENT ID** to this README file!

## 学生信息

- **NAME（姓名）**：宋志涛
- **STUDENT ID（学号）**：3120262312

## 实验环境

- 操作系统：Ubuntu 26.04.1 LTS
- SVF：课程提供的 SVF 3.2
- LLVM / Clang：16.0.4
- GCC：13.4.0
- 图形可视化工具：Graphviz

## 实现内容

补全 `Assignment-1-IR/SVFIR.cpp` 中的 TODO，构建并导出 SVFIR（PAG）、Call Graph 和 ICFG。

核心实现如下：

```cpp
SVFIR* pag = builder.build();
pag->dump("pag");
pag->getCallGraph()->dump("callgraph");
pag->getICFG()->dump("icfg");
```

其中：

- `builder.build()` 构建 SVFIR，以及关联的调用图和 ICFG。
- `pag->dump("pag")` 导出 `pag.dot`。
- `pag->getCallGraph()->dump("callgraph")` 导出 `callgraph.dot`。
- `pag->getICFG()->dump("icfg")` 导出 `icfg.dot`。

## 测试与观察

### 测试过程

在作业仓库根目录运行 `build.sh`，生成可执行文件 `Assignment-1-IR/svfir`。

使用 SVF 配套的 Clang 16，将 `Assignment-1-IR/Test-Cases` 中的 C 测试文件编译为 LLVM IR（`.ll` 文件），编译选项为：

```bash
clang -S -emit-llvm -O0 -g input.c -o input.ll
```

随后使用 `svfir` 读取生成的 `.ll` 文件，导出三种图，并使用 Graphviz 转换为 SVG 格式。不同测试用例的输出分别保存在 `results/测试名/` 目录中，避免图文件相互覆盖。

三种图的作用如下：

| 图文件 | 表示的内容 |
|---|---|
| `pag.dot` | 取址、赋值、加载和存储等关系 |
| `callgraph.dot` | 函数之间的调用关系 |
| `icfg.dot` | 跨函数的控制流 |

### 不同优化选项下的 LLVM IR 对比

对 `add.c` 分别使用 `-O0` 和 `-O1` 生成 LLVM IR，并使用 `diff` 比较结果。本组对比除优化级别外，其他编译选项保持一致。

在 `-O0` 下，函数为两个参数和局部变量分配了栈空间，通过 `store` 保存值、通过 `load` 读取值，然后完成加法并返回结果。

在 `-O1` 下，这些临时栈存储及相关的内存读写被消除，函数体简化为：

```llvm
%3 = add nsw i32 %1, %0
ret i32 %3
```

函数体中的指令数量对比如下：

| 指令 | `-O0` | `-O1` |
|---|---:|---:|
| `alloca` | 3 | 0 |
| `store` | 3 | 0 |
| `load` | 3 | 0 |
| `add` | 1 | 1 |
| `ret` | 1 | 1 |
| 合计 | 11 | 2 |

优化后，函数直接对参数进行加法并返回结果，保留了原程序的计算功能，同时消除了本例中不必要的栈空间分配和内存读写。

本次对比表明，不同优化级别会明显改变 LLVM IR 的结构。因此，在使用 SVF 分析程序时，需要关注生成输入 IR 时使用的编译选项。
