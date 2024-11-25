个人博客

    # 初始化一个新的git, 
    git init
    git config --global user.email "18713822182@163.com"
    git config --global user.name "zzc"
    git remote add origin https://gitee.com/zhang-zechang/myblog.git
    git add .  # 在执行的时候.deploy_git也有一个.git 不用管,强制执行
    git commit -m 'first commit'
    git push -u origin "master"
    
    # 克隆到其他地方后执行下面的操作
    npm install
    cd .\.deploy_git\
    # 先给.deploy_git配一个git及路径
    git init
    git remote add orange https://github.com/zhangzc-hub/zhangzc-hub.github.io.git
    cd ..
    # 执行打包和发布功能
    hexo g
    hexo d


获取远程仓库的最新更改：
git fetch origin
查看你的本地分支状态：
git status
将远程更改合并到你的本地分支：
git merge origin/master
这个命令会尝试将远程分支合并到你的本地分支。如果有任何冲突，Git 会提示你解决这些冲突。
如果你想要直接拉取（即合并）远程仓库的最新更改到你的本地分支，你可以使用 git pull 命令，它实际上是一个 fetch 和 merge 的组合。这里是具体的命令：
git pull origin master
这条命令会从远程仓库 origin 的 master 分支拉取最新的更改，并自动合并到你的本地 master 分支。
如果你的本地分支和远程分支之间有冲突，Git 会暂停合并过程，并要求你手动解决冲突。一旦解决了所有冲突，你可以提交这些更改以完成合并：
git add <conflicted-file>
git commit -m "Resolved merge conflicts"
完成以上步骤后，你的本地分支就包含了最新的远程更改。如果你需要再次推送你的更改到远程仓库，可以使用：

git push origin master
这将把你的更改推送到远程仓库的 master 分支
