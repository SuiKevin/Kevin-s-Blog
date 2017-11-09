### Pandora学习笔记

---

#### 架构图

<div align=center><img src="http://www.kevinsui.com/static/upload/arch.png" width="800px"/></div>

---

#### 控制台操作

> **实时计算工作流：**

**创建数据源**

在工作流编辑器中，我们首先看到的节点是数据源，这个节点用来接收用户的实时数据，也就是说，当这个节点被创建后，**用户需要将自己的数据推送至这个数据源中**，才可以继续进行下一步。

<img src="https://qiniu.github.io/pandora-docs/_media/flow1.gif" width="800px"/>

1. 定义字段及类型

2. 类型：

	* string

	* float

    * long

    * date

    * boolean

    * array[string/long/float]

    * map

    * jsonstring

**创建计算任务**

<img src="https://qiniu.github.io/pandora-docs/_media/flow2.gif" width="800px"/>

计算的方式分为两种：标准SQL计算和自定义代码计算，它们两者可以并存，执行的顺序是先执行自定义代码计算，后执行标准SQL计算；在一个计算任务中，至少需要指定一种计算方式。

1. 使用UDF

    UDF是在SQL计算中使用的方法

    * 系统UDF

        <img src="https://qiniu.github.io/pandora-docs/_media/flow11.gif" width="800px"/>

        系统默认提供了上百种udf，分别为：数学函数、日期函数、字符串函数、聚合函数和窗口函数，我们可以在工作流列表的右上角 UDF管理 中查看。

	* 自定义UDF

        <img src="https://qiniu.github.io/pandora-docs/_media/flow12.gif" width="800px"/>

        <img src="https://qiniu.github.io/pandora-docs/_media/flow13.gif" width="800px"/>

        下载 UDF-java 项目工程，在 src/main/java/com.pandora/ 目录下新建Class和方法，并在方法中编写udf逻辑，代码编写完成后，需要将这个工程打成Jar包并上传至Pandora，然后就可以注册并使用这个udf了。

2. 自定义计算（Plugin）- Java

    * 下载 -> Plugin-Java.zip

        <img src="https://qiniu.github.io/pandora-docs/_media/flow3.gif" width="800px"/>

    * 编写输入&输出类

    * 编写业务逻辑代码

    * 打包上传

        <img src="https://qiniu.github.io/pandora-docs/_media/flow4.gif" width="800px"/>

**数据导出**

  * 导出数据至HTTP

      <img src="https://qiniu.github.io/pandora-docs/_media/flow5.gif" width="800px"/>

  * 导出数据至对象存储

      <img src="https://qiniu.github.io/pandora-docs/_media/flow6.gif" width="800px"/>

  * 导出数据至日志检索服务

      <img src="https://qiniu.github.io/pandora-docs/_media/flow7.gif" width="800px"/>

  * 导出数据至时序数据库

      <img src="https://qiniu.github.io/pandora-docs/_media/flow8.gif" width="800px"/>

> **离线计算工作流：**

在工作流编辑器中，我们首先看到的节点是数据源，这个节点用来指定用户数据的位置，**用户必须指定一个正确的数据所在地**，并且成功加载后，才可以继续进行下一步。

**数据计算**

计算方式目前仅支持**SQL**。

<img src="https://qiniu.github.io/pandora-docs/_media/batch2.gif" width="800px"/>

**数据导出**

目前仅支持将数据导出到**对象存储服务**当中。

<img src="https://qiniu.github.io/pandora-docs/_media/batch3.gif" width="800px"/>

---

**七牛云官方教程：** https://qiniu.github.io/pandora-docs/#/?id=pandora
