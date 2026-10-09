# WSL2 环境搭建指南（Windows 10 / 11）

> 目标：30 分钟内获得一个能编译运行 CSAPP 全部 Lab 的 Linux 环境。
> CSAPP 的 Lab 都是 Linux 工具链产物，Windows 原生（MinGW/Cygwin）会踩大量坑，一律用 WSL2。

## 1. 安装 WSL2 + Ubuntu

以**管理员身份**打开 PowerShell，执行：

```powershell
wsl --install -d Ubuntu
```

- 该命令自动启用"适用于 Linux 的 Windows 子系统"和"虚拟机平台"，并安装 WSL2 内核 + Ubuntu；
- 如果你的系统较旧导致失败，先手动启用功能再装：

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
# 重启电脑后：
wsl --set-default-version 2
wsl --install -d Ubuntu
```

- 重启后 Ubuntu 会自动初始化，让你设置一个用户名和密码（密码用于 sudo，输错可重装，别紧张）；
- 若提示"虚拟化未启用"，进 BIOS 打开 Intel VT-x / AMD SVM。

验证：PowerShell 里 `wsl -l -v` 应显示 Ubuntu，VERSION 为 2。

## 2. 装编译工具链

进入 Ubuntu（开始菜单点 Ubuntu，或 PowerShell 输入 `wsl`），执行：

```bash
sudo apt update
sudo apt install -y build-essential gdb valgrind git make gcc-multilib
```

- `gcc-multilib` 很重要：datalab 的 btest 和 bomblab 的 32 位程序需要它；
- 验证：

```bash
gcc --version && gdb --version && valgrind --version
```

## 3. 接入 VS Code（可选但强烈推荐）

> **状态：✅ 已装（2026-10-07 实测）**。Windows 侧 VS Code 在 `C:\Users\<你的用户名>\AppData\Local\Programs\Microsoft VS Code\`，WSL 里 `code` 可直接解析到它的 `bin\code`，`~/.vscode-server/` 下已有 server。
> **每天的开工流程见【 Lab执行手册.md 】第零部分**，这里只留一次性接入步骤。

1. Windows 上安装 VS Code；（✅ 已完成）
2. 在 VS Code 里安装扩展 **WSL**（微软官方，搜 wsl 第一个）；（✅ 已可远程连接）
3. 在 Ubuntu 终端里进入你的工作目录后执行 `code .`，VS Code 会自动以 WSL 模式打开，终端、调试、git 全部在 Linux 侧运行。

- 判断成功：VS Code **左下角出现绿色 `WSL: Ubuntu`**。没有这个角标 = 你改的是 Windows 侧的文件，编译不会生效；
- 打开集成终端：`` Ctrl+` ``（默认就是 bash，不是 PowerShell）。

## 4. 目录约定与文件互访

- 学习项目建议放 WSL 内（I/O 快、工具链顺）：`~/csapp/`；
- Windows 资源管理器访问 WSL 文件：地址栏输入 `\\wsl$\Ubuntu\home\<你的用户名>\csapp`；
- 反过来，WSL 内访问 Windows 的 D 盘：`/mnt/d/`（例如下载的 Lab 包在 `/mnt/c/Users/<你>/Downloads/`）；
- 建议把演示代码拷到 WSL 内再编译，避免跨文件系统 I/O 慢。

## 5. git 初始配置（一次性）

```bash
git config --global user.name  "你的GitHub用户名"
git config --global user.email "你的GitHub邮箱"
```

## 6. 下载官方 Lab

浏览器打开 **http://csapp.cs.cmu.edu/3e/labs.html**，每个 Lab 下载 *Self-Study Handout*（自学版，无需学校账号）。以 datalab 为例：

```bash
mkdir -p ~/csapp/labs && cd ~/csapp/labs
# 把下载的 tar 包放到此处（Windows 下载目录在 /mnt/c/Users/<你>/Downloads/）
tar xf datalab-handout.tar && cd datalab-handout
make            # 编译通过即环境 OK
./btest         # datalab 的测试器
```

各 Lab 对应关系：datalab→Ch2、bomblab/attacklab→Ch3、cachelab→Ch6、shllab(tsh)→Ch8、malloclab→Ch9。

## 7. 常见问题

| 症状 | 处理 |
|---|---|
| `wsl --install` 报错或找不到命令 | 按第 1 节手动启用两个功能后重启再试；Win10 需 2004 以上版本 |
| datalab `make` 报 bits/libc 头文件错误 | `sudo apt install gcc-multilib` 后重新 make |
| bomb 运行报 No such file | 32 位运行库缺失，同样装 `gcc-multilib` |
| valgrind 跑得慢 | 正常现象，Lab 数据量不大，等即可 |
| 终端中文乱码 | Ubuntu 默认 UTF-8，一般不会出现；若出现 `sudo apt install language-pack-zh-hans` |
| 想彻底重来 | PowerShell: `wsl --unregister Ubuntu`（会删掉 WSL 内全部文件，慎用） |

## 8. 完成标志

在 Ubuntu 里成功执行下面三行，即环境搭建完成：

```bash
echo 'int main(){return 0;}' > /tmp/t.c && gcc /tmp/t.c -o /tmp/t && /tmp/t && echo OK
```
