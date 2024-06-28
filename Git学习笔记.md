# Git学习笔记

## 配置和初始化

参数：

1. Local：本地配置，只对本地仓库有效
2. golbal：全局配置，所有仓库生效
3. system：系统配置，对所有用户生效

~~~git
//配置用户名
git config --global user.name "name"
//配置邮箱
git config --global user.email "321321@outlook.com"
//保存
git config --global --list
~~~

## 新建版本库

有两种方式，一种在本地创建仓库，一种从网上克隆仓库

~~~git
//在本地创建仓库
//1、新建目录
mkdir learn-git
//2.新建仓库
git init
~~~

## 工作区域和文件状态

### 工作区域

1. 工作区：.git所在的目录
2. 暂存区：.git/index 临时存储区域，用于保存即将提交到git仓库的修改内容，
3. 本地仓库：.git/objects 包含完整的项目历史和版本工具

### 文件状态

1. 未跟踪：新创建还没有被Git管理起来的文件
2. 未修改：已经被Git管理但是文件内容没有发生变化
3. 已修改：已经修改过还没有添加到暂存区中的文件
4. 已暂存：修改过后已经添加到暂存区中的文件

## 添加和提交文件

~~~Git
//新建仓库
git init
//查看仓库状态
git status
//添加到暂存区
git add
//提交
git commit -m "提交参数信息"
~~~

## 回退版本——git reset命令

三种模式

1. git reset --soft 回退到某个版本并且**保留**工作区和暂存区的所有修改内容
2. git reset --hard 回退到某个版本并且**丢弃**工作区和暂存区的所有修改内容
3. git reset --mixed 默认参数 回退到某个版本保留工作区丢弃暂存区

## 查看差异——git diff

~~~Git
//比较工作区和暂存区的差异
git diff
//比较工作区和仓库的差异
git diff HEAD
//比较暂存区和仓库之间的差异
git diff --cached

//比较版本差异
//HEAD 指向最新提交节点
git diff '版本号1' '版本号2'
git diff '版本号1' HEAD	//对比最新版本
git diff HEAD~ HEAD		//HEAD~指向上一个版本
git diff HEAD^ HEAD		//HEAD^指向上一个版本
git diff HEAD~2 HEAD	//HEAD~2指向之前的两个版本
git diff HEAS~3 HEAD filename  //只查看这个文件的差异
~~~

## 删除文件

~~~Git
//直接删除
//使用这种删除只是删除工作区，暂存区中还没有删除，需要接着提交到暂存区然后提交到仓库才算删除
rm filename
//删除命令
//同时删除工作区和暂存区
git rm filename
//删除仓库中的文件并将这个文件放回暂存区
git rm --cached filename

~~~

## .gitignore 忽略文件

git会忽略在.gitignore中列出的文件

文件的匹配方式跟正则表达式很像

在这节课中用到的一些命令

~~~Git
//将工作区中的内容直接提交到仓库
git commit -am "massage"
//追加内容
echo "text" >> filename
//新建目录
mkdir temp
//显示被Git追踪的文件
git ls-files
//显示简短的文件状态
	1.?? 未跟踪
	2.M 已修改
	3.A 已添加暂存
	4.D 已删除
	5.R 重命名
	6.U 更新未合并
git status -s
~~~

## SSH配置和克隆仓库

~~~Git
//生成SSH密钥
ssh-keygen -t rsa -b 4096
//克隆仓库
git clone 'SSH地址'
//将文件推送到github上  需要在github上添加SSH的公钥
git push
~~~

回到根目录

**如果不是第一次创建密钥则执行以下操作**

输入SSH文件名再回车
添加config文件，并在文件中添加

~~~Git
Host github.com
HostName github.con
PreferredAuthentications publickey
IdentityFile ~/.ssh/name
~~~

## 关联本地仓库和远程仓库

~~~Git
//添加远程仓库
git remotr add '仓库别名' '仓库地址'
git push -u '远程仓库名' '分支名'

//列出别名对应的远程仓库
git remote -v

//与本地仓库关联
git push -u '远程仓库别名' 分支名:分支名

//拉取远程仓库内容与本地仓库合并
//拉去之后会自动执行一次合并操作
git pull '远程仓库名' '远程分支名':'本地分支名'
//获取远程仓库的修改，不会自动合并到本地仓库中
git fetch
~~~

## 分支和基本操作

> 使用分支名+序号命名文件
>
> 使用分支名+冒号+序号的方式编写提交记录

### 新建分支

~~~Git
git branch 分支名
~~~

### 切换分支

~~~Git
git switch 分支名
~~~

### 合并分支

~~~Git
//将指定分支合并到当前所在分支
git merge 分支名
~~~

### 删除分支

~~~Git
git branch -d 分支名
//如果分支没有合并
git branch -D 分支名
~~~

## 回退和 rebase

在rebase中，每个分支都有一个指针，指向当前分支的最新提交记录，在执行rebase时，git会先找到当前分支和目标分支的共同祖先，在把当前分支从共同祖先到最新提交记录的所有提交都移动到目标分支的最新提交后面

## Rebase 和 Merge有什么区别该如何区分使用

Merge

- 优点：不会破坏原分支的提交历史，方便回溯和查看
- 缺点：会产生额外的提交节点，分支图比较复杂

Rebase

- 优点：不会新增额外的提交记录，形成线性历史，直观干净
- 缺点：会改变提交历史，改变了当前分支 branch out 的节点

分支管理和工作流模型
