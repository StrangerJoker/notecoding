# Linux学习





## 文件、目录与磁盘



### 文件权限和目录配置

#### 用户和用户组

```shell
drwxrwxr-x. 2 wade wade   28 Jan  7 10:06 centOS-learn
```

第一个wade是用户，第二个wade是用户组。

LInux用户和和用户组记录文件：

- `/etc/passwd`：记录用户相关信息
- `/etc/shadow`：记录个人密码
- `/etc/group`：记录用户组信息

#### 文件属性

`ls -al`输出的结果是`[1][2][3][4][5][6][7]`

- [1]：文件权限

  `[d/-][rwx][rwx][rwx]`：d表示目录，-表示文件。后续的rwx分别表示 **文件拥有者、文件所属用户组、其他人**的**可读、可写、可执行**的权利

文件如果是`.fileName`，则其是隐藏文件

##### 目录的权限

- r：读取目录结构列表的权限
- w：改动该目录结构列表的权限
- x：进入目录的权限

#### 更改文件属性和权限

##### chgrp 修改所属用户组

change group

##### chown 修改文件拥有者

change owner

命令格式：`chown [-R] 账号名称 文件或者目录`、`chown [-R] 账号名称:用户组名称 文件或者目录`

##### chmod 修改文件的权限

（1）数字类型修改文件权限：`chmod [-R] xyz 文件或者目录`,`rwx`对应着`xyz`，即x=4 y=2 z=1

（2）符号类型修改文件权限：`chmod [-R] [u/g/o/a][+/-/=][r/w/x] 文件或者目录`。u表示当前用户user，g表示group，o表示others，a表示all。`+`表示添加，`-`表示移除，`=`表示设置。

#### Linux文件种类和扩展名

**任何设备在Linux上都是文件**

- 常规文件：
- 目录：
- 连接文件（link）：
- 设备与设备文件：
- 数据接口文件
- 数据传送文件
- 