# CSAPP 笔记与 Lab

《深入理解计算机系统》（CS:APP 3e）+ CMU 15-213 自学笔记与实验记录。
重点在**位级机制**与**可复现的实验过程**：每个 Lab 都附代码、踩坑记录和分数凭证。

## Lab 进度

| Lab | 章节 | 要改的文件 | 结果 |
| --- | --- | --- | --- |
| [Data Lab](labs/datalab/) | Ch2 信息的表示和处理 | `bits.c` | **62/62**（36 正确性 + 26 性能，`dlc` 零违规） |

### Data Lab 记录

```
Score = 62/62 [36/36 Corr + 26/26 Perf] (142 total operators)
```

用位运算实现 13 个函数（10 个整数谜题 + 3 个浮点谜题）：

- [实验记录](labs/datalab/README.md)：成绩构成、验证命令、踩过的五个坑
- [位级洞察](labs/datalab/REFLECTION.md)：每道题"为什么这么写"、边界怎么想
- [分数凭证](labs/datalab/driver_output.txt)：`./driver.pl` 原始输出
- [实现代码](labs/datalab/bits.c)：提交文件本身

最值得记的一条：**编译失败时 `btest` 不会报错**，它会安静地测上一次编译的旧二进制，
给出一个看起来很正常的假分数。`btest` 管结果、`dlc` 管写法，两条线都得过。

## 笔记

| 笔记 | 内容 |
| --- | --- |
| [datalab 笔记](notes/datalab.md) | 贯穿全程的五个位运算套路、13 题逐题思路、IEEE 754 三个位域、踩过的五个坑 |

这份笔记是自己动手做的记录：规律 → 逐题推导 → 边界怎么翻车。
**不抄书，也不放没有亲手验证过的内容**——每条结论都在 `bits.c` 里落地过。

## 环境与流程

- [WSL2 环境搭建指南](guides/WSL2环境搭建指南.md)：一次性装好能跑全部 Lab 的 Linux 环境
- [Lab 执行手册](guides/Lab执行手册.md)：从开机到 lab 跑通的固定流程（含故障排查表）

每个 Lab 的固定循环：

```bash
cd ~/csapp/labs/<lab>-handout
make btest && ./btest    # 结果对不对（改了源码必须重编）
./dlc bits.c             # 写法合不合规则（Data Lab 专有）：静默 = 通过
./driver.pl              # 官方口径的最终成绩
```

## 目录

```
.
├── notes/                 # 自己写的笔记（按主题，不按章节抄书）
├── labs/
│   ├── datalab/           # Lab 成果：代码 + 心得 + 分数凭证
│   └── datalab-handout/   # 官方 handout 源码（方便复现）
└── guides/                # 环境搭建与实验流程
```
