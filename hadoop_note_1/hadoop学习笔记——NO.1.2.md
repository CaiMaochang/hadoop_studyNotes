# hadoop学习笔记——NO.1.2
----------
## 安装JDK
### 1.上传jdk-7u45-linux-x64.tar.gz到Linux上
### 2.解压jdk到/usr/local目录
```
tar -zxvf jdk-7u45-linux-x64.tar.gz -C /usr/local/
```
### 3.设置环境变量，在/etc/profile文件最后追加相关内容
```
vi /etc/profile
```
```
export JAVA_HOME=/usr/local/jdk1.7.0_45
export PATH=$PATH:$JAVA_HOME/bin
```
### 4.刷新环境变量
```
source /etc/profile
```
### 5.测试java命令是否可用
```
java -version
```
## 安装Tomcat
### 1.上传apache-tomcat-7.0.68.tar.gz到Linux上
### 2.解压tomcat
```
tar -zxvf apache-tomcat-7.0.68.tar.gz -C /usr/local/
```
### 3.启动tomcat
```
/usr/local/apache-tomcat-7.0.68/bin/startup.sh
```
### 4.查看tomcat进程是否启动
```
jps
```
### 5.查看tomcat进程端口
```
netstat -anpt | grep 2465
```
### 6.通过浏览器访问tomcat
```
http://192.168.0.101:8080/
```
## 在CentOS7上安装MySQL
### 1.下载mysql的repo源
```
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
```
### 2.安装mysql-community-release-el7-5.noarch.rpm包
```
sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
```
安装这个包后，会获得两个mysql的yum repo源：/etc/yum.repos.d/mysql-community.repo，/etc/yum.repos.d/mysql-community-source.repo。

### 3.安装mysql
```
sudo yum install mysql-server
```
根据提示安装就可以了,不过安装完成后没有密码,需要重置密码

### 4.重置mysql密码
```
mysql -u root
```
登录时有可能报这样的错：ERROR 2002 (HY000): Can‘t connect to local MySQL server through socket ‘/var/lib/mysql/mysql.sock‘ (2)，原因是/var/lib/mysql的访问权限问题。下面的命令把/var/lib/mysql的拥有者改为当前用户：
```
sudo chown -R root:root /var/lib/mysql
```
### 5.重启mysql服务
```
$ service mysqld restart
```
### 6.接下来登录重置密码：
```
mysql -u root  #直接回车进入mysql控制台
mysql > use mysql;
mysql > update user set password=password('root') where user='root';
mysql > exit;
```
