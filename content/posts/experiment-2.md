---
title: "实验二：Linux 文件管理及文件过滤实践"
date: 2026-10-18
draft: false
categories: ["Linux课程"]
tags: ["文件管理", "Vim", "grep", "sed", "awk"]
---

## 实验目的

- 掌握文件管理操作（创建、复制、移动、删除、链接）；
- 掌握 Vim 编辑器的基本使用（进入、编辑、保存、查找、替换）；
- 掌握 grep、sed、awk 三种文本过滤工具的基本用法；
- 理解正则表达式与管道，并能组合使用这些工具完成检索、过滤与统计。

## 实验环境

Ubuntu 20.04，Vim 8.x，GNU grep / sed / awk。

## 实验内容与步骤

### 一、文件管理操作

```bash
mkdir -p ~/lab2/demo
touch ~/lab2/demo/file1.txt
cp ~/lab2/demo/file1.txt ~/lab2/demo/file2.txt
mv ~/lab2/demo/file2.txt ~/lab2/demo/file2.bak
rm -i ~/lab2/demo/file2.bak
```

**硬链接与软链接对比**：

```bash
ln ~/lab2/demo/file1.txt ~/lab2/demo/hard.txt      # 硬链接
ln -s ~/lab2/demo/file1.txt ~/lab2/demo/soft.txt   # 软链接（符号链接）
ls -li ~/lab2/demo
```

执行后可以看到：`hard.txt` 与 `file1.txt` 的 inode 号相同、链接计数为 2；`soft.txt` 是独立 inode，内容指向源文件路径。删除源文件后，硬链接仍可访问，软链接则失效（出现红色闪烁的断链）。

![图 1 硬链接与软链接的 inode 对比](/images/exp2-ln.png)

### 二、Vim 编辑器练习

```bash
vim ~/lab2/demo/note.txt
```

在 Vim 中依次练习：按 `i` 进入插入模式输入内容，按 `Esc` 回到普通模式，`:wq` 保存退出；用 `/关键字` 查找、`:%s/旧/新/g` 全文替换、`dd` 删除整行、`yy` 复制整行、`p` 粘贴、`u` 撤销。

![图 2 使用 Vim 编辑文件](/images/exp2-vim.png)

### 三、文件过滤实践

**grep 检索**：

```bash
grep -n "root" /etc/passwd          # 显示匹配行并标出所在行号
grep -v "nologin" /etc/passwd       # 反向匹配，排除 nologin 用户
grep -r "ssh" /etc 2>/dev/null      # 递归搜索目录
```

**sed 流编辑**：

```bash
sed 's/Ubuntu/ubuntu/g' /etc/os-release     # 仅输出到屏幕，不改文件
sed -i 's/file/demo/g' ~/lab2/demo/note.txt # 直接修改文件
```

**awk 字段处理**：

```bash
awk -F: '{print $1, $7}' /etc/passwd        # 以冒号分隔，取第 1、7 列
awk -F: '$3>=1000 {print $1}' /etc/passwd   # 筛选 UID 大于等于 1000 的普通用户
```

**管道组合**：

```bash
awk -F: '{print $7}' /etc/passwd | sort | uniq -c | sort -rn
```

以上命令统计了 `/etc/passwd` 中各类 shell 的使用次数，并按次数从高到低排序，输出类似：

```
      22 /usr/sbin/nologin
       1 /bin/bash
       1 /bin/sync
```

![图 3 grep / sed / awk 与管道组合结果](/images/exp2-filter.png)

## 遇到的问题与解决办法

1. `ln` 创建硬链接时报 "Invalid cross-device link"：硬链接不能跨文件系统，改到同一目录下创建。
2. `sed -i` 修改后中文出现乱码：文件编码不是 UTF-8，先用 `file -i` 确认编码再处理。
3. `grep -r /etc` 输出大量 "Permission denied"：加上 `2>/dev/null` 丢弃标准错误即可。

## 实验总结

本次实验练习了文件的创建、复制、移动与链接，理解了硬链接共享 inode、软链接类似快捷方式的本质区别；熟悉了 Vim 的普通模式与插入模式切换及常用编辑命令；掌握了 grep 检索、sed 流编辑、awk 字段统计三者的分工，并通过管道把它们组合起来完成复杂文本处理。核心体会是：grep 管"找哪几行"，sed 管"改这几行"，awk 管"按列算什么"。
