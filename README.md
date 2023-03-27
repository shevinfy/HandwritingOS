# HandwritingOS
This one is written and learned lunaix OS operating system, from 0 to complete. It will realize environment building, protection mode, interrupt, virtual memory, memory management, multi process &amp; system call, signal, multithreading, PCI and SATA drivers, file system, shell, network, GUI. You are welcome to watch, participate and correct the wrong knowledge（tr：这个一个手写并学习Lunaix OS操作系统与intel手册的handWriting个人操作系统，从0到完整，将实现环境搭建，保护模式，中断，虚拟内存，内存管理，多进程&amp;系统调用，信号，多线程，PCI与SATA驱动，文件系统，Shell，网络，GUI，欢迎大家观看和参与和指正不对的知识）



## 项目参考：

1. 深入理解计算机系统（Computer Systems）
2. Modern Operating Systems
3. inter手册
4. [姗姗来迟的系列引入与介绍 - 从零开始自制操作系统 EP-INTRO_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1zv4y1g7J3?spm_id_from=333.880.my_history.page.click&vd_source=54d88c3531bbb47aa4123072ef9c00aa)



## 项目安排：

1. 环境搭建
2. 引导
3. 保护模式
4. 中断
5. Hello World
6. 虚拟内存
7. 内存管理
8. 多进程&系统调用
9. 信号
10. 多线程
11. PCI与SATA驱动
12. 文件系统
13. Shell
14. 网络
15. GUI



## 支持的功能

该操作系统支持x86架构，运行在保护模式中，采用宏内核架构，目前仅支持单核心。内存结构采用经典的3:1划分，即低3GiB为用户地址空间（0x400000 ~ 0xBFFFFFFF），内核地址空间重映射至高1GiB（0xC0000000 ~ 0xFFFFFFFF）。

在下述列表中，则列出目前所支持的所用功能和特性。列表项按照项目时间戳进行升序排列。

+ 使用Multiboot进行引导启动
+ APIC/IOAPIC作为中断管理器和计时器
+ ACPI
+ 虚拟内存
+ 内存管理与按需分页（Demand Paging）
+ 键盘输入
+ 多进程
+ 47个常见的Linux/POSIX系统调用（[附录1](#appendix1)）
+ 用户模式
+ 信号机制
+ PCI 3.0
+ PCIe 1.1 (WIP)
+ Serial ATA AHCI
+ 文件系统
  + 虚拟文件系统
  + ISO9660
    + 原生
    + Rock Ridge拓展 (WIP)
+ 远程GDB串口调试 (COM1@9600Bd)

已经测试过的环境：

+ QEMU (>=7.0.0)
+ Bochs（SATA功能不支持）
+ Virtualbox
+ Dell G3 3779

## 编译与构建

### 环境搭建

构建该项目需要满足以下条件：

+ gcc (目标平台: i686-elf)
+ binutils
+ make
+ xorriso
+ grub-mkrescue

**注意：gcc不能是本机自带的，必须要从源码编译，并配置目标平台为：`i686-elf`，以进行交叉编译。配置过程可参考[附录二：编译gcc作为交叉编译器](#appendix2)。**

### Docker镜像

对于开发环境，本项目也提供了Docker镜像封装。开箱即用，无需配置，非常适合懒人或惜时者。详细使用方法请转到：[Lunaix OSDK项目](https://github.com/Minep/os-devkit)。

### 构建选项

假若条件满足，那么可以直接执行`make all`进行构建，完成后可在生成的`build`目录下找到可引导的iso。

本项目支持的make命令：

| 命令                     | 用途                                            |
| ------------------------ | ----------------------------------------------- |
| `make all`               | 构建镜像（`-O2`，但禁用CSE相关的优化项 **※** ） |
| `make instable`          | 构建镜像（`-O2`，开启CSE相关优化）              |
| `make all-debug`         | 构建适合调试用的镜像（`-Og`）                   |
| `make run`               | 使用QEMU运行build目录下的镜像                   |
| `make debug-qemu`        | 构建并使用QEMU进行调试                          |
| `make debug-bochs`       | 构建并使用Bochs进行调试                         |
| `make debug-qemu-vscode` | 用于vscode整合                                  |
| `make clean`             | 删除build目录                                   |

**※：由于在`-O2`模式下，GCC会进行CSE优化，这导致LunaixOS会出现一些非常奇怪、离谱的bug，从而影响到基本运行。具体原因有待调查。**

## 运行以及Issue

运行该操作系统需要一个虚拟磁盘镜像，可以使用如下命令快速创建一个：

```bash
qemu-img create -f vdi machine/disk0.vdi 128M
```

如果你想要使用别的磁盘镜像，需要修改`configs/make-debug-tool`

找到这一行：

```
-drive id=disk,file="machine/disk0.vdi",if=none \
```

然后把`machine/disk0.vdi`替换成你的磁盘路径。

有很多办法去创建一个虚拟磁盘，比如[qemu-img](https://qemu-project.gitlab.io/qemu/system/images.html)。

## 附录1：支持的系统调用<a id="appendix1"></a>

**Unix/Linux/POSIX**

1. `sleep(3)`
2. `wait(2)`
3. `waitpid(2)`
4. `fork(2)`
5. `getpid(2)`
6. `getppid(2)`
7. `getpgid(2)`
8. `setpgid(2)`
9. `brk(2)`
10. `sbrk(2)`
11. `_exit(2)`
12. `sigreturn(2)`
13. `sigprocmask(2)`
14. `signal(2)`
15. `kill(2)`
16. `sigpending(2)`
17. `sigsuspend(2)`
18. `read(2)`
19. `write(2)`
20. `open(2)`
21. `close(2)`
22. `mkdir(2)`※
23. `lseek(2)`
24. `readdir(2)`
25. `readlink(2)`※
26. `readlinkat(2)`※
27. `rmdir(2)`※
28. `unlink(2)`※
29. `unlinkat(2)`※
30. `link(2)`※
31. `fsync(2)`※
32. `dup(2)`
33. `dup2(2)`
34. `symlink(2)`※
35. `chdir(2)`
36. `fchdir(2)`
37. `getcwd(2)`
38. `rename(2)`※
39. `mount(2)`
40. `unmount` (a.k.a `umount(2)`)※
41. `getxattr(2)`※
42. `setxattr(2)`※
43. `fgetxattr(2)`※
44. `fsetxattr(2)`※
45. `ioctl(2)`
46. `getpgid(2)`
47. `setpgid(2)`

**LunaixOS自有**

1. `yield`
2. `geterrno`
3. `realpathat`
4. `syslog`

( **※**：该系统调用暂未经过测试 )

## 附录2：编译gcc作为交叉编译器<a id="appendix2"></a>

注意，gcc需要从源码构建，并配置为交叉编译器，即目标平台为`i686-elf`。你可以使用本项目提供的[自动化脚本](slides/c0-workspace/gcc-build.sh)，这将会涵盖gcc和binutils源码的下载，配置和编译（没什么时间去打磨脚本，目前只知道在笔者的Ubuntu系统上可以运行）。

**推荐**手动编译。以下编译步骤搬运自：<https://wiki.osdev.org/GCC_Cross-Compiler>

**首先安装构建依赖项：**

```bash
sudo apt update &&\
     apt install -y \
        build-essential \
        bison\
        flex\
        libgmp3-dev\
        libmpc-dev\
        libmpfr-dev\
        texinfo
```

**开始编译：**

1. 获取[gcc](https://ftp.gnu.org/gnu/gcc/)和[binutils](https://ftp.gnu.org/gnu/binutils)源码
2. 解压，并在同级目录为gcc和binutil新建专门的build文件夹

现在假设你的目录结构如下：

```
+ folder
  + gcc-src
  + binutils-src
  + gcc-build
  + binutils-build
```

3. 确定gcc和binutil安装的位置，并设置环境变量：`export PREFIX=<安装路径>` 然后设置PATH： `export PATH="$PREFIX/bin:$PATH"`
4. 设置目标平台：`export TARGET=i686-elf`
5. 进入`binutils-build`进行配置

```bash
../binutils-src/configure --target="$TARGET" --prefix="$PREFIX" \
 --with-sysroot --disable-nls --disable-werror
```

然后 `make && make install`

6. 确保上述的`binutils`已经正常安装：执行：`which i686-elf-as`，应该会给出一个位于你安装目录下的路径。
6. 进入`gcc-build`进行配置

```bash
../gcc-src/configure --target="$TARGET" --prefix="$PREFIX" \
 --disable-nls --enable-languages=c,c++ --without-headers
```

然后编译安装（取决性能，大约10~20分钟）：

```bash
make all-gcc &&\
 make all-target-libgcc &&\
 make install-gcc &&\
 make install-target-libgcc
```

8. 验证安装：执行`i686-elf-gcc -dumpmachine`，输出应该为：`i686-elf`

**将新编译好的GCC永久添加到`PATH`环境变量**

虽然这是一个常识性的操作，但考虑到许多人都会忽略这一个额外的步骤，在这里特此做出提示。

要想实现这一点，只需要在shell的配置文件的末尾添加：`export PATH="<上述的安装路径>/bin:$PATH"`。

这个配置文件是取决于你使用的shell，如zsh就是`${HOME}/.zshrc`，bash则是`${HOME}/.bashrc`；或者你嫌麻烦的，懒得区分，你也可以直接修改全局的`/etc/profile`文件，一劳永逸（但不推荐这样做）。

至于其他的情况，由于这个步骤其实在网上是随处可查的，所以就不在这里赘述了。

## 附录3：Issue的提交<a id="appendix3"></a>

由于目前LunaixOS没有一个完善强大的内核追踪功能。假若Lunaix的运行出现任何问题，还请按照以下的描述，在Issue里面提供详细的信息。

最好提供：

+ 可用于复现问题的描述和指引（如Lunaix运行平台的软硬件配置）
+ 错误症状描述
+ （如可能）运行截图
+ 错误消息（如果给出）
+ 寄存器状态的dump
+ （如可能）提供错误发生时，EIP附近的指令（精确到函数）。如果使用`make all-debug`，会提供`build/kdump.txt`，你可以在这里面定位。或者也可以直接`objdump`
+ （如可能）虚拟内存映射信息（QEMU下可使用`info mem`查看）。

## 附录4：串口GDB远程调试

LunaixOS内核集成了最基本的GDB远程调试服务器。可通过串口COM1在9600波特率上与之建立链接。但是，在将GDB与内核链接起来之前，还需要让内核处在调试模式下。

要进入调试模式，需要往串口（波特率如上）写入字节串 `0x40` `0x63` `0x6D` `0x63`。此时，如果屏幕底部出现一条品红色背景的`DEBUG` 字样，那么就说明LunaixOS已处在调试模式下。

注意，在这个时候，LunaixOS会开始在`COM1`上监听GDB协议信息，并且暂停一切的活动（如调度，以及对外部中断的一切响应）。用户此时需要将GDB与其挂载，并使用GDB的工作流来指示内核下一步的动作。

在目前，为了防止代码过于臃肿，LunaixOS实现的是GDB远程协议要求的最小服务端命令子集：`g`, `G`, `p`, `P`, `Q`, `S`, `k`, `?`, `m`, `M`, `X`。足以满足大部分的调试需求。

当结束调试的时候，请使用GDB的`kill`指令进行连接的断开。注意，这个指令会使得LunaixOS恢复所有暂停的活动，进入正常的运行序列，但并不会退出调试模式。GDB的挂载请求依然在LunaixOS中享有最高优先权。如果需要退出调试模式，需要往串口写入字节串：`0x40` `0x79` `0x61` `0x79`。

### GDB调试注意事项

在调试中，请避免使用`info stack`，`bt`或者任何涉及 **栈展开（Stack Unwinding）** 或者 **栈回溯（Stack Backtracing）** 的指令。否则，LunaixOS很有可能会出现 **不可预料的行为** 。