


# 奋斗吧，骚年！


-----------------------------------------------------


## umask用法

umask+default=666 | 777  
其中666是文件的权限，777是目录的权限  
1. -rw-r--r--. 1 root root 0 Sep 28 20:22 f1         
解析  ：umask+644=666,umask=002  
2. -rw-rw-r--. 1 jie  jie  0 Sep 28 20:23 f2  
3. -rw-rw-r--. 1 jie  jie  0 Sep 28 20:23 f2  
[jie@centos7 /data]$umask  
0002           
如果我想实现1中的权限，可以这莫做  
[jie@centos7 /data]$umask 022  
[jie@centos7 /data]$umask  
0022  
[jie@centos7 /data]$touch f4  
[jie@centos7 /data]$ll  
-rw-r--r--. 1 jie  jie  0 Sep 28 20:44 f4  
目录权限  
  [jie@centos7 /data]$mkdir fgdir  
[jie@centos7 /data]$ll  
drwxr-xr-x. 2 jie  jie  6 Sep 28 20:46 fgdir  
注意 : 如果想下次权限不变需要用nano命令保存，保存后通过输入.生效

如果按上面的说法的话就存在一个很尴尬的问题  
[jie@centos7 /data]$umask 321    
[jie@centos7 /data]$touch f5    
[jie@centos7 /data]$ll    
-rw-r--r--. 1 jie  jie  0 Sep 28 21:01 f5  
666-321=345而结果还是446  
umask  掩码   把最大值权限中有权限的位遮挡住，去掉，没有权限的就不遮挡  
110110110  
011010001     umask  
100100110  

666  
321  
345   

由此可得  
umask值 可以用来保留在创建文件权限     
&ensp;&ensp;&ensp;&ensp;新建FILE权限: 666-umask 如果所得结果某位存在执行（奇数）权限，则将其权限+1            
&ensp;&ensp;&ensp;&ensp;新建DIR权限: 777-umask  
&ensp;&ensp;&ensp;&ensp;非特权用户umask是 002   
&ensp;&ensp;&ensp;&ensp;root的umask 是 022   
&ensp;&ensp;&ensp;&ensp;umask: 查看   
&ensp;&ensp;&ensp;&ensp;umask #: 设定 umask 002   
&ensp;&ensp;&ensp;&ensp;umask –S 模式方式显示   
&ensp;&ensp;&ensp;&ensp;umask –p 输出可被调用   
&ensp;&ensp;&ensp;&ensp;全局设置： /etc/bashrc   
&ensp;&ensp;&ensp;&ensp;用户设置：~/.bashrc 

## 了解SUID（只对二进制程序有用，对脚本没用，作用在目录上没意义 ）
* suid:4 作用于二进制可执行的文件上，功能：当用户执行此文件，会继承此文件所有者的权限 

[root@centos7 ~]#su jie  
[jie@centos7 /root]$cat /etc/shadow  
cat: /etc/shadow: Permission denied  
[jie@centos7 /root]$passwd  
Changing password for user jie.  
Changing password for jie.  
(current) UNIX password:   
New password:  
 [root@centos7 /home]#which passwd  
/usr/bin/passwd
[root@centos7 ~]#ll /usr/bin/passwd  
-rwsr-xr-x. 1 root root 27832 Jun 10  2014 /usr/bin/passwd  
&ensp;&ensp;&ensp;&ensp;权限中的s指的就是SUID权限  
当jie用户执行passwd命令时，jie用户的身份就临时切换成了此文件所有者的身份  
[root@centos7 ~]#which cat  
/usr/bin/cat  
[root@centos7 ~]#ll /usr/bin/cat  
-rwxr-xr-x. 1 root root 54080 Apr 11 12:35 /usr/bin/cat  
[root@centos7 ~]#chmod u+s /usr/bin/cat  
[root@centos7 ~]#ll /usr/bin/cat  
-rwsr-xr-x. 1 root root 54080 Apr 11 12:35 /usr/bin/cat 
 [root@centos7 ~]#su jie  
[jie@centos7 /root]$cat /etc/shadow  
root：
Rokdbse1::0:99999:7:::
bin:*:17632:0:99999:7:::
daemon:*:17632:0:99999:7:::
adm:*:17632:0:99999:7:::
lp:*:17632:0:99999:7:::
[root@centos7 ~]#chmod 4755 /usr/bin/cat  
[root@centos7 ~]#ll /usr/bin/cat  
-rwsr-xr-x. 1 root root 54080 Apr 11 12:35 /usr/bin/cat  
[root@centos7 ~]#chmod 755 /usr/bin/cat  
[root@centos7 ~]#ll /usr/bin/cat  
-rwxr-xr-x. 1 root root 54080 Apr 11 12:35 /usr/bin/cat

## SGID权限
* 1）作用于二进制可执行的文件上，功能：当用户执行此文件，会继承此文件所属组的权限  
2）作用于目录上，功能：当用户在此目录建新文件时，此新文件的所属组继承目录的所属组
关于第二点举例  
[root@centos7 ~]#cd /data  
[root@centos7 /data]#mkdir webdir  
[root@centos7 /data]#ls  
webdir  
[root@centos7 /data]#groupadd webs  
[root@centos7 /data]#useradd li  
[root@centos7 /data]#gpasswd -a jie webs  
Adding user jie to group webs  
[root@centos7 /data]#gpasswd -a li webs  
Adding user li to group webs  
[root@centos7 /data]#groupmems -l -g webs  
jie  li   
[root@centos7 /data]#chgrp webs webdir  
[root@centos7 /data]#ll  
total 0  
drwxr-xr-x. 2 root webs 6 Sep 29 14:35 webdir  
[root@centos7 /data]#chmod 770 webdir  
[root@centos7 /data]#ll  
total 0  
drwxrwx---. 2 root webs 6 Sep 29 14:35 webdir  
[root@centos7 /data]#getfacl webdir  
 #file: webdir  
#owner: root  
#group: webs  
user::rwx  
group::rwx  
other::---  
[root@centos7 /data]#su jie  
[jie@centos7 /data]$cd webdir  
[jie@centos7 /data/webdir]$touch jie1  
[jie@centos7 /data/webdir]$ll -d  
drwxrwx---. 2 root webs 18 Sep 29 14:41 .  
[jie@centos7 /data/webdir]$ll jie1 -d  
-rw-rw-r--. 1 jie jie 0 Sep 29 14:41 jie1  
[jie@centos7 /data/webdir]$touch jie2  
[jie@centos7 /data/webdir]$exit  
exit  
[root@centos7 /data]#su li  
[li@centos7 /data]$cd webdir  
[li@centos7 /data/webdir]$touch li1   
[li@centos7 /data/webdir]$ll  
total 0  
-rw-rw-r--. 1 jie jie 0 Sep 29 14:41 jie1  
-rw-rw-r--. 1 jie jie 0 Sep 29 14:42 jie2  
-rw-rw-r--. 1 li  li  0 Sep 29 14:44 li1  
[li@centos7 /data/webdir]$echo xx>>jie1  
bash: jie1: Permission denied  
[li@centos7 /data/webdir]$echo ll>>li1  
[li@centos7 /data/webdir]$chgrp webs li1  
[li@centos7 /data/webdir]$ll li1 -d  
-rw-rw-r--. 1 li webs 3 Sep 29 14:49 li1  
[li@centos7 /data/webdir]$chgrp webs jie1  
chgrp: changing group of ‘jie1’: Operation not permitted  
[li@centos7 /data/webdir]$ll  
total 4  
-rw-rw-r--. 1 jie jie  0 Sep 29 14:41 jie1  
-rw-rw-r--. 1 jie jie  0 Sep 29 14:42 jie2  
-rw-rw-r--. 1 li  webs 3 Sep 29 14:49 li1  
[li@centos7 /data/webdir]$exit  
exit  
[root@centos7 /data]#su jie  
[jie@centos7 /data]$cd webdir  
[jie@centos7 /data/webdir]$chgrp webs jie*  
[jie@centos7 /data/webdir]$ll  
total 4  
-rw-rw-r--. 1 jie webs 0 Sep 29 14:41 jie1  
-rw-rw-r--. 1 jie webs 0 Sep 29 14:42 jie2  
-rw-rw-r--. 1 li  webs 3 Sep 29 14:49 li1  
[jie@centos7 /data/webdir]$echo fdhjjh >> li1  
[jie@centos7 /data/webdir]$cat li1    
ll  
fdhjjh  
[jie@centos7 /data/webdir]$touch jie3  
[jie@centos7 /data/webdir]$ll  
total 4  
-rw-rw-r--. 1 jie webs  0 Sep 29 14:41 jie1  
-rw-rw-r--. 1 jie webs  0 Sep 29 14:42 jie2  
-rw-rw-r--. 1 jie jie   0 Sep 29 15:01 jie3  
-rw-rw-r--. 1 li  webs 10 Sep 29 15:00 li1  
[jie@centos7 /data/webdir]$exit  
exit    
[root@centos7 /data]#chmod 2770 webdir  
[root@centos7 /data]#ll  
total 0  
drwxrws---. 2 root webs 53 Sep 29 15:01 webdir  
[root@centos7 /data]#cd webdir  
[root@centos7 /data/webdir]#touch root1  
[root@centos7 /data/webdir]#ll  
total 4  
-rw-rw-r--. 1 jie  webs  0 Sep 29 14:41 jie1  
-rw-rw-r--. 1 jie  webs  0 Sep 29 14:42 jie2  
-rw-rw-r--. 1 jie  jie   0 Sep 29 15:01 jie3  
-rw-rw-r--. 1 li   webs 10 Sep 29 15:00 li1  
-rw-r--r--. 1 root webs  0 Sep 29 15:06 root1  
[root@centos7 /data/webdir]#su li  
[li@centos7 /data/webdir]$touch li2  
[li@centos7 /data/webdir]$ll  
total 4  
-rw-rw-r--. 1 jie  webs  0 Sep 29 14:41 jie1  
-rw-rw-r--. 1 jie  webs  0 Sep 29 14:42 jie2  
-rw-rw-r--. 1 jie  jie   0 Sep 29 15:01 jie3  
-rw-rw-r--. 1 li   webs 10 Sep 29 15:00 li1  
-rw-rw-r--. 1 li   webs  0 Sep 29 15:07 li2  
-rw-r--r--. 1 root webs  0 Sep 29 15:06 root1  
&ensp;&ensp;&ensp;&ensp;默认情况下，用户创建文件时，其属组为此用户所属的主组  
一旦某目录被设定了SGID，则对此目录有写权限的用户在此目录中创建的文件 所属的组为此目录的属组  
通常用于创建一个协作目录   
SGID权限与SUID权限大同小异，只是s加在所属组上，用数字“2表示”，SUID权限s加在所有者上用数字“4”表示  
举个例子  
chmod 2755 就是SGID权限，chmod 4755就是SUID权限 ，chmod 6755就是既有SUID权限 又有SGID权限


 ## Sticky位（作用在目录上） 
 * sticky:1 作用于目录上，功能：对目录的文件，只能删除自已的文件

  &ensp;&ensp;&ensp;&ensp;具有写权限的目录通常用户可以删除该目录中的任何文件，无论该文件的权限 或拥有权  
   在目录设置Sticky 位，只有文件的所有者或root可以删除该文件 
   ·sticky 设置在文件上无意义   
 [root@centos7 ~]#ll -d /tmp    
drwxrwxrwt. 23 root root 4096 Sep 29 09:21 /tmp    
权限中的t指的就是 Sticky位  
[root@centos7 ~]#ll /data -d    
drwxrwxrwt. 3 root root 69 Sep 28 21:01 /data    
[root@centos7 ~]#cd /data    
[root@centos7 /data]#touch f1    
[root@centos7 /data]#ll f1    
-rw-r--r--. 1 root root 0 Sep 29 10:29 f1  
[root@centos7 /data]#su jie  
[jie@centos7 /data]$rm f1  
rm: remove write-protected regular empty file ‘f1’? y  
rm: cannot remove ‘f1’: Permission denied  
[jie@centos7 /data]#touch f2  
[jie@centos7 /data]$rm f2  
[jie@centos7 /data]$ls  
[jie@centos7 /data]$f1  
[jie@centos7 /data]$chmod 1777 /data  
[jie@centos7 /data]$ll -d  
drwxrwxrwt. 2 root root 26 Sep 16 13:47 .  
[jie@centos7 /data]$chmod 777 /data  
[jie@centos7 /data]$ll -d  
drwxrwxrwx. 2 root root 26 Sep 16 13:47 .  
 Sticky 权限使用数字“1”表示只可以修改自己文件，别人的文件修改不了
 ## chattr
  &ensp;&ensp;&ensp;&ensp;chattr +i 不能删除，改名，更改  
 举例  
   [root@centos7 /data/webdir]#ll  
total 4  
-rw-rw-r--. 1 jie  webs  0 Sep 29 14:41 jie1  
-rw-rw-r--. 1 jie  webs  0 Sep 29 14:42 jie2  
-rw-rw-r--. 1 jie  jie   0 Sep 29 15:01 jie3  
-rw-rw-r--. 1 li   webs 10 Sep 29 15:00 li1  
-rw-rw-r--. 1 li   webs  0 Sep 29 15:07 li2  
-rw-r--r--. 1 root webs  0 Sep 29 15:06 root1  
[root@centos7 /data/webdir]#echo abs >>jie1  
[root@centos7 /data/webdir]#ll jie1 -d  
-rw-rw-r--. 1 jie webs 4 Sep 29 15:44 jie1    
[root@centos7 /data/webdir]#chattr +i jie1  
[root@centos7 /data/webdir]#ll jie1  
-rw-rw-r--. 1 jie webs 4 Sep 29 15:44 jie1  
[root@centos7 /data/webdir]#lsattr jie1  
----i----------- jie1  
[root@centos7 /data/webdir]#echo djfshfj >> jie1  
-bash: jie1: Permission denied  
[root@centos7 /data/webdir]#rm -rf jie1  
rm: cannot remove ‘jie1’:   Operation not permitted  
[root@centos7 /data/webdir]#cd ..  
[root@centos7 /data]#rm -rf webdir  
rm: cannot remove ‘webdir/jie1’:   Operation not permitted  
[root@centos7 /data]#cd /webdir  
-bash: cd: /webdir: No such file or directory  
[root@centos7 /data]#cd webdir  
[root@centos7 /data/webdir]#ls  
jie1  
[root@centos7 /data/webdir]#
chattr -i jie1  
[root@centos7 /data/webdir]#lsattr jie1  
---------------- jie1  
chattr -i是取消不能删除，改名，更改 

 &ensp;&ensp;&ensp;&ensp;chattr +a 只能追加内容  
 举例  
 [root@centos7 /data/webdir]# mv jie1 jie6  
[root@centos7 /data/webdir]#chattr +a jie6  
[root@centos7 /data/webdir]#lsattr jie6  
-----a---------- jie6  
[root@centos7 /data/webdir]#mv jie6 jie9  
mv: cannot move ‘jie6’ to ‘jie9’: Operation not permitted  
[root@centos7 /data/webdir]#rm jie6  
rm: remove regular file ‘jie6’? y  
rm: cannot remove ‘jie6’:  
 Operation not permitted  
[root@centos7 /data/webdir]#echo 不怂 >> jie6  
[root@centos7 /data/webdir]#cat     jie6    
abs  
不怂  
[root@centos7 /data/webdir]#> jie6  
-bash: jie6: Operation not permitted  
[root@centos7 /data/webdir]#chattr -a jie6  
[root@centos7 /data/webdir]#lsattr jie6  
---------------- jie6  
[root@centos7 /data/webdir]#ls
jie6  
[root@centos7 /data/webdir]#mv jie6  jie9  
[root@centos7 /data/webdir]#ls
jie9  


 lsattr 显示特定属性

## setfacl
仅对jie用户设置权限，其他用户权限不变
[root@centos7 ~]#touch /data/f1  
[root@centos7 ~]#ll /data/f1  
-rw-r--r--. 1 root root 0 Sep 29 17:18 /data/f1  
[root@centos7 ~]#cd /data  
[root@centos7 /data]#setfacl -m u:jie:- f1  
[root@centos7 /data]#ll  
total 0  
-rw-r--r--+ 1 root root 0 Sep 29 17:18 f1
  [root@centos7 /data]#su jie  
[jie@centos7 /data]$cat f1  
cat: f1: Permission denied  
[jie@centos7 /data]$exit  
exit  
[root@centos7 /data]#echo ???? >>f1  
[root@centos7 /data]#su li  
[li@centos7 /data]$cat f1  
????  
由于li用户 是other用户，所以只能读不能写，那麽我们可以通过setfacl -m命令来实现在不影响其他用户的权限下修改li用户  
[li@centos7 /data]$ll f1  
-rw-r--r--+ 1 root root 5 Sep 29 17:33 f1  
[li@centos7 /data]$exit  
exit  
[root@centos7 /data]#setfacl -m u:li:rw f1  
[root@centos7 /data]#ll  
total 4  
-rw-rw-r--+ 1 root root 5 Sep 29 17:33 f1 
事实证明成功了

[root@centos7 ~]#useradd liu  
[root@centos7 ~]#su liu  
[liu@centos7 /root]$cd /data  
[liu@centos7 /data]$ll  
total 4  
-rw-rw-r--+ 1 root root 5 Sep 29 17:33 f1  
[liu@centos7 /data]$cat f1  
????  
[liu@centos7 /data]$nano f1  
会给你一个 [ Error writing f1: Permission denied ] 提示，因为liu用户是other用户只有读权限

如果想看f1文件中各个用户的权限可以通过getfacl这个命令即  
[liu@centos7 /data]$getfacl f1  
#file: f1  
#owner: root  
#group: root  
user::rw-  
user:jie:---  
user:li:rw-  
group::r--  
mask::rw-  
other::r-- 
ACL生效顺序：所有者，自定义用户，自定义组，其他人
如果你想去掉某一个用户，或者组的权限可以用setfacl -x 
[root@centos7 /data]#getfacl f1  
#file: f1    
#owner: wang  
#group: root  
user::rw-  
user:jie:rw-  
user:wang:---  
group::r--  
group:webs:-w-  
group:dbs:r--  
mask::rw-  
other::r--  

[root@centos7 /data]#setfacl -x g:dbs f1  
[root@centos7 /data]#getfacl f1  
#file: f1  
#owner: wang  
#group: root  
user::rw-  
user:jie:rw-  
user:wang:---  
group::r--   
group:webs:-w-  
mask::rw-  
other::r--  

[root@centos7 /data]#setfacl -x u:jie f1  
[root@centos7 /data]#getfacl f1  
#file: f1  
#owner: wang  
#group: root  
user::rw-  
user:wang:---  
group::r--  
group:webs:-w-  
mask::rw-  
other::r--  
如果想将setfacl-m设置的权限全部都清空可以用setfacl -b  
[root@centos7 /data]#ll f1  
-rw-rw-r--+ 1 wang root 35 Sep 30 10:20 f1  
[root@centos7 /data]#setfacl -b f1  
[root@centos7 /data]#ll f1  
-rw-r--r--. 1 wang root 35 Sep 30 10:20 f1  
用setfacl有一个莫大的好处就是不用担心权限恢复问题 
【1】 [root@centos7 /data]#mkdir feel  
[root@centos7 /data]#cp   /etc/fstab feel/f1  
[root@centos7 /data]#cp   /etc/fstab feel/f2  
[root@centos7 /data]#setfacl -R -m u:wang:rwx feel  
[root@centos7 /data]#getfacl feel  
#file: feel  
#owner: root  
#group: root  
user::rwx  
user:wang:rwx  
group::r-x  
mask::rwx  
other::r-x  

[root@centos7 /data]#getfacl feel/f1  
#file: feel/f1  
#owner: root  
#group: root  
user::rw-  
user:wang:rwx  
group::r--  
mask::rwx  
other::r--

【2】 chmod命令的重生之路  
[root@centos7 ~]#chmod -x /usr/bin/chmod  
[root@centos7 ~]#ll /usr/bin/chmod  
-rw-r--r--. 1 root root 58584 Apr 11 12:35 /usr/bin/chmod  
[root@centos7 ~]#ll /data  
total 4  
-rw-r--r--. 1 wang root 35 Sep 30 10:20 f1  
drwxr-xr-x. 2 root root 26 Sep 30 10:55 feel  
[root@centos7 ~]#chmod 744 /data/f1  
-bash: /usr/bin/chmod: Permission denied  
[root@centos7 ~]#chmod +x /usr/bin/chmod  
-bash: /usr/bin/chmod: Permission denied  
[root@centos7 ~]#setfacl -m u:root:rwx /usr/bin/chmod  
[root@centos7 ~]#getfacl /usr/bin/chmod  
getfacl: Removing leading '/' from absolute path names  
#file: usr/bin/chmod  
#owner: root  
#group: root  
user::rw-  
user:root:rwx  
group::r--  
mask::rwx  
other::r--  

[root@centos7 ~]#ll /usr/bin/chmod  
-rw-rwxr--+ 1 root root 58584 Apr 11 12:35 /usr/bin/chmod  
[root@centos7 ~]#chmod +x /usr/bin/chmod  
[root@centos7 ~]#ll /usr/bin/chmod  
-rwxrwxr-x+ 1 root root 58584 Apr 11 12:35 /usr/bin/chmod  
[root@centos7 ~]#chmod 744 /data/f1  
[root@centos7 ~]#ll /data/f1  
-rwxr--r--. 1 wang root 35 Sep 30 10:20 /data/f1

论d: u:中d的作用，其作用是不影响前面文件权限让用setfacl -R -m改完权限之后的新建文件拥有修改后的权限  
[root@centos7 /data]#touch feel/f3  
[root@centos7 /data]#getfacl feel/f3  
#file: feel/f3  
#owner: root  
#group: root  
user::rw-  
group::r--  
other::r--  
[root@centos7 /data]#setfacl -R -m d:u:wang:rwx feel    
[root@centos7 /data]#getfacl feel    
#file: feel  
#owner: root  
#group: root  
user::rwx  
user:jie:rwx  
group::r-x  
mask::rwx  
other::r-x  
default:user::rwx  
default:user:wang:rwx  
default:group::r-x  
default: mask ::rwx  
default:other::r-x  

[root@centos7 /data]#touch feel/f6  
[root@centos7 /data]#getfacl feel/f6  
#file: feel/f6  
#owner: root  
#group: root  
user::rw-  
user:wang:rwx			#effective:rw-  
group::r-x			#effective:r--  
mask::rw-  
other::r--  

[root@centos7 /data]#getfacl   feel/f3  
#file: feel/f3  
#owner: root  
#group: root  
user::rw-  
group::r--  
other::r--

mask作用
[root@centos7 /data]#getfacl f1  
#file: f1  
#owner: wang  
#group: root  
user::rw-  
user:wang:rw-  
group::r--  
mask::rw-  
other::r--  

[root@centos7 /data]#setfacl -m   mask::r f1  
[root@centos7 /data]#getfacl f1  
#file: f1  
#owner: wang  
#group: root  
user::rw-  
user:wang:rw-			#effective:r--  
group::r--  
mask::r--  
other::r--
mask起到限高线的作用，可以进行批量改权限

getfacl file1 | setfacl --set-file=- file2  
  复制file1的acl权限给file2

[root@centos7 /data]#ll f1  
-rw-rw-r--+ 1 wang root 35 Sep 30 10:20 f1  
[root@centos7 /data]#getfacl f1  
#file: f1  
#owner: wang  
#group: root  
user::rw-  
user:wang:rw-  
group::r--  
mask::rw-  
other::r--  
[root@centos7 /data]#touch f2  
[root@centos7 /data]#getfacl f2  
#file: f2  
#owner: root  
#group: root  
user::rw-  
group::r--  
other::r--  

[root@centos7 /data]#ll f2  
-rw-r--r--. 1 root root 0 Sep 30 17:41 f2  
[root@centos7 /data]#getfacl f1 >f1.acl  
[root@centos7 /data]#cat f1.acl  
#file: f1  
#owner: wang  
#group: root  
user::rw-  
user:wang:rw-  
group::r--  
mask::rw-  
other::r--  

[root@centos7 /data]#setfacl --set-file=f1.acl f2  
[root@centos7 /data]#getfacl f2  
#file: f2  
#owner: root  
#group: root  
user::rw-  
user:wang:rw-  
group::r--  
mask::rw-  
other::r--
上面这个方法有点啰嗦，可以这样写进行优化  
[root@centos7 /data]#touch f3  
[root@centos7 /data]#ll f3  
-rw-r--r--. 1 root root 0 Sep 30 17:55 f3  
[root@centos7 /data]#getfacl f1 | setfacl --set-file=- f3    
[root@centos7 /data]#getfacl f3   
#file: f3  
#owner: root  
#group: root  
user::rw-  
user:wang:rw-  
group::r--  
mask::rw-  
other::r--  

1、在/testdir/dir里创建的新文件自动属于webs组，组apps的成员如： tomcat能对这些新文件有读写权限，组dbs的成员如：mysql只能对新文 件有读权限，其它用户（不属于webs,apps,dbs）不能访问这个文件夹   
[root@centos7 ~]#mkdir testdir  
[root@centos7 ~]#cd testdir  
[root@centos7 ~/testdir]#mkdir dir  
[root@centos7 ~/testdir]#cd dir  
[root@centos7 ~/testdir/dir]#groupadd webs  
[root@centos7 ~/testdir/dir]#groupadd apps  
[root@centos7 ~/testdir/dir]#groupadd dbs  
[root@centos7 ~/testdir/dir]#useradd tomcat  
[root@centos7 ~/testdir/dir]#useradd mysql  
[root@centos7 ~/testdir/dir]#gpasswd -a tomcat -g apps  
Adding user tomcat to group apps  
[root@centos7 ~/testdir/dir]#usermod -aG dbs mysql  
准备工作完成  
[root@centos7 ~]#chmod -R g+s testdir/dir  
[root@centos7 ~]#chgrp webs testdir/dir  
[root@centos7 ~]#ll -d testdir/dir  
drwxr-sr-x. 2 root webs 6 Oct  1 10:57 testdir/dir  
[root@centos7 ~]#touch testdir/dir/f1  
[root@centos7 ~]#ll -d testdir/dir/f1  
-rw-r--r--. 1 root webs 0 Oct  1 11:27 testdir/dir/f1  
将testdir/dir的所属组换成webs，同时，组权限g+s可以让新建的文件继承主组的权限，从而实现/testdir/dir里创建的新文件自动属于webs组  
[root@centos7 ~]#setfacl -R -m g:apps:rw testdir/dir  
[root@centos7 ~]#setfacl -R -m g:dbs:r testdir/dir  
[root@centos7 ~]#getfacl testdir/dir  
#file: testdir/dir  
#owner: root  
#group: webs  
#flags: -s-  
user::rwx  
group::r-x  
group:apps:rw-  
group:dbs:r--  
mask::rwx  
other::r-x  
组apps的成员如： tomcat能对这些新文件有读写权限，组dbs的成员如：mysql只能对新文 件有读权限  
[root@centos7 ~]#setfacl -R -m o:- testdir/dir    
[root@centos7 ~]#getfacl testdir/dir    
#file: testdir/dir  
#owner: root  
#group: webs  
#flags: -s-  
user::rw-  
group::r--  
group:apps:rw-  
group:dbs:r--  
mask::rw-  
other::---  
其它用户（不属于webs,apps,dbs）不能访问这个文件夹  也就是说其他用户没有访问权限  
这样这道题就完成了

2、备份/testdir/dir里所有文件的ACL权限到/root/acl.txt中，清除 /testdir/dir中所有ACL权限，最后还原ACL权限    
[root@centos7 ~]#getfacl -R testdir/dir > /root/acl.txt    
[root@centos7 ~]#getfacl testdir/dir    
#file: testdir/dir    
#owner: root  
#group: webs  
#flags: -s-  
user::rwx  
group::r-x  
group:apps:rw-  
group:dbs:r--  
mask::rwx  
other::r-x  

[root@centos7 ~]#setfacl -R -b testdir/dir  
[root@centos7 ~]#getfacl testdir/dir  
#file: testdir/dir   
#owner: root   
#group: webs  
#flags: -s-   
user::rwx  
group::r-x  
other::r-x  

[root@centos7 ~]#setfacl -R --set-file=/root/acl.txt testdir/dir  
[root@centos7 ~]#getfacl testdir/dir  
#file: testdir/dir  
#owner: root  
#group: webs  
#flags: -s-  
user::rw-  
group::r--  
group:apps:rw-  
group:dbs:r--  
mask::rw-  
other::r--  

## head命令了解一下

head [OPTION]... [FILE]...   
-c #: 指定获取前#字节  
 -n #: 指定获取前#行   
 -#： 指定行数   

## tail命令  

tail [OPTION]... [FILE]...     
-c #: 指定获取后#字节     
-n #: 指定获取后#行     
-#：同上    
-f: 跟踪显示文件fd新追加的内容,常用日志监控   
&ensp;&ensp;&ensp;&ensp;相当于 --follow=descriptor   
-F: 跟踪文件名，相当于--follow=name --retry   
tail -f和tail -F，tailf的区别    
tail -f追踪的是描述字符，描述字符被删了就停止追踪    
tail -F追踪的是文件名，也就是说，文件被删之后，在重建一个文
件名相同的文件它会继续追加    
tailf 类似tail –f，当文件不增长时并不访问文件 

 ## cut命令 
cut [OPTION]... [FILE]...  
 -d DELIMITER: 指明分隔符，默认tab   
 -f FILEDS:   
 #: 第#个字段  
  #,#[,#]：离散的多个字段，例如1,3,6   
  #-#：连续的多个字段, 例如1-6   
  混合使用：1-3,7   
  -c 按字符切割   
  举例子 cut -d: -f 1,3 /etc/passwd 

 centos 7版本 从ifconfig命令中取Ip地址的方法  
    1) 第一种方法 
   [root@centos7 ~]#ifconfig ens33 | head -2 | tail -1 | tr -s ' ' % | cut -d% -f3  
   2）第一种方法的改进版  
  [root@centos7 ~]#ifconfig ens33 | head -2 | tail -1 | tr -s ' ' | cut -d' ' -f3  
   3）一种脑洞大开的办法  
   [root@centos7 ~]#ifconfig ens33 | head -2 | tail -1 | tr -s ' ' | cut -dt -f2 | cut -dn -f1  
   4) 利用grep命令  
  [root@centos7 ~]#ifconfig ens33 | grep -o "[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}"|head -1  
  5) 第四种方法的改进版 
[root@centos7 ~]#ifconfig ens33|grep -o "\<\([0-9]\{1,3\}\.\)\{3\}[0-9]\{1,3\}\>"|head -1

   centos 6版本 从ifconfig命令中取Ip地址有两种办法  
   1） [root@Centos6 ~]#ifconfig eth0 | head -2 |tail -1 | tr -dc "[0-9. ]" | tr -s " " | cut -d" " -f2  
2）[root@Centos6 ~]#ifconfig eth0 | head -2 |tail -1 | cut -d: -f2 | tr -dc '[0-9].'

--output-delimiter=STRING指定输出分隔符    
[root@centos7 ~]#cat /etc/passwd | cut -d: -f 1,3  
root:0  
bin:1  
[root@centos7 ~]#cat /etc/passwd | cut -d: -f 1,3--output-delimiter=%  
root%0  
bin%1

## paste横向合并
 
 paste 合并两个文件同行号的列到一行  
paste [OPTION]... [FILE]...   
&ensp;&ensp;&ensp;&ensp;-d 分隔符：指定分隔符，默认用TAB   
&ensp;&ensp;&ensp;&ensp;-s : 所有行合成一行显示   
例子：  
[root@centos7 ~]#echo {0..2} | tr " " "\n" >f1  
[root@centos7 ~]#echo {a..c} | tr " " "\n" >f2  
[root@centos7 ~]#paste f1 f2  
0	a  
1	b  
2	c  
[root@centos7 ~]#paste -d":" f1 f2  
0:a   
1:b  
2:c
[root@centos7 ~]#paste -s f1 f2  
0	1	2	  
a	b	c	
[root@centos7 ~]#paste -s -d":" f1 f2  
0:1:2  
a: b :c

## wc命令(统计个数)
 
 计数单词总数、行总数、字节总数和字符总数  
  可以对文件或STDIN中的数据运行 wc   story.txt 39 237 1901 story.txt
  &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;       行数 字数 字节数 
   常用选项     
 * -l 只计数行数   
  * -w 只计数单词总数   
  * -c 只计数字节总数   
  * -m 只计数字符总数  
  * -L 显示文件中最长行的长度 

 [root@centos7 ~]#paste f1 f2 >> f3  
[root@centos7 ~]#cat f3  
0	a  
1	b  
2	c  
3	d  
4	e  
5	f  
6	g  
7	h  
8	i  
9	j  
10	k  
[root@centos7 ~]#wc f3    
11 22 45 f3  
[root@centos7 ~]#wc -l f3  
11 f3  
[root@centos7 ~]#wc -w f3  
22 f3  
[root@centos7 ~]#wc -c f3  
45 f3  
[root@centos7 ~]#wc -m f3  
45 f3  
[root@centos7 ~]#wc -L f3  
9 f3

看别人连你用的IP地址，有三种方法  
1）[root@centos7 ~]#ss -nt | tr -s ' ' ':' | cut -d: -f6 | tr -dc '[0-9].\n'  
2）[root@centos7 ~]#ss -nt|tr -dc '[0-9]. \n:'|tr -s ' ' ':'|cut -d: -f6  
3) [root@centos7 ~]#ss -nt|cut -d: -f2|tr -s ' '|cut -d' ' -f2|tr -dc '[0-9].\n' 

查/etc/profile一共有多少个单词  
root@centos7 ~]#cat /etc/profile|tr -sc '[a-zA-Z]' '\n'|wc -l  

##  sort命令

把整理过的文本显示在STDOUT，不改变原始文件   
sort [options] file(s)  
 常用选项      
-r 执行反方向（由上至下）整理  
-R 随机排序  
-n 执行按数字大小整理  
-f 选项忽略（fold）字符串中的字符大小写  
-u 选项（独特，unique）删除输出中的重复行  
-t  c 选项使用c做为字段界定符   
-k  X 选项按照使用c字符分隔的X列来整理能够使用多次

63个数随机抽取  
[root@centos7 ~]#seq 63 | sort -R | head -1  
 
 ## uniq命令

 uniq命令：从输入中删除前后相接的重复的行   
 uniq [OPTION]... [FILE]...  
 -c: 显示每行重复出现的次数  
  -d: 仅显示重复过的行   
 -u: 仅显示不曾重复的行 注：连续且完全相同方为重复   
 常和sort 命令一起配合使用：   
  sort  userlist.txt  |  uniq -c
  [root@centos7 ~]#cat f1  
aba  
aba  
sdj  
dff  
 
dff  
ahd  
aba   
789  
789  
[root@centos7 ~]#uniq f1  
aba  
sdj  
dff  
 
dff  
ahd  
aba   
789  
[root@centos7 ~]#uniq -c f1  
      2 aba  
      1 sdj  
      1 dff  
      1    
      1 dff  
      1 ahd  
      1 aba   
      2 789  
[root@centos7 ~]#uniq -d f1  
aba  
789  
[root@centos7 ~]#uniq -u f1  
sdj  
dff  
 
dff  
ahd  
 aba 

——*面试题*—— 连接次数最多的前十个IP解决方法  
cut -d " " -f1 /var/log/httpd/access_log |sort |uniq -c|sort -nr|head 

[root@centos7 ~]#cat f1  
bbb  
dddd  
cce  
aaa  
[root@centos7 ~]#cat f2  
aaaa  
ccc  
ddd  
cce  
[root@centos7 ~]#cat f1 f2  
bbb   
dddd   
cce  
aaa  
aaaa  
ccc  
ddd   
cce  
对f1、f2取交集  
第一种：[root@centos7 ~]#cat f1 f2|sort|uniq -d  
cce   
第二种：[root@centos7 ~]#grep -f f1 f2   
对f1、f2取不同  
[root@centos7 ~]#cat f1 f2|sort|uniq -u  
aaa  
aaaa  
bbb  
ccc  
ddd  
dddd  
对f1、f2取并集  
[root@centos7 ~]#cat f1 f2|sort -u  
aaa  
aaaa  
bbb  
ccc  
cce  
ddd  
dddd  

## diff命令

diff 命令的输出被保存在一种叫做“补丁”的文件中  
 使用 -u 选项来输出“统一的（unified）”diff格式文件，最适用于补丁文件 
[root@centos7 ~]#cat>f1    
aaa    
bbb    
ccc  
[root@centos7 ~]#cat > f2  
ddd  
ccc  
bbb  
eee  
[root@centos7 ~]#diff f1 f2  
1,2c1  
< aaa  
< bbb  
_ _   
> ddd  
3a3  
> bbb  
[root@centos7 ~]#diff -u f1 f2  
--- f1	2018-10-02   16:55:23.236562203 +0800  
+++ f2	2018-10-02   16:56:59.547567674 +0800  
@@ -1,3 +1,3 @@  
-aaa  
-bbb  
+ddd  
 ccc  
+bbb  

## Linux文本处理三剑客

grep：文本过滤(模式：pattern)工具  
 &ensp;&ensp;&ensp;&ensp;grep, egrep, fgrep（不支持正则表达式搜索） 
sed：stream editor，文本编辑工具   
awk：Linux上的实现gawk，文本报告生成器

## grep
grep: Global search REgular expression and Print out the line  
 作用：文本搜索工具，根据用户指定的“模式”对目标文本逐行进行匹配检 查；打印匹配到的行  
  模式：由正则表达式字符及文本字符所编写的过滤条件  
  grep [OPTIONS] PATTERN [FILE...]  
   grep root /etc/passwd  
    grep "$USER"  /etc/passwd  
     grep '$USER'  /etc/passwd  
      grep `whoami`  /etc/passwd  
### grep命令选项
 *   --color=auto: 对匹配到的文本着色显示
  * -v: 显示不被pattern匹配到的行 
   * -i: 忽略字符大小写 
   * -n：显示匹配的行号 
   * -c: 统计匹配的行数 
   * -o: 仅显示匹配到的字符串 
   * -q: 静默模式，不输出任何信息 
   * -A #: after, 后#行 
  * -B #: before, 前#行 
   * -C #：context, 前后各#行
  * -e：实现多个选项间的逻辑or关系 grep –e ‘cat ’  -e ‘dog’  file 
   * -w：匹配整个单词 
   * -E：使用ERE 
    * -F：相当于fgrep，不支持正则表达式 
    * -f file: 根据模式文件处理

包含硬盘分区用的利用率，挑出来排大小 
[root@centos7 ~]#df|grep /dev/sda|tr -s ' ' '%'|cut -d% -f5|sort -nr 

## 正则表达式  (*重点)

REGEXP： Regular Expressions，由一类特殊字符及文本字符所编写的模式， 其中有些字符（元字符）不表示字符字面意义，而表示控制或通配的功能   
* 程序支持：grep,sed,awk,vim, less,nginx,varnish等   
* 分两类： 基本正则表达式：BRE 扩展正则表达式：ERE grep -E, egrep   
 * 正则表达式引擎： 采用不同算法，检查处理正则表达式的软件模块 PCRE（Perl Compatible Regular Expressions）   
* 元字符分类：字符匹配、匹配次数、位置锚定、分组   
* man  7 regex  

基本正则表达式元字符   

 字符匹配:  
 * . 匹配任意单个字符  
 * []  匹配指定范围内的任意单个字符，示例:  
  [wang]   [0-9]    [a-z]   [a-zA-Z]   
 * [^] 匹配指定范围外的任意单个字符   
 * [:alnum:] 字母和数字   
  * [:alpha:] 代表任何英文大小写字符，亦即 A-Z, a-z   
 * [:lower:] 小写字母   
 * [:upper:] 大写字母   
 * [:blank:] 空白字符（空格和制表符）   
 * [:space:] 水平和垂直的空白字符（比[:blank:]包含的范围广）   
 * [:cntrl:] 不可打印的控制字符（退格、删除、警铃...）   
 * [:digit:] 十进制数字   
 * [:xdigit:]十六进制数字   
 * [:graph:] 可打印的非空白字符   
 * [:print:] 可打印字符   
 * [:punct:] 标点符号  

正则表达式  

匹配次数：用在要指定次数的字符后面，用于指定前面的字符要出现的次数  
 * *匹配前面的字符任意次，包括0次 贪婪模式：尽可能长的匹配   
* .* 任意长度的任意字符   
 * \? 匹配其前面的字符0或1次   
 * \+ 匹配其前面的字符至少1次  
  * \{n\} 匹配前面的字符n次   
 * \{m,n\} 匹配前面的字符至少m次，至多n次   
  * \{,n\} 匹配前面的字符至多n次   
 * \{n,\} 匹配前面的字符至少n次  

正则表达式  

位置锚定：定位出现的位置  
 * ^ 行首锚定，用于模式的最左侧  
  * $ 行尾锚定，用于模式的最右侧  
  * ^PATTERN$  用于模式匹配整行   
   * ^$  空行   
  * ^[[:space:]]*$  空白行   
  * \< 或 \b 词首锚定，用于单词模式的左侧   
  * \> 或 \b 词尾锚定，用于单词模式的右侧   
  * \<PATTERN\> 匹配整个单词   

正则表达式  

* 分组：\(\) 将一个或多个字符捆绑在一起，当作一个整体处理，如：\(root\)\+   
* 分组括号中的模式匹配到的内容会被正则表达式引擎记录于内部的变量中，这些 变量的命名方式为: \1, \2, \3, ...   
* \1 表示从左侧起第一个左括号以及与之匹配右括号之间的模式所匹配到的字符   
示例：   
\(string1\+\(string2\)*\)   
\1 ：string1\+\(string2\)*  
 \2 ：string2   
* 后向引用：引用前面的分组括号中的模式所匹配字符，而非模式本身   
* 或者：\|   
示例：a\|b: a或b  C\|cat: C或cat   \(C\|c\)at:Cat或cat  

[root@centos7 ~]#cat >f1  
abc xyz abc xyz  
[root@centos7 ~]#grep "\(abc\).*\(xyz\).*\2" f1  
abc xyz abc xyz  
[root@centos7 ~]#grep "\(abc\).*\(xyz\).*\1" f1  
abc xyz abc xyz  

[root@centos7 ~]#cat f1  
123sfs123ddd123  
[root@centos7 ~]#grep "\(123\).*\1" f1  
123sfs123ddd123

[root@centos7 ~]#grep "\([0-9]\)\{3\}.*\1" f1  
123sfs123ddd123  
234hdhaha244  

以a或者是b开头  
* [root@centos7 ~]#grep "^\(a\|b\)" /etc/passwd  
bin: x :1:1:bin:/bin:/sbin/nologin  
adm: x :3:4:adm:/var/adm:/sbin/nologin  
abrt: x :173:173::/etc/abrt:/sbin/nologin  
avahi: x :70:70:Avahi mDNS/DNS-SD  Stack:/var/run/avahi-daemon:/sbin/nologin
* [root@centos7 ~]#grep -E "^(a|b)" /etc/passwd  
bin: x :1:1:bin:/bin:/sbin/nologin  
adm: x :3:4:adm:/var/adm:/sbin/nologin  
abrt: x :173:173::/etc/abrt:/sbin/nologin  
avahi: x :70:70:Avahi mDNS/DNS-SD   Stack:/var/run/avahi-daemon:/sbin/nologin



























































































