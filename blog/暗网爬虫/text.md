# 暗网爬虫

## 1.前言
（暂略

## 2.tor环境配置
使用机器：CentOS7
安装tor
```
yum install tor
```

编辑配置文件
```
sudo vim /etc/tor/torrc
```

添加如下三行代码
```
Socks5Proxy 127.0.0.1:1080 # 将 shadowsocks 作为 Tor 的前置代理

ExcludeNodes {cn},{hk},{mo},{kp},{ir},{sy},{pk},{cu},{vn} # 表示排除这些国家／地区的节点

StrictNodes 1 # 表示强制执行
```

运行
```
systemctl start tor
```


## 3.Scrapy框架介绍及安装使用

### 3.1Scrapy框架介绍

### 3.2安装
通过python命令进行安装Scrapy
```
pip install scrapy
```

在命令行模式下，切换到项目存放目录， 创建爬虫项目
```
scrapy startproject [项目名]
```

根据提示，通过提供的模板，创建爬虫(注意：爬虫名称，不能跟项目名称一致，否则会报错)
```
scrapy genspider [爬虫名称] [域名]
```

