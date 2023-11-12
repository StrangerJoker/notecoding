# Git Demo

## Git 新建仓库

`git init`：在当前文件夹下新建一个git仓库，仓库名就是文件夹名

`git init repo-name`：在当前文件夹下新建一个名为`repo-name`的仓库

`git clone url`：从远程仓库拉取一个仓库

## 工作区域与文件状态

工作区 => `git add` => 暂存区 ->  `git commit` =>本地仓库：

**查看暂存区的内容:`git ls-files`**

文件状态：

![](D:\0-笔记以及思维导图\md\java\3-basic\pic\0-文件状态.png)



## git添加和提交文件

`git add`:

- `git add <file1> <file2>`：暂存指定文件
- `git add *.file-suffix`:通过通配符暂存指定文件
- `git add .`：暂存所有的文件

`git commit -m "your commit info msg"`：

`git commit -a -m "your commit info msg"`：暂存和提交修改到本地仓库

## git reset 回退版本

`git reset --param <commit-id>`

![](D:\0-笔记以及思维导图\md\java\3-basic\pic\1-git reset.png)



## git diff查看差异

1. 查看 工作区、暂存区 和 本地仓库之间的差异

   `git diff`：查看工作区和暂存区的差异

   `git diff HEAD`：比较工作区和版本库的差异

   `git diff --cached`：比较暂存区和版本库的差异

2. 查看不同版本的差异

   `git diff <commitI-id1> <commit-id2>`：查看指定两个提交的差异

   `git diff <last-commit-id> HEAD`：比较当前和上一提交的差异

   `git diff HEAD~ HEAD`:比较当前和上一提交的差异

   `git diff HEAD^ HEAD`:比较当前和上一提交的差异

   `git diff HEAD~2 HEAD`:比较当前和前两个提交版本的差异

   `git diff HEAD~2 HEAD file3`:比较当前和前两个提交版本的`file3文件`差异

3. 查看不同分支的差异

   `git diff branch-name1 branch-name2`：

## git rm删除文件

`git rm <file>`：删除 工作区，暂存区，本地库的file文件，删除本地文件

`git rm --cached <file>` ：删除本地库的文件，并不删除本地文件

`git rm -r <folder>`：删除文件阿基

## .gitignore忽略文件



![](D:\0-笔记以及思维导图\md\java\3-basic\pic\2-gitignore忽略规则.png)

注意：空文件夹不会被git管理

git官网匹配规则：http://git-scm.com/docs/gitignore

## 创建远程仓库并克隆到本地

`git clone <url>`

## 关联本地仓库到远程仓库

`git remote add origin <remote-url>`

`git push -u origin <branch-name>`

`git pull `：拉取远程仓库到本地，并合并（无冲突则合并，有冲突合并失败）

`git fetch`：拉取远程仓库到本地，未合并

## 分支及其基本操作

`git branch `：查看分支列表

`git branch <branch-name>`：创建一个新的分支

`git checkout <branch-name>`：可以用来切换分支 和 恢复文件

`git switch <branch-name>`：专门用来切换分支

`git merge dev`:在当前分支合并`dev`分支（将`dev`分支合并到当前分支上）,合并之后会默认产生一次提交

`git log --graph --oneline --decorate --all`：展示分支合并图

`git branch -d <branch-name>`：删除一个**已经合并**的分支，没有合并则不能删除

`git branch -D <branch-name>`：删除一个分支

##  解决分支合并的冲突

场景：两分支修改同一文件的同一行代码

 ## merge和rebase

![](D:\0-笔记以及思维导图\md\java\3-basic\pic\3-rebase.png)

merge和rebase的优缺点

merge：

- 优点：不会破坏原分支的提交历史，方便回溯和查看
- 缺点：会产生额外的提交节点，分支图比较复杂

rebase：

- 优点：不会新增额外的提交分支，形成线性历史，比较直观和干净
- 缺点：会改变提交历史，改变了当前分支branch out的节点，避免在共享分支使用。

