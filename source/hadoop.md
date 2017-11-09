#Hadoop Command Note
## Hadoop path
``` sh
/usr/hdp
```
## spark
``` sh
spark  /usr/hdp/2.4.3.0-277/spark/bin
spark history server  slave1.kevin.com:18080
```

## storm
``` sh
/usr/hdp/current/storm-client
http://storm.apache.org/releases/0.10.0/Setting-up-a-Storm-cluster.html
```
**start storm**

1. Nimbus: Run the command "bin/storm nimbus" under supervision on the master machine.

2. Supervisor: Run the command "bin/storm supervisor" under supervision on each worker machine. The supervisor daemon is responsible for starting and stopping worker processes on that machine.

3. UI: Run the Storm UI (a site you can access from the browser that gives diagnostics on the cluster and topologies) by running the command "bin/storm ui" under supervision. The UI can be accessed by navigating your web browser to http://{ui host}:8080.


## running spark on yarn-cluster
``` sh
HADOOP_USER_NAME=hdfs spark-submit --class jie.com.myspark2.mySparkExample --master yarn --deploy-mode cluster --driver-memory 512m --executor-memory 512m --executor-cores 1 --proxy-user dfs --queue thequeue /data/hadoop/myspark2-0.0.1-SNAPSHOT.jar

sudo -u hdfs spark-submit --class com.cloudera.sparkwordcount.JavaWordCount --master local target/sparkwordcount-0.0.1-SNAPSHOT.jar /user/cloudera/data/inputfile.txt 2

./spark-submit --class org.apache.spark.examples.SparkPi --master yarn --deploy-mode cluster --driver-memory 512m --executor-memory 512m --executor-cores 1 --queue default /home/bigdata/spark/examples/jars/spark-examples_2.11-2.1.1.jar

./bin/spark-submit --class org.apache.spark.examples.SparkPi --master yarn     --deploy-mode cluster --driver-memory 2g --executor-memory 1g --executor-cores 1 --queue default examples/jars/spark-examples_2.11-2.1.1.2.6.1.0-129.jar 10
```
## remote
``` sh
HADOOP_USER_NAME=hadoop ./bin/spark-submit --class org.apache.spark.examples.SparkPi --master spark://47.93.144.37:6066 --deploy-mode cluster --driver-memory 4g --executor-memory 2g --executor-cores 1 --queue default /home/bigdata/spark/examples/jars/spark-examples_2.11-2.1.1.jar 10
```

## running python_spark on yarn-cluster
``` sh
HADOOP_USER_NAME=hdfs spark-submit --master yarn --deploy-mode cluster --driver-memory 512m --executor-memory 512m --executor-cores 1 --proxy-user hdfs python_spark_test.py

./bin/spark-submit --class org.apache.spark.examples.SparkPi --master yarn --deploy-mode cluster --driver-memory 4g --executor-memory 2g --executor-cores 1 --queue default     examples/src/main/python/pi.py 10

./bin/spark-submit --class org.apache.spark.examples.SparkPi --master spark://47.93.144.37:7077 --deploy-mode cluster --driver-memory 4g --executor-memory 2g --executor-cores 1 --queue default     examples/src/main/python/pi.py 10

./bin/spark-submit --class org.apache.spark.examples.SparkPi --master spark://192.168.31.101:7077 --deploy-mode cluster --driver-memory 500mb --executor-memory 500mb --executor-cores 1 --queue default examples/src/main/python/pi.py 10

./bin/spark-submit --master yarn --deploy-mode cluster --driver-memory 4g --executor-memory 2g --executor-cores 1 --queue default     examples/src/main/python/pi.py 10

./bin/spark-submit --master yarn --deploy-mode cluster --driver-memory 500mb --executor-memory 500mb --executor-cores 1 --queue default examples/src/main/python/pi.py 10
```
## python zip
``` sh
HADOOP_USER_NAME=hdfs ./bin/spark-submit --master yarn --deploy-mode cluster --driver-memory 2g --executor-memory 1g --executor-cores 1 --queue default --py-files /root/import.zip /root/import.py --verbose
```
