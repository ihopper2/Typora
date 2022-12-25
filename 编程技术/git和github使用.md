# 发展历史

Linus在1991年创建了开源的Linux，从此，Linux系统不断发展，已经成为最大的服务器系统软件了。Linus虽然创建了Linux，但Linux的壮大是靠全世界热心的志愿者参与的，这么多人在世界各地为Linux编写代码，那Linux的代码是如何管理的呢?事实是，在2002年以前，世界各地的志愿者把源代码文件通过diff的方式发给Linus，然后由Linus本人通过手工方式合并代码!你也许会想，为什么Linus不把Linux代码放到版本控制系统里呢?不是有CVS、SVN这些免费的版本控制系统吗?因为Linus坚定地反对CVS和SVN，这些集中式的版本控制系统不但速度慢，而且必须联网才能使用。有一些商用的版本控制系统，虽然比CVS、SVN好用，但那是付费的，和Linux的开源精神不符。不过，到了2002年，Linux系统已经发展了十年了，代码库之大让Linus很难继续通过手工方式管理了，社区的弟兄们也对这种方式表达了强烈不满，于是Linus选择了一个商业的版本控制系统BitKeeper，BitKeeper的东家BitMover公司出于人道主义精神，授权Linux社区免费使用这个版本控制系统。安定团结的大好局面在2005年就被打破了，原因是Linux社区牛人聚集，不免沾染了一些梁山好汉的江湖习气。开发Samba的Andrew试图破解BitKeeper的协议(这么干的其实也不只他一个)，被BitMover公司发现了(监控工作做得不错!)，于是BitMover公司怒了，要收回Linux社区的免费使用权。Linus可以向BitMover公司道个歉，保证以后严格管教弟兄们，嗯，这是不可能的。实际情况是这样的：Linus花了两周时间自己用C写了一个分布式版本控制系统，这就是Git!一个月之内，Linux系统的源码已经由Git管理了!牛是怎么定义的呢?大家可以体会一下。Git迅速成为最流行的分布式版本控制系统，尤其是2008年，GitHub网站上线了，它为开源项目免费提供Git存储，无数开源项目开始迁移至GitHub，包括jQuery，PHP，Ruby等等。历史就是这么偶然，如果不是当年BitMover公司威胁Linux社区，可能现在我们就没有免费而超级好用的Git了。

# git常用命令

| 命令名称                       | 作用                           |
| ------------------------------ | ------------------------------ |
| git config --global user.name  | 用户名 设置用户签名            |
| git config --global user.email | 邮箱 设置用户名                |
| git init                       | 初始化本地库                   |
| git status                     | 查看本地库状态                 |
| git add                        | 文件名 添加到暂存区            |
| git commit -m                  | "日志信息" 文件名 提交到本地库 |
| git reflog                     | 查看历史纪录                   |
| git log                        | 查看历史详细信息               |
| git reset --hard               | 版本号 版本穿梭                |

## 设置用户签名

```javascript
git config --global user.name 用户名
git config --global user.email 邮箱

```

全局范围签名的设置

签名的作用是区分不同操作者身份。用户的签名信息在每一个版本的提交信息中能够看到，以此确认本次提交是谁做的。Git 首次安装必须设置一下用户签名，否则无法提交代码。

这里设置用户签名和将来登录 GitHub（或其他代码托管中心）的账号没有任何关系。

## 初始化本地库

```
git init
```



![在这里插入图片描述](https://kangrui-pictures.oss-cn-beijing.aliyuncs.com/img/20210619135040286.png)

##  添加暂存区

基本语法   git add 文件名

在暂存区删除 git rm --cached <file>

## 提交到本地库

基本语法 git commit -m "日志信息" 文件名

查看状态 git status

## 历史版本

git reflog 查看版本信息

git log 查看版本详细信息

git reset --hard 版本号    回溯为之前的版本

# 分支操作

| 命令名称            | 作用                         |
| ------------------- | ---------------------------- |
| git branch 分支名   | 创建分支                     |
| git branch -v       | 查看分支                     |
| git checkout 分支名 | 切换分支                     |
| git merge 分支名    | 把指定的分支合并到当前分支上 |

##  远程仓库操作

| 命令名称                     | 作用                                                     |
| ---------------------------- | -------------------------------------------------------- |
| git remote -v                | 查看所有的远程地址别名                                   |
| git remote add 别名 远程地址 | 起别名                                                   |
| git push 别名 分支           | 推送本地分支的内容到远程仓库                             |
| git clone 远程地址           | 将远程仓库的内容克隆到本地                               |
| git pull 别名 远程分支名     | 将远程仓库对于分支最新内容拉下来后与当前本地分支直接合并 |

6.4

