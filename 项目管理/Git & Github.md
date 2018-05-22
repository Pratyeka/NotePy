### Git版本控制基本图示
![image.png](https://upload-images.jianshu.io/upload_images/9112509-10b11ec72a0da989.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Git基本命令 ###
- `git init`: 初始化代码仓库，在当前目录下生成.git目录 = 本地代码仓库
    - `.git`所在的目录就是workspace，一般是项目的根目录
- `git add <fileName>`: 在暂存区中添加文件，版本库开始跟踪该文件（!=提交）
- `git commit -m "message"`: 将暂存区中代码提交提交到分支中，
    - -m 参数表示文件说明

- `git status`: 查看当前仓库的状态
- `git diff`: 查看文件的修改(对比暂存区与工作空间文件)
    - `git diff HEAD -- fileName`(对比版本库与工作空间文件)
    - `git diff --cached`(对比版本库与暂存区文件)

- `git log`: 查看全部的提交历史
- `git reflog`: 记录每一次命令
- `git reset --hard HEAD^`: 跳转到上一次提交的位置处
    - HEAD^: 每个托字符表示回退一级
    - HEAD~100: 表示退回100次
- `git reset --hard ID`: 跳转到指定ID处

- `git checkout -- fileName`: 从版本库或者暂存区中一键还原文件
- `git reset HEAD fileName` : 文件从暂存区中撤销，然后使用checkout命令完成进一步的撤销
- `git reset --hard HEAD^`  : 提交到版本库中，使用版本回退方式来撤销
- `git rm fileName`: 从版本库中删除文件

### Git远程GitHub ###
- 方便多人协作以及代码发布所以使用GitHub
- GitHub支持SSH协议
- 建立连接的方式
    - `ssh-keygen -t rsa -C 'message'`:生成秘钥和公钥 
        - -t 生成rsa方式加密的秘钥
        - -C 替换注释信息
        - 根据秘钥配对GitHub确保是作者提交代码的依据

- 添加远程库：
    - 在GitHub中新建仓库
    - 关联远程库:  `git remote add origin git@github.com:Pratyeka/learnPush.git`：设置origin == url
        - `git remote rm origin` 删除对应的远程库文件夹
    - 推送master分支所有文件到远程库: `git push -u origin master`
        - `-u`：表示设置默认,下次可以直接使用git push命令完成推送

- 从远程库克隆代码
    - `git clone git@github.com:Pratyeka/GitSkill.git`
- 拿到最新提交，本地合并后提交
    - `git pull`: 把最新的提交从从某一分支中拿到
    - `git branch --setupstream dev origin/dev`: 指定本地dev分支与远程origin/dev分支的连接

### Git标签 ###
- `git tag name [HEAD]`: 新建标签，默认为HEAD，可以指定commit ID
- `git tag tagname -m "message"`: -m 用来指定注释信息
- `git tag`: 可以查看所有的标签
- `git tag -d name`: 删除本地标签
- `git push origin name`: 将某标签推送到远程
- `git push origin --tags`: 一次将全部未推送的标签完成推送
- `git push origin :refs/tags/<tagname>`: 删除一个远程标签
    - 前提是完成本地标签的删除


### Git的分支命令管理 ###
分支方式保证主版本代码的安全与稳定
- `git branch`:           查看仓库中所有的分支
- `git branch name`:      新建分支（在需要修改的分支上新建分支）
- `git checkout name`:    切换分支
- `git checkout -b name`: 新建并切换分支
    - `git checkout -b div origin/div`：clone代码仓库后切换分支
- `git merge name`:       将name分支合并到当前分支
- `git branch -d name`:   删除name分支
    - `git branch -D name`: 强制删除name分支

### 新建bug分支 ###
用于解决的情况：当前分支的工作未完成（不想commit），有需要紧急处理的问题时
- `git stash`: 将当前的工作状态保存
- `git stash pop`: 恢复未完的工作状态，并删除stash中的内容
- `git stash list`: 查看stash空间中内容
    - 可以多次使用stash，恢复时首先查看list，然后恢复指定的stash
- `git stash apply`：恢复工作状态，但是stash内容不删除
- `git stash drop`： 删除stash中的内容 

### 新建feature分支 ###
- 每次添加新功能都新建一个分支，为了保证主分支的安全与稳定
