## Installing Spark stanalone
### OS version
These steps were reproduced on: CentOS 7.7
### Before You Install
Make sure that you have passwordlees ssh access to localhost (it is also Spark host)
### Disabling the Firewall
Disable firewall for ports 8080,8081. 
Optionally 4040+ (4041,4042,...) ports for spark App Web UI.
### Required Privileges
The installation is under regular user, but sudo needed for 'yum install'
### Updare repo
```sh
$ sudo yum update -y
```
### Install wget
```sh
$ sudo yum install wget -y
```
### Edit hosts file 
Add a record to /etc/hosts file to be able to address Spark Host by hostname.
Attetntion! Avoid hostnames with [underscores](https://stackoverflow.com/questions/2180465/can-domain-name-subdomains-have-an-underscore-in-it).
### Install JDK
Install Java, set env virables and renew env virables for current session
```sh
$ sudo yum install java-1.8.0-openjdk-1.8.0.232.b09-0.el7_7.x86_64 -y
$ echo 'export JAVA_HOME=/usr/lib/jvm/jre-1.8.0-openjdk' >> ~/.bash_profile
$ echo 'export JRE_HOME=/usr/lib/jvm/jre' >> ~/.bash_profile
$ source ~/.bash_profile
```
### Install Spark
Start executing commands being at home directory of non-root user
```sh
$ wget http://apache-mirror.rbc.ru/pub/apache/spark/spark-2.4.4/spark-2.4.4-bin-hadoop2.7.tgz
$ tar -xvzf spark-2.4.4-bin-hadoop2.7.tgz 
$ sudo mv spark-2.4.4-bin-hadoop2.7 /usr/local/spark
```
### Set Spark environment
Set SPARK_HOME and add path to Spark executables to PATH.
```sh
$ echo "export SPARK_HOME=/usr/local/spark" >> ~/.bash_profile
$ echo "export PATH=$PATH:/usr/local/spark/bin" >> ~/.bash_profile
$ source ~/.bash_profile
```
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
export SPARK_MASTER_HOST='Spark Host IP'
export JAVA_HOME=/usr/lib/jvm/jre-1.8.0-openjdk
```
### Add Workers
Edit the configuration file slaves in (/usr/local/spark/conf).
```sh
sudo vim slaves
```
And add the following entries.
```sh
<Spark Host name>
```
### Start Spark
To start the spark cluster, run the following command on master.
```sh
cd /usr/local/spark
./sbin/start-all.sh
```
### Spark Web UI
Browse the Spark UI to know about worker nodes, running application, cluster resources.
Spark Master UI
```sh
http://<Spark host>:8080/
```
