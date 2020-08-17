### Git

#### 1.Git

##### 进行全局配置

配置用户名和邮箱

git config --global user.name "用户名"

git config --global user.email "xxxxx@qq.com"

##### 

##### 基本命令

###### 1.初始化仓库（将当前目录初始化为一个git本地仓库）

git init

###### 2.查看git状态

git status

###### 3.将文件从工作区添加到缓冲区

git add 文件1  文件2...

git add .  （.代表添加当前）

###### 4.将文件从缓冲区清除

git rm --cache 文件1 文件2...git 

###### 5.把文件从缓存区提交到版本库

git commit -m "注释信息"



##### 分支管理

一个分支可以认为是一个模块

###### 1.创建分支

git branch 分支名

###### 2.查看分支

git branch

###### 3.切换分支

git checkout  分支名

###### 创建分支后切换到新创建的分支

git checkout -b 分支名

###### 5.删除分支

git branch -d 分支名

###### 6..合并分支

git merge 被合并的分支名



##### 版本回退

###### 查看版本

显示详细日志：

git log 

```yml
 #(HEAD -> master):代表当前版本 
 #abeb1d52507e30a3e224eeee85ce081119761268：提交编号
commit abeb1d52507e30a3e224eeee85ce081119761268 (HEAD -> master)
Author: ywb <1808500763@qq.com> #修改用户
Date:   Fri May 1 10:15:26 2020 +0800 #时间

    test push #注释信息

```



显示一行日志：

git log --pretty=oneline

```yml
abeb1d52507e30a3e224eeee85ce081119761268 (HEAD ->master) test push
```

###### 回退版本

git reset --hard 提交编号

###### 回退后查看历史版本

（*git log 只能查看当前版本之前的版本，若1 2 3 回退到2后还想回退到3可以使用一下命令查看*）

git reflog



##### 忽略文件

有的文件基本不变（js、css、html）或者部分文件不想提交到线上仓库，可以在创建 .gitighore文件，用于声明忽略文件

步骤:

1.在仓库下创建 .gitignore

touch .gitignore

2.编辑内容

vi .gitignore

```
#过滤规则
/test/          忽视整个文件夹
*.zip           忽视所有zip文件
/test/a.txt     忽视具体文件
!/test/a.txt    不忽视某个具体文件
```



#### 2.github

##### 克隆项目到本地

git clone https://github.com/ywb-create/ywb-create.git



###### clone时如果报403，则将 .git/config 的url进行如下更改

```
 更改前：url = https://github.com/ywb/Learn-note.git

 更改后：url = https://同户名:密码@github.com/ywb/Learn-note.git
```



##### 拉取线上版本

当项目更改时，拉取github的最新版本到本地git中

git pull



##### 上传到github

git push 



###### push到远程分支

git push origin test:master         // 提交本地test分支作为远程的master分支，远程的github就会自动创建一个test分支 

git push origin test:test              // 提交本地test分支作为远程的test分支

删除远程的分支： 如果:左边的分支为空，那么将删除:右边的远程的分支。

 git push origin :test              // 刚提交到远程的test将被删除，但是本地还会保存的，不用担心



##### git连接github

###### 1.连接

git remote add origin https://github.com/ywb-create/ywb-create.git

###### 4.查看连接

git remote -v

###### 5.删除连接

git remote rm origin

