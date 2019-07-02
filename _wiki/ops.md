---
layout: wiki
title: OPS
categories: OPS
description: 运维场景中遇到的一些问题集合
keywords: OPS
---


> 运维过程中会遇到很多问题，这些问题可能都是随手完成的小事情，但是下次来了又忘记了，脑子里面记那么多肯定很累，好记性不如烂笔头，以下都是我运维中遇到的一些场景问题，还有很多涉及到隐私没有公开的。

**目录**

* TOC
{:toc}


### Salt-minion启动失败

- python3 salt-minion install
```shell
sudo yum remove salt-minion -y
sudo yum install https://repo.saltstack.com/py3/redhat/salt-py3-repo-2018.3-1.el7.noarch.rpm 
sudo yum install salt-minion -y
cat > /etc/salt/minion <<EOF
master : salt-tx.xxxxx.com
id : $(hostname)
EOF
systemctl restart salt-minion
```

```
yum install salt-minion -y
cat >> /etc/salt/minion <<EOF
master : salt.xxxxxx.com
id : $(hostname)
EOF
/etc/init.d/salt-minion start
原因：之前指向了别的机器Master,删除重新启动
解决:rm -f /etc/salt/pki/minion/minion_master.pub;rm -f /var/run/salt-minion.pid
```


### 更改默认Yum源为阿里源
```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo;yum clean all

curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```

### VCenter机器调整磁盘空间大小
  3.1 基于单块磁盘单分区磁盘扩展
    
    
  在单磁盘单分区进行扩展磁盘容量需要重启机器，实例重启会导致业务中断，请谨慎操作。
  ```
  
1、操作之前需要备份此实例磁盘/执行磁盘快照。
2、新增磁盘扩展后，使用fdisk -l 查看磁盘容量是否已经增加
3、确保此磁盘未被挂载，若被挂在使用umount /dev/sde1;df -h
4、由于我们/dev/sde只有一个主分区/dev/sde1，首先删除此分区，新将分区
      fdisk /dev/sde  p查看分区，d删除分区 n创建分区 p主分区 1 默认 wq保存
5、检查文件系统，并重新设置文件系统大小
      e2fsck -f /dev/vdb1 # 检查文件系统
      resize2fs /dev/vdb1 # 变更文件系统大小  
 
6、 挂载磁盘，确认数据是否正常
         mount /dev/sde1 /resize/ ； df -h ; tree /data1 
  ```
  
  
  
  
  3.2  基于LVM磁盘容量进行扩展
  
```
pvcreate /dev/sdb #挂载一块sdb的磁盘格式化，格式转化为PV
vgdisplay   ##记住Name的名字，加入VG的时候要用到。
vgextend VolGroup /dev/sdb #磁盘加入VG组
lvextend -L +10G /dev/mapper/VolGroup-lv_root  #拓展10G到LVM_root
e2fsck -f /dev/mapper/VolGroup-lv_root  #检查root卷
resize2fs /dev/mapper/VolGroup-lv_root  #不需重启，resize下就可以了
xfs_growfs /dev/centos/root    #centos 7下实用这个
df -h  #查看root卷已经被扩展了

 Vcenter添加硬盘后，重新扫描SCSI总线来添加设备

  1. [root@centos7 ~]# echo "- - -">/sys/class/scsi_host/host0/scan
  2. [root@centos7 ~]# echo "- - -">/sys/class/scsi_host/host1/scan
  3. [root@centos7 ~]# echo "- - -">/sys/class/scsi_host/host2/scan



```
	

### 修改Linux时区为CST
```
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

### history优化（操作时间+用户IP）
```
UserIP=$(who -u am i | cut -d"("  -f 2 | sed -e "s/[()]//g")
export HISTTIMEFORMAT="[%F %T] [`whoami`] [${UserIP}] " 
```


### 查找 \ 目录下占用哪个目录占用磁盘空间大
```
for i in /*; do echo $i; find $i | wc -l; done
```


### Linux系统中新增数据硬盘操作步骤
```
1、停服务，关闭机器，新增/dev/sdb 100G
2、mv /data1 /bak   ##记住，最好机器做一个备份
3、mkfs.ext4 /dev/sdb    #格式化新加的磁盘
4、mkdir -p /data1; rsync -a /bak/* /data1/  ##将原先的数据全部同步到新磁盘上
5、将新挂载的磁盘设置为开机自动挂载， /etc/fstab文件
```


### 山石防火墙禁止外网IP访问WAN口
```
SG-6000# con
host-blacklist ip from 52.70.53.108 to 52.70.53.108 vrouter trust-vr enable
```
### MySQL从库1062主键冲突错误解决
```
 mysql>slave stop; 
 mysql> set GLOBAL SQL_SLAVE_SKIP_COUNTER=1;
 mysql> slave start;
vim /etc/my.cnf   #配置文件 [mysqld]添加，重启从库
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
#slave-skip-errors=1062
```


### GitLab安装完成后没有CSS样式，报500错误问题
```
cd  /home/git/gitlab
bundle exec rake assets:precompile RAILS_ENV=production RAILS_RELATIVE_URL_ROOT=/gitlab
跑完后会在  /home/git/gitlab/public 目录下生成 assets 个目录 
这个目录里面就是 css 跟js文件 ，一开始是没有的。
```


### GitLab迁移后访问500错误问题
```
#db_key不一致导致的
cd /home/git/gitlab ; cp config/secure.yml config/secure-bak-`date +%F`  #先备份
rsync -e ssh -avz /home/git/gitlab/config/secure.yml root@gitlabtest.xxxx.net.cn:/home/git/gitlab/config/secure.yml 

```
### GitLab迁移后访问代码库显示“repository is empty”问题
```
#MySQL数据库导入，权限正常，代码库能看到，当点击进去就显示为空
sudo -u git -H bundle exec rake gitlab:check  RAILS_ENV=production  #检查状态
sudo -u git -H bundle exec rake cache:clear  RAILS_ENV=production  #清除下缓存就OK
sudo -u git -H bundle exec rake gitlab:update_commit_count RAILS_ENV=production    #gitlab版本库字节显示为0问题

```
### Linux free -m 参数
```
[root@Intranet_Ops_YangHongFeiTest01 ~]#free -m
             total       used       free     shared    buffers     cached
Mem:          2005       1733        272         18        186        466
-/+ buffers/cache:       1080        924
Swap:          991          0        991
```

- Tocal1（表示总内存容量）
- Used1 (系统已分配的内存) userd1 = buffers1+caced1+used2
- free1 (系统未分配的内存)
- shared1（共享内存）
- buffers1 (系统已分配内存中，但未使用的buffers数量)
- cached (系统已分配内存中，但未使用的cached数量)
- used2(系统已分配的内存中，系统使用的总量)
- free2(未使用系统已分配的内存，和系统未分配内存的之和) free2 = buffers1+cached1+free1
- 



### Aws ec2用户认证问题


```
15.1.1. ssh permission denied失败
修改/etc/ssh/sshd_config    #PermitRootLogin yes  取消注释
Command：  sed -i  's/#PermitRootLogin yes/PermitRootLogin yes/g'  /etc/ssh/sshd_config

15.1.2. 允许密码登陆设置
修改/etc/ssh/sshd_config   #PasswordAuthentication yes  改为Yes
Command：  sed -i  's/PasswordAuthentication no/PasswordAuthentication yes/g'  /etc/ssh/sshd_config  

service sshd restart

```


### Python3 Pip3 install 
```
在 CentOS 7 中安装 Python 依赖
$ yum -y groupinstall development 
$ yum -y install zlib-devel 
$ yum install -y python3-devel openssl-devel libxslt-devel libxml2-devel libcurl-devel 

运行下面的命令来安装 Python 3.6：

$ wget https://www.python.org/ftp/python/3.6.3/Python-3.6.3.tar.xz
$ xz -d  Python-3.6.3.tar.xz
$ tar xvf Python-3.6.3.tar
$ cd Python-3.6.3/
$ ./configure
$ make && make install

# 查看安装
$ python3 -V

pip3安装

$ curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
$ python3 get-pip.py

# 查看安装
$ pip3 -V

```

### Unix System Command（flushcache\MacBrew\GetIP\Iteam2\PID_Time）
####  刷新MacDNS缓存
```
sudo dscacheutil -flushcache
```
####  Mac系统安装Brew
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
#### 获取公网IP地址
```
curl members.3322.org/dyndns/getip
curl whatismyip.akamai.com
```
#### Mac系统Iteam2配置Lrzsz
```
expression:\*\*B0100
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-send-zmodem.sh

Regular expression:\*\*B00000000000000
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-recv-zmodem.sh
```
#### grep过滤空行、注释
```
grep -Ev "^$|^[#;]" php.ini
```
#### MySQL查看UUID
```
show variables like '%server_uuid%';
```
#### Linux 释放内存
```
Linux释放内存的命令：
sync   #释放之前sync下，防止数据丢失
echo 1 > /proc/sys/vm/drop_caches

drop_caches的值可以是0-3之间的数字，代表不同的含义：
0：不释放（系统默认值）
1：释放页缓存
2：释放dentries和inodes
3：释放所有缓存

释放完内存后改回去让系统重新自动分配内存。
echo 0 >/proc/sys/vm/drop_caches

free -m #看内存是否已经释放掉了。
```
#### Linux查看进程开始时间和运行时间
```
ps -p 24936 -o pid,cmd,stime,uid,gid   
ps -p 24936 -o pid,cmd,etime,uid,gid
```
#### MySQL主从配置信息
```
grant replication slave on *.* to 'replication'@'%' identified by 'nfH56BmsQ';  
UPDATE user SET password=PASSWORD('nfH56BmsQ') WHERE user='replication';
CHANGE MASTER TO MASTER_HOST='10.0.3.xx',MASTER_USER='replication',MASTER_PASSWORD='xxxxxxxxx',MASTER_LOG_FILE='mysql-bin.005473',MASTER_LOG_POS=189,m
```

#### Linux批量修改多个文件内容
```
sed -i "s/domain.key/doamin.com.key/g" `grep domain.key -rl ./*`
``` 
#### Linux 终端登陆显示信息
```
cat /etc/motd
```

#### Linux VIM 粘贴的时候保留格式
```
: set paste
```

### Nginx配置支持JS的gzip压缩
- 在nginx.conf 中 http字段添加
- application/javascript 表示支持JS的gzip
```
   gzip on;
   gzip_min_length  1k;
   gzip_buffers     4 16k;
   gzip_http_version 1.0;
   gzip_comp_level 2;
   gzip_types       text/plain application/x-javascript text/css application/xml application/javascript;
   gzip_vary on;
```
- 查看是否压缩成功
- ChormF12---Network---Name右键---ResponseHeaders---Content-Encoding


### nginx开启目录浏览美化
```
$ yum install nginx -y
- 这里配置到vhost里面即可
       location ~/shinezoneit {
            autoindex on;
            autoindex_localtime on;
            autoindex_exact_size off;
   	        charset utf-8,gbk;
  	        add_after_body /autoindex.html;
            auth_basic "ShinezoneIT Authorized";
            auth_basic_user_file /etc/nginx/yanghongfei.conf;
       }
       
       
       
       


```
### SSH免密登陆
```shell

ssh-keygen -t rsa -C "k8s"
ssh-copy-id -i id_rsa.pub root@10.10.10.132

```




