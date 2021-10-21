# 杂笔记
## 1.RtlSetProcessIsCritical
```c
NTSTATUS 
RtlSetProcessIsCritical (
    BOOLEAN bNew,// new setting for process
    BOOLEAN *pbOld,// pointer which receives old setting (can be null)
    BOOLEAN bNeedScb);// need system critical breaks
```

RtlSetProcessIsCritical是将进程设置为系统关键。进程终止时，Windows 本身也会终止。  
bNew设置为TRUE时，表示将当前进程标记为critical process；设置为FALSE时，当前进程不是critical process  
例：RtlSetProcessIsCritical(1,0,0)//设置  
RtlSetProcessIsCritical(0,0,0)//取消设置
## 2.ZwQueryDefaultLocale
```c
NTSYSCALLAPI NTSTATUS NTAPI ZwQueryDefaultLocale	(	
    _In_ BOOLEAN 	UserProfile,
    _Out_ PLCID 	DefaultLocaleId 
)	
```
获取当前系统的本地语言ID  
例 ZwQueryDefaultLocale(0, &localeID)
## 3.ZwContinue
```c

```
windows x86程序下软件断点到本质是在下断点处到内存里写入0xcc,而step into单步调试到本质则是通过设置cpu的flag 寄存器里面陷阱标志T位来实现的。
程序在进入到ZwContinue这个函数之后会通过系统调用启动一个新的线程并清空我们flag寄存器里面的陷阱标志T位，导致接下来我们的程序就不能单步调试了，相当于go继续执行。

ZwContinue这个函数的作用是启动新线程，其中第一个参数IN PCONTEXT ThreadContext是一个结构体，表示的是新线程的信息，而在这个结构体里面，有个成员叫做eip会表示新线程的起始地址
```c
typedef struct _CONTEXT {
  ULONG              ContextFlags;
  ULONG              Dr0;
  ULONG              Dr1;
  ULONG              Dr2;
  ULONG              Dr3;
  ULONG              Dr6;
  ULONG              Dr7;
  FLOATING_SAVE_AREA FloatSave;
  ULONG              SegGs;
  ULONG              SegFs;
  ULONG              SegEs;
  ULONG              SegDs;
  ULONG              Edi;
  ULONG              Esi;
  ULONG              Ebx;
  ULONG              Edx;
  ULONG              Ecx;
  ULONG              Eax;
  ULONG              Ebp;
  ULONG              Eip;
  ULONG              SegCs;
  ULONG              EFlags;
  ULONG              Esp;
  ULONG              SegSs;
  UCHAR              ExtendedRegisters[MAXIMUM_SUPPORTED_EXTENSION];
} CONTEXT;
```
这个结构体里面的ULONG都是4字节的,这里的SIZE_OF_80387_REGISTERS是80大小。
```c
typedef struct _FLOATING_SAVE_AREA {
    ULONG   ControlWord;
    ULONG   StatusWord;
    ULONG   TagWord;
    ULONG   ErrorOffset;
    ULONG   ErrorSelector;
    ULONG   DataOffset;
    ULONG   DataSelector;
    UCHAR   RegisterArea[SIZE_OF_80387_REGISTERS];
    ULONG   Cr0NpxState;
} FLOATING_SAVE_AREA;
```
经计算EIP在结构体偏移量就是0xb8，那么我们只要在这个地址下断点就好了。