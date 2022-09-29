# Hadoop3.3.1 Pseudo Distributed Mode on Ubuntu 18.04

The operating system in this tutorial is Ubuntu18.04.6. The steps to install Ubuntu18.04 is omitted.

## 1. Create user hadoop

Open the terminal and type in command below to create new user:

```bash
sudo useradd -m hadoop -s /bin/bash
```

This command creates a log-in user hadoop and uses /bin/bash as shell.

Set up password for user hadoop:

```bash
sudo passwd hadoop
```

Give sudo permission to user hadoop:

```bash
sudo adduser hadoop sudo
```
**ATTENTION!!!**

Switch Linux login user (via Ubuntu UI) to **hadoop** to process steps below.

upgrade apt

```bash
sudo apt-get update
```

install vim

```bash
sudo apt-get install vim
```

install ssh, set up ssh none-key login

```bash
sudo apt-get install openssh-server
```

login localhost

```bash
ssh localhost
```

exit localhost

```bash
exit
```

authorize the key

```bash
cd ~/.ssh/
ssh-keygen -t rsa
cat ./id_rsa.pub >> ./authorized_keys
```

## 2. Install Java

```bash
sudo apt-get install openjdk-8-jre openjdk-8-jdk
```

change environment variables

```bash
cd ~
vim ~/.bashrc
```

add details below to it

```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

save changes, exit vim and refresh .bashrc

```bash
source ~/.bashrc
```

check java installing successfully

```bash
java -version
```

If details are shown on the screen, the installation is successful.

## 3. Install Hadoop3.3.1

change directory

```bash
cd /usr/local
```

download hadoop3.3.1

```bash
sudo wget http://dlcdn.apache.org/hadoop/common/hadoop-3.3.1/hadoop-3.3.1.tar.gz
```

unzip and change filename, ownership

```bash
tar xzf hadoop-3.3.1.tar.gz
mv hadoop-3.3.1 hadoop
sudo chown -R hadoop:hadoop ./hadoop
```

## 4. Configure pseudo distributed mode

```bash
cd ~
vim ~/.bashrc
```

add details below to .bashrc

```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:${HADOOP_HOME}/bin
export PATH=$PATH:${HADOOP_HOME}/sbin
export HADOOP_MAPRED_HOME=${HADOOP_HOME}
export HADOOP_COMMON_HOME=${HADOOP_HOME}
export YARN_HOME=${HADOOP_HOME}
```

refresh to activate new environment variables

```bash
source ~/.bashrc
```

Configure several files in path :/usr/local/hadoop/etc/hadoop/

- hadoop-env.sh

```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

- core-site.xml

```bash
<configuration>
	<property>
    	<name>hadoop.tmp.dir</name>
    <value>file:/usr/local/hadoop/tmp</value>
    	<description>Abase for other temporary directories.</description>
	</property>
	<property>
    	<name>fs.defaultFS</name>
    	<value>hdfs://localhost:9000</value>
	</property>
</configuration>
```

- hdfs-site.xml

```bash
<configuration>
	<property>
    	<name>dfs.replication</name>
    	<value>1</value>
	</property>
	<property>
    	<name>dfs.namenode.name.dir</name>
    	<value>file:/usr/local/hadoop/tmp/dfs/name</value>
	</property>
	<property>
    	<name>dfs.datanode.data.dir</name>
    	<value>file:/usr/local/hadoop/tmp/dfs/data</value>
	</property>
</configuration>
```

- mapred-site.xml

```bash
<configuration>
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
	<property>
		<name>yarn.app.mapreduce.am.env</name>
	  <value>HADOOP_MAPRED_HOME=/usr/local/hadoop</value>
	</property>
	<property>
	  <name>mapreduce.map.env</name>
	  <value>HADOOP_MAPRED_HOME=/usr/local/hadoop</value>
	</property>
	<property>
	  <name>mapreduce.reduce.env</name>
	  <value>HADOOP_MAPRED_HOME=/usr/local/hadoop</value>
	</property>
</configuration>
```

- yarn-site.xml

```bash
<configuration>
<property>
	<name>yarn.nodemanager.aux-services</name>
	<value>mapreduce_shuffle</value>
</property>
<property>
	<name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
	<value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>
</configuration>
```

format NameNode:

```bash
hdfs namenode -format
```

start hdfs

```bash
start-dfs.sh
```

start yarn

```bash
start-yarn.sh
```

to certify that dfs and yarn started successfully:

```bash
jps
```

if it looks like below (all parts should show up), previous steps are successful

```bash
$jps
2961 ResourceManager
2482 DataNode
3077 NodeManager
2366 NameNode
2686 SecondaryNameNode
3199 Jps
```
if namenode doesn't show up, you need tou format the namenode. Please make sure there is no important files in HDFS, beacause everything will be cleared.
```
cd /usr/local/hadoop
./sbin/stop-dfs.sh   # stop HDFS
rm -r ./tmp     # note: this will clear all files in HDFS
./bin/hdfs namenode -format # format namenode  
./sbin/start-dfs.sh  # restart
```
## 5. Job Test

try to run an example-jar file

```bash
cd usr/local/hadoop/share/hadoop/mapreduce/
hadoop jar ./hadoop-mapreduce-examples-3.3.1.jar pi 5 10
```

open browser to see web UI pages

urls:

```bash
localhost:9870
```

```bash
localhost:8088
```
You can also access the web page on other PC. First, find the IP address of the ubuntu.
```bash
ifconfig -a
```
The information showed up gives ip address. You can access the web page on other PC's browsers.
```bash
${IP ADDRESS}:9870
${IP ADDRESS}:8088
```
>note: the job history port is 19888

## 6. Shut down

use command separately :

```bash
stop-dfs.sh
stop-yarn.sh
```

Or stop them together:

```bash
stop-all.sh
```
