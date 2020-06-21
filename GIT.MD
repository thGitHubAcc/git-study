#GIT下载
官网：git-scm.com

#初始化本地仓库
```
1.设置用户名和邮箱

2.初始化仓库
```

#提交文件
```
 add 命令（文件提交到暂存区）

 commit 命令 (将暂存区内容提交到本地仓库)

```

# 合并
```
1.将分支切换到主分支

2.合并操作 merge

```

# 冲突
```
1. 查看文件

2. 删除冲突的标记符


3. 重新commit提交 此时不需要带文件名 
```


#本地库和远程库交互
```
远程仓库创建好后，将远程仓库地址保存到本地git 创建别名
```

#GIT命令

## 查看git版本、
```
git --version
```
## 清屏
```
clear
```

## 设置用户名和邮箱
```
git config --global user.name "fth"
git config --global user.email "364870745@qq.com"
```

## 初始化仓库
```
git init(.git目录是隐藏的)
```

## 查看目录
```
ll(查看目录)
ll -la(查看所有目录)
```

## 查看文件
```
cat [filename]
```

## 添加文件
```
git add [filename]
[filename] 文件名
```

## 提交文件
```
git commit -m [memo] [filename]
-m 消息
[memo] 注释
```

## 查看工作区和暂存区状态
```
git status
```

## 查看日志
```
git log
<空格> 下一页
<b> 上一页
<q> 退出

日志展示方式
git log 默认 分页
git log --pretty=oneline 一行展示一个日志内容
git log --online 一行展示（日志索引缩写）
git log --reflog （多了 HEAD@{数字}） 指针回到当前历史版本需要多少步
```

##版本回退
```
git reset --hard [logindex]
[logindex] 版本索引

hard 参数：本地库版本指针移动的同时，重置暂存区，工作区
mixed 参数：本地库版本指针移动的时候，暂存区移动，工作区不动
soft 参数：只移动本地库的版本指针

找回被删除文件，也可通过版本回退实现
```

## 文件删除
```
rm [filename] (只删除工作区中的文件)
再执行 add ，commit 操作 将删除命令同步到本地库 

恢复暂存区数据
```

##比对
```
git diff [filename] （将工作区文件和暂存区文件进行比较）
不加文件名 则显示所有有差异的文件信息


git diff [HEAD] [filename] （暂存区与本地库进行比较）
[HEAD] 版本索引号
```

## 分支

### 查看分支
```
git branch -v 查看各个分支
```

### 创建分支
```
git breanch [branch]
[branch] 分支名
```

### 切换分支
```
git checkout [branch]
```

### 合并分支
```
git merge [branch]
```

## 创建别名
```
git remote -v
```

## 查看别名
```
git remote add [别名] [URL]
[URL] 远程仓库的地址
```