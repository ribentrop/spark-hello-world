
# Installing a three node Spark cluster
## Before You Install
### Updare repo
```sh
sudo yum update -y
```
### Install wget
```sh
sudo yum install wget -y
```
### System prereq
### Ports
### Disabling the Firewall
### Required Privileges
### Edit hosts file  (master only)
```sh
sudo vi /etc/hosts
<MASTER-IP> master
<SLAVE01-IP> slave01
<SLAVE02-IP> slave02
```
### Configure passwordless SSH access  (master only)
### Install JDK (master and slaves)
Install Java, set env virables and renew env virables for current session
```sh
sudo yum install java-1.8.0-openjdk-1.8.0.232.b09-0.el7_7.x86_64 -y
sudo cp /etc/profile /etc/profile_backup
echo 'export JAVA_HOME=/usr/lib/jvm/jre-1.8.0-openjdk' | sudo tee -a /etc/profile
echo 'export JRE_HOME=/usr/lib/jvm/jre' | sudo tee -a /etc/profile
source /etc/profile
```
## Install Scala (master and slaves) - optional
## Install Spark
### Install Spark (master and slaves)
Start executing commands being at home dir of non-root user
```sh
wget http://apache-mirror.rbc.ru/pub/apache/spark/spark-2.4.4/spark-2.4.4-bin-hadoop2.7.tgz
tar -xvzf spark-2.4.4-bin-hadoop2.7.tgz 
sudo mv spark-2.4.4-bin-hadoop2.7 /usr/local/spark
```
### Set environment (master only)
Set SPARK_HOME
```sh
echo "export SPARK_HOME=/usr/local/spark" >> ~/.bash_profile
echo "export PATH=$PATH:/usr/local/spark/bin" >> ~/.bash_profile
Edit spark-env.sh
```
source ~/.bash_profile
Move to spark conf folder and create a copy of template of spark-env.sh and rename it.
```sh
$ cd /usr/local/spark/conf
$ cp spark-env.sh.template spark-env.sh
```
Now edit the configuration file spark-env.sh.
```sh
sudo vim spark-env.sh
```
And set the following parameters.
```sh
export SPARK_MASTER_HOST='<MASTER-IP>'
export JAVA_HOME=<Path_of_JAVA_installation>
```
### Add Workers (master only)
Edit the configuration file slaves in (/usr/local/spark/conf).
```sh
sudo vim slaves
```
And add the following entries.
```sh
master
slave01
slave02
```
### Start Spark Cluster
To start the spark cluster, run the following command on master.
```sh
cd /usr/local/spark
./sbin/start-all.sh
```
### Spark Web UI
Browse the Spark UI to know about worker nodes, running application, cluster resources.
Spark Master UI
```sh
http://<MASTER-IP>:8080/
```
