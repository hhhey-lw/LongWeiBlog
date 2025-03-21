---
author: "LongWei"
title: "Linux&Docker&Shell命令"
date: "2025-03-16"
tags: ["Commands"]
categories: ["Learning"]
series: ["Learning"]
aliases: ["common_commands"]
ShowToc: true
TocOpen: true
---


### docker

```bash
service docker start

docker pull <镜像名称>:<标签>

docker rmi <镜像名称或ID>

docker images

docker run [选项] <镜像名称>:<标签>

docker start <容器名称或ID>

docker ps

docker exec -it <容器名称或ID> /bin/bash # execute -it:Interactive and Teletypewriter 交互式&伪终端
```





### **Linux**

文件和目录

```bash
ls -l # -l:详细信息
cd
pwd # 当前目录
mkdir/vim
rm/rmdir
cp mv
```

文件权限

```bash
chmod  # Change Mode  # chmod u+rwx file.txt x:Execute
chown
```

系统命令

```bash
ps -ef  # e:all -f full info
ps -u $USER
ps -ef | grep <info>  # global regular expression print

htop  # 显示系统资源使用情况
```

网络管理

```bash
ping
ifconfig
netstat  # 网络连接信息
```

压缩解压

```bash
tar -czvf archive.tar.gz folder_name  # 压缩 
tar -xzvf archive.tar.gz  # 解压

-c: # create 创建
-x: # extract解压
-z: # 使用gzip压缩
-v: # verbose显示[详细]信息
-f：# file 指定文件名
```

杂项

```bash
history 
echo
clear
```





### **Git**

Git 是一个分布式版本控制系统，支持多人协作开发。



```bash
git init
git clone --link
git status
添加文件：git add file  或者 git add .
提交文件：git commit -m "commit msg"
git push 
git pull
git log

// branch 分支
git branch <分支名>
git checkout <分支名>  // 切换分支
git merge <分支名>
-- 手动编辑冲突文件，然后使用 git add 标记冲突已解决，最后提交更改。
```



### **Shell**

条件判断

```bash
#!/bin/bash
if [ -f "file.txt" ]; then
        echo "file.txt exists."
else
        echo "file.txt not exists."
fi

# -f 检查文件是否存在

#!/bin/bash
num=10
if [ $num -gt 5 ]; then
        echo "$num is greater than 5."
fi

-gt # greater than
```

循环

```bash
#!/bin/bash
num=(1 2 3 4 5)
for i in "${num[@]}"; do  # 是用于引用数组 num 中所有元素的方式
        echo "number: $i"
done
------------------------
#!/bin/bash
if [ $# -eq 0 ]; then
        echo "Usage: $0 <n1> <n2>"
        exit 1
fi

for i in "$@"; do
        echo "number: $i"
done    

# $# 表示传递给脚本或函数的命令行参数的数量  $@ 所有参数
=> ./script.sh 1 2 3
```

Where

```bash
#!/bin/bash
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    count=$((count + 1))
done
```

函数

```bash
#!/bin/bash
greet() {
    echo "Hello, $1!"
}
greet "Alice"
greet "Bob"
```

读写文件

```bash
echo "Hello, File!" > output.txt  # write：> 覆盖 >> 追加

while read line; do
    echo "Line: $line"
done < file.txt     # 读文件
```

读取输入

```bash
echo "Enter your name:"
read name  # 类似scanf 控制台阻塞等待
echo "Hello, $name!"

read -s -p "Enter your password: " password
echo "Password entered."

# s 静默，不显示信息  p显示提示信息
```

