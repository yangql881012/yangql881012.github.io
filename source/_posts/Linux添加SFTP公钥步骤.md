---
title: Linux添加SFTP公钥步骤
date: 2017/01/01
tags: 免密钥
categories: Linux
---
1.首先需要在目录(/home/yangql)创建.ssh文件夹

```
[yangql@hadoop01 ~]$ mkdir .ssh
```

2.在客户端生成公钥和私钥
```
[yangql@hadoop01 ~]$ cd .ssh
[yangql@hadoop01 .ssh]$ ssh-keygen -t rsa
```
执行创建密钥对命令(一路回车)，直至完成公钥与私钥生成。
<!-- more -->
3.把.ssh目录下的公钥文件：/home/yangql/.ssh/id_rsa.pub文件传输到服务器上
```
[yangql@hadoop01 .ssh]$ scp /home/yangql/id_rsa.pub yangql@hadoop02
```
注: hadoop01（对应IP192.168.1.231）
    hadoop02（对应IP192.168.1.232）

4、检查服务器（hadoop02）权限 /home/yangql/ 必须是755

5.公钥拷贝到authorized_keys(第一次添加时将公钥重命名为authorized_keys)
```
[yangql@hadoop01 .ssh]$ cp id_rsa.pub authorized_keys
```
如果已经存在authorized_keys文件，执行命令：
```
[yangql@hadoop01 .ssh]$ cat id_rsa.pub >> /home/dxpt/.ssh/authorized_keys
```

6.公钥文件的权限必须是644
```
[yangql@hadoop01 .ssh]$  chmod 644 authorized_keys
```
7.客户端ssh验证(如果不需要输入密码则公钥设置成功,只有第一次连接需要输入YES确认：)
```
[yangql@hadoop02 home]$ sftp yangql@hadoop02
Connecting to hadoop02...
sftp>

```
8.验证通过后，则可通过免密传输文件

注意事项：
- 权限：如果配置完对等信任公钥，仍提示输入密码或者访问拒绝，则需要查看服务器的目录权限是否正确，家目录权限755，.ssh目录权限是755,authorized_keys文件权限是644
- 备份：authorized_keys不能出现空格等不是公钥的信息，否则公钥文件就会失效，每次附加新公钥时，养成变更前备份的好习惯
