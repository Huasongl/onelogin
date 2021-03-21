# 代码分析

## 代码结构

- 定义文件系统数据结构
  - 基本表项数据
  - 表项索引函数
  - 根目录索引内核文件名函数
  - 从硬盘读取数据到内存函数
- 定义保护模式数据结构
  - 系统数据结构（gdt），全局描述符表，包含代码段和数据段描述符，需要使用LGDT汇编指令加载gdt到GDTR寄存器
  - 中断和异常
  - 分页机制
  - 多任务机制
- 部分开启地址线A20功能

## 相关代码

- 开启地址线A20功能，信号线为低电平时，地址线低20位有效。

```置位0x92端口第一位,此处使用I/O端口0x92来处理A20信号线
push ax 
in al, 92h
or al, 00000010b
out 92h, al
pop
```

- 开启保护模式

```置位cr0第一位，开启前需要加载gdt，
cli
db 0x66
lgdt [GdtPtr]
mov eax, cr0
or eax, 1
mov cr0, eax
```

- 开启Big Real Mode模式，即扩展FS寄存器寻址空间，查询1M以上的内存空间，设置后，关闭保护模式

```通过cr0第一位置0
mov ax, SelectorData32
mov fs, ax
mov eax, cr0
and al, 11111110b
mov cr0, eax
sti
```
