- [Boot引导启动程序](#Boot引导启动程序)
	- [总结](#总结)
	- [2-1代码-定义FAT12文件系统数据结构](#2-1代码-定义FAT12文件系统数据结构)
		- [相关代码](#相关代码)
		- [功能描述](#功能描述)
	- [2-2代码-搜索硬盘中是否存在文件LOADER.BIN](#2-2代码-搜索硬盘中是否存在文件LOADER.BIN)
        - [相关代码](#相关代码)
		- [功能描述](#功能描述)
	- [2-3代码-从硬盘中读取LOADER.BIN到指定内存](#2-3代码-从硬盘中读取LOADER.BIN到指定内存)
		- [功能描述](#功能描述)
        - [相关代码](#相关代码)


# Boot引导程序

## 总结

1. 按下计算机启动按钮，主板上电，初始化CS（指向ROM）和IP寄存器,cpu开始执行ROM（只读存储器）上的程序，即BIOS（基本输入输出系统）。
2. BIOS系统功能：硬件检查；初始化中断向量表和中中断服务程序；初始化显示器显示字符程序；检测硬盘中第一扇区结尾地址有无数据0xAA55，有则加载数据（boot.asm）到内存地址0x7c00来运行。
3. boot.asm代码作用，从硬盘中搜索并加载Loader.asm文件。

## 1-1代码-定义FAT12文件系统数据结构

### 相关代码
	org	0x7c00	

BaseOfStack	equ	0x7c00

BaseOfLoader	equ	0x1000
OffsetOfLoader	equ	0x00

RootDirSectors	equ	14
SectorNumOfRootDirStart	equ	19
SectorNumOfFAT1Start	equ	1
SectorBalance	equ	17	

	jmp	short Label_Start
	nop
	BS_OEMName	db	'MINEboot'
	BPB_BytesPerSec	dw	512
	BPB_SecPerClus	db	1
	BPB_RsvdSecCnt	dw	1
	BPB_NumFATs	db	2
	BPB_RootEntCnt	dw	224
	BPB_TotSec16	dw	2880
	BPB_Media	db	0xf0
	BPB_FATSz16	dw	9
	BPB_SecPerTrk	dw	18
	BPB_NumHeads	dw	2
	BPB_HiddSec	dd	0
	BPB_TotSec32	dd	0
	BS_DrvNum	db	0
	BS_Reserved1	db	0
	BS_BootSig	db	0x29
	BS_VolID	dd	0
	BS_VolLab	db	'boot loader'
	BS_FileSysType	db	'FAT12   '
## 2-2代码-搜索硬盘中是否存在文件LOADER.BIN

## 2-3代码-从硬盘中读取LOADER.BIN到指定内存