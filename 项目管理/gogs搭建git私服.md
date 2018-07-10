#### 文件提供
链接: https://pan.baidu.com/s/1GvdOLAeFrw5i14_rD3U5Aw 密码: tm2r

### 安装依赖
- 解压安装包
    - `tar xf Percona-Server-5.5.45-37.4-r042e02b-el6-x86_64-bundle.tar`
    - Percona是改进版的Mysql数据库
- 安装如下依赖包：
```
    yum install Percona-Server-client-55-5.5.45-rel37.4.el6.x86_64.rpm
    yum install Percona-Server-server-55-5.5.45-rel37.4.el6.x86_64.rpm
    yum install Percona-Server-shared-55-5.5.45-rel37.4.el6.x86_64.rpm
```
- centos7数据库安装错误解决：
    - `rpm -qa | grep mariadb`:找出mariadb的文件
    - `rpm -e --nodeps mariadb-libs-5.5.56-2.el7.x86_64`: 卸载

- 设置数据库开机自启动：`chkconfig mysql on`
- 启动数据库服务：`service mysql start`
- 初始化数据库：`/user/bin/mysql_secure_installation`
- 登陆mysql：`mysql -u root -p`
- 显示数据库：`show databases;`
- 创建gogs库：`mysql -uroot -p < /home/git/gogs/scripts/mysql.sql`
- 为gogs库创建用户：`grant all on gogs.* to 'gogs'@'%' identified by 'xdliubin';`  
    - 最后的字符串是密码
- 切换数据库：`use gogs`
- 显示数据库内容：`show tables`


### 安装gogs
- 添加默认用户git：`useradd git`
    - 修改用户密码: `passwd git`
- 切换用户：`su - git`
- 解压gogs安装包：`tar xf gogs0.11.4_amd64.tar.gz`,并`cd gogs`
- 创建文件夹: `mkdir custom/conf -p`,
    - 传入配置文件：`cp ../app.ini custom/conf/`
- 编辑app.ini文件
```ini
    APP_NAME = xd:lb
    RUN_USER = git
    RUN_MODE = dev
    [server]
    HTTP_ADDR = 0.0.0.0
    HTTP_PORT = 3000

    [database]
    DB_TYPE = mysql
    HOST = 127.0.0.1:3306
    NAME = gogs
    USER = gogs
    PASSWD = gogs

    [security]
    INSTALL_LOCK = false
    SECRET_KEY = xdliubin@126.com&liubinguet@gmail.com.(python).GIT:gogs
```

### 启动gogs服务
- 复制启动项：`cp /home/git/gogs/scripts/init/centos/gogs /etc/init.d`
- 添加执行权限：`cd /etc/init.d` `chmod +x gogs`
- 添加开机自启动：`chkconfig gogs on`
- 启动gogs服务： `service gogs start`   / `systemctl start gogs`
    - `ss -tanl`: 检查服务是否已经启动
        - 在git用户下：在gogs目录下新建log文件夹
- 获取web地址：ip a   或者 ifconfig
- *注：上面操作要求在root权限下*


