# Boot引导程序

- [总结](#总结)
- [2-1代码-定义FAT12文件系统数据结构](#2-1代码-定义FAT12文件系统数据结构)
- [2-2代码-相关寄存器的初始化](#2-2代码-相关寄存器的初始化)
- [2-3代码-搜索硬盘中是否存在文件LOADER.BIN](#2-2代码-搜索硬盘中是否存在文件LOADER.BIN)
- [2-4代码-从硬盘中读取LOADER.BIN到指定内存](#2-3代码-从硬盘中读取LOADER.BIN到指定内存)

## 总结

1. 按下计算机启动按钮，主板上电，初始化CS（指向ROM）和IP寄存器,cpu开始执行ROM（只读存储器）上的程序，即BIOS（基本输入输出系统）。
2. BIOS系统功能：硬件检查；初始化中断向量表和中中断服务程序；初始化显示器显示字符程序；检测硬盘中第一扇区结尾地址有无数据0xAA55，有则加载数据（boot.asm）到内存地址0x7c00来运行。
3. boot.asm代码作用，从硬盘中搜索并加载Loader.asm文件。
4. boot.asm上装载FAT12文件系统。FAT12文件系统作用，将硬盘分成多个分区，分为引导区，FTA表项区，目录区，数据区。FTA表项中最基本的单位是簇，一个簇可以是2的整数次方个扇区。

## 2-1代码-定义FAT12文件系统数据结构

### 相关代码

```定义初始地址，Loader文件的段地址和偏移地址，段地址左移四位等于实际基地址，因为实模式下有20根地址总线，而段地址只有16位
 org 0x7c00
    BaseOfStack equ 0x7c00
    BaseOfLoader equ 0x1000
    OffsetOfLoader equ 0x00
    RootDirSectors equ 14
    SectorNumOfRootDirStart equ 19
    SectorNumOfFAT1Start equ 1
    SectorBalance equ 17

```

```FAT12的数据成员
 jmp short Label_Start
 nop
 BS_OEMName db 'MINEboot'
 BPB_BytesPerSec dw 512
 BPB_SecPerClus db 1
 BPB_RsvdSecCnt dw 1
 BPB_NumFATs db 2
 BPB_RootEntCnt dw 224
 BPB_TotSec16 dw 2880
 BPB_Media db 0xf0
 BPB_FATSz16 dw 9
 BPB_SecPerTrk dw 18
 BPB_NumHeads dw 2
 BPB_HiddSec dd 0
 BPB_TotSec32 dd 0
 BS_DrvNum db 0
 BS_Reserved1 db 0
 BS_BootSig db 0x29
 BS_VolID dd 0
 BS_VolLab db 'boot loader'
 BS_FileSysType db 'FAT12   '
```

## 2-2代码-搜索硬盘中是否存在文件LOADER.BIN

```init
 org 0x7c00
 BaseOfStack equ 0x7c00
 Label_Start:
 mov ax, cs
 mov ds, ax
 mov es, ax
 mov ss, ax
 mov sp, BaseOfStack
 mov ax, 0600h
 int 10h
 mov ax, 0200h
 mov bx, 0000h
 mov dx, 0000h
 int 10h
 mov ax, 1301h
 mov bx, 000fh
 mov dx, 0000h
 mov cx, 10
 push ax
 mov ax, ds
 mov es, ax
 pop ax
 mov bp, StartBootMessage
 int 10h
 xor ah, ah
 xor dl, dl
 int 13h
 StartBootMessage: db "Start Booting"
 times 510-（$-$$）db 0
 dw 0xaa55
```

### 相关描述

1. mov ax,0600h int 10h触发中断服务程序，实现清屏的功能，al不为零时，实现范围滚动窗口的功能，al=滚动的列数，bh=滚动后空出的位置的颜色属性（bit0~2为字体颜色，bit3为字体亮度，bit4~6为背景颜色，bit7为字体是否闪烁）
2. mov ax,0200h int 10h设置屏幕光标位置，bh=光标的列树，dl为光标的行数，bh=页码。
3. mov ah,13h int10h实现字符串的显示功能，al=写入模式（al=00h（00光标位置不变，01光标至末尾）：bl提供字符串属性，cx提供字符串长度。al=02h：字符串属性由每个支付后面紧跟的字节提供，cx改为以word为单位，光标位置不变，03光标至末尾。）
4. int 13h，ah 00h，软盘复位功能，dl=00代表第一个驱动器，dl=80h代表第一个硬盘驱动器
5. times//填充数据

## 2-3代码-从硬盘中读取LOADER.BIN到指定内存
