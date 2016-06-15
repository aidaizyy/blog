title: "Hadoop安装运行"
date: 2016-05-29 14:18:57
tags:
- hadoop
- java
categories: hadoop 
toc: true
---

Hadoop：是一个由Apache基金会所开发的处理计算机集群上大数据的开源分布式系统基础框架。
本文基于Mac OSX 10.11操作系统和Hadoop 2.6.4。

<!--more-->
**Title: [Hadoop安装运行](https://aidaizyy.github.io/hadoop)**
**Author: [Yunyao Zhang(张云尧)](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2016-05-29](http://aidaizyy.github.io)**

## 准备

### JAVA
Hadoop是由Java实现的，所以首先确入是否已经安装Java。
``` bash
java -version
```
在终端中输入以上命令，确认是否安装Java。如果没有，可以去[Java官方网站](http://www.java.com)下载，并参照说明配置环境变量。如果已经安装Java，可以进行下一步。
本文基于JDK 1.8安装Hadoop。官方要求至少java 1.5及其以上版本。

### SSH
其次，确认ssh安装，且sshd一直运行，以便Hadoop脚本管理远端Hadoop守护进程。
Mac系统已自带ssh，可通过以下三条命令验证。
``` bash
which ssh
which sshd
which ssh-Keygen
```
得到ssh路径。
我们要实现无密码登录ssh。
``` bash
ssh-keygen -t rsa
```
输入以上命令，需要输入密码时按enter键。
成功后进入ssh目录。
``` bash
cd ~/.ssh
cp id_rsa.pub authorized_keys
```
下面验证是否成功。
``` bash
ssh localhost
```
如果出现
``` bash
ssh: connect to host localhost port 22: Connection refuse
```
说明用户没有权限，需要去“系统偏好设置”——“共享”——“远程登录”，勾选并选择允许访问：“所有用户”。

注意，在Windows系统下，需要安装Cygwin，提供shell支持。

## 下载

### 下载Hadoop
前往[Apache Hadoop官方网站](http://hadoop.apache.org/releases.html)下载Hadoop发行版。本文下载了最新修改发布的2.6.4版本（2016年2月），选择[binary版本](http://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-2.6.4/hadoop-2.6.4.tar.gz)，如果选择了source版本，压缩包后会有-src后缀，表示源码，需要额外编译。

### 解开压缩包
``` bash 
tar -xvzf hadoop-2.6.4.tar.gz
```
我这里将其放置的路径是~/Applications（本文的路径是/Users/zhangyunyao/Applications/hadoop-2.6.4）。

### 设置环境变量
在~/.bash_profile文件或~/.profile文件中添加环境变量（本文在zsh环境下，在~/.zshrc中添加环境变量）。

**添加JAVA_HOME环境变量**
```
export JAVA_HOME=`/usr/libexec/java_home`
```
将上面句子加在环境变量文件末尾，并用source命令更新。
``` bash
source .zshrc
```
验证JAVA_HOME环境变量是否设置成功。
``` bash
echo $JAVA_HOME
```
如果返回Java路径则说明设置成功。

**添加HADOOP_HOME环境变量**
``` 
export HADOOP_HOME=/Users/zhangyunyao/Applications/hadoop-2.6.4
```
将上面句子加载环境变量文件末尾，同样可以用source命令更新和echo命令验证。

**添加其他可能用到的环境变量** 
``` 
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib
```

## 配置
进入hadoop-2.6.4目录后，主要配置四个文件（hadoop-env.sh，core-site.xml，mapred-site.xml，hdfs-site.xml），很多资料显示这四个文件在conf目录里。但在Hadoop 2.5以后，这四个文件都在etc/hadoop文件夹下。
``` bash
cd hadoop-2.6.4/etc/hadoop
```

### 编辑hadoop-env.sh文件
该文件涉及Hadoop的配置。
``` 
export JAVA_HOME=${JAVA_HOME}
```
因为我们在环境配置中已经设置了JAVA_HOME的值，所以这里可以不用修改。
```
#export HADOOP_HEADSIZE=
```
表示Hadoop堆可用的最大大小，默认是1000MB。没有特殊需求，可以不用修改。修改的话去掉注释符号“#”，并在“=”后面加上数字，比如“=2000”，表示2000MB。
```
export HADOOP_OPTS="$HADOOP_OPTS -Djava.net.preferIPv4Stack=true"
```
已经有默认值，暂时不需要修改。

### 编辑core-site.xml文件
参考官方提供的默认文件[core-default.xml](http://hadoop.apache.org/docs/r2.6.4/hadoop-project-dist/hadoop-common/core-default.xml)，其他版本都可以在官网找到对应的默认文件。
“hadoop.tmp.dir”表示临时目录，“fs.default.name”表示缺省的文件URI标识，这里设置了主机名和端口。
```
<configuration>
 	<property>
    		<name>hadoop.tmp.dir</name>
    		<value>/Users/zhangyunyao/Applications/hadoop-2.6.4/tmp/hadoop-${user.name}</value>
    		<description>A base for other temporary directories.</description>
  	</property>
  	<property>
    		<name>fs.default.name</name>
    		<value>hdfs://localhost:9000</value>
  	</property>
</configuration>
```

### 编辑hdfs-site.xml文件
参考官方提供的默认文件[hdfs-default.xml](http://hadoop.apache.org/docs/r2.6.4/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml)，其他版本都可以在官网找到对应的默认文件。
“dfs.replication”表示缺省的块复制数量，因为这里只有一个节点，所以值设为1。
```
<configuration>
    	<property>
        	<name>dfs.replication</name>
        	<value>1</value>
    	</property>
</configuration>
```

### 编辑mapred-site.xml文件
参考官方提供的默认文件[mapred-default.xml](http://hadoop.apache.org/docs/r2.6.4/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml)，其他版本都可以在官网找到对应的默认文件。
“mapreduce.jobtracker.address”表示JobTracker作业跟踪器的地址，这里设置了它的主机名和端口。
```
<configuration>
	<property>
		<name>mapreduce.jobtracker.address</name>
		<value>localhost:9001</value>
		<description>The address of JobTracker</description>
	</property>
</configuration>
```
注意hadoop-2.6.4/etc/hadoop目录下是没有mapred-site.xml文件的，只有mapred-site.xml.temple。我们需要把该文件复制一份，并命令为mapred-site.xml。
``` bash
cp mapred-site.xml.tmple mapred-site.xml
```

## 安装HDFS
``` bash
hadoop namenode -format
```
在Hadoop目录内输入以上命令。

## 运行
运行hadoop-2.6.4/sbin目录下的start-all.sh脚本。
``` bash
./start-all.sh
```
输入命令``jps``验证是否成功运行Hadoop。
在本机上出现如下信息：
``` 
30417 SecondaryNameNode
30610 Jps
30324 DataNode
30571 NodeManager
29660 ResourceManager
30253 NameNode
```
说明Hadoop已经成功运行。

运行自带的例子验证。
``` bash
hadoop jar $HADOOP_HOME/share/hadoop/madreduce/hadoop-mapreduce-examples-2.6.4.jar pi 10 100
```
出现以下信息：
```
Number of Maps  = 10
Samples per Map = 100
Wrote input for Map #0
Wrote input for Map #1
Wrote input for Map #2
Wrote input for Map #3
Wrote input for Map #4
Wrote input for Map #5
Wrote input for Map #6
Wrote input for Map #7
Wrote input for Map #8
Wrote input for Map #9
Starting Job
……
Job Finished in 1.749 seconds
Estimated value of Pi is 3.14800000000000000000
```
说明自带的例子hadoop-mapreduce-examples-2.6.4.jar已经成功运行，至此确认Hadoop已经成功安装。

