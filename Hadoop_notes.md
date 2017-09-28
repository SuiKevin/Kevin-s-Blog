# Hadoop Notes

## Note1
### 问题描述
Yarn不能同时运行多个application

### 问题分析
经观察[ResourceManager UI](http://master:8088/cluster)

<div align=center><img src="http://www.kevinsui.com/static/upload/yarn scheduler metrics.png" width="800px"/></div>

与执行命令

> HADOOP_USER_NAME=hdfs ./bin/spark-submit --master yarn --deploy-mode cluster `--driver-memory 2g --executor-memory 2g` --executor-cores 1 --queue default examples/src/main/python/pi.py 10

发现单个application申请的资源（内存、CPU）超过单个node上最大资源一半，导致没有足够资源供其余application运行，所以多个application同时运行时，后提交的application会等待有足够的资源时再运行。

### 解决方法
在Ambari中设置YARN的Memory和CPU
<div align=center><img src="http://www.kevinsui.com/static/upload/yarn memory cpu.png" width="800px"/></div>

调整执行命令中的内存大小，使之不超过YARN配置中设置的大小，合理安排内存，`Memory for Node / Application Memory = Num of Application`
