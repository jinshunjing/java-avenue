# Linux Command

## 设置环境变量
vi /etc/profile <br/>
PATH=/usr/local/bin:$PATH <br/>
export PATH <br/>
LIBRARY_PATH=/lib64:$LIBRARY_PATH <br/>
export LIBRARY_PATH
source /etc/profile

## find
find . -iname "key"

## awk
先按行读取文件，然后按域分隔符划分单词

### 指定域分隔符
awk -F ':' 'BEGIN {print "user,app"} {print $1","$5} END {print "end,end"}'

### 匹配关键字来过滤行
awk -F ':' '/root/ {print $1}' /etc/passwd

### 使用Hash
awk -F ':' '{users[$1]=$5} END {for (user in users) {print user","users[user]}}' /etc/passwd

### 批量修改文件后缀
ls | awk -F "." '{print "mv "$0" "$1".txt"}' | sh

## tmux
列表：tmux ls <br/>
进入：tmux at -t xxx <br/>
临时退出：ctr+b d

## ssh
### 文件的下载与上传
scp user@0.0.0.0:/usr/dir/file local <br/>
scp local user@0.0.0.0:/usr/dir/

rsync user@0.0.0.0:/usr/dir/file local <br/>
rsync local user@0.0.0.0:/usr/dir/

## 内核态用户态
* 内核: CPU/进程, RAM/内存，Disk/文件，Network/IO，设备驱动
* 系统调用: c语言实现
* 库函数：封装了系统调用，glibc, syscall
* shell脚本
* Intel X86 CPU 提供了0-3共4个特权等级，0级表示内核态，3级表示用户态
* 有些资源只有在内核态才能被访问

### 从用户态切换到内核态
* 系统调用：0x80中断进入系统调用（软中断）
* 异常中断
* 外围设备的中断


## netstat
### Server Socket
netstat -ltnp

### Socket
netstat -tnp

查询某个端口
netstat -tna | grep {port}
统计各中状态的数量
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a,S[a]}'


## 查看线程数量
查看进程里所有的线程
ps -o nlwp {pid}
查看运行中的线程
ps -eLo pid,stat | grep {pid} | grep running | wc -l





