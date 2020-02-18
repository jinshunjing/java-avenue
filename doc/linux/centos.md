# centos

## 安装cmake 3.13.2
1. 下载安装包 wget https://cmake.org/files/v3.13/cmake-3.13.2.tar.gz
2. 解压 tar -xzvf cmake-3.13.2.tar.gz
3. cd cmake-3.13.2
4. ./bootstrap --prefix=/usr
5. gmake
6. make install

## 安装Oracle JDK
1. 浏览器里找到JDK下载页面：https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
2. 点击Accept License Agreement，然后复制下载地址
3. wget --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie;" https://download.oracle.com/otn-pub/java/jdk/8u201-b09/42970487e3af4f5aa5bca3f542482c60/jdk-8u201-linux-x64.rpm
4. rpm -ivh jdk-8u201-linux-x64.rpm
5. vi /etc/profile

export JAVA_HOME=/usr/java/jdk1.8.0_201-amd64
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin

