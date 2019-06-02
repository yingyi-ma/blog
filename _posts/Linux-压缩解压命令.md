---
title: Linux 压缩解压命令
date: 2019-05-15 16:18:51
tags: Linux
keywords:
description:
---
<script type="text/javascript" src="/js/src/bai.js"></script>

## tar.gz

### 压缩
```
tar -zcvf webfile.tar.gz webfile
```

### 解压
#### 使用tar命令进行解压

```
tar -zxvf java.tar.gz 
```

#### 解压到指定的文件夹 

```
tar -zxvf java.tar.gz -C /usr/java
```
#### 参数说明
x 解压缩 
z 具有gzip属性 
v 屏显信息 
f 后面接文件 老的版本f 后面要直接跟文件名 现在的版本好像很好有这个限制了。
C 指定文件夹


## gz
### 解压 gzip 命令
```
 gzip -b java.gz
```

### 也可使用zcat 命令，然后将标准输出 保存文件
```
zcat java.gz > java.java
```

## zip

### 压缩
#**压缩目录**data
```
zip -r data.zip data
```
#压缩**文件夹**和**文件**abc.zip
```
zip -r abc123.zip abc 123.txt
```

### 解压
**解压到指定目录**
```
unzip data.zip -d datadir
```
**直接解压**
```
unzip data.zip
```
**同时解压多个**
```
unzip data\*.zip
```
**查看zip里面的内容**
```
unzip -v data.zip
```
**验证zip是否完整**
```
unzip -t data.zip
```
**所有文件解压到第一级目录**
```
unzip -j data.zip
```

### 主要参数
-c：将解压缩的结果
-l：显示压缩文件内所包含的文件
-p：与-c参数类似，会将解压缩的结果显示到屏幕上，但不会执行任何的转换
-t：检查压缩文件是否正确
-u：与-f参数类似，但是除了更新现有的文件外，也会将压缩文件中的其它文件解压缩到目录中
-v：执行是时显示详细的信息
-z：仅显示压缩文件的备注文字
-a：对文本文件进行必要的字符转换
-b：不要对文本文件进行字符转换
-C：压缩文件中的文件名称区分大小写
-j：不处理压缩文件中原有的目录路径
-L：将压缩文件中的全部文件名改为小写
-M：将输出结果送到more程序处理
-n：解压缩时不要覆盖原有的文件
-o：不必先询问用户，unzip执行后覆盖原有文件
-P：使用zip的密码选项
-q：执行时不显示任何信息
-s：将文件名中的空白字符转换为底线字符
-V：保留VMS的文件版本信息
-X：解压缩时同时回存文件原来的UID/GID


### 参考
[https://www.cnblogs.com/wangshouchang/p/7748527.html](https://www.cnblogs.com/wangshouchang/p/7748527.html)
[https://www.jb51.net/LINUXjishu/105916.html](https://www.jb51.net/LINUXjishu/105916.html)
