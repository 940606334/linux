##linux自启动jar文件

#1.第一步

编写shell脚本,如hello.sh

	#!/bin/bash
	# program
	# test java open

	export JAVA_HOME=/usr/java/jdk1.8.0_112
	export JRE=/usr/java/jdk1.8.0_112/jre
	export CLASSPATH=$JAVA_HOME/lib:$JRE/lib:.
	export PATH=$PATH:$JAVA_HOME/bin/:$JRE/bin

	nohup java -jar /root/java/hello.jar >hello.log &

这就是执行jar包，运行一下，执行这个sh文件，没有问题：可以通过： 
ps -ef|grep java查询一下进程。

#2.第二步
输入命令`vi /etc/rc.local` 在最后一行加上那个脚本的绝对路径.