### Pyenv开发环境
- linux平台下的Python多版本管理工具
    - 管理Python解释器
    - 管理Python版本
    - 管理Python的虚拟环境

### Pyenv的安装
- 安装前准备：pip包管理器
    - pip配置：vim ~/.pip/pip.conf 添加如源
    ```ini
        [global]
        index-url=https://mirrors.aliyun.com/pypi/simple/
        trusted-host=mirrors.aliyun.com
    ```
    - 安装常用软件包
        - pip install redis ipython
        - pip install jupyter
            - jupyter notebook password
            - jupyter notebook -ip=... --port=8888
        - pip -V
    - 包管理
        - pip freeze > requirement
        - pip install -r requirement
- linux平台下Pyenv安装
    1. 安装git：yum install git -y
    2. 安装Python编译依赖项：yum -y install gcc make patch gdbm-devel openssl-devel sqlite-devel readline-devel zlib-devel bzip2-devel
    3. 创建用户Python： useradd python
    4. 使用python用户登陆后安装Pyenv：
    ```shell
        curl -L https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer | bash
    ```
        - 出现SSL connect error，是nss版本太低，配置新的yum源
        ```ini
            [updates]
            name=CentOS-Updates
            baseurl=https://mirrors.aliyun.com/centos/6.9/os/x86_64
            gpgcheck=0
        ```
        - 执行更新：yum update nss
    5. 在pythoh用户的~/.bash_profile中追加:
        ```ini
        export PATH="/home/python/.pyenv/bin:$PATH"
        eval "$(pyenv init -)"
        eval "$(pyenv virtualenv-init -)"
        ```
        - 更新：source ~/.bash_profile    (添加了启动项)
- Pyenv命令
    - 列出所有可用版本：pyenv install --list
    - 在线安装指定版本：pyenv install 3.5.3
        - cache方式安装：
            1. 在 ~/.pyenv目录下，新建cache目录，放入下载好的安装版本文件
            2. pyenv install 3.5.3 -v
    - pyenv版本控制
        - global全局设置：pyenv global 3.5.3(root下不可用)
        - shell会话：pyenv shell 3.5.3（只影响当前会话）
        - local本地设置：pyenv shell 3.5.3（从当前目录开始向下递归继承该设置）
- virtualenv虚拟环境
    - 没有虚拟环境时，所有的Python都在一个公用的空间中，为了应对如下问题：
        - 多个项目使用不同Python版本开发
        - 使用不同的Python版本部署运行
        - 使用同样的版本开发，但是不同项目使用不同版本的库
    - 使用命令：pyenv virtualenv 3.5.3 dl353
        - 创建的dl353是一个独立的虚拟空间（真实目录：~/.pyenv/versions）
        - Python可以针对特定的目录设置虚拟环境