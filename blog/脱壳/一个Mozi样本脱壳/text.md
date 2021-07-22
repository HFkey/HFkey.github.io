# 一个Mozi样本脱壳

    IOC  
    MD5:59ce0baba11893f90527fc951ac69912  
    SHA1:5857a7dd621c4c3ebb0b5a3bec915d409f70d39f
    SHA256:4293c1d8574dc87c58360d6bac3daa182f64f7785c9d41da5e0741d2b1817fc7  

该文件属于Mozi僵尸网络家族一个样本，mips下使用的upx加壳。且将p_info结构中的p_file_size和p_blocksize值擦除为了0。让一般脱壳脚本失效。  
## 查壳
加壳UPX3.95，但"upx -d"脱壳失败。  
![脱壳失败](/blog/脱壳/一个Mozi样本脱壳/pic/1脱壳失败.png)

根据报错信息"upx: Mozi.m: CantUnpackException:p_info corrupted",查询到p_info结构体
```c
struct p_info{//12-byte
    unsigned p_progid;
    unsigned p_filesize;
    unsigned p_blocksize;
} p_info;
```
对应位置，p_filesize和p_blocksize被清0。  
![p_info清0](/blog/脱壳/一个Mozi样本脱壳/pic/3p_info被清除.png)

p_filesize和p_blocksize应该是分别表示加壳前的文件大小和占用空间大小。不过实际看下来，两个值相同，均是加壳前文件的大小，这里被清空，但在文件结尾处，仍有p_filesize的值。  
![查找p_filesize](/blog/脱壳/一个Mozi样本脱壳/pic/4p_filesize.png)

修改并保存  
![修复](/blog/脱壳/一个Mozi样本脱壳/pic/5修复.png)

再脱壳，成功。  
![脱壳成功](/blog/脱壳/一个Mozi样本脱壳/pic/6脱壳成功.png)  
![脱壳成功2](/blog/脱壳/一个Mozi样本脱壳/pic/7脱壳成功2.png)

样本具体行为分析，以后有机会再说= =