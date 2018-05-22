## liunx基础命令汇总整理 ##

**本文记录最常用的 linux 命令，会存在一些命令选项理解有偏差的情况**。
***
### 系统基本信息查询命令 ###

- 查看内存大小： `free -h`  或者  `cat /proc/meminfo`
- 查看分区大小： `lsblk`  或者  `cat /proc/partitions`  或者  `df -h`
- 查看shell类型： `echo $SHELL`  或者  `cat /etc/shells`
- 查看内核版本： `uname -r`  或者  `cat etc/centos-release`
- 获取主机名： `hostname`
- 修改主机名： `hostname newname`
- 查看主板（硬件）时间：`clock`
- 查看系统（内核）时间：`date`
- 设置时间：`date 030719502018.30`  >   格式：月日时分年.秒
- 以字符串的方式显示时间信息：`date -d now`  或者  `date -d "+1 day"`
- 设置时间同步：`clock -w` (硬件时间=系统时间)  以及 `clock -s` (软件时间=硬件时间)
- 查看日历命令：`cal`
- ctrl+alt+f2: 进入到字符界面  ctrl+alt+f6：切换会图形界面
- 查看文件状态：`stat file` 主要信息（创建时间；内容修改时间；元数据修改时间）
- mail -s "标题" 收件人   编写邮件内容以. 来退出  输入mail命令可以查看收到的邮件

**注：在proc文件夹中存放的是硬件的一些设置信息 ； etc文件夹中存放的是软件的配置信息**
***
### 网络相关命令 ###

- 获取ip地址： `ifconfig`  或者 `ip -a`
- 修改网卡名称： `vim /boot/grub2.cfg`, 在第一个linux16后添加 `net.ifnames=0`
- 与服务器时间临时同步：`ntpdate IP` (及时同步时间命令)
- 与服务器时间永久同步：`vim /etc/ntp.conf`  添加  `service IP iburst`后执行：
- ntpd开关
	- CentOS6:
		- `service ntpd start`   开启同步服务
		- `service ntpd restart` 重启服务
		- `chkconfig ntpd on`    开机自启动
	- CentOS7:   
		- `systemctl start ntpd`    开启同步服务
		- `systemctl restart ntpd`  重启同步服务
		- `systemctl enable ntpd`   开机自启动
- 防火墙开关
	- CentOS6:    
		- `service iptables status` 查看防火墙状态   
		- `service iptables stop`   关闭防火墙 
		- `service iptables start`  打开防火墙        
		- `chkconfig iptables on/off`  永久开启关闭防火墙：   
	- CentOS7:    
		- `systemctl stop firewalld`     关闭防火墙        
		- `systemctl start firewalld`    打开防火墙    
		- `systemctl enable/disable firewalld`   开机启动/不启动防火墙   


***
### 配置相关 ###

- 提示符格式：`echo $PS1`
- 修改提示符格式：在 `~/.bash_profile`文件中添加 `PS1="\[\e[1;34m\][\u@\h \W]\\$\[\e[0m\]"`
- 临时别名：`alias name="commands" ` (只能在当前shell中使用，退出时便消失)  
- 永久定义别名：将命令存储在家目录的`.bashrc`文件中可以一直保持。
- 临时取消别名：`unalias`（只取消当前shell中的别名作用）   
- 跳过别名执行原始命令： `command s` 或者 `\s` 或者 `“s”` 或者 `‘s’` 
- 文件夹颜色配置文件： `/etc/DIR_COLOR`

***
### 帮助相关 ###
- 内部命令：集成在shell中，开机时会加载到进程中，运行效率高
- 外部命令：独立的文件，在运行中使用后会存储在hash文件中从而提高其运行速度，但是效率相对比较低
- 命令功能简介：`whatis cmd`
- 内部命令列表：`enable`
- 判定命令类型：`type cmd`
- 内部命令查看帮助：`help cmd`     
- 外部命令查看帮助：`man [章节号] cmd` 或者 `cmd --help `
- 查看外部命令路径：`which cmd`   `whereis cmd`
- 存储外部命令的搜索路径：`hash`    （可以提高搜索效率，不用每次都按PATH路径搜索）
- linux软件发布的帮助文档在`/usr/share/doc`中


***
### 软件安装 ###

##### rpm #####
- rpm方式安装screen：`rpm -ivh path/to/screen.rpm` 
    - 软件使用： (确保两个用户都登录到同一个服务器)
		1. 求助人发起请求：`screen -S help`
		2. 援助人查看请求：`screen -ls`
		3. 援助人接受请求：`screen -x help`
		4. 撤销帮助请求 ：`exit`          
		5. 临时退出帮助 ：`ctrl+a+d` 再使用`screen -x help` 加入帮助
- 软件安装管理
- 源码包文件命名规则： 名称-主版本.次版本.发布版本.压缩格式     
- rpm包文件命名规则：  名称-主版本.次版本.发布版本.编译版本.硬件平台.rpm 
- 拆包：主包、开发子包、工具子包、库文件子包  
- 包的组成文件：rpm包内文件、rpm元数据、安装卸载脚本
- rpm包的公共数据库：`/var/lib/rpm`（本机中rpm包的安装信息）
- rpm包管理命令：  `rpm [选项] /path/to/rpm`
   - `-ivh`： 安装选项
   - `--replacepkgs`：覆盖包安装（解决程序被破坏时的问题）
   - `--replacefile`：覆盖文件安装（解决两个包有相同文件禁止安装的问题）
   - `-q`           ：包查询命令
   - `-qpi --scripts`：查看脚本文件
   - `-qf`          ：查看文件来自那个包
   - `-Uvh`         ：升级或者安装
   - `-Fvh`         ：升级
   - `--oldpackage` ：降级
   - `--force`      ：强制安装
   - `-e`           ：包卸载
   - `-V`           ：包校验选项（元数据有变化都可以看出包是否被破坏）
   - `-K`           ：检查下载到的安装包是否被破坏（根据签名来判断，签名文件一般会在CD的根目录下）  
   - `--import key` ：在`-K`选项之前导入才可以验证安装包完整性
- 预览rpm包中的文件：`rpm2cpio 包文件 | cpio -itv`
- 释放rpm包中的文件：`rpm2cpio 包文件 | cpio -id "*.conf"`

##### yum #####
- yum：rpm包管理器的前端工具（解决依赖性问题）
- yum仓库：*.rpm文件、 rpm包的元数据（repodata文件夹）
- yum配置：
    1. 首先备份 `/etc/yum.repo.d`文件夹中的所有文件
    2. 新建`base.repo`文件（必须以repo为后缀，光盘文件默认使用base为名）
    3. 编辑文件内容如下：
>	
		[base]
		name=base   (ps:名字随便起,标示而已)
		baseurl=file:///misc/cd [见注释]
		gpgcheck=1
		gpgkey=/path/to/keyfile
		enabled=0（禁用该yum源）
- baseurl注释：
    1. 开启autofs工具：`systemctl start autofs`
    2. 设置开机自启动： `systemctl enable autofs`
    3. 执行下面的命令完成自动挂载 `cd /misc && cd /cd` 
    4. 路径名以repodata文件的父目录为准       
- yum执行的命令历史：`yum history`
- 查看对应num的安装细节：`yum history info num`
- 撤销yum安装num的全部文件：`yum history undo num`（可以用来完全卸载安装的文件）
- 重新执行num的安装：`yum history redo num`
- 可以自动去安装依赖：`yum install name`
- 列出所有可以使用的包名称：`yum list `
- 查看指定的包文件的信息：`yum list name` （可以查看是否安装以及向导）
- 清除yum元数据的缓存：`yum clean all`（在更换了同名源的情况下使用）        
- 查看仓库列表：`yum repolist`（本地没有缓存数据的时候会自动下载更新元数据库）        
- 根据name搜索相关的包文件：`yum search name`
- 重新安装如软件：`yum reinstall name`
- 卸载软件：`yum remove name`
- yum仓库创建：`creatrepo dir`
- yum安装软件错误：一般是配置文件中的路径错误或者元数据的缓存文件没有清空
- 查看二进制程序的依赖库文件：      ldd /path/to/binary_file
- 查看当前系统正在使用的库文件：    ldconfig -v

#### CentOS 源码编译安装软件 ####
- 安装步骤
	1. 下载源文件   `wget IP`
	2. 解压缩源码   `tar xvf 包名 -C /path/to/file`
	3. 切换到解压缩路径中，看`README`和`INSTALL`文件了解软件的功能以及安装对应的选项卡
	4. 安装基本的编译依赖项 `yum groupinstall "Development tools"` 等
	5. 开始编译 `./configure [optional]`
	6. `make -j4 && make install`
	7. 根据INSTALL中的说明指示开启软件
	8. 常用软件可以将路径加入到PATH中:
>
		echo 'PATH=/apps/name/bin:$PATH' > /etc/profile.d/nameAddPath.sh   
		source /etc/profile.d/nameAddPath.sh    
- 卸载步骤：
	1. 退出软件的服务（`ss -ntl`可以查看httpd的运行状态，根据INSTALL中的指令关闭软件）
	2. 删除安装时指定的文件夹  
	3. 删除`/etc/profile.d/nameAddPath.sh`文件                                    


***                               
### 文件目录管理 ###

- 新建空文件：`touch file`(不会覆盖文件)； `> file`(会覆盖旧文件内容)  
- 新建文件夹：`mkdir dir`
- 显示当前路径：`pwd`  ;  `pwd -P`：显示真实的物理路径（针对于链接的情况
- 提取文件名： `basename path`
- 提取路径名： `dirname path`
- 目录跳转： `cd [~user]`(转到user的家目录) ； `cd -`：回到上一次的目录（记录在环境变量OLDPWD变量)
- 文件复制：`cp file0 file1` 默认情况会丢失元数据信息，常用 `-av` 选项
  - `cp -p file0 file1`：保留元数据信息    
  - `cp -v file0 file1`：显示复制过程的详细信息   
  - `cp -a file0 file1`：保留全部的信息    
  - `cp -r file0 file1`：可以用来复制文件夹
  - `cp -v file0 file1`：显示复制过程的详细信息 
- 文件移动：`mv /old/path/file0 /new/path/file1` (包含了改名的功能)
- 目录文件查看：`ls [dir]` ; dir 不指定时默认是当前目录
  - `ls -i [dir]`：显示文件的节点号（同一分区的文件节点号不同，分区的节点标号是有限资源）    
  - `ls -ld [dir]`: 显示文件夹本身的信息
  - `ll --time=atime file` : 显示文件的详细信息（权限）以及元数据
- 硬链接：`ln f1 f11` （硬链接不能跨分区）
  - **注：两个文件属性完全一致（是同一个文件），不支持文件夹操作，删除f1不影响f11文件的访问**
- 软链接：`ln -s f1 f11`（软连接可以跨分区）  
  - **注：不是同一个文件，支持文件夹操作，删除f1影响f11对文件的访问**
  - **注：f1要写绝对路径 或者相对于f11文件的相对路径**
- 查看历史命令记录：`history`  存储在 `~/.bash_history`中（非实时更新 在终端退出时才更新）    
   - `！num` ： 执行标号为num的命令   
   - `！char`： 执行最近的包含char字符的命令
   - `!? str`： 执行最近的包含str的历史命令
   - `!$` ： 前一个命令的最后一个参数
- 查看文件类型： `file` (通过文件头信息（魔数）来确定类型)
  - `file -f`： 将一个文档中列出的所有文档名的文件类型列出  
  - `file -L`： 显示软链接指向的文件的类型   
- 标准输出以及标准错误的默认设备是当前终端窗口，标准输入的默认设备是键盘输入
- I/O重定向
  - `> file`： 把标准输出重定向到文件，会将文件内容覆盖    
  - `2> file`：把标准错误重定向到文件     
  - `&> file`：把标准输出以及标准错误都重定向到文件
  - `set -C` ：禁止覆盖文件内容，换成追加模式
  - `set +C` ：允许覆盖文件内容
  - `>| file`：强制覆盖文件内容，无视`set -C`
  - `>> file`：在原有内容的基础上追加
- 多行重定向： `<<结束词` (在独立一行中出现结束词的情况下才会完成输入)
- 管道：`cmd1 | cmd2` 将命令1的标准输出发送给命令2的标准输入，要求cmd1有标准输出
- 标准错误洗白：`2>&1`表示将标准错误转成标准输出，一般用法：`cmd >/dev/null 2>&1`
- 同时重定向：`cmd1 |& cmd2`表示将标准输出以及标准错误都作为cmd2的标准输入
- 重定向到多处： `cmd |tee log` 将标准输出重定向到log文件以及终端 
- ls /proc/$$/fd 可以显示终端对应的编号信息

***
### 属性权限 ###
- 完全切换用户身份：`su - username` 退出：`exit`
- 查看用户的id信息： `id username` （用户编号、组编号、以及主组和辅助组的列表）
- 进程权限：进程权限取决于进程的运行者的身份，跟进程的权限没有关系
- 每个用户必须属于一个组且只有一个主组，一个用户可以属于零个或者多个辅助组
- 打印组列表信息：`groups UID`
- 管理user的组信息：`groupmems user`
- 新建用户：`useradd name`，新建用户时会在/home中创建家目录，可在/etc/passwd 文件中查看
	- `useradd -u ID name`： 新建用户的时候指定对应的ID号码， 
	- `useradd -r name`：将用户设置为系统用户（不会创建对应的家目录） 
    - `useradd -G groupname name`：指定name用户的辅助组  
    - `useradd -g grpupname name`：指定name用户的主组
    - `useradd -N name`：在新建用户时不新建同名组
- 系统在创建用户时的默认设定：
	- 默认配置信息：`/etc/default/useradd`文件中    
	- 邮箱：`/var/spool/mail`目录下    
	- 家目录中的默认文件： `/etc/skel`（模板文件夹）
    - 默认账号信息：`/etc/login.defs`
- 修改用户密码：`passwd username` (终端输入密码)，可在`/etc/shadow`文件中查看
- 新建组：`groupadd name`
- 可在`/etc/gpasswd`中查看用户信息，对应的口令可在文件`/etc/gshadow`文件中查看
- 在生产环境中：为了便于管理，会首先创建一个组再创建一个用户并加入组中
	1. `groupadd -u ID gname` ：新建编号为ID的组 
	2. `useradd -u ID -g gname -r name` : 创建ID用户加入到ID组并设置为系统用户)
- 修改用户信息：`usermod`
	-   `usermod -L`：可以锁定用户，禁止登陆
	-   `usermod -U`: 选项可以将用户解锁  
	-   `usermod -G`:选项重新设置辅助组 
	-   `usermod -a`:选项可保证原来的辅助组不变
- 删除已有用户：`userdel username`
	- 注：默认删除保留家目录文件，并且权限属于该用户的ID，当新建用户使用了该ID号权限就会转移到新建用户 
	- 注：`userdel -r username`：将用户信息以及家目录全部删除 
- 临时切换主组：`newgrp gname` , 在用户退出后无效，并且只能在主组和辅助组之间切换
- 文件所有者修改：`chown u+r g+w o+x file`
- 文件所属组修改：`chgrg`
- 同时修改所属：`chown username.groupname file` 可以同时修改file的所有者以及所属组  
	- `-R`选项可以修该目录下的所有文件以及文件夹的所有者以及所属组
- 文件权限修改： `chmod who opt perfile file`    
	- **注：opt： +（添加权限） -（取消权限） =（重新设置权限）  `chmod u+r，g-w，o=x file`**
	- **注：文件权限对象(who)：u（所有者） g（所属组） o（其他**    
	- **注：文件访问权限(profile)：r(可读性) w(可写性) x(可执行) X (仅文件夹可执行权限)**
- `s`权限： 用户执行程序时（用户必须有对程序执行的权限），临时切换到程序所有者的权限
	- `suid`权限只有在二进制程序中才有效 （一般也必须包括可执行权限）
	- `sgid`作用在程序中与suid相同
	- `sgid`权限作用在目录上，目录中新建文件的所属组自动变为目录所在组
- `t`权限(粘滞位)： 针对文件夹的管理权限，该目录下用户只可以删除自己新建的文件
- `s、t`权限的数字法跟普通权限不混合计算，放在首位 
- 访问控制列表： 对特定的用户(超过who的范围)在该文件的权限做限制 （要求系统支持ACL功能）
	- `setfacl -m who:name:profile file`：可以针对单独的用户或者单独的组进行权限管理
	- `getfacl file` ： 查看该文件的全部管理权限
	- `setfacl -x who:name file` ： 删除file文件中who的权限
	- `setfacl -b file`： 删除file的所有acl权限               

***
### 文档处理 ###

#### 文本处理工具 ####
- 列拼接并打印输出：`cat -n file1 file2`(将文件内容拼接输出并显示行号)
- 行拼接并打印输出：`paste file1 file2` 按行合并两个文件内容并打印输出     
	- `paste -d% file1 file2`：指定分隔符的形式 “%” 为分隔符
	- `paste -s file`：file文件中的所有行合并成一行输出
- 输出文件的前几行：`head -n num` ，选项`-c#`：获取指定的前#个字节 
- 输出文件的后几行：`tail -n num` ，选项 `-f`：跟踪命令，可以时时打印输出文件增加的内容
- 按列取出文件内容：`cut`（要求有明确的分隔符）  
	- 例子：`cut -d: -f1,3-5 file` (取出file中按：分割的第1列，3-5列)
	- 例子：`cut -c34-43 file`     (取出file文件中第34到43列的所有字符）
- 文件内容按行反序输出：`tac file` (从最后一行到第一行)
- 文件内容按列逆向输出：`rev file` (从行尾到行首)
- 文本数据统计：`wc -lLc file`
- 文件内容排序：`sort -rnu file` (文件内容逆序、按数值、去重排序，不修改文件)
- 重复行报告处理：`uniq -cdu file`(连续出现并且完全相同定义为重复)（重复数、显重、显不重）


#### 通配符以及正则 ####
通配符针对文件名的匹配操作

- 括号扩展： `{}`表示括号内的内容全部展开; `[]` 表示括号中的任意一个
- 文件通配符：`*` 匹配零个或者多个字符;  `？`匹配任何单个字符； `[^]`匹配指定范围外的任意单个字符
- 查看预定义的通配符 `man 7 glob`

正则针对文件内容的匹配操作

* 正则表达式：基本正则表达式 与 扩展正则表达式相差`\`符号
* 匹配任意单个字符： `.`（与通配符有区别）
* 字符字数匹配：
	* `*`：表示前面的字符任意次，`.*`表示任意长度的字符串
	* `\?`：表示前面的字符出现0次或者1次
	* `\+`：表示前面的字符出现至少一次
	* `\{m\}`：前面的字符出现m次
	* `\{m,n\}`：前面的字符出现至少m次，最多n次
	* `\{m,\}`：前面的字符至少出现m次
* 字符位置锚定：
	* `^`：行首锚定
	* `$`：行尾锚定
	* `^PATTERNS`：行首出现正则匹配的锚定
	* `\<或者\b`：词首锚定
	* `\>或者\b`：词尾锚定
	* `\<PATTERN>\`：单词锚定
* 分组：`\(keyword\)`: 表示将一个或者多个字符捆绑在一起，当做一个整体进行处理
* 后向引用：`\1` 从左起第一个`\`(到右侧第一个与之匹配的)`\`符号之间模式匹配到的*字符*（字符不是模式）
* 或者关系：`a\|b` a或者b

### grep sed awk  ###
grep命令：文本搜索过滤工具，根据用户指定的"模式"对目标文本进行匹配检测，将匹配的行打印输出（检索工具）
 
- 命令格式：`grep [选项...] 模式 [文件名...]`
- 常用选项：         
	- `-e PATTERN1 -e PATTERN2` ： 表示或者的关系
	- `-w` ：按单词匹配 （数字、字母、“_”不是分割符，“-”是分割符）
	- `-n` ：显示匹配的行号码
	- `-v` ：显示没有匹配到的行
	- `-o` ：只显示匹配到的行
	- `-i` ：忽略匹配到的字符大小写
	- `-ABC`：显示匹配到的前后几行
	- `-E` ： 扩展的正则表达式（= egrap）

sed命令：一种在线行编辑器，每次处理一行并送入标准输出中（搜索匹配处理工具）
 
- 命令格式：`sed [选项...] “脚本” 输入文件...`              
- 常用选项：
	- `-r` ： 支持扩展的正则表达式
	- `-e 脚本1 -e 脚本2`：指定多脚本运行
	- `-f 脚本` ： 从指定文件中读取并运行脚本
	- `-i.bak` ： 直接修改源文件，但是有备份
- 脚本：由***地址***和***编辑命令***两部分联合构成，没有分隔符，连起来写
- 地址定界：
	- 省略地址：表示对全文进行处理
	- `#`：指定行号
	- `$`：最后一行
	- `/regexp/`：模式匹配到的行
	- `#,/regexp/`：从指定行到模式匹配的行
	- `#1，#1`：从指定行1到指定行2
	- `/regexp1/,/regexp2/`：从模式匹配的行1到模式匹配的行2
	- `#,+n`：从指定的行开始，到向下的第n行
	- `1~2`：从1开始步进2 
- 编辑命令：
	- `d` ： 删除模式空间中的行  
	- `p` ： 打印模式空间中的行
	- `=` ： 为模式空间中的行打印行号
	- `a [\]text`：在行后添加内容（text中有空格的情况下使用\符号）可使用`\n`添加多行   
	- `w /path/to/somewhere`：指定的内容另存到指定文件中
	- `！`：模式空间中匹配行取反操作
	- `s/old/new/替换标记`  查找替换命令，分隔符无所谓    （这里可以使用后向引用操作 & \1）
			- `g` 全局替换  
			- `i` 不匹配大小写
**注意： 在正则表达式中匹配的字符有通配符的作用，需要使用转义符。**       

awk命令：报表生成器,格式化文本的输出

- 命令格式：
	- `awk [选项]  'program' file`
	- `awk [选项]  'BEGIN{action;...}pattern{action;...}END{action;...}' file`   
- 常用选项：
	- `-F` ：直接跟着分隔符号（省略的时候表示以空格为分割符号）  
	- `-v`或者`var=value`：用来设置program中使用的参数
	- `-f /path/from/awk-脚本` ： 从脚本中载入
- program的基本格式： `pattern{action statement} `  
	- patten部分决定动作语句何时触发事件
	- action部分表示对数据的处理，放在{}内      
- pattern（正则表达式 && 判断）
	- 省略不写的情况： 匹配文件的所有行
	- /regexp/ ： 处理匹配到的行 
	- /reg1/,/reg2/ ：不能直接使用数字来指定行数（可以使用NR变量构建关系表达式来指定行） 
	- 关系表达式：
		- 算数操作符 （x+y, x-y, x*y, x/y, x^y, x%y）
		- 赋值操作符 （=, +=, -=, *=, /=, %=, ^=, ++, --）
		- 比较操作符  (==, !=, >, >=, <, <=)
		- 模式匹配符  (~ 左侧是否匹配包含右侧,!~ 左侧是否不匹配包含右侧) 
		- 逻辑操作符  (与&&，或||，非!)
	- BEGIN（只在开始时执行一次）  END（在文本结束时执行一次）     
- action
	- print ： “,”隔开多个输出，打印默认空格是分隔符，使用“；”自定义分隔符 `print $1 ";" $2`
	- printf ： 格式化输出命令   `printf “格式1 格式2” ，变量1，变量2...`    
	- 条件表达式（三目表达式） 条件？符合时执行：不符合时执行
	- if(条件) {statement;...} [else{statement;...}]
	- [do{statement;...}]while(条件) {statement;...}
	- for(init;条件;变量修改){statement;...}

- 内置变量
    - `FS` :（输入字段分隔符）用法`-v FS=";"`,功能与-F选项完全一致,但是可以在program中使用
    - `OFS`:（输出字段分隔符）用法`-v OFS="%"`, 默认情况下是空格
    - `RS` :（输入记录分隔符，指定输入时的换行符）用法`-v RS=" "` , 按照设定将输入数据划分成记录段
    - `ORS`:（输出记录分隔符，指定输出时的换行符）用法`-v ORS=" "` ,按照设定将输出时的换行符替换
    - `NF` :（字段数量）    
    - `NR` :（记录号）
    - `FNR`:（在处理多文件的情况下，对每个文件的记录号分别计数）
    - `FILENAME` : （当前的文件名）
    - `ARGC` :（命令行参数个数）  包括awk自身
    - `ARGV` :（命令行参数矩阵）  
              
- awk数组：
	- name[$i]++ 的主要功能是将每一行的第一个字段作为索引信息，然后++完成次数的统计
    - for(var in name){print var, name[var]}
         
- awk函数
	- srand():随机数函数种子       
	- rand():随机数生成
	- sub(r,s,[t]) :对字符串t搜索r表示的模式匹配的内容，并将第一个匹配到的内容替换成s
	- gsub(r,s,[t]):对字符串t搜索r表示的模式匹配的内容，并将所有匹配到的内容替换成s
	- split(s,array,[t])：以t为分隔符切割字符串s，逐个放入到array数组中，索引值依次为1,2,3...
	- length() 输出字符串的长度


#### SHELL脚本 ####
- 切换shell：`usermod -s /bin/csh` 
- 查看当前shell的编号  `echo $$`         
- 查看shell之间的嵌套状态：`pstree -P`（显示进程树）    
- 当前的shell的层级：`echo $SHLVL`
- shell脚本：语法检查：`bash -n name.sh` ;  调试检查： `bash -x name.sh`
- 普通变量：只能对当前的shell中有效，在父shell以及子shell中都无效 
- 环境（全局）变量： 使用`export var`或者`declare -x var`定义变量
- 打印全部环境变量：`export/declaer -x `或者`/env/`或者`printenv`
- 显示全部变量：`set`   
- 撤销变量：`unset var`（建议在shell脚本结束时删除变量避免干扰，因为是在当前shell中定义的变量）
- 远程备份文件： `scp -arv /etc/  IP:/date/`
- **注：在脚本中调用另外的shell脚本相当与开启了子shell**                

- 双引号：弱类型表达，可执行引号中的命令引用
- 单引号：强类型表达，直接输出引号中的字符串
- 变量的引用：  使用$var 或者 ${var}
- 命令的引用：  `command` 或者 $(command)  
	- 注：在`var=$(command)`时，如果有多行内容
	-	 ${var} 或者 $var的打印，会将内容按一行打印，没有换行
	-	 “${var}” 或者 "$var"的打印，会将空白分隔符转成换行显示
- bash中的算数运算（关键字let），语法形式如下： 
	- `let var=算数表达式`   
	- `var=$[算数表达式]` 
	- `var=$((算数表达式))` 
	- `echo '算数表达式' | bc`
- 支持+=, -=, *=, /=, %=, --, ++ 运算    逻辑运算
      
- bash中的条件测试命令
	- test EXPRESSION
	- [ EXPRESSION ]     ：标准写法是对其中的变量引用以及字符串都加双引号
	- [[ EXPRESSION ]]   ：一般用在包含正则表达式的情况下
	- **注意：EXPRESSION前后必须有空白字符**
    - && 代表条件性的AND THEN （等价于-a） ;  || 代表条件性的OR ELSE （等价于-o）
    - `“==”`  表示字符串间的判定    
    - `“-eq”` 表示数字间的判定（也需要在变量引用两侧使用双引号）

- 接受终端的输入，`read`
    -p： 指定要显示的提示信息
    -s： 静默输入，用于密码输入
- 条件语句以及循环语句
* if语句的格式为：	
```

      	if[ 判断条件 ];then
        	statement
      	elif[ 判断条件 ];then
        	statement
      	else
        	statement
      	fi
```
* case语句的格式为：
```
	
      	case 变量引用 in
      	PAT1)
        	statement
        	;;
      	PAT2)
        	statement
        	;;
      	esac
```
* for语句的格式为：
```

		for 变量名 in 列表;do          for ((init;条件表达式;控制变量的修正));do
			statement                         statement 
		done                          done
```
* while语句的格式为：
```

		while 条件; do
			statement
		done        
```      
* 函数的格式：
```

		fName(){
			statement;
		}      
```   

- 调用函数与调用脚本的区别： 
     - 调用函数不会开启子shell （因此函数定义的变量会影响脚本中的变量，给函数中的变量加local关键字）
     - 调用脚本会开启子shell                        
- 一般将函数写入特定的文件中：`functions`     
- 在脚本中调用函数的时候首先输入 `source /path/to/functions`
- `declare -f`：显示当前系统中的所有程序

- 位置参数变量：$1、 $2 、、、${10}  用来表示传递给命令的参数
- 特殊变量：    
	- $? 用来表示最近一次命令的执行结果（可以判定命令是否成功执行 0）
	- $0 表示脚本名称本身     
	- $@: 表示全部的输入变量，并且是按空格分开的
	- $# 表示变量数量         
	- $*: 表示全部的输入变量，并且是按空格分开的
	- $@ 和 $* 在带了双引号的时候有区别 
		- “$@” 表示按空格分开的字符  
		- “$*” 表示将全部的输入整合成一个字符串
- 数组操作：
	- 引用数组元素： ${ARRAY_NAME[INDEX]} 注意：省略[INDEX]表示引用下标为0的元素
	- 引用数组所有元素：  ${ARRAY_NAME[*]}   ${ARRAY_NAME[@]}
	- 数组的长度(数组中元素的个数)： ${#ARRAY_NAME[*]}    ${#ARRAY_NAME[@]}
	- 删除数组中的某元素：unset ARRAY[INDEX]
	- 删除整个数组： unset ARRAY                      
