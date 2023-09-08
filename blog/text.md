# 一个EHSTOR木马分析

样本来源网络

    IOC
    MD5:88d9e88e61e538f89688f26ab43fc3a5
    SHA1:7e77b925973755da363ad876c31d3552e91ed725
    SHA256:ac38403a3188bfe31850a3710cdd1311abe9f7bdaa0e23add7eda6196

## 1.概述
该样本为一个窃密远控木马，会创建EhStorAuthn.exe进程注入执行恶意代码。对语言1俄语，乌克兰，白俄罗斯语，亚美尼亚语，哈萨克语，罗马尼亚语 - 摩尔多瓦，俄语 - 摩尔多瓦设置白名单，推测作者应该是其中地区。外联域名main21.space，备用main21.site、main21.xyz。
![概述](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/一个EHSTOR木马分析报告.png)

## 2.详细分析
休眠5秒后，尝试创建does_not_exist.exe程序，创建成功即退出。不成功继续执行。  
![尝试创建标识程序](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/1.png)
### 动态解密
该样本通过动态解密方式载入API函数和所需字符串，可以调试后修改相关地址名为所存API函数便于分析，主体代码执行流程如下  
![整体代码](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/2.png)

### 运行环境检测
sub_4023AA()中分三个函数监测，满足任一即删除自身。  
![运行环境检测](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/3.png)

第一个函数sub_402232检测是否被调试。(原API函数为一段地址，编者为便于分析已做名称修改)  
![调试检测](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/4.png)

第二个函数sub_4021DD检测系统语言。排除1049俄语，1058乌克兰，1059白俄罗斯语，1067亚美尼亚语，1087哈萨克语，2072罗马尼亚语 - 摩尔多瓦，2073俄语 - 摩尔多瓦。  
![语言检测](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/5.png)

第三个函数sub_40226D检测虚拟机。对注册表"SYSTEM\CurrentControlSet\Services\disk\Enum\0"的键值匹配字串QEMU、VIRTIO、VMWARE、VBOX、XEN来识别虚拟机，不过匹配上会区分大小写，如遇到类似VMWare会识别失败。  
![虚拟机检测](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/6.png)

如果满足任一函数条件，进入sub_4043F1函数，创建并执行删除脚本del.bat  
![删除自身](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/7.png)

### 判断主机是否被感染
sub_404BEC函数通过检测"C:\Users\%USERNAME%\AppData\Local\z_%USERNAME%\wallpaper.mp4"文件是否存在判断主机是否被感染，wallpaper.mp4其实是运行时被拷贝重命名的"C:\windows\system32\ntdll.dll"文件，后面会看到。  
![删除自身](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/8.png)

### 首次创建文件
相应路径下无wallpaper.mp4文件则跳转sub_403835函数。  
遍历注册表"SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Uninstall"键，满足第v11(当前时间秒值加2)个键存在"DispalyName"键时，将该键"DisplayName"值作为文件名。不过基本上，是用后续生成的随机5位数字(实际用kernel32.GetSystemTime函数获得当前时间秒和毫秒计算获得，秒*1000+毫秒)，作为之后复制自身的文件名。  
![生成文件名](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/9.png)

一系列字符串解码拼接后，创建文件夹"C:\Users\%USERNAME%\AppData\Local\z_%USERNAME%"并设置隐藏属性，  
复制ntdll.dll到"C:\Users\%USERNAME%\AppData\Local\z_%USERNAME%\wallpaper.mp4"，  
复制自身到开机自启动目录，文件名为之前生成的5位数字  
![释放文件](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/10.png)

创建并写入%USERNAME%.vbs文件，用于执行%USERNAME%.bat脚本。  
![创建%USERNAME%.vbs](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/11.png)

创建并写入%USERNAME%.bat文件，如果当前进程有EhStorAuthn.exe在执行则退出，没有则执行被拷贝后的本体。  
![创建%USERNAME%.bat](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/12.png)

然后执行被拷贝后的本体，调用之前提过的sub_4043F1函数删除自身。  
![创建%USERNAME%.bat](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/13.png)

### EhStorAuthn进程注入
识别当前进程不为EhStorAuthn.exe后，进程注入  
![进程注入](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/14.png)  
![进程注入2](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/15.png)

### 提权
创建互斥体，设置%USERNAME%.vbs注册表开机自启动，提权Debug权限，如果提权失败，再设置%USERNAME%.vbs计划任务，定时执行。  
![互斥体](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/16.png)  
![提权](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/17.png)  
![计划任务](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/18.png)

### 窃密远控
在功能函数执行过程中，使用RtlSetProcessIsCritical函数设置为系统关键进程，强制退出会导致蓝屏。  

获取本机和用户信息，先用自身算法加密，再通过base64编码，作为transfer值，发送给main21.space，备用域名main21.site和main21.xyz  
![窃密](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/19.png)

向"/adm2021/gate.php"发起POST请求  
![窃密](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/20.png)

由于域名现均访问不到，后续功能只能根据静态代码推测  
接受返回数据并解密  
![接受指令](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/21.png)

进入sub_401183函数执行指令，提取指令和参数  
![提取参数](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/22.png)

有"de"，"de:loadmemory"，"update"，"uninstall"四种指令
#### "de"指令  
调用函数sub_404A0F根据参数，选择路径，下载文件  
![路径](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/23.png)  
![下载](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/24.png)

如果是exe文件，则执行；是dll文件，则注册。  
![执行或注册](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/25.png)

#### "de:loadmemory"
用http请求GET获得代码，再创建新的EhStorAuthn.exe进程再注入。  
![主体](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/26.png)  
![GET请求](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/27.png)  
![新进程注入](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/28.png)

#### "update"
下载并执行update.exe,并卸载老文件本体。  
![更新](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/29.png)

#### "uninstall"
使用SHFileOperationW函数删除文件。本体和全部生成文件以及注册表项被删除。  
删除"C:\Users\User\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup"目录下，随机5为数字命名的exe本体。  
删除%USERNAME%.vbs开机自启动注册表项。  
删除%USERNAME%.vbs的计划任务。  
删除z_%USERNAME%文件夹。  
![卸载](/blog/恶意样本分析/一个EHSTOR木马分析报告/pic/30.png)

## 3.总结