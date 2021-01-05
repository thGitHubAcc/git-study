# Shell

### hello world 
```sh
1. 创建test.sh (vi test.sh)

2. 编辑内容
echo "Hello World !"

3. 执行
  方式1 /bin/sh test.sh
  方式2 chmod +x ./test.sh  #使脚本具有执行权限
      - ./test.sh  #执行脚本
```


### 1.变量
命名只能使用英文字母，数字和下划线，首个字符不能以数字开头。
中间不能有空格，可以使用下划线（_）。
不能使用标点符号。
不能使用bash里的关键字（可用help命令查看保留关键字）。


#### 1.1 使用变量
变量名外面的花括号是可选的，加不加都行，加花括号是为了帮助解释器识别变量的边界
```
your_name="fth"
echo $your_name
echo ${your_name}
```

#### 1.2 只读变量
```sh
myUrl="https://www.google.com"
readonly myUrl
myUrl="https://www.runoob.com"
```
运行结果：
```
/bin/sh: NAME: This variable is read only.
```

#### 1.3 删除变量
变量被删除后不能再次使用。unset 命令不能删除只读变量。
```
unset variable_name
```

### 2.字符串

#### 2.1 获取字符串长度
```sh
string="abcd"
echo ${#string} #输出 4
```

#### 提取子字符串
```sh
string="runoob is a great site"
echo ${string:1:4} # 输出 unoo
```

#### 查找子字符串
查找字符 i 或 o 的位置(哪个字母先出现就计算哪个)：

以上脚本中 \` 是反引号，而不是单引号 '
```sh
string="runoob is a great site"
echo `expr index "$string" io`  # 输出 4
```