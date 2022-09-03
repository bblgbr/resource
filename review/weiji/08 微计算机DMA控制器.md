## 第八章 微计算机DMA控制器
---

#### 1.DMA基本概念
DMA(direct memory accsee)是一种内存访问技术。它允许某些计算机内部的硬件子系统，可以独立地直接读写系统存储器，而不需绕道中央处理器（CPU）。在同等程度的处理器负担下，DMA 是一种快速的数据传送方式。很多硬件的系统会使用DMA，包含硬盘控制器、显卡、网卡和声卡。

具体工作流程见P209


#### 2.DMA控制器8237A
见书P211页
- 4个独立编程的DMA通道，40引脚双列直插
- 控制逻辑单元
- 缓冲器
- 内部寄存器：8237 内部有12 种寄存器
- 两种状态：DMA周期和空闲周期
- 引脚定义
- 工作模式：单字节传送方式，数据块传送方式，请求传送方式，级联方式
- 完整时序：空转状态SI,请求状态S0,获得总线控制权tAV,输出地址并发出读/写控制信号S2，等待状态SW,锁存地址高8位S3，测试传输模式S


8237A编程
1. 输出主清除命令
2. 将传送数据的地址写入基地址与现行地址寄存器；
3. 将传送数据的字节数写入基字节数与现行字节数寄存器；
4. 将传送模式写入模式寄存器；
5. 写入屏蔽寄存器；
6. 写入命令寄存器；
7. 写入请求寄存器。

初始化编程
根据要求查表P213中方式寄存器，控制寄存器，

```asm

;---------------检测前禁止8237A工作----------------
MOV AL,04
OUT DMA+08H,AL
MOV AL,00 ;复位
OUT DMA+0DH,AL
;----------------写入测试数据 4组0FFH写入DMA----------------
MOV DX,DMA
MOV CX,0004
WRITE:
	MOV AL,0FFH
	OUT DX,AL
	OUT DX,AL
	INC DX
	INC DX
	LOOP WRITE

;---------------方式(模式)寄存器设置4个通道----------------
MOV AL,40H ;CH0通道
OUT DMA+0BH,AL
MOV AL,41H ;CH1通道
OUT DMA+0BH,AL
MOV AL,84H ;84H为设CH2的方式寄存器的值
OUT DMA+0BH,AL
MOV AL,43H ;CH3通道
OUT DMA+0BH,AL
;--------------控制(命令)寄存器、屏蔽寄存器与方式寄存器同理只是DMA+的数不同-------

MOV AL,A0H;计算每个通道值替换A0H
OUT DMA+08H,AL

;-------------读出数据查看结果---------------------
MOV DX,DMA
MOV CX,0004
READ:
	IN AL,DX
	MOV AH,AL
	IN AL,DX
	CMP AX,0FFFFH
	JNZ FAIL
	INC DX
	INC DX
	LOOP READ
;-------------关闭屏蔽寄存器---------------------
MOV AL,00
OUT DMA+0AH,AL
MOV AL,01
OUT DMA+0AH,AL
MOV AL,02
OUT DMA+0AH,AL
MOV AL,03
OUT DMA+0AH,AL

FAIL: HLT

```



