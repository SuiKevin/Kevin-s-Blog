# 关于`SparkLivy`和`ZeppelinNotebook`访问`HDFS`时的多用户权限控制

`SparkLivy`在`Yarn`上运行应用时，默认使用的用户是`Livy`。

`ZeppelinNotebook`在使用`%Spark`等非`%Livy`的Interpreter时，使用的用户是`Zeppelin`。

`Livy`和`Zeppelin`用户默认在HDFS上的用户组是`hdfs`，默认具有`HDFS`上所有目录及文件的`r-x`权限。

这样就会存在以下问题。

## 问题一：多用户

### 描述

所有用户在使用`SparkLivy`或者使用`Zeppelin`时都使用的是同一个用户（`Livy`或`Zeppelin`），
这样就无法控制用户应用（代码级别）对HDFS的访问权限，所有用户都使用的是`Livy`或`Zeppelin`的用户权限。

### 解决方案

**SparkLivy**

在创建`batch`或者`session`的时候添加`proxyUser`参数，指定当前用户。
详见：[SparkLivy REST API](https://livy.incubator.apache.org/docs/latest/rest-api.html)

**ZeppelinNotebook**

>关于ZeppelinNotebook自身的多用户配置，请阅读本人的另一篇文章[配置Zeppelin使用Apache Shiro进行鉴权（多用户登录）](http://www.kevinsui.com/posts/ambari_zeppelin_shiro/)。

當把interpreter設定為isolated的時候某方面的確達到了多使用者共同使用Zeppelin的環境
([详细Zeppelin配置](https://github.com/apache/zeppelin/blob/5e0aacf8a8f187702452d7cd2ee83b26c56dec90/docs/manual/userimpersonation.md))，
但是那只限於Zeppelin這個平台本身，後來遇到了Zeppelin所有使用者在YARN裏面會用相同user啟動的狀況，
這樣一來對於使用者的追蹤變的困難。

後來找了一下Zeppelin的討論目前對於使用者權限區隔的問題，Zeppelin採用Livy這個Spark Proxy當作解決方案，
再搭配Zeppelin內建的Livy Interpreter，能夠對Zeppelin的使用者做到區隔的動作。
(转自：[Zeppelin 支援多用戶-Livy Server篇](http://terrence.logdown.com/posts/1172854-zeppelin-livy-server-supports-multiple-users-review))

>目前仍存在一个问题，在作者`qwemicheal`的文章
[Zeppelin 和livy结合实现代理用户中如何代理ldap邮箱用户](http://blog.csdn.net/qwemicheal/article/details/70306104)
中提到`“使用zeppelin+livy的方式实现用户代理和后台权限控制．优点在于livy支持cluster模式
并seesion 客户端一段时间(默认６０分钟)不活动会自动中止spark任务”`。但是当多用户同时使用时，仍然会出现Spark集群无资源可用的情况。
那么就需要将session客户端的终止时间缩短。本人经过多次试验终究没有设置成功，最终在Web项目中添加轮询线程，调用SparkLivy的API结束掉
状态为Idle的session，以释放Spark集群资源。

### 总结

经过以上方法，解决了多用户的用户隔离问题，以及Zeppelin资源占用问题。但轮询线程仍然会消耗系统资源，未能找到最优解决方案。
希望有想法的道友能指点一二。

## 问题二：HDFS用户权限

问题一结果了多用户的问题，使Web项目的用户可以和Hadoop生态关联，但仍未控制用户在Hadoop平台上的权限（**本文仅介绍关于HDFS的权限控制**）。

### 解决方案

**启用HDFS的ACLs**

在hdfs-site.xml文件中添加配置项，以在HDFS上启用ACL：

```
<property>
  <name>dfs.namenode.acls.enabled</name>
  <value>true</value>
</property>
```

将此属性设置为“true”以启用对ACL的支持。ACL默认是禁用的。当禁用ACL时，NameNode拒绝所有对ACL的设置。

**使用CLI命令创建和列出ACLs**

两个新的子命令被添加到FsShell：`setfacl`和`getfacl`。这些命令是在相同的Linux shell命令之后建模的，但实现的标志较少。如果需要，可以稍后添加对其他标志的支持。

* **setfacl**

设置文件和目录的ACL。

例：

-setfacl [-bkR] {-m | -x} <acl_spec> \<path>

-setfacl --set <acl_spec> <path>

选项：

**表6.1. ACL选项**
|选项|描述|
|---|---|
|-b|删除所有条目，但保留基本ACL条目。“用户”，“组”和“其他”的条目保留为与“权限位”兼容。|
|-k|删除默认ACL。|
|-R|将操作递归地应用于所有文件和目录。|
|-m|修改ACL。新的条目被添加到ACL中，并保留现有的条目。|
|-x|删除指定的ACL条目。所有其他ACL条目都保留。|
|--set|完全替换ACL并放弃所有现有的条目。acl_spec必须包含User，Group和Others的条目，以便与Permission Bits兼容。|
|<acl_spec>|逗号分隔的ACL条目列表。|
|\<path>|要修改的文件或目录的路径。|

例子：

```
hdfs dfs -setfacl -m user:hadoop:rw- /file
hdfs dfs -setfacl -x user:hadoop /file
hdfs dfs -setfacl -b /file
hdfs dfs -setfacl -k /dir
hdfs dfs -setfacl --set user::rw-,user:hadoop:rw-,group::r--,other::r-- /file
hdfs dfs -setfacl -R -m user:hadoop:r-x /dir
hdfs dfs -setfacl -m default:user:hadoop:r-x /dir
```

**退出代码：**

成功返回0，错误返回非零。

* **getfacl**

显示文件和目录的ACL。如果某个目录具有默认ACL，则getfacl还会显示默认ACL。

用法：

```
 -getfacl [-R] <path>
```

选项：
​
**表6.2. getfacl选项**

|选项|描述|
|----|----|
|-R|以递归方式列出所有文件和目录的ACL。|
|\<path>|要列出的文件或目录的路径。|

例子：

```
hdfs dfs -getfacl /file
hdfs dfs -getfacl -R /dir
```

**退出代码：**

成功返回0，错误返回非零。

>**为每个用户指定需要访问的目录或者文件，指定文件的ACL时，需要指定所有父级目录的ACL！！！**
