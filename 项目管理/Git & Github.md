### Git版本控制基本图示
![image.png](https://upload-images.jianshu.io/upload_images/9112509-10b11ec72a0da989.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Git基本命令
- 用户名和邮箱
    - `git config [--global] user.name`: 查看本地[全局]用户名
    - `git config [--global] user.email`: 查看本地[全局]用户邮箱
        - 本地用户名以及用户邮箱信息在 .git/config文件中
        - 全局用户名以及用户邮箱在 ~/.gitconfig文件中
        - *注意：本地有优先级高于全局的优先级*
    - `git config [--global] user.name Alec Bruks`: 设置本地[全局]用户名
    - `git config [--global] user.email xd@126.com`: 设置本地[全局]用户邮箱
    - 查看项目的git配置信息：`cat .git/config`
- 初始化、暂存、提交
    - `git init`: 初始化代码仓库，在当前目录下生成.git目录 = 本地代码仓库
        - `.git`所在的目录就是workspace，一般是项目的根目录
    - `git add <fileName>`: 在暂存区中添加文件，版本库开始跟踪该文件（!=提交）
        - `git add .`: 表示递归添加当前目录及子目录所有文件
        - `git rm --cached filename`: 将filename从追踪变为未追踪态
    - `git commit -m "message"`: *将暂存区中代码提交*到当前分支中
        - `-m` 选项表示本次提交的说明
        - commit提交的是暂存区中的改动，不是物理文件目前的改动
    - `git commit index.html`: 提交文件到暂存区并提交到版本库
    - `git commit -a`: 将*所有跟踪文件的改动*自动暂存，然后commit
    - `git commit --amend`: 修改上一次的提交补充信息

***
#### git文件分类
- 追踪的Tracked，已经加入到版本库中的文件
- 未追踪的UNtracked，未加入到版本库中的文件
- 忽略的Ignored，git不再关注的文件
    - .gitignore文件中，目录以/结尾，行开始的！表示取反
    - Python项目的忽略文件在git找文件

***
#### 仓库状态查看
- `git status`: 查看当前仓库的状态
- `git log`: 查看全部的提交历史
- `git reflog`: 记录每一次命令

#### 文件内容对比
- `git diff`: 对比暂存区与工作空间文件的修改
    - `git diff HEAD [-- fileName]`: 对比版本库与工作空间文件的修改
        - HEAD：指代最后一次commit
        - HEAD^: 指代上一次提交
        - HEAD^^: 指代上上一次提交
        - 上n次提交：HEAD~n
    - `git diff --cached`: 对比版本库与暂存区文件的修改
    - `git diff <$id1> <$id2>`: 比较两次提交之间的修改
    - `git diff branch1 branch2`: 比较两个分支最新提交之间的修改

#### checkout 检出
- 切换
    - `git checkout branch1`: 切换到分支branch1
        - `git checkout -b name`: 新建并切换分支
        - `git checkout [-b] div origin/div`: 克隆远程div分支[切换分支div]
    - `git checkout v1.0`: 切换tag
    - `git checkout commit_id`: 切换到某一次的提交
- 撤销
    - `git checkout`: 列出暂存区可以被检出的文件
    - `git checkout [--] filename`: 使用暂存区的文件覆盖工作空间中的文件
        - *注：checkout 命令只能撤销还没有 add 进暂存区的文件*
    - `git checkout .`: 检出暂存区的所有文件到工作区
    - `git checkout HEAD^^ sor.py`: 从HEAD前两次提交恢复sor到暂存区和工作空间
        - *注：HEAD可以指定为某次提交ID*

***
#### reset 重置
- `git reset`: 列出将被reset的文件
- `git reset file` : 将暂存区中的文件恢复到上次的commit状态，工作区不受影响
- `git reset --hard HEAD^`: 暂存区和工作区文件都恢复到前一次的commit状态
    - HEAD^: 每个托字符表示回退一级
    - HEAD~100: 表示退回100次
- `git reset ID`: 重置当前分支的HEAD为指定ID，同时重置暂存区，工作区不变
- `git reset --hard ID`: 重置当前分支的HEAD为指定ID，同时重置暂存区与工作区
- `git reset --keep ID`: 重置当前分支的HEAD为指定ID，暂存区与工作区不变
- `git reflog`: 显示commit的信息，只要HEAD发生变化，就可以在这里看到

***
#### 新建bug分支
用于解决的情况：当前分支的工作未完成（不建议commit），有需要紧急处理的问题时
- `git stash`: 将当前的工作状态保存(要求这些代码没有commit)
- `git stash list`: 查看stash空间中记录信息
    - 可以多次使用stash，恢复时首先查看list，然后恢复指定的stash
- `git stash pop`: 恢复未完的工作状态，并删除stash中的内容
    - `git stash apply`：恢复工作状态，但是stash内容不删除
    - `git stash drop`： 删除stash中的内容
- `git stash clear`: 清空所有暂存区的记录

### Git的分支管理
分支方式保证主版本代码的安全与稳定
- `git branch`: 查看本地仓库中所有的分支
- `git branch -r`: 查看远程仓库中的所有分支
- `git branch name`: 新建分支（在需要修改的分支上新建分支）
    - 此时分支内容与当前分支内容完全一致
- `git branch -d name`: 删除name分支
    - `git branch -D name`: 强制删除name分支
- `git merge name`: 将name分支合并到当前分支
- `git rebase name`: 将name分支合并到当前分支

### Git远程GitHub
- 方便多人协作以及代码发布所以使用GitHub
- GitHub支持SSH协议, ssh建立连接:
    - `ssh-keygen -t rsa -C 'message'`:生成秘钥和公钥 
        - -t 生成rsa方式加密的秘钥
        - -C 替换注释信息
        - 根据秘钥配对GitHub确保是作者提交代码的依据
- GitHub中添加公钥：
    - 打开公户设置-> SSH秘钥
    - 打开秘钥文件，将内容贴入“秘钥内容”框中，点击增加秘钥
    - 可以使用SSH方式登陆
- 添加远程库：
    - 在GitHub中新建仓库
    - 关联远程库: `git remote add origin url`：设置origin == url
        - `git remote rm origin` 删除对应的远程库文件夹
    - 查看当前的远程仓库：`git remote -v`
    - 推送branch分支所有文件到远程库: `git push -u origin branch`
        - `-u`：表示设置默认,下次可以直接使用git push命令完成推送
- 从远程库克隆代码
    - `git clone git@github.com:Pratyeka/GitSkill.git`
- 拿到最新提交，本地合并后提交
    - `git pull origin master`: 将远程最新的代码更新到本地(会造成文件覆盖)
    - `git branch --setupstream dev origin/dev`: 指定本地dev分支与远程origin/dev分支的连接

***
### Git标签
tag 指定版本号，方便版本回退调试，默认标签是打在最新提交的commit上的
- `git tag name [HEAD] -m "message"`: 新建标签，默认为HEAD，可以指定commit ID
    - `-m`：选项用来指定注释信息
    - `git checkout tagname`: 版本回退
- `git tag`: 可以查看所有的标签
- `git tag -d name`: 删除本地标签
- `git show tagname`: 显示tagname对应的信息
- *标签推送到远程*
    - `git push origin name`: 将某标签推送到远程
    - `git push origin --tags`: 一次将全部未推送的标签完成推送
    - `git push origin :refs/tags/<tagname>`: 删除一个远程标签
        - 前提是完成本地标签的删除

#### 移动和删除
- `git mv src dest`: 直接把改名的改动放入暂存区
- `git rm fileName`: 同时在版本库和工作目录中删除文件，真删除
- `git rm --cache file`: 将文件从版本库中删除，但不删除工作目录中的文件(不追踪)
- *注：只有在commit后才算是真改动了*

### 新建feature分支 ###
- 每次添加新功能都新建一个分支，为了保证主分支的安全与稳定
