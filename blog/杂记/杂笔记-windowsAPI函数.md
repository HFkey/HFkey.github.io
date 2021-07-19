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