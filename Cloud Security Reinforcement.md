一、防火墙
```
# 开启防火墙
systemctl start firewalld

# 关闭防火墙【临时】
systemctl stop firewalld

# 关闭防火墙【永久】
systemctl disable firewalld

# 查看防火墙状态
systemctl status firewalld 或者 firewall-cmd --state

# 开放指定端口
firewall-cmd --zone=public --add-port=80/tcp --permanent

# 关闭指定端口
firewall-cmd --zone=public --remove-port=80/tcp --permanent

# 重新载入
firewall-cmd --reload

# 查看开放的端口
firewall-cmd --zone=public --list-ports

# 查看所有规则
firewall-cmd --list-all

# 开通指定地址端口
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="x.x.x.x" port protocol="tcp" port="8080" accept"

# 禁用指定地址端口
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="x.x.x.x" port protocol="tcp" port="8080" reject"
```

二、用户与用户组
```
# 创建用户
useradd user1  //创建用户user1
useradd –e 12/30/2009 user2  //创建user2,指定有效期2009-12-30到期。用户的缺省UID从500向后顺序增加，500以下作为系统保留账号，可以指定UID
useradd –u 600 user3

# 使用 passwd 命令为新建用户设置密码
passwd user1  //注意：没有设置密码的用户不能使用

# 命令 usermod 修改用户账户
usermod –l u1 user1  //将用户 user1的登录名改为  u1
usermod –g users user1  //将用户 user1 加入到 users组中
usermod –d /users/us1 user1  //将用户 user1 目录改为/users/us1

# 使用命令 userdel 删除用户账户
userdel user2  //删除用户user2
userdel –r user3  //删除用户 user3，同时删除他的工作目录

# 查看用户信息
finger user4  //可以查看用户的主目录、启动shell、用户名、地址、电话等信息  
id user4  //id命令查看一个用户的UID和GID, 查看user4的id  

# groupadd命令创建用户组
groupadd –g 888 users  //创建一个组users，其GID为888

# 命令 gpasswd为组添加用户，只有root和组管理员能够改变组的成员
gpasswd –a user1 users  //把 user1加入users组
gpasswd –d user1 users  //把 user1退出users组  

# 命令groupmod修改组
groupmod –n user users  //修改组名user为users

# groupdel删除组  
groupdel users   //删除组users
```

三、限制账号
```
# 只允许test组用户su到root
vi /etc/pam.d/su
添加auth required pam_wheel.so group=test。

# 禁止root用户直接登录
创建普通权限账号并配置密码,防止无法远程登录;
vi /etc/ssh/sshd_config
修改配置文件将PermitRootLogin的值改成no
service sshd restart
```

四、检查特殊账号，禁用或删除
```
查看空口令和root权限账号，确认是否存在异常账号：
# 查看空口令账号
awk -F: '($2=="")' /etc/shadow

# 查看UID为零的账号
使用命令 awk -F: '($3==0)' /etc/passwd

# 加固空口令账号
passwd <用户名> 为空口令账号设定密码
确认UID为零的账号只有root账号

# 删除不必要的账号
userdel <用户名> 

# 锁定不必要的账号
passwd -l <用户名> 

# 解锁必要的账号
passwd -u <用户名> 
```

五、SSH服务加固
```
vi /etc/ssh/sshd_config

# 不允许root账号直接登录系统
设置 PermitRootLogin 的值为 no

# 修改SSH使用的协议版本
设置 Protocol 的版本为 2

# 修改允许密码错误次数（默认6次）
设置 MaxAuthTries 的值为 3

配置文件修改完成后，重启sshd服务生效
```

六、日志记录
```
Linux系统默认启用以下类型日志：
1、系统日志（默认）/var/log/messages
2、cron日志（默认）/var/log/cron
3、安全日志（默认）/var/log/secure

# 通过脚本代码实现记录所有用户的登录操作日志
vim /etc/profile
 history
 USER=`whoami`
 USER_IP=`who -u am i 2>/dev/null| awk '{print $NF}'|sed -e 's/[()]//g'`
 if [ "$USER_IP" = "" ]; then
 USER_IP=`hostname`
 fi
 if [ ! -d /var/log/history ]; then
 mkdir /var/log/history
 chmod 777 /var/log/history
 fi
 if [ ! -d /var/log/history/${LOGNAME} ]; then
 mkdir /var/log/history/${LOGNAME}
 chmod 300 /var/log/history/${LOGNAME}
 fi
 export HISTSIZE=4096
 DT=`date +"%Y%m%d_%H:%M:%S"`
 export HISTFILE="/var/log/history/${LOGNAME}/${USER}@${USER_IP}_$DT"
 chmod 600 /var/log/history/${LOGNAME}/*history* 2>/dev/null

# 加载配置生效
source /etc/profile

# 日志存放位置
/var/log/history

每次用户退出后都会产生以用户名、登录IP、时间的日志文件，包含此用户本次的所有操作
```



