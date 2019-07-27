# 包管理工具
RPM 和 DPKG 为最常见的两类软件包管理工具：
- RPM 全称为 Redhat Package Manager，最早由 Red Hat 公司制定实施，随后被 GNU 开源操作系统接受并成为很多 Linux 系统 (RHEL) 的既定软件标准。
- 与 RPM 竞争的是基于 Debian 操作系统 (Ubuntu) 的 DEB 软件包管理工具 DPKG，全称为 Debian Package，功能方面与 RPM 相似。

YUM 基于 RPM，具有依赖管理和软件升级功能。

# VIM三个模式
- 一般指令模式（Command mode）：VIM 的默认模式，可以用于移动游标查看内容；
- 编辑模式（Insert mode）：按下 "i" 等按键之后进入，可以对文本进行编辑；
- 指令列模式（Bottom-line mode）：按下 ":" 按键之后进入，用于保存退出等操作。

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/14b9d1ffc34ee9e91d5dff2c38520fb9_591x498.png" />

在指令列模式下，有以下命令用于离开或者保存文件。

| 命令 | 作用 |
| - | -|
| :w | 写入磁盘 |
| :w! | 当文件为只读时，强制写入磁盘。到底能不能写入，与用户对该文件的权限有关 |
| :q | 离开 |
| :q! | 强制离开不保存 |
| :wq | 写入磁盘后离开 |
| :wq! | 强制写入磁盘后离开 |

# 常用指令
## 压缩与打包
Linux 底下有很多压缩文件名，常见的如下：

| 扩展名 | 压缩程序 |
| --- | --- |
| \*.Z | compress |
| \*.zip | zip |
| \*.gz | gzip |
| \*.bz2 | bzip2 |
| \*.xz | xz |
| \*.tar | tar 程序打包的数据，没有经过压缩 |
| \*.tar.gz | tar 程序打包的文件，经过 gzip 的压缩 |
| \*.tar.bz2 | tar 程序打包的文件，经过 bzip2 的压缩 |
| \*.tar.xz | tar 程序打包的文件，经过 xz 的压缩 |

### 压缩指令
1\. gzip
gzip 是 Linux 使用最广的压缩指令，可以解开 compress、zip 与 gzip 所压缩的文件。

经过 gzip 压缩过，源文件就不存在了。

有 9 个不同的压缩等级可以使用。

可以使用 zcat、zmore、zless 来读取压缩文件的内容。

```shell
$ gzip [-cdtv#] filename
-c ：将压缩的数据输出到屏幕上
-d ：解压缩
-t ：检验压缩文件是否出错
-v ：显示压缩比等信息
-# ： # 为数字的意思，代表压缩等级，数字越大压缩比越高，默认为 6
```

2\. bzip2

提供比 gzip 更高的压缩比。

查看命令：bzcat、bzmore、bzless、bzgrep。

```shell
$ bzip2 [-cdkzv#] filename
-k ：保留源文件
```

3\. xz

提供比 bzip2 更佳的压缩比。

可以看到，gzip、bzip2、xz 的压缩比不断优化。不过要注意的是，压缩比越高，压缩的时间也越长。

查看命令：xzcat、xzmore、xzless、xzgrep。

```shell
$ xz [-dtlkc#] filename
```
### 打包
压缩指令只能对一个文件进行压缩，而打包能够将多个文件打包成一个大文件。tar 不仅可以用于打包，也可以使用 gzip、bzip2、xz 将打包文件进行压缩。

```shell
$ tar [-z|-j|-J] [cv] [-f 新建的 tar 文件] filename...  ==打包压缩
$ tar [-z|-j|-J] [tv] [-f 已有的 tar 文件]              ==查看
$ tar [-z|-j|-J] [xv] [-f 已有的 tar 文件] [-C 目录]    ==解压缩

-z ：使用 zip；
-j ：使用 bzip2；
-J ：使用 xz；
-c ：新建打包文件；
-t ：查看打包文件里面有哪些文件；
-x ：解打包或解压缩的功能；
-v ：在压缩/解压缩的过程中，显示正在处理的文件名；
-f : filename：要处理的文件；
-C 目录 ： 在特定目录解压缩。
```

| 使用方式 | 命令 |
| -| - |
| 打包压缩 | tar -jcv -f filename.tar.bz2 要被压缩的文件或目录名称 |
| 查看 | tar -jtv -f filename.tar.bz2 |
| 解压缩 | tar -jxv -f filename.tar.bz2 -C 要解压缩的目录 |

## 管道指令
管道是将一个命令的标准输出作为另一个命令的标准输入，在数据需要经过多个步骤的处理之后才能得到我们想要的内容时就可以使用管道。

在命令之间使用 | 分隔各个管道命令。

```shell
$ ls -al /etc | less
```

## 进程管理
1\. ps
查看某个时间点的进程信息

示例一：查看自己的进程
```hell
$ ps -l
```
示例二：查看系统所有进程

```shell
$ ps aux
```
示例三：查看特定的进程

```shell
$ ps aux | grep threadx
```
2\. pstree

查看进程树

示例：查看所有进程树
```shell
$ pstree -A
```
3\. top

实时显示进程信息

示例：两秒钟刷新一次
```shell
$ top -d 2
```
4\. netstat

查看占用端口的进程

示例：查看特定端口的进程
```shell
$ netstat -anp | grep port
```
其他示例
```shell
$ ps -ef  显示当前所有进程环境变量及进程间关系
$ ps -A  显示当前所有进程
$ ps -aux | grep apache  与 grep 联用查找某进程
$ ps aux | grep '(cron|syslog)'  找出与 cron 与 syslog 这两个服务有关的 PID 号码
```
## 文件查找
find 搜索文件的命令格式

find \[搜索范围\] \[匹配条件\]

选项:
- name 根据名字查找
- size    根据文件大小查找, +, -\: 大于设置的大小,直接写大小是等于
- user   查找用户名的所有者的所有文件
- group 根据所属组查找相关文件
- type    根据文件类型查找(f 文件,d 目录,l 软链接文件)
- inum   根据 i 节点查找
- amin   访问时间 access
- cmin    文件属性 change
- mmin   文件内容 modify

示例：

``` shell
$ find / -name filename.txt 根据名称查找/目录下的 filename.txt 文件。  
$ find . -name "*.xml" 递归查找所有的xml文件  
$ find . -name "*.xml" |xargs grep "hello world" 递归查找所有文件内容中包含 hello world的 xml 文件  
$ grep -H 'spring'*.xml 查找所以有的包含 spring 的 xml 文件  
$ find ./ -size 0 | xargs rm -f & 删除文件大小为零的文件  
```
## 日常操作
查看文件： ls ll

进入目录： cd

显示当前路径： pwd

创建文件夹： mkdir 文件名

rm： 删除 ，删除以 -f 开头的文件 rm -- -f*

mv：重命名&移动文件
- 将文件 test.log 重命名为 test1.txt
```shell
$ mv test.log test1.txt
```

- 将文件 log1.txt，log2.txt，log3.txt 移动到根的 test3 目录中
```shell
$ mv llog1.txt log2.txt log3.txt /test3
```
- 将文件 file1 改名为 file2，如果 file2 已经存在，则询问是否覆盖
```shell
$ mv -i log1.txt log2.txt
```
- 移动当前文件夹下的所有文件到上一级目录
```shell
$ mv * ../
```


cp：复制

将源文件复制至目标文件，或将多个源文件复制至目标目录。

注意：命令行复制，如果目标文件已经存在会提示是否覆盖，而在 shell 脚本中，如果不加 \-i 参数，则不会提示，而是直接覆盖！
```shell
-i 提示
-r 复制目录及目录内所有项目
-a 复制的文件与原文件时间一样
```
实例：

1）复制 a.txt 到 test 目录下，保持原文件时间，如果原文件存在提示是否覆盖
```shell
$ cp -ai a.txt test
```
2）为 a.txt 建立一个链接（快捷方式）
```shell
$ cp -s a.txt link_a.txt
```

sudo chmod -R 777 目录名

-R     是指级联应用到目录里的所有子目录和文件

777   是所有用户都拥有最高权限（可自定权限码）


# 关键操作记录
1.如何找到一个占用某个端口的后台进程并关闭这个进程？

使用`netstat -tunlp`查看端口被占用的情况，可以看到占用端口的进程的PID

<img src="https://github.com/ChenMingK/ImagesStore/blob/master/imgs/f87248c4099f7e6b529b739075336ba3_1014x417.png" />

然后使用`kill pid`命令杀死这个进程即可

`netstat -tunlp|grep 端口号`这个命令可以直接找到占用对应端口的进程

*****
2.使用 curl 进行网络检测

curl 是一个发送请求的命令行工具。可使用 HTTP(s)、FTP，以及一些你可能从未听过的协议发送请求。它可以下载文件，检查响应头，自由地访问远程数据。

示例：

1）获取一个 URL 的 HTTP HEADER
```shell
$ curl -I http://google.com
HTTP/1.1 302 Found
Cache-Control: private
Content-Type: text/html; charset=UTF-8
Referrer-Policy: no-referrer
Location: http://www.google.com/?gfe_rd=cr&ei=0fCKWe6HCZTd8AfCoIWYBQ
Content-Length: 258
Date: Wed, 09 Aug 2017 11:24:01 GMT
```
2）向远程API发出GET请求
```shell
$ curl http://numbersapi.com/random/trivia
```
*****
3.使用 tree 展示文件目录结构

[每个 web 开发者都应该了解的 12 个命令行](https://blog.csdn.net/powertoolsteam/article/details/90764150#comments)
