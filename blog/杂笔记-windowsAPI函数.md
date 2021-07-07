# 杂笔记
## 1.RtlSetProcessIsCritical
```c
NTSTATUS 
RtlSetProcessIsCritical (
    BOOLEAN bNew,// new setting for process
    BOOLEAN *pbOld,// pointer which receives old setting (can be null)
    BOOLEAN bNeedScb);// need system critical breaks
```

ntdll未公开的函数,RtlSetProcessIsCritical是将进程设置为系统关键。进程终止时，Windows 本身也会终止。  
bNew设置为TRUE时，表示将当前进程标记为critical process；设置为FALSE时，当前进程不是critical process
