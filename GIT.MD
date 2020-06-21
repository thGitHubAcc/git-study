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
```