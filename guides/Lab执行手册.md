# CSAPP Lab 执行手册

> **定位**：从「按下电源」到「lab 跑通」的固定流程。datalab 是第一个实例；**第零、二、三、六部分对后面 5 个 lab 直接复用**。  
> **入口**：每天第一件事 → 第零部分（VS Code + WSL 四步开机）｜环境搭建 →《WSL2环境搭建指南.md》｜Lab 包说明 →《Lab/README.md》  
> **可信度**：本手册所有命令与输出，均为 **2026-10-06 / 10-07 在你这台机器上实测**（Ubuntu 26.04.1 LTS / GCC 15.2.0 / WSL 2.7.12.0 / WSL 内核 6.18.33.2；10-07 补测 VS Code + WSL 接入）。凡标注「实测」的数字都不是抄来的。

---

## 目录

- [第零部分 · 每天开工](#第零部分--每天开工)（VS Code + WSL，每天第一步）
- [第一部分 · 开机到开工](#第一部分--开机到开工)
- [第二部分 · 通用工作循环](#第二部分--通用工作循环)
- [第三部分 · datalab 专项](#第三部分--datalab-专项)
- [第四部分 · 故障排查表](#第四部分--故障排查表)
- [第五部分 · 复用到后面 5 个 lab](#第五部分--复用到后面-5-个-lab)
- [第六部分 · 归档到 GitHub（两个仓库）](#第六部分--归档到-github两个仓库)
- [附录 A · 本机环境快照](#附录-a--本机环境快照)
- [附录 B · 命令速查卡](#附录-b--命令速查卡)

---

# 第零部分 · 每天开工

> 每天做 lab 的固定起手式。**四步，一分钟**。后面 5 个 lab 全一样，只换目录名。

## 0.1 四步打开工作台

| 步   | 在哪敲             | 敲什么                                    | 看到什么算对                         |
| --- | --------------- | -------------------------------------- | ------------------------------ |
| 1   | 开始菜单 / 任务栏      | 点 **Ubuntu**（不是 VS Code，不是 PowerShell） | 提示符 `csapp@<你的主机名>:~$` |
| 2   | Ubuntu 终端       | cd ~/csapp/labs/datalab-handout        | 无输出即成功                         |
| 3   | 同一终端            | code .                                 | VS Code 窗口弹出来                  |
| 4   | VS Code **左下角** | 看有没有绿色 `WSL: Ubuntu`                   | 有 = 服务器在 Linux 侧，对了            |

第 4 步是关键：**没有 `WSL: Ubuntu` 那个角标，就说明你开的是 Windows 侧的文件**，改了也不进 Linux，编译出来还是旧代码。

> **为什么先终端后 `code .`？** 因为 `code .` 里的 `.` = "当前目录"。先 `cd` 进 lab 目录再执行，VS Code 才是以这个目录为工作区打开的；直接点 VS Code 图标打开的是空窗口，你还得自己 File → Open Folder 找路径。
>
> 已验证（2026-10-07）：这台机器 WSL 里 `code` 解析到 `/mnt/c/Users/<你的用户名>/AppData/Local/Programs/Microsoft VS Code/bin/code`，`~/.vscode-server/bin/` 下已有 server，中文语言包已装 → **`code .` 现在就能直接用**。

**第二天想省掉 1–3 步：**

- VS Code 里 `文件 → 打开最近的文件` → 点 `datalab-handout`；
- 或者 Ubuntu 终端里按 `↑` 翻出 `code .` 再回车。

这两种都会自动把 WSL 连接带回来（VS Code 会自动重连上次的远程窗口）。

## 0.2 打开的目录和文件

左侧资源管理器里应该看到这些（Windows 侧的 `\\wsl$\Ubuntu\home\csapp\csapp\labs\datalab-handout\` 是同一个目录）：

```text
bits.c        ← 唯一要改的文件
bits.h        ← 函数声明，不动
btest / btest.c / btest.h / tests.c / decl.c / ishow.c / fshow.c   ← 测试器，不动
dlc / driver.pl / Makefile / README                                ← 判分与说明，不动
bits.c.orig / bits.c.save                                          ← 你自己留的备份，不参与判分
```

要改 `bits.c`，三个进法任选：

- 资源管理器里**双击** `bits.c`，再按 **`Ctrl+G`** 跳到具体行号；
- `Ctrl+P` → 输 `bits.c` → 回车；
- Ubuntu 终端里 `code -g ~/csapp/labs/datalab-handout/bits.c:146`（直接停在第 146 行）。

> **题目在哪？** 没有"做题窗口"——每道题的题面就是函数头顶那段注释（要求 / 例子 / `Legal ops` / `Max ops` / `Rating`）。把 `return 2;` 换成你的实现。逐题行号见第 3.2 节总表。

## 0.3 VS Code 里的终端怎么用

```text
Ctrl + `      （反引号，Esc 下面那个键）
```

按这个组合键打开**集成终端**。它默认就是 **bash**（不是 PowerShell），提示符和 §0.1 里那个一样。之后改代码 → 编译 → 测试，全在这一个窗口里，不用来回切。

VS Code 里必备的四个快捷键：

| 按键 | 作用 |
| --- | --- |
| `Ctrl+反引号` | 打开 / 关闭集成终端（菜单：终端 → 新建终端） |
| `Ctrl+G` | 跳到指定行（用来找题目） |
| `Ctrl+S` | 保存（**没保存 = 没改**） |
| `Ctrl+P` | 按文件名快速打开 |

## 0.4 一分钟闭环（第零部分到第五部分的接口）

```bash
make btest && ./btest -f bitXor     # 编译 + 只测这一题
./dlc bits.c                        # 查规则：没输出 = 通过
```

改完必须 `make`，否则 `./btest` 测的还是上一次编译的二进制。完整循环见第二部分。

## 0.5 收工

- 直接关 VS Code 窗口即可，代码已经在 `/home/csapp/csapp/labs/datalab-handout/` 里落盘了；
- 想彻底退出 WSL：在 Ubuntu 终端敲 `exit`（或在 PowerShell 里 `wsl --shutdown`，那会关掉整个 WSL）；
- 关窗口**不会**丢代码；`~/csapp/` 在 WSL 里，重启电脑也还在。

## 0.6 `code .` 不出窗口怎么办

| 现象 | 原因 | 处理 |
| --- | --- | --- |
| `code: command not found` | WSL 里看不到 Windows PATH | 在 VS Code 里装 **WSL** 扩展（微软官方），然后 `wsl --shutdown` 重进；再不行用 VS Code 左下角绿色角标 `连接到 WSL` |
| VS Code 弹出来但没有 `WSL: Ubuntu` | 你在 Windows 侧的终端里敲的 `code .` | 回 §0.1 第 1 步，确认提示符是 `csapp@` |
| 改了 `bits.c` 但 `./btest` 结果不变 | 没保存 / 没 `make` | `Ctrl+S`，然后 `make btest && ./btest -f <名字>` |
| 窗口卡在 "正在打开远程" | WSL 在冷启动 | 等十几秒；仍不行就在 PowerShell 敲 `wsl --shutdown` 再重来 |

---

# 第一部分 · 开机到开工

## 1.1 你面前有两个房间

| 提示符长相                        | 你在哪                      | 能敲什么                                   |
| ---------------------------- | ------------------------ | -------------------------------------- |
| `PS C:\Users\<你的用户名>>`         | **Windows PowerShell**   | 只有 Windows 命令（`dir`、`cd C:\...`）       |
| `csapp@<你的主机名>:...$` | **Linux（Ubuntu / WSL2）** | 只有 Linux 命令（`uname`、`ls`、`gcc`、`sudo`） |

这两个房间的**同名命令不一样**，这是本项目最贵的坑（详见第四部分）。**VS Code 不在这张表里**：它的窗口在 Windows 显示，但集成终端里敲的是 Linux 命令（提示符 `csapp@...$`）——见第零部分：

| 名字     | Windows 里是什么                                      | Linux 里是什么                  |
| ------ | ------------------------------------------------- | --------------------------- |
| `sudo` | `C:\WINDOWS\system32\sudo.exe`（Win11 自带，**默认禁用**） | `/usr/bin/sudo`（真 sudo，要密码） |
| `~`    | `C:\Users\<你的用户名>`                                  | `/home/csapp`               |
| 路径写法   | `<盘符>:\<你的项目目录>`                                     | `/mnt/<盘符>/<你的项目目录>/...`           |

## 1.2 三条铁律

> **1. 提示符本身，永远不用你打。**  
> **2. `$` 或 `>` 左边的所有东西，都是电脑打给你的；你只打右边的。**  
> **3. 从别处复制命令前，先看行里有没有 `$` —— 有，就说明整行不能直接贴。**

## 1.3 开机四步（每天照做）

> **现在有 VS Code 了，日常照第零部分走**（Ubuntu 终端 → `cd ~/csapp/labs/datalab-handout` → `code .`）。下面这四步是**没有 VS Code 时**的纯终端进法，也是第零部分第 1–2 步的展开版——照样要会，因为 VS Code 出问题时就靠它。

| 步 | 动作       | 进法 A（键盘）                          | 进法 B（鼠标）          |
| - | -------- | --------------------------------- | ----------------- |
| 1 | 打开终端     | `Win` → 输 `wt` → 回车               | 开始菜单搜 `Ubuntu` 点开 |
| 2 | 进入 Linux | 敲 `wsl` → 回车                      | 已经在了              |
| 3 | **看提示符** | 必须变成 `csapp@<你的主机名>:...$` | 同左                |
| 4 | 确认位置     | 敲 `pwd`                           | 同左                |

**第 3 步不能跳。** 没变成 `csapp@` 开头，后面所有命令都会报错。

> **为什么第 4 步要 `pwd`？** 实测：`wsl` 会**继承 PowerShell 当前的目录**并翻译成 `/mnt/...` 形式。
>
> ```
> PowerShell 在 C:\Users\<你的用户名>          → wsl → /mnt/c/Users/<你的用户名>
> PowerShell 在 <你的项目目录> → wsl → <你的项目目录>
> ```
>
> 所以敲 `wsl` 进去**不是**在家目录。用鼠标进法（开始菜单 / 终端标签页）才是 `/home/csapp`。

## 1.4 一条命令到位

```
cd ~/csapp/labs/datalab-handout && pwd
```

看到 `/home/csapp/csapp/labs/datalab-handout` 就对了。

> `~` 在 Linux 里 = `/home/csapp`。所以 `~/csapp` = `/home/csapp/csapp`，读起来有点绕（用户名恰好也叫 csapp），但这就是《Lab/README.md》规则 1 定的路径。

## 1.5 环境体检（换机器 / 重装 / 觉得不对劲时跑一次）

整段贴进 Linux 终端：

```bash
echo "--- 我是谁 ---"; whoami; id -u; pwd
echo "--- 系统 ---"; uname -sr; . /etc/os-release && echo "$PRETTY_NAME"
echo "--- 工具链 ---"
for c in gcc g++ gdb make git valgrind perl python3; do
  printf '%-9s ' "$c"; command -v "$c" >/dev/null 2>&1 && echo "$(command -v "$c")" || echo "缺失"
done
echo "--- 32 位编译（datalab / bomb 的命门）---"
printf 'int main(){return 0;}\n' > /tmp/t32.c
gcc -m32 /tmp/t32.c -o /tmp/t32 && /tmp/t32 && echo "32 位: OK" || echo "32 位: 失败"
rm -f /tmp/t32.c /tmp/t32
echo "--- 资料可见吗 ---"; ls <你的项目目录>/Lab/*.tar | sed 's|.*/|  |'
```

**期望输出**（实测基线，2026-10-06）：

```
csapp
1000
Linux 6.18.33.2-microsoft-standard-WSL2
Ubuntu 26.04.1 LTS
gcc /usr/bin/gcc   g++ /usr/bin/g++   gdb /usr/bin/gdb
make /usr/bin/make git /usr/bin/git   valgrind /usr/bin/valgrind
perl /usr/bin/perl python3 /usr/bin/python3
32 位: OK
```

**缺什么就补什么：**

```bash
sudo apt update
sudo apt install -y build-essential gdb valgrind git make gcc-multilib
```

---

# 第二部分 · 通用工作循环

## 2.1 四步循环（6 个 lab 全是这个骨架）

```
① 改       nano -l +<行号> <文件>      或 VS Code
② 编       make                        ← 改了源码必须重编（VS Code 里用集成终端敲，见第零部分）
③ 测       ./<测试器>                  ← 看错在哪
④ 查规则   ./dlc bits.c  （datalab 专有） ← 没输出 = 通过
```

**最容易忘的是 ②。** datalab 官方 README 第 65-66 行原话：

> You will need to recompile btest each time you change your bits.c program.

忘了 `make`，你测的是**上一次编译的旧代码**，会对着假结果怀疑人生。养成一条命令连着打：

```
make btest && ./btest -f bitXor
```

## 2.2 编辑器：VS Code 为主，nano 备用

**默认用 VS Code**（每天开工流程见第零部分）：双击 `bits.c` → `Ctrl+G` 跳行号 → 改 → `Ctrl+S` 保存 → 集成终端里 `make btest && ./btest -f <名字>`。

VS Code 相比 nano 多出来的好处：错误能让 `gcc` 直接跳到源码行、`Ctrl+F` 全文搜函数名、`Ctrl+Z` 撤销、改错了能看出 diff。

**nano 仍留着当救命稻草**（VS Code 连不上、或者你只想在纯终端里改两行）：

```
nano -l +145 bits.c
```

- `-l` → 显示行号
- `+145` → **光标直接停在第 145 行**（省掉翻页找函数的功夫）

| 按键            | 作用               |
| ------------- | ---------------- |
| `Ctrl+O` → 回车 | 保存               |
| `Ctrl+X`      | 退出               |
| `Ctrl+W`      | 搜索               |
| `Ctrl+K`      | 删掉整行             |
| `Ctrl+_`      | 跳到指定行            |
| `Ctrl+C`      | 显示当前光标位置（第几行第几列） |

> **两个编辑器只能选一个当主场**：VS Code 和 nano 同时开同一个 `bits.c`，后保存的会把先保存的覆盖掉。规则很简单——**用 VS Code 时就别开 nano**。

## 2.3 怎么判断"我改的文件真的生效了"

| 症状                         | 原因                      | 处理                                  |
| -------------------------- | ----------------------- | ----------------------------------- |
| 改了代码但测试结果不变                | 忘了 `make`               | `make btest && ./btest -f <名字>`     |
| `make` 说 `up to date` 但你没改 | 正常，说明源码没变               | 确认你保存了（nano 是 `Ctrl+O`）             |
| 结果诡异，怀疑编译缓存                | ——                      | `make clean && make btest`          |
| `git diff` 看不到改动           | 你在 Windows 侧改了 WSL 里的文件 | 用 `\\wsl$\Ubuntu\home\csapp\...` 访问 |

> Windows 侧访问 Linux 文件的路径：`\\wsl$\Ubuntu\home\csapp\` —— 资源管理器地址栏直接贴。实测可用。

## 2.4 收工动作（每天结束前 2 分钟）

```bash
# 1. 把成果拷进仓库目录
mkdir -p ~/csapp/notes/labs/datalab
cp ~/csapp/labs/datalab-handout/bits.c ~/csapp/notes/labs/datalab/

# 2. 在仓库里写心得
nano ~/csapp/notes/labs/datalab/README.md

# 3. 退出 Linux
exit
```

> **当前状态提醒**：启动期第 2 项"GitHub 建 `csapp-notes` 仓库"和《WSL2环境搭建指南》§5"git 初始配置"**都还没做** —— 实测 `git config --global --list` 为空、`~/.ssh` 不存在。归档这步要么先补这两项，要么暂时只拷到 `~/csapp/notes/`（在 WSL 里，不会丢）。

---

# 第三部分 · datalab 专项

## 3.1 这个 lab 在干什么

**一句话**：把整数和浮点数当成「一串位」来操作 —— 不许用算术，只许用位运算。

- 你要改的文件：**只有 `bits.c`**（最终也只交这一个）
- 判分器两把尺子：
  - `btest` → **结果对不对**
  - `dlc` → **你怎么写的**（有没有用禁用的运算符、有没有偷写 `if`、运算符个数超没超）
- `driver.pl` → 把两者合成最终成绩

## 3.2 13 道题总表（行号、限制、分值均从 `bits.c` 逐个抄录）

| #   | 函数               | 行号  | 允许的运算符                      | Max ops | Rating | 梯队  |
| --- | ---------------- | --- | --------------------------- | ------- | ------ | --- |
| 1   | `bitXor`         | 146 | 只有 `~ &`                    | 14      | 1      | ①   |
| 2   | `tmin`           | 155 | 全整数                         | 4       | 1      | ①   |
| 3   | `isTmax`         | 168 | `! ~ & ^ \| +`（**无移位**）     | 10      | 1      | ①   |
| 4   | `allOddBits`     | 178 | 全整数                         | 12      | 2      | ②   |
| 5   | `negate`         | 189 | 全整数                         | 5       | 2      | ①   |
| 6   | `isAsciiDigit`   | 202 | 全整数                         | 15      | 3      | ②   |
| 7   | `conditional`    | 212 | 全整数                         | 16      | 3      | ②   |
| 8   | `isLessOrEqual`  | 222 | 全整数                         | 24      | 3      | ②   |
| 9   | `logicalNeg`     | 234 | 全整数，**禁 `!`**               | 12      | 4      | ③   |
| 10  | `howManyBits`    | 249 | 全整数                         | 90      | 4      | ③   |
| 11  | `floatScale2`    | 264 | **任意**（含 `if`/`while`/`&&`） | 30      | 4      | ④   |
| 12  | `floatFloat2Int` | 279 | **任意**                      | 30      | 4      | ④   |
| 13  | `floatPower2`    | 295 | **任意**                      | 30      | 4      | ④   |

> 「全整数」= `! ~ & ^ | + << >>`。  
> 行号 = 该函数 `int xxx(...) {` 那一行（也就是 `return 2;` 的上一两行），**2026-10-07 对着 `bits.c` 逐个复核过**（旧版抄行号时 `allOddBits`/`negate` 的顺序是反的，已修正）。纯终端用 `nano -l +<行号> bits.c`，VS Code 里双击 `bits.c` 后 `Ctrl+G` 输行号。  
> **Rating 相加 = 36**，就是 `Total points: 0/36` 那个分母。

**四个陷阱先记住：**

- `bitXor` 禁 `|` 和 `^`（只能用 `~ &`）→ 逼你学德摩根律
- `isTmax` 禁移位（只能用 `! ~ & ^ | +`）
- `logicalNeg` 禁 `!` —— 题目就是叫你**手工实现 `!`**
- 反过来 **浮点三题最自由**，随便用 `if`/`while`，只是要先懂 IEEE 754（§2.4）

## 3.3 判分机制（读 `btest.c` 源码确认的三条）

**① 单题全对才有分，没有部分分。**  
`btest.c:441`：`tscore = terrors == 0 ? 1.0 : 0.0;` —— 错一个用例，这道题的 Rating 分**整个归零**。

**② `Errors` 列常年是 1，不代表"只错了一处"。**  
`btest.c:380-382`：

```c
/* Stop testing if there is an error */
if (errors)
    return errors;
```

**撞到第一个失败用例就停了，不会往下测。** 所以别拿这个数字估进度。官方 README 也这么说：「it prints out the test that failed ... and then terminates the testing for that function」。

**③ `dlc` 通过时静默。**  
官方 README：「dlc returns silently if there are no problems with your code.」**没输出就是好消息**，别以为它没反应。

## 3.4 每道题的意义

### 逐题：学什么 → 书在哪 → 以后在哪复用

| 题                | 真正学到的东西                                                    | 书                       | 以后在哪复用                                                            |
| ---------------- | ---------------------------------------------------------- | ----------------------- | ----------------------------------------------------------------- |
| `bitXor`         | **德摩根律**：`x\|y ≡ ~(~x & ~y)`；用受限运算符拼出任意布尔函数                | §2.1.6 布尔代数、§2.1.7 位级运算 | 后面 `conditional`/`logicalNeg`/`allOddBits` 全靠它；写算子时"没有某个指令怎么等效实现" |
| `tmin`           | 补码最小值 `TMin = 0x80000000`；**不能用 `1<<31` 字面量**（大常量不许用，得移位拼） | §2.2.3 补码编码             | `INT_MIN` 的经典坑：`-INT_MIN` 还是它自己；溢出边界处理                            |
| `isTmax`         | `TMax = 0x7FFFFFFF`；`TMax + 1` 溢出成 `TMin`                  | §2.2.3、§2.3.2           | 边界值检测、溢出判断                                                        |
| `negate`         | 补码取负 = `~x + 1`                                            | §2.3.3 补码的非             | **CPU 里减法器的原理**；取绝对值                                              |
| `allOddBits`     | 构造掩码常量：`0xAAAAAAAA` 不能直接写，要用 `0xAA` 移位拼                    | §2.1.7                  | 位掩码、SIMD 向量掩码、**量化里的 bit packing**                                |
| `isAsciiDigit`   | 区间判断 = 两个不等式；用符号位替代比较                                      | §2.2.2、§2.3.1           | 字符分类、词法分析器                                                        |
| `conditional`    | **无分支编程**：用全 0 / 全 1 掩码做选择                                 | §2.1.7（配合 §3.6.5 条件传送）  | 消除分支预测失败（高性能代码的核心手法）；SIMD 的 blend                                 |
| `isLessOrEqual`  | 比较的位级实现；**符号位不同时不能直接相减**（会溢出）                              | §2.3.1、§2.3.2           | 溢出安全比较；**这就是 CPU `CMP` 指令干的事**                                    |
| `logicalNeg`     | 用 `x \| (-x)` 的符号位判断「是不是零」                                 | §2.3.3                  | 零检测；理解 `!` 的硬件实现                                                  |
| `howManyBits`    | 二分法 + **符号扩展**，预算 90 个运算符                                  | §2.2.6 扩展位表示            | 位宽压缩、**量化时搜最优 bit-width**                                         |
| `floatScale2`    | IEEE 754 单精度位段：`1 位符号 + 8 位阶码 + 23 位尾数`；**非规格化数**的处理       | §2.4.2                  | 浮点硬件；混合精度训练里的指数处理                                                 |
| `floatFloat2Int` | 浮点转整数的截断、舍入、溢出（超范围返回 `0x80000000`）                         | §2.4.4 舍入               | **量化 cast**；类型转换的隐蔽陷阱                                             |
| `floatPower2`    | 阶码偏移量 `bias = 127`；非规格化下溢、上溢到 INF                          | §2.4.2、§2.4.3           | 数值范围分析、log 域计算                                                    |

### 中观：这个 lab 在 CSAPP 体系里的位置

它是**全书唯一一个纯位级 lab**，训练的是同一个眼光：**把数据看成位模式，而不是数。**

这个眼光是后面每一个 lab 的地基：

- **bomblab**（Ch3）→ 要读汇编里的位运算，才知道那几条 `and`/`sar` 在干什么；
- **cachelab**（Ch6）→ 要手算 `s`/`E`/`b` 位段划分，**就是 datalab 里"移位 + 取位"那套动作**；
- **malloclab**（Ch9）→ 地址对齐和块头里的位标记，同一个手法；
- **tshlab**（Ch8）→ 信号位掩码。

### 宏观：对你两条长期路线的价值

对照学习路径规划：

- **408 考研**：Ch2 直接对应「计组·数据的表示和运算」。datalab 做透了，408 里补码/溢出/浮点那部分基本是复习。
- **AI Infra**：int8 / int4 量化的 bit packing、BitNet 的 1.58bit、算子里的位技巧 —— **都是这一套**。`howManyBits` 和 `floatFloat2Int` 尤其直接：量化做的就是"用多少位表示这个数"和"浮点怎么截成整数"。

## 3.5 分数阶梯与推荐顺序

**不要按文件从上到下做。** 按这个顺序，每一步都有可交付的分数：

| 阶段    | 题目                                                        | 本阶段得分            | 累计    | 前置             |
| ----- | --------------------------------------------------------- | ---------------- | ----- | -------------- |
| ① 入门  | `bitXor` `tmin` `isTmax` `negate`                         | 1+1+1+2 = **5**  | 5/36  | §2.1–2.2       |
| ② 中档  | `allOddBits` `isAsciiDigit` `conditional` `isLessOrEqual` | 2+3+3+3 = **11** | 16/36 | §2.2–2.3       |
| ③ 硬骨头 | `logicalNeg` `howManyBits`                                | 4+4 = **8**      | 24/36 | §2.3 读透        |
| ④ 浮点  | `floatScale2` `floatFloat2Int` `floatPower2`              | 4+4+4 = **12**   | 36/36 | **§2.4 精读完再动** |

> ④ 放在最后，是你《Lab/README.md》里定的（"浮点 3 题读完 2.4 再补做"）。  
> **一天一道题**是正常节奏；13 道题分布在 W3（9/21–9/27）那一周。

## 3.6 起手式：`bitXor` 该怎么想

`bits.c` 第 138-144 行的要求：

```
bitXor - x^y using only ~ and &
  Example: bitXor(4, 5) = 1
  Legal ops: ~ &
  Max ops: 14
  Rating: 1
```

**核心方法：逐位列真值表。** 一位的情况只有四种：

| x | y | x^y（要的） | x\&y | x|y |  
| --- | --- | --- | --- | --- | --- |  
| 0 | 0 | **0** | 0 | 0 |  
| 0 | 1 | **1** | 0 | 1 |  
| 1 | 0 | **1** | 0 | 1 |  
| 1 | 1 | **0** | 1 | 1 |

盯着这两个规律：

- `x & y` 只在「**两个都是 1**」时为 1 —— 而这时异或恰恰要 **0**。**它精确标出了"要排除"的位置。**
- `x | y` 在「**至少一个 1**」时为 1 —— 覆盖了异或该为 1 的三行，但多算了「两个都是 1」那行。

于是结构出来了：

```
异或 = 「至少一个 1」 且 非「两个都是 1」
     = (x | y) & ~(x & y)
```

**但 `|` 不在允许列表里。** 这就是这道 Rating 1 的题真正要教的东西 —— **德摩根律**：

```
x | y   ≡   ~(~x & ~y)
```

（读一遍就通了：「x 或 y」等价于「并非（x 和 y 都不成立）」。）

**把它代进上式，就只剩 `~` 和 `&` 了 —— 最后这一步，你自己写进第 146 行。**

## 3.7 完成标准（打勾用的）

- [ ] `./btest` → `Total points: 36/36`
- [ ] `./dlc bits.c` → **无输出**（静默 = 通过）
- [ ] `./dlc -e bits.c` → 每题的运算符个数都 ≤ 该题 Max ops
- [ ] `./driver.pl` → 记录最终分
- [ ] `bits.c` 拷进 `~/csapp/notes/labs/datalab/`
- [ ] 心得写进 `~/csapp/notes/labs/datalab/README.md`
- [ ] 在进度表里把「datalab 全部通过」勾上 `[x]`

---

# 第四部分 · 故障排查表


**全部为本次实测踩过的坑**，不是从网上抄的。

## 4.1 房间搞错类（最高频）

| 症状                                            | 真实原因                                                                                 | 处理                                        |
| --------------------------------------------- | ------------------------------------------------------------------------------------ | ----------------------------------------- |
| `tar: Failed to open '/mnt/d/...'`            | 在 **PowerShell** 里写了 Linux 路径                                                        | 先 `wsl` 进 Linux；或在 PowerShell 里写 `D:\...` |
| `已在此计算机上禁用 Sudo`                              | 打到了 **Windows 自带的 `sudo.exe`**（默认禁用），不是 Linux sudo                                   | **别去开设置里那个开关**（开了也没用）；先 `wsl` 进 Linux     |
| `uname : 无法将"uname"项识别为...`                   | `uname` 是 Linux 命令，Windows 没有                                                        | 先进 WSL                                    |
| `cd : 找不到路径"C:\Users\<你的用户名>\csapp\..."`        | PowerShell 把 `~` 翻成 `C:\Users\<你的用户名>`；但 `~/csapp` 是 **Linux 家目录** `/home/csapp/csapp` | 先进 WSL 再 `cd ~/csapp`                     |
| `csapp@<你的主机名>:~$: command not found` | **把提示符一起复制**进 shell 了                                                                | 只打 `$` 右边的（铁律 2）                          |
| `Linux: command not found`                    | 把**上一条命令的输出**当命令敲了                                                                   | 只打你该打的，输出是电脑回的                            |

## 4.2 环境类

| 症状                                                                  | 真实原因                                                          | 处理                                                                       |
| ------------------------------------------------------------------- | ------------------------------------------------------------- | ------------------------------------------------------------------------ |
| `wsl --list --online` 超时 `Wsl/WININET_E_TIMEOUT`                    | 它要读 `raw.githubusercontent.com`，被墙                            | 绕开：`wsl --install -d Ubuntu --web-download`                              |
| `command not found: gcc`                                            | 工具链没装                                                         | `sudo apt install -y build-essential gdb valgrind git make gcc-multilib` |
| `make` 报 `bits/libc-header-start.h` 找不到                             | 缺 32 位头文件                                                     | `sudo apt install gcc-multilib`                                          |
| apt 每次刷 `N: Ignoring file 'xxx.bak' ... invalid filename extension` | 备份文件放在 `/etc/apt/sources.list.d/` 里，apt 只认 `.sources`/`.list` | 备份挪出去（如 `/root/`）                                                        |
| apt 下载慢到怀疑人生                                                        | 用的是国外官方源                                                      | 换清华源，见《WSL2环境搭建指南.md》                                                    |
| 想看 `dlc` 帮助却卡住不动                                                    | **`dlc` 不带参数时会读 stdin**，把你的输入吃掉了                              | `./dlc bits.c` 或 `./dlc < /dev/null`                                     |

## 4.3 lab 类

| 症状                                | 真实原因                                         | 处理                              |
| --------------------------------- | -------------------------------------------- | ------------------------------- |
| `./btest` 全红，`Total points: 0/36` | **正常起点**，13 个函数都是桩代码 `return 2;`             | 开始做题                            |
| 改了 `bits.c` 但分数不变                 | 忘了 `make`                                    | `make btest && ./btest -f <名字>` |
| `./btest -f bitxor` 报找不到函数        | **函数名区分大小写**                                 | 必须写 `bitXor`                    |
| `btest` 全绿但 `driver.pl` 扣分        | `dlc` 规则检查没过（用了禁用的运算符 / 超 Max ops / 偷写 `if`） | 每道题都跑 `./dlc bits.c`            |
| `./dlc bits.c` 什么都不打印             | **这是通过**（静默 = 无问题）                           | 看 `echo $?` 应为 0                |
| 不知道每题用了几个运算符                      | ——                                           | `./dlc -e bits.c`，会列出每题计数       |

---

# 第五部分 · 复用到后面 5 个 lab

## 5.1 通用套用四步

```bash
# ① 在 WSL 家目录解压（不要解在 /mnt/d 下：《Lab/README.md》规则 1）
mkdir -p ~/csapp/labs && cd ~/csapp/labs
tar xf <你的项目目录>/Lab/<包名>.tar

# ② 进目录，先看有什么
cd <解出的目录>
ls -la

# ③ 先读 README（每个 lab 都有官方 README，写了构建和测试方式）
#    VS Code：Ctrl+P 输 README；纯终端：nano -l README
code README

# ④ 用 VS Code 打开这个 lab，找"你要改的那一个文件"，进入四步循环
code .
```

> **规律**：每个 lab 都有**唯一一个你要改的文件**。认准它，别乱动别的。
>
> **每天重新开工**：不用重解压，照第零部分四步走（`cd` 到该 lab 目录 → `code .`）。

## 5.2 六个 lab 对照总表

| Lab           | 章   | 包名                      | 解出目录                 | **你要改的文件**                                  | 构建     | 测试                                             | 交什么                |
| ------------- | --- | ----------------------- | -------------------- | ------------------------------------------- | ------ | ---------------------------------------------- | ------------------ |
| **datalab**   | Ch2 | `datalab-handout.tar`   | `datalab-handout/`   | **`bits.c`**                                | `make` | `./btest` / `./dlc` / `./driver.pl`            | `bits.c`           |
| **bomblab**   | Ch3 | `bomb.tar`              | `bomb/`              | （无源码）读 `bomb.c` 了解框架                        | 无（预编译） | `./bomb` 交互输答案                                 | 答案清单               |
| **attacklab** | Ch3 | `target1.tar`           | `target1/`           | 自己写 exploit 文件                              | 无      | `./ctarget -q < exploit.txt`                   | exploit 文件         |
| **cachelab**  | Ch6 | `cachelab-handout.tar`  | `cachelab-handout/`  | **`csim.c`**（Part A）+ **`trans.c`**（Part B） | `make` | `./test-csim` / `./test-trans` / `./driver.py` | `csim.c` `trans.c` |
| **tshlab**    | Ch8 | `shlab-handout.tar`     | `shlab-handout/`     | **`tsh.c`**                                 | `make` | `make test01`…`test16` / `./sdriver.pl`        | `tsh.c`            |
| **malloclab** | Ch9 | `malloclab-handout.tar` | `malloclab-handout/` | **`mm.c`**                                  | `make` | `./mdriver -V`                                 | `mm.c`             |

> 各包的实际内容（实测 `tar tf`）：cachelab 还带 `csim-ref` 参考实现、`traces/*.trace`、`driver.py`；tshlab 带 `tshref` 参考 shell、`trace01-16.txt`、`sdriver.pl`、`myspin/mysplit/mystop/myint` 测试辅助程序；malloclab 带 `mdriver.c`、`memlib.c`、`clock.c`、`fcyc.c` 及 `.o` 和两个测试 trace。

## 5.3 每个 lab 的特有坑

### bomblab（Ch3，W4–W6）

- **没有 Makefile**，`bomb` 是预编译二进制，直接 `./bomb`。
- 拆弹的**唯一手段是 `gdb`**：`objdump -d bomb > bomb.s` 拿汇编，或 `gdb bomb` 单步看 `explode_bomb` 前的比较。
- 32 位程序，**依赖 `gcc-multilib`**（你已装好）。
- 有防爆机制，输错就 `BOOM`；建议先在 gdb 里跑，别裸跑。

### attacklab（Ch3，选做）

- 解出目录叫 `target1/`（不是 `attacklab/`）。
- `./ctarget` 有服务器校验，**必须加 `-q` 跳过**：`./ctarget -q < exploit.txt`。
- 用自带的 `./hex2raw` 把十六进制文本转成原始字节：`./hex2raw < exploit.txt | ./ctarget -q`。
- `farm.c` 是找 gadget 的"农场"（ROP 用）。

### cachelab（Ch6，W8–W9）

- Part A 写 `csim.c`（缓存模拟器），用 `./test-csim` 和官方 `csim-ref` 对拍。
- Part B 写 `trans.c`（矩阵转置优化），用 `./test-trans -M 32 -N 32`。
- **`./driver.py` 需要 Python。** 你机器上 `python3` 有（3.14.4），但 **`python` 这个命令没有**（实测）。如果 `./driver.py` 报 `python: not found`，就改用：
  ```bash
  python3 driver.py
  ```
- Part B 靠 `valgrind` 数 cache miss，**跑得慢是正常的**（你《WSL2环境搭建指南》故障表里也写了）。

### tshlab（Ch8，W11–W12）

- 写 `tsh.c` 实现 tiny shell。
- 官方测试：`make test01` … `make test16`（每个 trace 一个目标）。
- 也可以手动：`./sdriver.pl -t trace01.txt -s ./tsh -a "-p"`，跟参考实现 `tshref` 的输出比。
- 信号处理是重头（`SIGINT`/`SIGTSTP`/`SIGCHLD`），务必把自测验收清单逐条走一遍。

### malloclab（Ch9，W13–W14）

- 写 `mm.c` 实现 `malloc`/`free`/`realloc`。
- 官方测试：`./mdriver -V`（`-V` 打印详细统计）。
- **评分是双指标**：吞吐率 × 峰值利用率。别只追一个 —— 这个权衡正是「用 Ch9 理论复盘我的内存池」那篇复盘要写的东西。

---

# 第六部分 · 归档到 GitHub（两个仓库）

> 目标：**技术笔记公开、个人计划留本地**。踩坑与实验记录值得给人看，计划类文档不适合放公网。
> 2026-10-09 已按本节搭好并本地提交，只剩 GitHub 网页上的两步（见 6.2）。

## 6.1 两个仓库的分工

| 仓库 | 位置（WSL） | 内容 | 去处 |
| --- | --- | --- | --- |
| **csapp-public** | `~/csapp-public` | `ch02/` 笔记、`labs/datalab/`（代码 + 心得 + 分数凭证）、`guides/`（环境与流程） | **推 GitHub** |
| **csapp-notes** | `~/csapp-notes` | 上面全部 **+** 个人计划类文档 | 本地保留 |

两个仓库各自独立 `git init`；公开版是从完整版**净化后复制**出来的，所以：

- 公开版里的本机路径已替换成 `<你的项目目录>`；
- 指向个人计划类文档的引用已删净；
- 有自动隐私自检脚本，会扫描私人文档名、本机路径、账号邮箱。

**改动顺序**：先改完整版 → 再重跑净化脚本 → 公开版重新提交。别反过来，否则公开版会漏掉后续修改。

## 6.2 GitHub 上要做的两步（只需一次）

**第一步：建仓库**

1. 打开 <https://github.com/new>
2. Repository name 填 **`csapp-notes`**
3. 选 **Public**；**不要**勾 "Add a README file"（本地已有）
4. 点 **Create repository**

**第二步：配置 SSH 认证**

```bash
ssh -T git@github.com
```

看到 `Hi <你的用户名>! You've successfully authenticated...` 就说明可以推了。

- **没有 key** → `ssh-keygen -t ed25519 -C "your@email"`，把 `~/.ssh/id_ed25519.pub` 的内容
  粘到 <https://github.com/settings/keys>；
- **只想授权这一个仓库** → 用仓库的 **Deploy key**（Settings → Deploy keys），
  **必须勾上 `Allow write access`**，否则只能拉不能推；
- **22 端口被墙** → 在 `~/.ssh/config` 让 github.com 走 443：`HostName ssh.github.com` / `Port 443`。

**第三步：推送**

```bash
cd ~/csapp-public
git remote add origin git@github.com:<你的GitHub用户名>/csapp-notes.git
git push -u origin main
```


## 6.3 以后每做完一个 lab

```bash
# ① 成果归档进完整版
mkdir -p ~/csapp-notes/labs/bomblab
cp <bomblab 的答案/代码> ~/csapp-notes/labs/bomblab/
cd ~/csapp-notes && git add -A && git commit -m "bomblab: 拆完 6 关"

# ② 净化出公开版（工具在 <你的项目目录>\工具\，已装到 ~/tools/）
python3 ~/tools/01-抽出公开版.py     # 抽内容（自动保留 .git 历史）
python3 ~/tools/02-净化公开版.py     # 换占位符 + 隐私复检，不干净就拒绝继续
python3 ~/tools/03-提交并推送.py     # 两个仓库提交 + 推送

# 也可以一条命令走完（等价于上面三步，任一步失败就停）
python3 ~/tools/归档到GitHub.py "bomblab: 拆完 6 关"

# ③ 推公开版
cd ~/csapp-public && git push
```

---

# 附录 A · 本机环境快照

**几个非显然的事实：**

1. `Microsoft-Windows-Subsystem-Linux` 这个 Windows 可选功能**是禁用的** —— 但这**不影响**你。你装的是 Store 版 WSL 2.x，它只依赖 `VirtualMachinePlatform`（已启用）。禁用那个组件只意味着**不能用 WSL1**。
2. `wsl -d Ubuntu -u root` 是**免密**的（WSL 的 root 默认无密码）—— 忘了用户密码时可以用它重置：`wsl -d Ubuntu -u root -- passwd csapp`。
3. Windows Terminal 的**默认配置是 PowerShell**，但里面**已经有 `Ubuntu` 配置**（紫色 `#300A24` 底、起始目录 `~`）。想换默认：`Ctrl+,` 打开设置 → 改「默认配置文件」。
4. 发行版装在 C 盘。现在挪最便宜：`wsl --shutdown` 后 `wsl --manage Ubuntu --move D:\WSL\Ubuntu`。不挪也够用（C 盘还剩约 90 GB）。

---

# 附录 B · 命令速查卡

## B.1 进出发 Linux

| 目的             | 命令                                 |
| -------------- | ---------------------------------- |
| 进 Linux        | `wsl`（在 PowerShell 里敲）             |
| 出 Linux        | `exit` 或 `Ctrl+D`                  |
| 直接发一条命令给 Linux | `wsl -d Ubuntu -- <命令>`            |
| 直接发一串 Linux 脚本 | `wsl -d Ubuntu -- bash -lc '<脚本>'` |
| 以 root 身份发命令   | `wsl -d Ubuntu -u root -- <命令>`    |

## B.2 目录与文件

| 目的        | 命令                                       |
| --------- | ---------------------------------------- |
| 我在哪       | `pwd`                                    |
| 有什么       | `ls` / `ls -la`                          |
| 去哪        | `cd <路径>` / `cd ~` / `cd ..` / `cd -`    |
| 看文件       | `cat <文件>` / `less <文件>`（`q` 退出）         |
| 头几行 / 尾几行 | `head -20 <文件>` / `tail -20 <文件>`        |
| 找文件       | `find . -name "*.c"`                     |
| 找内容       | `grep -n "关键词" <文件>`                     |
| 建目录       | `mkdir -p <路径>`                          |
| 拷 / 移 / 删 | `cp` / `mv` / `rm`（⚠️ `rm -rf` 慎用，没有回收站） |

## B.3 datalab 专用

| 目的     | 命令                            |
| ------ | ----------------------------- |
| 编译     | `make btest`                  |
| 测全部    | `./btest`                     |
| 只测一道   | `./btest -f bitXor`           |
| 只看分数   | `./btest -g`                  |
| 指定参数测  | `./btest -f bitXor -1 5 -2 3` |
| 查编码规则  | `./dlc bits.c`（静默 = 通过）       |
| 看运算符计数 | `./dlc -e bits.c`             |
| 官方评分   | `./driver.pl`                 |
| 重新来过   | `make clean && make btest`    |

## B.4 编辑

| 目的              | 命令                    |
| --------------- | --------------------- |
| 在 VS Code 里打开整个 lab | `code .`（先在 lab 目录里）  |
| VS Code 里跳到第 146 行 | 双击 `bits.c` → `Ctrl+G` |
| 命令行直接停在某文件某行   | `code -g bits.c:146`  |
| 带行号打开并跳到第 145 行 | `nano -l +145 bits.c` |
| 保存              | `Ctrl+O` → 回车         |
| 退出              | `Ctrl+X`              |
| 搜索              | `Ctrl+W`              |
| 跳到指定行           | `Ctrl+_`              |
| 看当前行号           | `Ctrl+C`              |

## B.5 求助与自检

| 目的        | 命令                                  |
| --------- | ----------------------------------- |
| 这个命令怎么用   | `<命令> --help` / `man <命令>`          |
| 看上条命令成功没有 | `echo $?`（0 = 成功）                   |
| 刚才那条命令是什么 | `history`                           |
| 全环境体检     | 见 [1.5](#15-环境体检换机器--重装--觉得不对劲时跑一次) |
| 每天开工      | 见 [第零部分](#第零部分--每天开工)（Ubuntu 终端 → `cd` 到 lab 目录 → `code .`） |

---

*本手册随 lab 推进更新。做完 datalab 后，把 3.6 的「起手式」换成你自己总结的"这类题怎么想"，第五部分补上每个 lab 的真实卡点。*
