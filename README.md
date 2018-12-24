# rsync
调用rsync进行服务器数据同步-无密码方式

以centos6.1为例



【服务端】

1，安装rsync  

yum  -y install rsync

2,创建配置文件 rsync.conf

mkdir /test/rsync
cd /test/rsync

touch rsync.conf
vi  rsync.conf

增加下列内容
uid = 0
gid = 0
use chroot = no
strict modes = false
hosts allow = *
port = 8002
pid file=/test/rsync/rsyncd.pid
lock file=/test/rsync/rsync.lock
log file=/test/rsync/rsync.log

[task1]                         
path=/test/host/task1

[task2]                         
path=/test/host/task2

3，关闭防火墙或者开放指定端口

关闭防火墙 

service iptables stop  

chkconfig iptables off

开放端口 

vi /etc/sysconfig/iptables   

添加下列内容
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8002 -j ACCEPT

4，创建执行脚本并后台运行 rsync

cd /test/rsync
touch rsync_server.sh
vi rsync_server.sh
增加下列内容
/usr/bin/rsync --daemon --config=/test/rsync/rsync.conf

增加权限
chmod 755 rsync_server.sh

开始执行

/test/rsync/rsync_server.sh
 
5，为防止卡死之类，添加定时任务crontab 每日重启

30 6  * * * /usr/bin/killall -9  rsync
31 6  * * * /usr/bin/rsync --daemon --config=/test/rsync/rsync.conf

【客户端】

1，安装rsync  

yum  -y install rsync

2，拷贝文件

mkdir /test/rsync
cp /usr/bin/rsync /test/rsync/rsync1
cp /usr/bin/rsync /test/rsync/rsync2

3，创建并执行脚本
cd /test/rsync
touch   rsync_client.sh
vi  rsync_client.sh

增加下列内容
/usr/bin/killall -9 rsync1
/usr/bin/killall -9 rsync2
/test/rsync/rsync1 -a --port=8002  10.0.72.221::task1 /test/host/task1 >> /test/rsync/task1.log
/test/rsync/rsync2 -a --port=8002  10.0.72.221::task2 /test/host/task2 >> /test/rsync/task2.log

增加权限
chmod 755 rsync_client.sh

执行 

/test/rsync/rsync_client.sh

4,加入定时任务crontab 

*/5 * * * * /test/rsync/rsync_client.sh
