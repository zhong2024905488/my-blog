---
title: "实验三：Linux Shell 编程及 Makefile 实践"
date: 2026-11-08
draft: false
categories: ["Linux课程"]
tags: ["Shell", "脚本", "Makefile"]
---

## 实验目的

- 掌握用 Vim 编写 Bash 脚本的完整流程（新建、编辑、赋权、运行）；
- 理解并运用 if/elif/else 条件判断与 case 分支；
- 掌握 read 读取输入、date 生成时间戳、文件与目录操作等语法；
- 学会把运行信息同时输出到屏幕和带日期的日志文件；
- 掌握 for 循环遍历命令行参数，以及用 bash -x 调试脚本；
- 掌握 Makefile 的目标、依赖与伪目标，实现 C 程序的自动化编译。

## 实验环境

Ubuntu 20.04，GNU bash 5.x，gcc 9.x，GNU make 4.x。

## 实验内容与步骤

### 一、脚本 1：age_room.sh（年龄分房）

```bash
#!/bin/bash
read -p "Input your age! " age
if [ "$age" -ge 60 ]; then
    echo "Please go to room 501!"
elif [ "$age" -ge 50 ]; then
    echo "Please go to room 401!"
elif [ "$age" -ge 40 ]; then
    echo "Please go to room 301!"
elif [ "$age" -ge 30 ]; then
    echo "Please go to room 201!"
elif [ "$age" -ge 20 ]; then
    echo "Please go to room 101!"
else
    echo "Please wait at the door!"
fi
```

赋予执行权限并运行：

```bash
chmod +x age_room.sh
./age_room.sh
```

同一功能用 case 改写，核心写法如下：

```bash
read -p "Input your age! " age
case $age in
    [2][0-9]) echo "Please go to room 101!" ;;
    [3][0-9]) echo "Please go to room 201!" ;;
    [4][0-9]) echo "Please go to room 301!" ;;
    [5][0-9]) echo "Please go to room 401!" ;;
    [6][0-9]) echo "Please go to room 501!" ;;
    *)        echo "Please wait at the door!" ;;
esac
```

![图 1 年龄分房脚本运行结果](/images/exp3-age.png)

### 二、脚本 2：test2_backup.sh（带时间戳的备份）

```bash
#!/bin/bash
# 参数个数检查：没有传入待备份文件时提示用法并退出
if [ $# -eq 0 ]; then
    echo "错误：未指定要备份的文件或目录！"
    echo "用法：$0 <文件1> [文件2] ..."
    exit 1
fi

TS=$(date +%Y%m%d_%H%M%S)
LOG="backup_${TS}.log"

for item in "$@"; do
    cp -r "$item" "${item}.bak.${TS}" 2>&1 | tee -a "$LOG"
    echo "[$(date +%H:%M:%S)] 已备份 $item" | tee -a "$LOG"
done
```

关键点说明：

- `for item in "$@"` 遍历所有命令行参数，`"$@"` 加引号可保留带空格的路径；
- `date +%Y%m%d_%H%M%S` 生成唯一时间戳，避免多次备份互相覆盖；
- `tee -a` 同时输出到屏幕并追加写入日志，若改用 `>` 或 `>>` 则屏幕上不再有输出。

调试脚本：

```bash
bash -x test2_backup.sh ~/lab2/demo
```

若要把副本改成压缩包，把 `cp -r` 换成：

```bash
tar -zcvf "${item}.bak.${TS}.tar.gz" "$item"
```

![图 2 备份脚本运行与日志输出](/images/exp3-backup.png)

### 三、Makefile 实践

准备源文件 `main.c`，依次完成基础版、变量版、增强版 Makefile：

```makefile
# 变量版
CC = gcc
CFLAGS = -Wall -O2
TARGET = main
OBJS = main.o

$(TARGET): $(OBJS)
	$(CC) -o $@ $^

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f $(TARGET) $(OBJS)

.PHONY: clean
```

编译与清理：

```bash
make          # 只编译有改动的文件
./main
make clean    # 删除生成文件，伪目标必须声明，否则会被同名文件干扰
```

![图 3 make 编译过程与运行结果](/images/exp3-make.png)

## 遇到的问题与解决办法

1. 脚本执行报 "Permission denied"：忘记 `chmod +x`，或直接写 `bash 脚本名` 也可运行。
2. Makefile 中用空格缩进命令行报 "missing separator"：命令必须以 **Tab 开头**，不能用空格。
3. 备份带空格的目录名失败：`$item` 未加双引号，被 shell 按空格拆成多个参数。
4. `./one2n`、`bash one2n` 与 `source one2n` 的区别：前两者都新建子 shell 执行，`source` 在当前 shell 中执行，因此其中的变量会留在当前终端。

## 实验总结

本次实验掌握了 Bash 脚本的基本骨架：shebang、变量、read 输入、if/elif 与 case 分支、for 遍历参数，以及 `bash -x` 调试和 `tee` 双路输出；同时通过 Makefile 理解了目标、依赖、自动变量与伪目标的关系，实现了 C 程序的增量编译。Shell 负责把重复的人工操作变成可复用脚本，Makefile 负责按依赖关系自动构建，二者结合是 Linux 下工程化的基础。
