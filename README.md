spark-yarn-hadoop-cluster-vagrant
============================

# 1. Introduction
### Vagrant project to spin up a cluster of 4, 64-bit CentOS6.5 Linux virtual machines with Hadoop v2.6.0 and Spark v1.6.1.

Ideal for development cluster on a machine with at least 8GB of memory for the VM and tested on MacBook Pro with 16GB of RAM.

1. node1 : HDFS NameNode + Spark Master
2. node2 : YARN ResourceManager + JobHistoryServer + ProxyServer
3. node3 : HDFS DataNode + YARN NodeManager + Spark Slave
4. node4 : HDFS DataNode + YARN NodeManager + Spark Slave

# 2. Prerequisites 
1. At least 2GB memory for each VM node. Default script is for 4 nodes, so you need 8GB for the nodes, in addition to the memory for your host machine.
2. Vagrant 1.7 or higher, Virtualbox 4.3.2 or higher
3. Preserve the Unix/OSX end-of-line (EOL) characters while cloning this project; scripts will fail with Windows EOL characters.
4. Project is tested on Mac OSX 10.9 host OS on Macbook Pro with 16GB RAM
5. The Vagrant box is downloaded to the ~/.vagrant.d/boxes directory. 


# 3. Downlooads
1. [Download and install VirtualBox](https://www.virtualbox.org/wiki/Downloads)
2. [Download and install Vagrant](http://www.vagrantup.com/downloads.html).
3. Run ```vagrant box add centos65 https://github.com/2creatives/vagrant-centos/releases/download/v6.5.1/centos65-x86_64-20131205.box 
4. Git clone this project, and change directory (cd) into this project (directory).
5. [Download Hadoop 2.6 into the /resources directory ] 
6. [Download Spark 1.6.1 into the /resources directory]
7. [Download Java 1.8 into the /resources directory]


# 4. Modifying scripts for adapting to your environment
You need to modify the scripts to adapt the VM setup to your environment and the version of the downloads  

1. Vagrant File 

./Vagrantfile  

- To add/remove slaves, change the number of nodes:  
line 5: ```numNodes = 4```  

- To modify VM memory change the following line:  
line 13: ```v.customize ["modifyvm", :id, "--memory", "1024"]```  

3. /scripts/common.sh  

- Java :Based on the version you have to change this as follows
```JAVA_ARCHIVE=jdk-8u91-linux-x64.tar.gz```

- Hadoop: To use a different version of Hadoop you've already downloaded to /resources directory, change the following line:  
```HADOOP_VERSION=hadoop-2.6.0```  
To use a different version of Hadoop to be downloaded, change the remote URL in the following line:  
```HADOOP_MIRROR_DOWNLOAD=http://apache.crihan.fr/dist/hadoop/common/stable/hadoop-2.6.0.tar.gz```  

- Spark: To use a different version of Spark, change the following lines:  
```SPARK_VERSION=spark-1.6.1```  
```SPARK_ARCHIVE=$SPARK_VERSION-bin-hadoop2.6.tgz```  
```SPARK_ARCHIVE_DIR=$SPARK_VERSION-bin-hadoop2.6
```SPARK_MIRROR_DOWNLOAD=../resources/spark-1.6.1-bin-hadoop2.6.tgz```  

3. /scripts/setup-java.sh  
To install from Java downloaded locally in /resources directory, if different from default version (jdk1.8.0_91), change the version in the following line:  
```ln -s /usr/local/jdk1.8.0_91 /usr/local/java```  
```yum install -y jdk-8u25-linux-i586```  

4. /scripts/setup-centos-ssh.sh  
To modify the version of sshpass to use, change the following lines within the function installSSHPass():  
```wget http://pkgs.repoforge.org/sshpass/sshpass-1.05-1.el6.rf.i686.rpm```  
```rpm -ivh sshpass-1.05-1.el6.rf.i686.rpm```  

5. /scripts/setup-spark.sh  
To modify the version of Spark to be used, if different from default version (built for Hadoop2.6), change the version suffix in the following line:  
```ln -s /usr/local/$SPARK_VERSION-bin-hadoop2.6 /usr/local/spark```  

# 3. Getting Started
- Run ```vagrant up``` to create the VM.
- Run ```vagrant ssh``` to get into your VM.
- Run ```vagrant halt``` to gracefully shutdown the VM's. You can later startup the cluster with ```vagrant up``` but have to repeat the #5 Post Provisioning again.
- Run ```vagrant destroy``` when you want to destroy and get rid of the VM to free up your disk space.

# 5. Post Provisioning
After you have provisioned the cluster, you need to run some commands to initialize your Hadoop cluster. 

- SSH into node1 using  ```vagrant ssh node-1```

Commands below require root permissions. 

- Change to root access using ```sudo su``` (or create a new user and grant permissions if you want to use a non-root access. In such a case, you'll need to do this on VMs)

Issue the following command. 

1. $HADOOP_PREFIX/bin/hdfs namenode -format myhadoop

## Start Hadoop Daemons (HDFS + YARN)
SSH into node1 and issue the following commands to start HDFS.

1. $HADOOP_PREFIX/sbin/hadoop-daemon.sh --config $HADOOP_CONF_DIR --script hdfs start namenode
2. $HADOOP_PREFIX/sbin/hadoop-daemons.sh --config $HADOOP_CONF_DIR --script hdfs start datanode

SSH into node2 and issue the following commands to start YARN.

1. $HADOOP_YARN_HOME/sbin/yarn-daemon.sh --config $HADOOP_CONF_DIR start resourcemanager
2. $HADOOP_YARN_HOME/sbin/yarn-daemons.sh --config $HADOOP_CONF_DIR start nodemanager
3. $HADOOP_YARN_HOME/sbin/yarn-daemon.sh start proxyserver --config $HADOOP_CONF_DIR
4. $HADOOP_PREFIX/sbin/mr-jobhistory-daemon.sh start historyserver --config $HADOOP_CONF_DIR

### Test YARN
Run the following command to make sure you can run a MapReduce job.

```
yarn jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0.jar pi 2 100
```

## Start Spark in Standalone Mode
SSH into node1 and issue the following command.

1. $SPARK_HOME/sbin/start-all.sh

### Test Spark on YARN
You can test if Spark can run on YARN by issuing the following command. Try NOT to run this command on the slave nodes.
```
$SPARK_HOME/bin/spark-submit --class org.apache.spark.examples.SparkPi \
    --master yarn-cluster \
    --num-executors 10 \
    --executor-cores 2 \
    $SPARK_HOME/lib/spark-examples*.jar \
    100
```
	
### Test Spark using Shell
Start the Spark shell using the following command. Try NOT to run this command on the slave nodes.

```
$SPARK_HOME/bin/spark-shell --master spark://node1:7077
```

Then go here https://spark.apache.org/docs/latest/quick-start.html to start the tutorial. You will have to load data into HDFS to make the tutorial work.

# 6. Web UI
You can check the following URLs to monitor the Hadoop daemons.

1. [NameNode] (http://10.211.55.101:50070/dfshealth.html)
2. [ResourceManager] (http://10.211.55.102:8088/cluster)
3. [JobHistory] (http://10.211.55.102:19888/jobhistory)
4. [Spark] (http://10.211.55.101:8080)

# 7. References
This project was put together with great pointers from all around the internet. All references made inside the files themselves.
Primaily this project is forked from [dnafrance's vagrant project](https://github.com/dnafrance/vagrant-hadoop-spark-cluster)

# 8. Copyright Stuff
Copyright 2016 IntelliSignals.com

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
