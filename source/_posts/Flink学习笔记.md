

# Flink学习笔记



## 运行时架构

### 1. flink运行时的组件

![img](https://img1.baidu.com/it/u=1363017092,178586132&fm=15&fmt=auto&gp=0.jpg)

#### 作业管理器（JobManager）

- 控制一个应用程序执行的主进程，也就是说，每个应用程序都会被一个不同的JobManager所控制执行
- JobManager会现接收到要执行的应用程序，这个应用程序会包括：作业图（JobGraph）、逻辑数据流图（logical dataflow graph）和打包了所有的类、库和其他资源的jar包
- JobManager会把JobGraph转换成一个物理层面的数据流图，这个图被叫做“执行图”（ExecutionGraph），包含了所有可以并发执行的任务。
- JobManager会向资源管理器（ResourceManager）请求执行任务必要的资源，也就是任务管理器（TaskManager）上面的插槽（slot）。一旦它获取了足够的资源，就会将执行图分发到真正运行它们的TaskManager上。而在运行过程中，JobManager会负责所有需要中央协调的操作，比如说检查点（checkpoint）的协调。



#### 任务管理器（TaskManager）

- flink中的工作进程。通常在flink中会有多个TaskManager运行，每一个TaskManager都包含了一定数量的插槽（slot）。插槽的数量限制了TaskManager能够并发执行的任务数量。
- 启动子后，TaskManager会向资源管理器注册它的插槽；收到资源管理器的指令后，TaskManager就会将一个或多个插槽提供给JobManager调用。JobManager就可以向slot分配任务（tasks）来执行了。
- 在执行过程中，一个TaskManager可以跟其他运行同一应用程序的TaskManager交换数据。

#### 资源管理器（ResourceManager）

- 主要负责管理任务管理器（TaskManager）的插槽（slot），TaskManager插槽是Flink中定义的处理资源单元。
- Flink为不同的环境和资源管理工具提供了不同资源管理器，比如yarn、Mesos、k8s以及standalone部署。
- 当JobManager申请插槽资源时，ResourceManager会将有空闲插槽的TaskManager分配给JobManager。如果ResourceManager没有足够的插槽来满足JobManager的请求，它还可以向资源提供平台发起会话，以提供启动TaskManager进程的容器。

#### 分发器（Dispatcher）

- 可以跨作业运行，它为应用提交提供了REST接口
- 当一个应用被提交执行时，分发器就会启动并将应用移交给一个JobManager。
- Dispatcher也会启动一个web UI，用来方便地展示和监控作业执行的信息。
- Dispatcher在架构中可能并不是必需的，这取决于应用提交执行的方式。

### 2. 任务提交流程

#### 总览

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg-blog.csdnimg.cn%2F20200524212126844.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMTgwMjI5%2Csize_16%2Ccolor_FFFFFF%2Ct_70&refer=http%3A%2F%2Fimg-blog.csdnimg.cn&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1629178209&t=dba3c336b51674fdb35f9f86eccfadd7)

#### 任务提交流程（Yarn）

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fwww.pianshen.com%2Fimages%2F597%2F9112d7f39d3f178daa84fda2f75a9815.png&refer=http%3A%2F%2Fwww.pianshen.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1629178307&t=b26d75e5a47e8ae65ce87280ce6d4bf0)







### 3. 任务调度原理

![The processes involved in executing a Flink dataflow](https://ci.apache.org/projects/flink/flink-docs-release-1.13/fig/processes.svg)

Flink 运行时由两种类型的进程组成：一个 *JobManager* 和一个或者多个 *TaskManager*。

*Client* 不是运行时和程序执行的一部分，而是用于准备数据流并将其发送给 JobManager。之后，客户端可以断开连接（*分离模式*），或保持连接来接收进程报告（*附加模式*）。客户端可以作为触发执行 Java/Scala 程序的一部分运行，也可以在命令行进程`./bin/flink run ...`中运行。

可以通过多种方式启动 JobManager 和 TaskManager：直接在机器上作为[standalone 集群](https://ci.apache.org/projects/flink/flink-docs-release-1.13/zh/docs/deployment/resource-providers/standalone/overview/)启动、在容器中启动、或者通过[YARN](https://ci.apache.org/projects/flink/flink-docs-release-1.13/zh/docs/deployment/resource-providers/yarn/)或[Mesos](https://ci.apache.org/projects/flink/flink-docs-release-1.13/zh/docs/deployment/resource-providers/mesos/)等资源框架管理并启动。TaskManager 连接到 JobManagers，宣布自己可用，并被分配工作。



### 4. 并行度（Parallelism）

![Operator chaining into Tasks](https://ci.apache.org/projects/flink/flink-docs-release-1.13/fig/tasks_chains.svg)

一个特定算子的子任务（subtask）的个数被称之为其其并行度（parallelism）

一般情况下，一个stream的并行度，可以认为就是其所有算子中最大的并行度



### 5. TaskManager和Slots

![A TaskManager with Task Slots and Tasks](https://ci.apache.org/projects/flink/flink-docs-release-1.13/fig/tasks_slots.svg)

每个 worker（TaskManager）都是一个 *JVM 进程*，可以在单独的线程中执行一个或多个 subtask。为了控制一个 TaskManager 中接受多少个 task，就有了所谓的 **task slots**（至少一个）。

每个 *task slot* 代表 TaskManager 中资源的固定子集。例如，具有 3 个 slot 的 TaskManager，会将其托管内存 1/3 用于每个 slot。分配资源意味着 subtask 不会与其他作业的 subtask 竞争托管内存，而是具有一定数量的保留托管内存。注意此处没有 CPU 隔离；当前 slot 仅分离 task 的托管内存。

通过调整 task slot 的数量，用户可以定义 subtask 如何互相隔离。每个 TaskManager 有一个 slot，这意味着每个 task 组都在单独的 JVM 中运行（例如，可以在单独的容器中启动）。具有多个 slot 意味着更多 subtask 共享同一 JVM。同一 JVM 中的 task 共享 TCP 连接（通过多路复用）和心跳信息。它们还可以共享数据集和数据结构，从而减少了每个 task 的开销。

![TaskManagers with shared Task Slots](https://ci.apache.org/projects/flink/flink-docs-release-1.13/fig/slot_sharing.svg)

默认情况下，Flink 允许 subtask 共享 slot，即便它们是不同的 task 的 subtask，只要是来自于同一作业即可。结果就是一个 slot 可以持有整个作业管道。允许 *slot 共享*有两个主要优点：

- Flink 集群所需的 task slot 和作业中使用的最大并行度恰好一样。无需计算程序总共包含多少个 task（具有不同并行度）。
- 容易获得更好的资源利用。如果没有 slot 共享，非密集 subtask（*source/map()*）将阻塞和密集型 subtask（*window*） 一样多的资源。通过 slot 共享，我们示例中的基本并行度从 2 增加到 6，可以充分利用分配的资源，同时确保繁重的 subtask 在 TaskManager 之间公平分配。
- Task Slot是静态的概念，是指TaskManager具有的并发执行能力



### 6. 并行子任务的分配

![img](https://img2.baidu.com/it/u=2280219054,4248526086&fm=26&fmt=auto&gp=0.jpg)









![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fss.csdn.net%2Fp%3Fhttps%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FadI0ApTVBFVEicLjjV1Ye9EdgGBJGzzibf5t5iaXUCFx52A4x88Cok5tUgkXk4pqhTdTMAB4EVWkZDW0iaVWn1Ka6g%2F640%3Fwx_fmt%3Dpng&refer=http%3A%2F%2Fss.csdn.net&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1629180212&t=1e62d7035afbbc12e4e1215dd95ec0b4)



![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fupload-images.jianshu.io%2Fupload_images%2F16177192-b180d5636433f157.png&refer=http%3A%2F%2Fupload-images.jianshu.io&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1629180234&t=69834520f36f495104904898bc387e8e)



### 7.程序与数据流（DataFlow）

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg.it610.com%2Fimage%2Finfo8%2Fb6208b0c1f91485c950269d469c2cf07.jpg&refer=http%3A%2F%2Fimg.it610.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1629180804&t=2bdb07510edffe42079397a2c959d836)



所有的Flink程序都是由三部分组成的：Source、Transformation和Sink。

- Source负责读取数据源，Transformation利用各种算子进行处理加工，Sink负责输出
- 在运行时，Flink上运行 的程序会被映射成“逻辑数据流”（dataflows），它包含了这三部分
- 每一个dataflow以一个或多个sources开始以一个或多个sinks结束。dataflow类似于任意的邮箱无环图（DAG）
- 再大部分情况下，程序中的转换运算（transformation）跟dataflow中的算子（operator）是一一对应的关系

### 8. 执行图（ExecutionGraph）

- Flink中的执行图可以分成四层：StreamGraph -> JobGraph -> ExecutionGraph -> 物理执行图
- StreamGraph：是根据用户通过StreamAPI编写的代码生成的最初的图。用来表示程序的拓扑结构。
- JobGraph：StreamGraph经过优化后生成了JobGraph，提交给JobManager的数据结构。主要的优化为：将多个符合条件的节点chain在一起作为一个节点。
- ExecutionGraph：JobManager根据JobGraph生成ExecutionGraph。ExecutionGraph是JobGraph的并行化版本，是调度层最核心的数据结构。
- 物理执行图：JobManager根据ExecutionGraph对Job进行调度后，在各个TaskManager上部署Task后形成的“图”，并不是一个具体的数据结构。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210408231604597.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nbG9uZ180NDQ0,size_16,color_FFFFFF,t_70)



![在这里插入图片描述](https://img-blog.csdnimg.cn/20210408231751844.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nbG9uZ180NDQ0,size_16,color_FFFFFF,t_70)





![在这里插入图片描述](https://img-blog.csdnimg.cn/20210408231836577.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nbG9uZ180NDQ0,size_16,color_FFFFFF,t_70)



### 9. 数据传输形式

- 一个程序中，不同的算子可能具有不同的并行度
- 算子之间传输数据的形式可以是one-to-one（forwarding）的模式也可以是redistributing的模式，具体是哪一种形式，取决于算子的种类。
- One-to-One：stream维护着分区以及元素的顺序（比如source和map之间）。这意味着map算子的子任务看到的个数以及顺序跟source算则的子任务生产的元素的个数、顺序相同。map、fliter、flatMap等算子都是one-to-one的对应关系。
- Redistribution：stream的分区会发生改变。每一个算子的子任务依据选择的transformation发送数据到不同的目标任务。例如，keyBy基于hashCode充分去。而broadcast和rebalance会重新分区（rebalance会轮询的方式进行分区，shuffle算子随机分区），这些算子都会引起redistribute过程，而redistribute的过程类似于spark中的shuffle过程。

### 10. 任务链（Operator Chains）

![Operator chaining into Tasks](https://ci.apache.org/projects/flink/flink-docs-release-1.13/fig/tasks_chains.svg)

- Flink采用了一种称为任务链（chains）的优化技术，可以在特定条件下减少本地通信的开销。为了满足任务链的要求，必须将两个或多个算子设为相同的并行度，并通过本地转发（local forward）的方式进行连接
- **相同并行度的one-to-one**操作，Flink这样相连的算子链成在一起形成一个task，原来的算子成为里面subtask
- 并行度相同，并且one-to-one操作，两个条件缺一不可





## Flink流处理API

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200509164232473.png#pic_center)



### 1. Environment

#### 1. getExecutionEnvironment

创建一个执行环境，表示当前执行程序的上下文。如果程序时独立调度用的，则此方法返回本地执行环境：如果从命令行客户端调用程序以提交到集群，则此方法返回此集群的执行环境，也就是说，getExecutionEnvironmen会根据查询运行的方式决定返回什么样的运行环境，是最常用的一种创建方式。

```java
// 批
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
// 流
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
```

如果没有设置并行度，会议flink-conf.yaml中的配置为准，默认是1.

#### 2 createLocalEnvironment

返回本地执行环境，需要在调用时指定默认的并行度。

```java
LocalStreamEnvironment env = StreamExecutionEnvironment.createLocalEnvironment(1);
```

#### 3 createRemoteEnvironment

返回集群执行环境，将 Jar 提交到远程服务器。需要在调用时指定 JobManager
的 IP 和端口号，并指定要在集群中运行的 Jar 包。

```java
StreamExecutionEnvironment env =
StreamExecutionEnvironment.createRemoteEnvironment("jobmanage-hostname", 6123,"YOURPATH//WordCount.jar");
```

### 2. Source



#### 1. 从集合读取数据

```java
package com.yulu.apitest.source;

import com.yulu.apitest.beans.SensorReading;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import java.util.Arrays;

public class SorceTest1_Collection {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //从集合中读取数据
        DataStream<SensorReading> data = env.fromCollection(Arrays.asList(
                new SensorReading("sensor1", 1547718199L, 35.8)
                , new SensorReading("sensor2", 1547718200L, 36.8)
                , new SensorReading("sensor3", 1547718211L, 34.6)
        ));
        DataStream<Integer> source = env.fromElements(1, 2, 3, 4, 5);
        data.print("SensorReading");
        source.print("int");
        env.execute();
    }
}
```

#### 2. 从文件读取数据

```java
DataStream<String> dataStream = env.readTextFile("YOUR_FILE_PATH");
```

#### 3. 以kafka消息队列的数据作为来源

```xml
<--!需要添加kafaka的flink的插件-->
pom.xml文件中需要添加依赖
<!--
https://mvnrepository.com/artifact/org.apache.flink/flink-connector-kafka-0.11
-->
<dependency>
<groupId>org.apache.flink</groupId>
<artifactId>flink-connector-kafka-0.11_2.12</artifactId>
<version>1.10.1</version>
</dependency>

```

```java
// kafka配置项
Properties properties = new Properties();
properties.setProperty("bootstrap.servers", "localhost:9092");
properties.setProperty("group.id", "consumer-group");
properties.setProperty("key.deserializer",
"org.apache.kafka.common.serialization.StringDeserializer");
properties.setProperty("value.deserializer",
"org.apache.kafka.common.serialization.StringDeserializer");
properties.setProperty("auto.offset.reset", "latest");
// 从 kafka 读取数据
DataStream<String> dataStream = env.addSource( new
FlinkKafkaConsumer011<String>("sensor", new SimpleStringSchema(), properties));
```



####  4. 自定义Source

除了以上的source数据来源，我们还可以自定义source。需要做的，只是传入一个SourceFunction就可以了。具体调用如下：

```java
DataStream<SensorReading> dataStream = env.addSource( new MySensor());
```

测试例子：

```java
package com.yulu.apitest.source;

import com.yulu.apitest.beans.SensorReading;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.SourceFunction;
import java.util.HashMap;
import java.util.Random;

public class  SourceTest2_UDF {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStreamSource<SensorReading> data = env.addSource(new MySourceFunction());
        SingleOutputStreamOperator<String> map = data.map(new MapFunction<SensorReading, String>() {
            @Override
            public String map(SensorReading sensorReading) throws Exception {
                String tempStr = sensorReading.getId() + ": " + sensorReading.getTimestamp() + ": " + sensorReading.getTemperature();
                return tempStr;
            }
        });
        map.print("自定义数据源");
        env.execute();
    }
}

class MySourceFunction implements SourceFunction<SensorReading>{
    private boolean flag = true;
    private HashMap<String, Double> sensorMap;
    private Random random;

    public MySourceFunction(){
        sensorMap = new HashMap<String, Double>();
        random = new Random();
        for (int i = 0; i < 10; i++) {
            String key = "sensor_"+(i+1);
            sensorMap.put(key, 60 + random.nextGaussian() * 20);
        }
    }

    public void run(SourceContext<SensorReading> sourceContext) throws Exception {
        while (flag) {
            for(String key : sensorMap.keySet()){
                Double temp_value = sensorMap.getOrDefault(key, 0.0) + random.nextGaussian();
                sensorMap.put(key, temp_value);
                Long timestamp = System.currentTimeMillis();
                sourceContext.collect(new SensorReading(key, timestamp, temp_value));
            }
        }
    }

    public void cancel() {
        flag = false;
    }
}
```



### Transform

转换算子

#### map

#### flatMap

#### filter

#### keyBy

DataStream---》KeyedStream：逻辑地将一个流拆分成不相交的分区，每个分区包含具有相同key的元素，在内部以hash的形式实现的。

只有使用了分组转换算子之后，才可以使用聚合类算子



#### 滚动聚合算子



##### 1. sum()



##### 2. min()





##### 3. max()



##### 4. minBy()



##### 5. maxBy()





##### 6. Reduce

####  

#### 多流转换算子

##### split&select

**DataStream** -----》 **SplitStream**：根据某些特征把一个DataStream拆分成两个或者多个DataStream

![Flink 分流](source/Flink%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Flink%20%E5%88%86%E6%B5%81.png)



SplitStream ----》 DataStream：从一个SplitStream中获取一个或者多个DataStream。

![Flink 合流](source/Flink%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Flink%20%E5%90%88%E6%B5%81.png)





##### connect&coMap

![Connect算子](source/Flink%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/Connect%E7%AE%97%E5%AD%90.jpg)

DataStream ----》ConnectedStreams：连接两个保持他们类型的数据流（两条数据流类型可以不一致），两个数据流被Connect之后，只是被放在了同一个流中，内部依然各自保持各自的数据和形式不发生任何变化，两个流相互独立。



![ConMap&ConflatMap](source/Flink%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/ConMap&ConflatMap.jpg)

ConnectedStreams  -----》 DataStream：作用于ConnectedStreams上，功能与map和flatMap一致，对ConnectedStreams中的每一个Stream分别进行map和flatMap处理。

其中传入的map方法会有两个，分别对应上述两个不同的流分别做各自的map处理。





##### union

![union合流](source/Flink%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/union%E5%90%88%E6%B5%81.jpg)

DataStream  ---》 DataStream ： 对两个或者两个以上的DataStream进行union操作，产生一个包含所有DataStream元素的新DataStream



##### Connect与Union的区别

1. Union之前两个流的类型必须是一样的，Connect可以不一样，在之后的coMap中再去调整成一样的。
2. Connect只能操作两个流，Union可以操作多个。



#### 支持的数据类型

Flink 流应用程序处理的是以数据对象表示的事件流。所以在 Flink 内部，我们 需要能够处理这些对象。它们需要被序列化和反序列化，以便通过网络传送它们； 或者从状态后端、检查点和保存点读取它们。为了有效地做到这一点，Flink 需要明 确知道应用程序所处理的数据类型。Flink 使用类型信息的概念来表示数据类型，并 为每个数据类型生成特定的序列化器、反序列化器和比较器。

 Flink 还具有一个类型提取系统，该系统分析函数的输入和返回类型，以自动获 取类型信息，从而获得序列化器和反序列化器。但是，在某些情况下，例如 lambda 函数或泛型类型，需要显式地提供类型信息，才能使应用程序正常工作或提高其性 能。

 Flink 支持 Java 和 Scala 中所有常见数据类型。使用最广泛的类型有以下几种。

##### 基础数据类型

Flink支持所有的Java和Scala基础数据类型：Integer， Double， Long， String........

###### 元组：Java和Scala中的元组（Tuples）

java中的元组类型是Flink中自己实现的，最多支持25个元素的Tuple

###### Scala中的样例类（case class）



###### Java简单对象（POJOs）

**JAVA  的 POJOs对象要求：**

1. 必须有空参构造方法
2. 如果字段类型不是public，必须有getter和setter方法。

###### 其他（Arrays，Lists， Maps， Enums等等）



#### 实现UDF函数----更细粒度的控制流



##### 函数类（Function Classes）

Flink暴露了所有的UDF函数的接口（实现方式为接口或者抽象类）。例如MapFunction，FillterFunction、ProcessFunction等等。



##### 匿名函数类（Lambda Functions）

（Lambda表达式）



##### 富函数（Rich Functions）

“富函数”是DataStream API提供的一个函数类的接口，所有Flink函数类都有其Rich版本。它与常规函数的不同在于，可以获取运行环境的上下文，并拥有一些生命周期方法，所以可以实现更复杂的功能。

- RichMapFunction
- RichFlatMapFunction
- RichFilterFunction
- 。。。。

Rich Funntion有一个生命周期的概念。典型的声明周期方法有：

- open()方法是rich function的初始化方法，当一个算子例如map或者filter被调用之前open()会被调用。
- close()方法是生命周期中的最后一个调用的方法，做一些清理工作
- getRuntimeContext()方法提供了函数的RuntimeContext的一些信息，例如函数执行的并行度，任务的名字，以及state状态。







### Sink

Flink没有类似于java中的foreach方法，让用户进行迭代的操作。虽然有对外的输出操作，但是都要利用Sink完成。最后通过类似如下方式完成任务的最终输出操作。

```java
stream.addSink(new MySink(XXXX));
```













### Window API

#### window概述

streaming流式计算是一种被设计用于处理无限数据集的数据处理引擎，而无限数据集是指一种不断增长的本质上无限的数据集，而**window**是一种**切割无限数据为有限块进行处理**的手段。

window是无限数据流处理的核心，Window将一个无限的stream拆分成有限大小的“buckets”桶，我们可以在这些桶上做计算操作。



#### 窗口的声明周期

简而言之，一旦应该属于这个窗口的第一个元素到达，就会创建一个窗口，并且当时间(这里的时间可分为两种：**事件时间**：EventTime或**处理时间：ProcessTime**)超过其结束时间戳和用户指定的允许迟到时间(参见允许迟到)时，该窗口将被完全删除。

**Flink 保证只对基于时间的窗口进行删除**，而不对其他类型的窗口进行删除，例如全局窗口(参见窗口分配器)。

例如，使用基于**事件时间**的窗口策略，每5分钟创建一个不重叠(或滚动)窗口，并允许迟到1分钟，Flink 将为12:00到12:05之间的间隔创建一个新窗口，当时间戳为此间隔的第一个元素到达时，它将在水印通过12:06时间戳时删除它。

此外，每个窗口将有一个触发器(**触发器**)和一个函数(ProcessWindowFunction、 ReduceFunction 或 AggregateFunction)(**窗口函数**)附加到它。

该函数将包含应用于窗口内容的计算，而触发器指定窗口在何种条件下被认为已准备好应用该函数。触发策略可能类似于“**当窗口中的元素数量超过4时**”，或者“**当水印通过窗口的末端时**”。触发器也可以决定在窗口创建和移除之间的任何时间清除窗口的内容。在这种情况下，清除仅仅指向窗口中的元素，而不是窗口元数据。这意味着仍然可以向该窗口添加新数据。

除此之外，还可以指定一个 Evictor (类似于过滤器) ，它将能够在触发器触发之后以及应用函数之前和/或之后从窗口中移除元素。

#### Keyed vs Non-Keyed Windows 键控窗口与非键控窗口

>  The first thing to specify is whether your stream should be keyed or not. This has to be done before defining the window. Using the `keyBy(...)` will split your infinite stream into logical keyed streams. If `keyBy(...)` is not called, your stream is not keyed.
>
> In the case of keyed streams, any attribute of your incoming events can be used as a key (more details [here](https://ci.apache.org/projects/flink/flink-docs-release-1.13/zh/docs/dev/datastream/fault-tolerance/state/#keyed-datastream)). Having a keyed stream will allow your windowed computation to be performed in parallel by multiple tasks, as each logical keyed stream can be processed independently from the rest. All elements referring to the same key will be sent to the same parallel task.
>
> In case of non-keyed streams, your original stream will not be split into multiple logical streams and all the windowing logic will be performed by a single task, *i.e.* with parallelism of 1.

#### window类型

Window可以分成两种类型：

- **TimeWindow**：按照时间生成Window（处理时间 、 时间时间）
  - 滚动时间窗口（Tumbling window）
  - 滑动时间窗口（Sliding Window）
  - 会话窗口（Session Window）
- **CountWindow**：按照指定的数据条数生成一个Window，与时间无关。
  - 滚动计数窗口
  - 滑动计数窗口

每种窗口类型下面都有一定相应的窗口分配器



#### Window Assigners 窗口分配器

在指定流是否为键控之后，下一步就是定义窗口分配器。窗口分配器定义如何将元素分配给指定的窗口。这是通过在`.window(WindowAssigner)`调用中指定选择的**WindowAssigner**来实现的。

**Windowwassigner** 负责将每个传入元素分配给一个或多个窗口。Flink 提供了预先定义的窗口分配，用于最常见的用例，即滚动窗口、滑动窗口、会话窗口和全局窗口。您还可以通过扩展 windowwassigner 类来实现自定义窗口分配器。所有内置的窗口分配器(全局窗口除外)根据时间将元素分配给窗口，可以是处理时间，也可以是事件时间。

基于时间的窗口有一个起始时间戳(包括起始时间戳)和一个结束时间戳(排他时间戳) ，它们共同描述了窗口的大小。在代码中，Flink 在处理基于时间的窗口时使用 **TimeWindow**，该窗口有查询开始和结束时间戳的方法，还有一个额外的方法 maxTimestamp () ，它返回给定窗口允许的最大时间戳。



##### 1. 滚动窗口（Tumbling Windows）

![Tumbling Windows](https://ci.apache.org/projects/flink/flink-docs-release-1.13/fig/tumbling-windows.svg)



将数据依据固定的窗口长度对数据进行切片。（**不指定步长，或者步长为窗口的宽度**）

<font color=green>其实本质上来讲，滚动窗口就是步长为窗口宽度的滑动窗口</font>

**特点：**

> 时间对其，窗口长度固定， 没有重叠。

```java
DataStream<T> input = ...;

// 基于事件时间的滚动窗口分配器
input
    .keyBy(<key selector>)
    .window(TumblingEventTimeWindows.of(Time.seconds(5)))
    .<windowed transformation>(<window funciton>);

// 基于处理时间的滚动窗口分配器
input
    .keyBy(<key selector>)
    .window(TumblingProcessingTimeWindows.of(Time.seconds(5)))
    .<windowed transformation>(<window funciton>);

// daily tumbling event-time windows offset by -8 hours.
// 指定窗口事件时间的偏移量
input
    .keyBy(<key selector>)
    .window(TumblingEventTimeWindows.of(Time.days(1), Time.hours(-8)))
    .<windowed transformation>(<window function>);
```

滚动窗口的of方法还提供了另外一个可选的参数：offset；这个参数可用于改变窗口的对其方式。

For example, without offsets hourly tumbling windows are aligned with epoch, that is you will get windows such as `1:00:00.000 - 1:59:59.999`, `2:00:00.000 - 2:59:59.999` and so on. If you want to change that you can give an offset. With an offset of 15 minutes you would, for example, get `1:15:00.000 - 2:14:59.999`, `2:15:00.000 - 3:14:59.999` etc. An important use case for offsets is to adjust windows to timezones other than UTC-0. For example, in China you would have to specify an offset of `Time.hours(-8)`.

可用此参数更改其他时区。



##### 2. 滑动窗口（Sliding Windows）

The *sliding windows* assigner assigns elements to windows of fixed length. Similar to a tumbling windows assigner, the size of the windows is configured by the *window size* parameter. An additional *window slide* parameter controls how frequently a sliding window is started. Hence, sliding windows can be overlapping if the slide is smaller than the window size. In this case elements are assigned to multiple windows.

![sliding windows](https://ci.apache.org/projects/flink/flink-docs-release-1.13/fig/sliding-windows.svg)

这样，你可以有10分钟大小的窗口，可以滑动5分钟。有了这个，您每5分钟就会得到一个窗口，其中包含在过去10分钟内到达的事件，

<font color=green>滑动窗口可以说是一种更为广义的一种窗口分配器，上述我们了解的滚动窗口其实质上就是滑动步长定滑动窗口宽度的一种滑动窗口。</font>

**特点：**

> 窗口长度固定， 可以有重叠

```java
DataStream<T> input = ...;

// sliding event-time windows
input
    .keyBy(<key selector>)
    .window(SlidingEventTimeWindows.of(Time.seconds(10), Time.seconds(5)))
    .<windowed transformation>(<window function>);

// sliding processing-time windows
input
    .keyBy(<key selector>)
    .window(SlidingProcessingTimeWindows.of(Time.seconds(10), Time.seconds(5)))
    .<windowed transformation>(<window function>);

// sliding processing-time windows offset by -8 hours
// 同理，依然可以传入一个偏移量，控制窗口的对齐时间
input
    .keyBy(<key selector>)
    .window(SlidingProcessingTimeWindows.of(Time.hours(12), Time.hours(1), Time.hours(-8)))
    .<windowed transformation>(<window function>);
```



##### 3. 会话窗口（Session Windows）

会话窗口分配器按照会话对元素进行分窗。相比于滚动窗口与滑动窗口，会话窗口没有一个固定的开始时间和结束时间。

相反，当会话窗口在一段时间内没有接收到元素时，即出现非活动间隔时，它将关闭。会话窗口分配器可以配置为静态会话间隙或者会话间隙提取器函数，该函数定义了不活动的周期有多长。当此期限过期时，当前会话关闭，并将后续元素分配给新的会话窗口。

![session windows](https://ci.apache.org/projects/flink/flink-docs-release-1.13/fig/session-windows.svg)

由一系列事件组合一个指定时间长度的timeout间隙组成，也就是一段时间没有接收到新数据就会生成新的窗口

**特点：**

> 时间无对齐

```java
DataStream<T> input = ...;

// event-time session windows with static gap
input
    .keyBy(<key selector>)
    .window(EventTimeSessionWindows.withGap(Time.minutes(10)))
    .<windowed transformation>(<window function>);
    
// event-time session windows with dynamic gap
input
    .keyBy(<key selector>)
    .window(EventTimeSessionWindows.withDynamicGap((element) -> {
        // determine and return session gap
    }))
    .<windowed transformation>(<window function>);

// processing-time session windows with static gap
input
    .keyBy(<key selector>)
    .window(ProcessingTimeSessionWindows.withGap(Time.minutes(10)))
    .<windowed transformation>(<window function>);
    
// processing-time session windows with dynamic gap
input
    .keyBy(<key selector>)
    .window(ProcessingTimeSessionWindows.withDynamicGap((element) -> {
        // determine and return session gap
    }))
    .<windowed transformation>(<window function>);
```

<font color=blue>特别说明:</font>

由于会话窗口没有固定的开始和结束，它们的评估不同于滚动窗口和滑动窗口。在内部，会话窗口操作符会为每个到达的纪录**创建一个新窗口**，然后进行窗口之间的距离比对，如果距离比我们指定的间隔更近，就将两个窗口进行合并。



指定的最小间隔，可以通过实现`SessionWindowTimeGapExtractor `接口来动态指定间隔。



##### 4. 全局窗口（Global Windows）

A *global windows* assigner assigns all elements with the same key to the same single *global window*. This windowing scheme is only useful if you also specify a custom [trigger](https://ci.apache.org/projects/flink/flink-docs-release-1.13/zh/docs/dev/datastream/operators/windows/#triggers). Otherwise, no computation will be performed, as the global window does not have a natural end at which we could process the aggregated elements.

![global windows](https://ci.apache.org/projects/flink/flink-docs-release-1.13/fig/non-windowed.svg)

```java
DataStream<T> input = ...;

input
    .keyBy(<key selector>)
    .window(GlobalWindows.create())
    .<windowed transformation>(<window function>);
```



#### window API节

窗口函数：window()方法

> 我们可以用window()来定义一个窗口，然后基于这个window去做一些聚合或者其他处理操作。
>
> <font color=red>但是需要注意：window()方法必须在keyBy()方法之后才能调用</font>
>
> Flink提供了更加简单的.timeWindow() 和 .countWindow()方法，用于定义时间窗口和计数窗口。

```java
DataStream<Tuple2<String, Double>> minTempPerWindowStream = dataStream.map(new MapFunction<>{.....}).keyBy(data->data.f0)
    .timeWindow(Time.seconds(15))
    .minBy(1);
```

#### 创建不同类型的窗口(简化版)

- 滚动时间窗口（tumbling time window）

  - ```java
    .timeWindow(Time.seconds(15))
    ```

- 滑动时间窗口（sliding time window）

  - ```java
    .timeWindow(Time.second(15), Time.second(5))
    ```

- 会话窗口（Session window）

  - ```java
    .window(EventTimeSessionWindows.withGap(Time.minutes(10)))
    ```

- 滚动计数窗口（tumbling count window）

  - ```java
    .countWindow(5)
    ```

- 滑动计数窗口（sliding count window）

  - ```java
    .countWindow(10, 2)
    ```

  - 当窗口中的数据不足窗口长度的时候，每次满足步长就会产生一个窗口，相当于自动的往前补齐了10个元素；

  - 当窗口中的数据满足了窗口长度的时候，就不会触发往前的自动补齐了。

#### 窗口函数（window function）

开窗之后就需要调用窗口函数来对窗口中收集的数据做计算操作

窗口函数一般可分为两类：

- 增量聚合函数（incremental aggregation function）
  - 每条数据到来就进行计算，保持一个简单的状态
  - **ReduceFunction**：聚合前后的类型需要保持一致
  - **AggregateFunciton**：聚合前后的类型可以不一致，中间还多一个累加器acc
- 全窗口函数（full window functions）
  - 先把窗口所有数据收集起来，等到计算的时候会遍历所有数据
  - **ProcessWindowFunction**
  - **WindowFunction**

##### ReduceFunction 增量窗口函数

ReduceFunction 指定如何将输入中的两个元素组合起来，以生成同类型的输出元素。使用 ReduceFunction 增量聚合窗口的元素。

```java
DataStream<Tuple2<String, Long>> input = ...;

input
    .keyBy(<key selector>)
    .window(<window assigner>)
    .reduce(new ReduceFunction<Tuple2<String, Long>>() {
      public Tuple2<String, Long> reduce(Tuple2<String, Long> v1, Tuple2<String, Long> v2) {
        return new Tuple2<>(v1.f0, v1.f1 + v2.f1);
      }
    });
// 上面的示例总结了一个窗口中所有元素的元组的第二个字段的和。
```



##### AggregateFunciton 增量窗口函数

AggregateFunction 是 ReduceFunction 的通用版本，它有三种类型: 输入类型(IN)、累加器类型(ACC)和输出类型(OUT)。输入类型是输入流中元素的类型，而 AggregateFunction 有一个将一个输入元素添加到累加器的方法。该接口还具有用于创建初始累加器、将两个累加器合并为一个累加器以及从累加器中提取输出(OUT 类型)的方法。

和 ReduceFunction 一样，AggregateFunction  会在窗口的输入元素到达时增量地聚合它们。

```java
/**
 * The accumulator is used to keep a running sum and a count. The {@code getResult} method
 * computes the average.
 */
private static class AverageAggregate
    implements AggregateFunction<Tuple2<String, Long>, Tuple2<Long, Long>, Double> {
  @Override
  public Tuple2<Long, Long> createAccumulator() {
    return new Tuple2<>(0L, 0L);
  }

  @Override
  public Tuple2<Long, Long> add(Tuple2<String, Long> value, Tuple2<Long, Long> accumulator) {
    return new Tuple2<>(accumulator.f0 + value.f1, accumulator.f1 + 1L);
  }

  @Override
  public Double getResult(Tuple2<Long, Long> accumulator) {
    return ((double) accumulator.f0) / accumulator.f1;
  }

  @Override
  public Tuple2<Long, Long> merge(Tuple2<Long, Long> a, Tuple2<Long, Long> b) {
    return new Tuple2<>(a.f0 + b.f0, a.f1 + b.f1);
  }
}

DataStream<Tuple2<String, Long>> input = ...;

input
    .keyBy(<key selector>)
    .window(<window assigner>)
    .aggregate(new AverageAggregate());

// 上面的示例计算窗口中元素的第二个字段的平均值。
```



##### ProcessWindowFunction 全量窗口函数

一个 ProcessWindowFunction 获得一个包含窗口所有元素的 Iterable，以及一个访问时间和状态信息的 Context 对象，这使得它比其他窗口函数提供更多的灵活性。这是以性能和资源消耗为代价的，因为不能对元素进行递增聚合，而是需要在内部缓冲，直到认为可以处理窗口为止。

内部定义：

```java
public abstract class ProcessWindowFunction<IN, OUT, KEY, W extends Window> implements Function {

    /**
     * Evaluates the window and outputs none or several elements.
     *
     * @param key The key for which this window is evaluated.
     * @param context The context in which the window is being evaluated.
     * @param elements The elements in the window being evaluated.
     * @param out A collector for emitting elements.
     *
     * @throws Exception The function may throw exceptions to fail the program and trigger recovery.
     */
    public abstract void process(
            KEY key,
            Context context,
            Iterable<IN> elements,
            Collector<OUT> out) throws Exception;

   	/**
   	 * The context holding window metadata.
   	 */
   	public abstract class Context implements java.io.Serializable {
   	    /**
   	     * Returns the window that is being evaluated.
   	     */
   	    public abstract W window();

   	    /** Returns the current processing time. */
   	    public abstract long currentProcessingTime();

   	    /** Returns the current event-time watermark. */
   	    public abstract long currentWatermark();

   	    /**
   	     * State accessor for per-key and per-window state.
   	     *
   	     * <p><b>NOTE:</b>If you use per-window state you have to ensure that you clean it up
   	     * by implementing {@link ProcessWindowFunction#clear(Context)}.
   	     */
   	    public abstract KeyedStateStore windowState();

   	    /**
   	     * State accessor for per-key global state.
   	     */
   	    public abstract KeyedStateStore globalState();
   	}

}
```

key参数是通过为 keyBy ()调用指定的 KeySelector 提取的key。在元组索引键或字符串字段引用的情况下，该键类型始终是 Tuple，您必须手动将其转换为大小正确的元组以提取key字段。



例子：

```java
DataStream<Tuple2<String, Long>> input = ...;

input
  .keyBy(t -> t.f0)
  .window(TumblingEventTimeWindows.of(Time.minutes(5)))
  .process(new MyProcessWindowFunction());  // 通过调用process方法调用ProcessWindowFunction全量窗口函数。

/* ... */

public class MyProcessWindowFunction 
    // 泛型的类型： 输入类型， 输出类型， key的类型， window类型
    extends ProcessWindowFunction<Tuple2<String, Long>, String, String, TimeWindow> {

  @Override
  public void process(String key, Context context, Iterable<Tuple2<String, Long>> input, Collector<String> out) {
    long count = 0;
    for (Tuple2<String, Long> in: input) {
      count++;
    }
    out.collect("Window: " + context.window() + "count: " + count);
  }
}

```

该示例显示了计算窗口中元素数量的 ProcessWindowFunction。此外，window 函数将有关窗口的信息添加到输出中。

<font color=green>这种全量的窗口函数非常耗费资源，计算的过程中需要保持这个窗口的所有元素。</font>



###### 具有增量功能的ProcessWindowFunction函数

ProcessWindowFunction可以与ReduceFunction或者AggregateFunction组合，以便在元素到达窗口时对其进行增量聚合。当窗口关闭时，ProcessWindowFunction将提供聚合结果。这允许在访问ProcessWindowFunction的附加窗口元信息时增量计算窗口。

```java
DataStream<SensorReading> input = ...;

input
  .keyBy(<key selector>)
  .window(<window assigner>)
  .reduce(new MyReduceFunction(), new MyProcessWindowFunction());

// Function definitions

private static class MyReduceFunction implements ReduceFunction<SensorReading> {

  public SensorReading reduce(SensorReading r1, SensorReading r2) {
      return r1.value() > r2.value() ? r2 : r1;
  }
}

private static class MyProcessWindowFunction
    extends ProcessWindowFunction<SensorReading, Tuple2<Long, SensorReading>, String, TimeWindow> {

  public void process(String key,
                    Context context,
                    Iterable<SensorReading> minReadings,
                    Collector<Tuple2<Long, SensorReading>> out) {
      SensorReading min = minReadings.iterator().next();
      out.collect(new Tuple2<Long, SensorReading>(context.window().getStart(), min));
  }
}


```



```java
DataStream<Tuple2<String, Long>> input = ...;

input
  .keyBy(<key selector>)
  .window(<window assigner>)
  .aggregate(new AverageAggregate(), new MyProcessWindowFunction());

// Function definitions

/**
 * The accumulator is used to keep a running sum and a count. The {@code getResult} method
 * computes the average.
 */
private static class AverageAggregate
    implements AggregateFunction<Tuple2<String, Long>, Tuple2<Long, Long>, Double> {
  @Override
  public Tuple2<Long, Long> createAccumulator() {
    return new Tuple2<>(0L, 0L);
  }

  @Override
  public Tuple2<Long, Long> add(Tuple2<String, Long> value, Tuple2<Long, Long> accumulator) {
    return new Tuple2<>(accumulator.f0 + value.f1, accumulator.f1 + 1L);
  }

  @Override
  public Double getResult(Tuple2<Long, Long> accumulator) {
    return ((double) accumulator.f0) / accumulator.f1;
  }

  @Override
  public Tuple2<Long, Long> merge(Tuple2<Long, Long> a, Tuple2<Long, Long> b) {
    return new Tuple2<>(a.f0 + b.f0, a.f1 + b.f1);
  }
}

private static class MyProcessWindowFunction
    extends ProcessWindowFunction<Double, Tuple2<String, Double>, String, TimeWindow> {

  public void process(String key,
                    Context context,
                    Iterable<Double> averages,
                    Collector<Tuple2<String, Double>> out) {
      Double average = averages.iterator().next();
      out.collect(new Tuple2<>(key, average));
  }


```



##### Window Function（旧版本）

在一些可以使用 ProcessWindowFunction 的地方，您也可以使用 WindowFunction。这是 ProcessWindowFunction 的较老版本，它提供的上下文信息较少，并且没有一些先进的特性，比如按窗口键控状态。此接口将在某个时候被弃用。

定义如下：

```java
public interface WindowFunction<IN, OUT, KEY, W extends Window> extends Function, Serializable {

  /**
   * Evaluates the window and outputs none or several elements.
   *
   * @param key The key for which this window is evaluated.
   * @param window The window that is being evaluated.
   * @param input The elements in the window being evaluated.
   * @param out A collector for emitting elements.
   *
   * @throws Exception The function may throw exceptions to fail the program and trigger recovery.
   */
  void apply(KEY key, W window, Iterable<IN> input, Collector<OUT> out) throws Exception;
}
```

例子：

```java
DataStream<Tuple2<String, Long>> input = ...;

input
    .keyBy(<key selector>)
    .window(<window assigner>)
    .apply(new MyWindowFunction());
```







#### 其他可选API

##### 触发器Trigger

A `Trigger` determines when a window (as formed by the *window assigner*) is ready to be processed by the *window function*. Each `WindowAssigner` comes with a default `Trigger`. If the default trigger does not fit your needs, you can specify a custom trigger using `trigger(...)`.

触发器确定窗口(由窗口分配者形成)什么时候可以被窗口函数处理。每个 windowwassigner 都有一个默认触发器。如果默认触发器不符合您的需要，可以使用`trigger(...)`方法指定自定义触发器。

触发器接口有5个方法，允许触发器对不同的事件做出反应：

- The  `onElement()` method is called for each element that is added to a window.  对于添加到窗口中的每个元素，都会调用
- The `onEventTime()` method is called when a registered event-time timer fires. 当注册的事件时间计时器 触发时，调用
- The `onProcessingTime()` method is called when a registered processing-time timer fires. 当注册的处理时间计时器触发时，调用
- The `onMerge()` method is relevant for stateful triggers and merges the states of two triggers when their corresponding windows merge, 方法与有状态触发器相关，并在两个触发器对应的窗口合并时合并它们的状态,*e.g. 例如:* when using session windows. 当使用会话窗口时
- The `clear()` method performs any action needed upon removal of the corresponding window. 方法在移除相应的窗口时执行所需的任何操作



##### 



##### evictor()——移除器

定义移除某些数据的逻辑

##### .allowedLateness()——允许处理迟到的数据

在使用事件时间窗口时，可能会发生元素延迟到达的情况，例如，Flink 用于跟踪事件时间进度的水印已经超过了元素所属窗口的结束时间戳。查看事件时间，特别是后期元素，可以更彻底地讨论 Flink 如何处理事件时间。

默认情况下，当水印超过窗口的末尾时，后期元素将被删除。但是，flink 允许指定窗口操作符允许的最大迟到时间。允许迟到指定元素在被删除之前可以迟到多少时间，其默认值为0。在水印通过窗口末端之后、但在通过窗口末端之前以及允许的迟到之后到达的元素仍然添加到窗口中。根据所使用的触发器，延迟但未删除的元素可能会导致窗口再次启动。这是 EventTimeTrigger 的情况。

```java
DataStream<T> input = ...;

input
    .keyBy(<key selector>)
    .window(<window assigner>)
    .allowedLateness(<time>)
    .<windowed transformation>(<window function>);
```



##### .sideOutputLateData()——将迟到的数据放入侧输出流

因为仅仅是设置了延迟时间也不能保证所有的数据都会如期的达到，例如实习生产环境消息会积压，造成消息延迟一天都是有可能的。但是我们不能直接把允许的延迟时间调的很大，这样很耗性能。

<font color=green>侧输出流就是一个很好的兜底方案</font>

侧输出流可以获得一个数据流，这些数据已经被丢弃了（或者可以说是水印超过了窗口的最大时间限制，被丢弃的数据）。

```java
final OutputTag<T> lateOutputTag = new OutputTag<T>("late-data"){};

DataStream<T> input = ...;

SingleOutputStreamOperator<T> result = input
    .keyBy(<key selector>)
    .window(<window assigner>)
    .allowedLateness(<time>)
    .sideOutputLateData(lateOutputTag)
    .<windowed transformation>(<window function>);

DataStream<T> lateStream = result.getSideOutput(lateOutputTag); // 从已经处理完的结果中获取侧输出流
```

#### 处理窗口结果 







#### 小总结

**Keyed Windows**

```java
stream
       .keyBy(...)               <-  keyed versus non-keyed windows
       .window(...)              <-  required: "assigner"
      [.trigger(...)]            <-  optional: "trigger" (else default trigger)
      [.evictor(...)]            <-  optional: "evictor" (else no evictor)
      [.allowedLateness(...)]    <-  optional: "lateness" (else zero)
      [.sideOutputLateData(...)] <-  optional: "output tag" (else no side output for late data)
       .reduce/aggregate/apply()      <-  required: "function"
      [.getSideOutput(...)]      <-  optional: "output tag"
```

**Non-Keyed Windows**

```java
stream
       .windowAll(...)           <-  required: "assigner"
      [.trigger(...)]            <-  optional: "trigger" (else default trigger)
      [.evictor(...)]            <-  optional: "evictor" (else no evictor)
      [.allowedLateness(...)]    <-  optional: "lateness" (else zero)
      [.sideOutputLateData(...)] <-  optional: "output tag" (else no side output for late data)
       .reduce/aggregate/apply()      <-  required: "function"
      [.getSideOutput(...)]      <-  optional: "output tag"
```





## Flink中的时间语义与WaterMark

![Event Time and Processing Time](https://ci.apache.org/projects/flink/flink-docs-release-1.13/fig/event_processing_time.svg)

在Flink的流式处理中，会涉及到时间的不同概念，如上图：

- **Event Time**：是事件创建的时间。它通常有事件中的时间戳描述，例如采集的日志数据中，每条日志都会记录自己的生成时间，Flink通过时间戳分配访问事件时间戳。
- **Ingestion(摄食，吸收，吞入) Time**：是数据进入Flink的时间。
- **Processing Time**：是每一个执行基于时间操作的算子的本地系统时间，与机器相关，flink1.2之前默认的时间属性就是Process Time。

举个例子来区分Event time  和 Processing Time：星球大战



![事件时间 & 处理时间](source/Flink%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/%E4%BA%8B%E4%BB%B6%E6%97%B6%E9%97%B4%20&%20%E5%A4%84%E7%90%86%E6%97%B6%E9%97%B4.png)

在例如：一条日志进入Flink的时间为10:00:000.123， 到达Window的系统时间为10:00:01:123

对于业务来说，要统计1min内的故障日志个数，哪个时间最有意义？——当然是eventTime，因为我们要根据日志的生成时间进行统计。

在Flink的流式处理中，绝大部分的业务都会使用eventTime，一般只在eventTime无法使用时，才会被迫使用ProcessingTime或者IngestionTime。

如果要使用EventTime，那么需要引入EventTime的时间属性，引入方式如下：

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
```

<font color=green>但是上述设置EventTime的操作在1.12版本之后就变成了默认</font>



**乱序数据的影响**



![乱序数据的影响（顺序数据）](source/Flink%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/%E4%B9%B1%E5%BA%8F%E6%95%B0%E6%8D%AE%E7%9A%84%E5%BD%B1%E5%93%8D%EF%BC%88%E9%A1%BA%E5%BA%8F%E6%95%B0%E6%8D%AE%EF%BC%89.png)

![乱序数据的影响（乱序数据）](source/Flink%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/%E4%B9%B1%E5%BA%8F%E6%95%B0%E6%8D%AE%E7%9A%84%E5%BD%B1%E5%93%8D%EF%BC%88%E4%B9%B1%E5%BA%8F%E6%95%B0%E6%8D%AE%EF%BC%89.png)

- 当Flink以Event Time模式处理数据流时，它会根据数据里的时间戳来处理基于时间的算子
- 由于网络、分布式等原因，会导致乱序数据的产生
- 乱序数据会让窗口计算不准确

因此Flink为解决乱序数据的情况，引入了重要的**WaterMark**

### WaterMark

支持事件时间的流处理器需要一种方法来度量事件时间的进度， 以便窗口能够根据事件时间的进度决定是否关闭窗口。

在Flink中测量事件时间进度的机制就是**watermark**（水印/水位线）。watermark会作为输入流的一部分数据进行流动，并带有时间戳t；这个时间戳t表示事件时间在流中到达的时间（或者称之为事件时间的进度）；这就意味着，**watermark**所带的这个时间戳t之前的事件时间到达时间都要**小于等于**这个时间戳t(即时间所带的时间戳早于或等于水印watermark的时间戳)

![A data stream with events (out of order) and watermarks](https://ci.apache.org/projects/flink/flink-docs-release-1.13/fig/stream_watermark_out_of_order.svg)

水印对于无序流至关重要，如上述图所示：w(11)watermark 代表时间戳小于11的事件时间都已经达到了。

<font color=green>自我理解：watermark就是事件时间戳的时间戳。</font>

每个事件或者流入flink中的每个元素都可能带有这个事件时间，而watermark就是事件时间戳的时间戳，不过对于无序来说，这个时间戳是递增的。



<img src=".\图片\watermark处理乱序数据.png" alt="watermark处理乱序数据" style="zoom:80%;" />

举个例子：如上述图

窗口大小为4s， watermark延迟时间为2s；

其窗口关闭的时机是在watermark为4的时候，第一个窗口才会关闭。



### WaterMark Strategy（水印策略）

为了处理事件时间，Flink需要知道事件时间戳，这就意味着流中的每个元素都需要分配其事件时间戳。1.12版本之后，通常通过使用**TimestampAssigner**从元素中的某个字段提取时间戳来完成。

时间戳分配与生成水印watermark密切相关，matermark告诉系统事件时间的进度。可以通过指定一个**WatermarkGenerator**来配置watermark。

Flik API 期望一个同时包含 TimestampAssigner 和 WatermarkGenerator 的 **WatermarkStrategy**。一些常见的策略可以作为 WatermarkStrategy 的静态方法开箱即用，但用户也可以在需要时构建自己的策略。 

```java
package org.apache.flink.api.common.eventtime;

import java.time.Duration;
import org.apache.flink.annotation.Public;
import org.apache.flink.api.common.eventtime.WatermarkGeneratorSupplier.Context;
import org.apache.flink.util.Preconditions;

@Public
public interface WatermarkStrategy<T> extends TimestampAssignerSupplier<T>, WatermarkGeneratorSupplier<T> {
    WatermarkGenerator<T> createWatermarkGenerator(Context var1);

    default TimestampAssigner<T> createTimestampAssigner(org.apache.flink.api.common.eventtime.TimestampAssignerSupplier.Context context) {
        return new RecordTimestampAssigner();
    }

    default WatermarkStrategy<T> withTimestampAssigner(TimestampAssignerSupplier<T> timestampAssigner) {
        Preconditions.checkNotNull(timestampAssigner, "timestampAssigner");
        return new WatermarkStrategyWithTimestampAssigner(this, timestampAssigner);
    }

    default WatermarkStrategy<T> withTimestampAssigner(SerializableTimestampAssigner<T> timestampAssigner) {
        Preconditions.checkNotNull(timestampAssigner, "timestampAssigner");
        return new WatermarkStrategyWithTimestampAssigner(this, TimestampAssignerSupplier.of(timestampAssigner));
    }

    default WatermarkStrategy<T> withIdleness(Duration idleTimeout) {
        Preconditions.checkNotNull(idleTimeout, "idleTimeout");
        Preconditions.checkArgument(!idleTimeout.isZero() && !idleTimeout.isNegative(), "idleTimeout must be greater than zero");
        return new WatermarkStrategyWithIdleness(this, idleTimeout);
    }

    static <T> WatermarkStrategy<T> forMonotonousTimestamps() {
        return (ctx) -> {
            return new AscendingTimestampsWatermarks();
        };
    }

    static <T> WatermarkStrategy<T> forBoundedOutOfOrderness(Duration maxOutOfOrderness) {
        return (ctx) -> {
            return new BoundedOutOfOrdernessWatermarks(maxOutOfOrderness);
        };
    }

    static <T> WatermarkStrategy<T> forGenerator(WatermarkGeneratorSupplier<T> generatorSupplier) {
        return generatorSupplier::createWatermarkGenerator;
    }

    static <T> WatermarkStrategy<T> noWatermarks() {
        return (ctx) -> {
            return new NoWatermarksGenerator();
        };
    }
}

```

正如前面提到的，通常不会自己实现这个接口，而是使用WaterMarkStrategy上的静态辅助方法来实现常见的水印策略，或者将定制的TimestampAssigner和WaterMarkGenerator绑定在一起。

例子：

```java
WaterMarkStrategy
    .<Tuple2<Long, String>>forBoundedOutOfOrderness(Duration.ofSeconds(20))
    .withTiemstampAssigner((event, timestamp)->event.f0);
```

使用WatermarkStrategy这种方法获取一个流，并生成一个带有时间戳元素和水印的新流。如果原始流已经具有时间戳/水印，时间戳分配器将覆盖他们。



### Dealing With Idle Sources（处理闲置资源）

如果其中一个输入（分区）有一段时间不携带任何事件，这意味着waterMark Generator也不能获得任何新的信息来源为水印做基础。这种情况我们称之为空闲输入（空闲源）。这是一个问题，因为一些分区可能仍然带有事件，但是最小的水印可能会保持不变，下游任务的水印就不会改变。

为了解决这个问题，可以使用一个WaterMaterStrategy来检测闲置状态并将输入标记为空闲。

```java
WatermarkStrategy
        .<Tuple2<Long, String>>forBoundedOutOfOrderness(Duration.ofSeconds(20))
        .withIdleness(Duration.ofMinutes(1));
```



### 水印生成器WaterMark Generator

```java
/**
 * The {@code WatermarkGenerator} generates watermarks either based on events or
 * periodically (in a fixed interval).
 *
 * <p><b>Note:</b> This WatermarkGenerator subsumes the previous distinction between the
 * {@code AssignerWithPunctuatedWatermarks} and the {@code AssignerWithPeriodicWatermarks}.
 */
@Public
public interface WatermarkGenerator<T> {

    /**
     * Called for every event, allows the watermark generator to examine 
     * and remember the event timestamps, or to emit a watermark based on
     * the event itself.
     */
    void onEvent(T event, long eventTimestamp, WatermarkOutput output);

    /**
     * Called periodically, and might emit a new watermark, or not.
     *
     * <p>The interval in which this method is called and Watermarks 
     * are generated depends on {@link ExecutionConfig#getAutoWatermarkInterval()}.
     */
    void onPeriodicEmit(WatermarkOutput output);
}
```

waterMark的生成有两种不同的方式：**周期性水印（periodic）**  和 **间断性水印（punctuated）**

周期性生成器通常通过onEvent()观察传入的事件， 然后框架调用onPeriodicEmit()时发出水印。

Puncutated 生成器将查看 onEvent ()中的事件，并等待在流中携带水印信息的特殊标记事件或标点。当它看到这些事件中的一个时，它会立即发出水印。通常，间断性生成器不会从 onPeriodicEmit ()发出水印。

#### 自定义周期性watermark生成器

周期生成器定期地观察流事件并生成水印(可能取决于流元素，也可能纯粹取决于处理时间)。

生成水印的时间间隔(每 n 毫秒)是通过 ExecutionConfig.setAutoWatermarkInterval (...)定义的。每次都会调用生成器的 onPeriodicEmit ()方法**，如果返回的水印非空且大于以前的水印，则会发出新的水印**。

```java
/**
 * This generator generates watermarks assuming that elements arrive out of order,
 * but only to a certain degree. The latest elements for a certain timestamp t will arrive
 * at most n milliseconds after the earliest elements for timestamp t.
 */
public class BoundedOutOfOrdernessGenerator implements WatermarkGenerator<MyEvent> {

    private final long maxOutOfOrderness = 3500; // 3.5 seconds

    private long currentMaxTimestamp;  

    @Override
    public void onEvent(MyEvent event, long eventTimestamp, WatermarkOutput output) {
        // onEvent每来一个事件，就会调用一次， 选取元素流中的当前最大时间戳
        currentMaxTimestamp = Math.max(currentMaxTimestamp, eventTimestamp);
    }

    @Override
    public void onPeriodicEmit(WatermarkOutput output) {
        // emit the watermark as current highest timestamp minus the out-of-orderness bound
        // 当周期一到，就会调用一次此方法
        output.emitWatermark(new Watermark(currentMaxTimestamp - maxOutOfOrderness - 1));
    }

}

/**
 * This generator generates watermarks that are lagging behind processing time 
 * by a fixed amount. It assumes that elements arrive in Flink after a bounded delay.
 */
public class TimeLagWatermarkGenerator implements WatermarkGenerator<MyEvent> {

    private final long maxTimeLag = 5000; // 5 seconds

    @Override
    public void onEvent(MyEvent event, long eventTimestamp, WatermarkOutput output) {
        // don't need to do anything because we work on processing time
    }

    @Override
    public void onPeriodicEmit(WatermarkOutput output) {
        output.emitWatermark(new Watermark(System.currentTimeMillis() - maxTimeLag));
    }
}
```

#### 自定义一个间断性watermark Generator

```java
public class PunctuatedAssigner implements WatermarkGenerator<MyEvent> {

    @Override
    public void onEvent(MyEvent event, long eventTimestamp, WatermarkOutput output) {
        if (event.hasWatermarkMarker()) {
            output.emitWatermark(new Watermark(event.getWatermarkTimestamp()));
        }
    }

    @Override
    public void onPeriodicEmit(WatermarkOutput output) {
        // don't need to do anything because we emit in reaction to events above
    }
}
```

<font color=red>可以在每个事件上生成水印。然而，由于每个水印都会导致一些下行计算，过多的水印会降低性能</font>



### WaterMark的传递

一个任务可能有多个并行的分区子任务，那么waterMark如何在任务之间传递的呢？

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg-blog.csdnimg.cn%2F20200205213258357.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMxODY2Nzkz%2Csize_16%2Ccolor_FFFFFF%2Ct_70&refer=http%3A%2F%2Fimg-blog.csdnimg.cn&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1629787474&t=adc8126790231c8f42f5fa85869c4663)

每个分区都会维护一个partition WM， 每次都会选取这些分区中waterMark**最小值**往下游传递，通过**广播的方式**，会传递给下游所有任务



### EventTime在window中的使用

```java
import com.yulu.apitest.beans.SensorReading;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;

import java.time.Duration;

public class WindowDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        // 1.12版本之后，默认为EventTime
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

        DataStreamSource<String> source = env.socketTextStream("192.168.0.110", 7777);

        SingleOutputStreamOperator<SensorReading> sensorReadingSingleOutputStreamOperator = source.map(new MapFunction<String, SensorReading>() {
            @Override
            public SensorReading map(String s) throws Exception {
                String[] fields = s.split(",");
                return new SensorReading(fields[0], Long.parseLong(fields[1]), Double.parseDouble(fields[2]));
            }
        })
            	// 这里分配时间时间戳字段和 waterMark策略
                .assignTimestampsAndWatermarks(WatermarkStrategy
                        .<SensorReading>forBoundedOutOfOrderness(Duration.ofSeconds(2)) // 指定watermark的延迟时间为2s
                        .withTimestampAssigner((sensorReading, l) -> sensorReading.getTimestamp() * 1000L
                        ) // 指定eventTime字段
                        .withIdleness(Duration.ofSeconds(1))
                );
        //.withTimestampAssigner((sensorReading, l) -> sensorReading.getTimestamp() * 1000)

        sensorReadingSingleOutputStreamOperator.keyBy(sensorReading -> sensorReading.getId())
                .window(TumblingEventTimeWindows.of(Time.seconds(5)))   // 滚动窗口大小：5s
                .min("temperature")
                .print("max Temperature");
        env.execute();

    }
}
```

```java
// 数据测试：
sensor1,1547718192,35.8   
sensor1,1547718193,35.8   
sensor1,1547718194,34.8   
sensor1,1547718195,35.8   
sensor1,1547718196,35.8   
sensor1,1547718197,35.8   
sensor1,1547718198,35.8
sensor1,1547718199,35.8
sensor1,1547718200,35.8
sensor1,1547718201,35.8
sensor1,1547718202,35.8

    
输出：
max Temperature:1> SensorReading{id='sensor1', timestamp=1547718192, temperature=34.8}
max Temperature:1> SensorReading{id='sensor1', timestamp=1547718195, temperature=35.8}
```

### 窗口的起点和偏移量

默认窗口的起点是窗口的大小的整数倍

```java
sensor1,1547718192,35.8   
sensor1,1547718193,35.8   
sensor1,1547718194,34.8   
sensor1,1547718195,35.8   
sensor1,1547718196,35.8 
    
针对于上述数据，当窗口大小为5时，那么其实watermark时间
第一个滚动窗口  [190~196)  最开右闭
第二个滚动窗口	[196~201)  一次类推

watermark的延迟时间设置了2s，那么watermark的时间达到197才会关闭第一窗口， 但是窗口中只有[190~196)的数据。
```

```java
.window(TumblingEventTimeWindows.of(Time.seconds(5)))
    默认的offset=0，我们可以指定offset，可以使得窗口的起点为我们自定义的时间点
    也可以通过这个偏移量控制时区UTC
```

### 允许延迟时间(allowedLateness)

虽然watermark在一定程度上可以解决数据乱序的问题，但是如果数据乱序特别严重，即使使用了watermark也是会有出现丢失数据的可能给。

针对上述问题，flink允许我们设置延迟时间，延迟窗口关闭的时间。

```java
sensorReadingSingleOutputStreamOperator.keyBy(sensorReading -> sensorReading.getId())
                .window(TumblingEventTimeWindows.of(Time.seconds(5)))
                .allowedLateness(Time.seconds(1))  //设置允许延迟时间
                .min("temperature")
                .print("max Temperature");
```

<font color=blue>但是即使设置了允许延迟时间，依然还有可能会出现延迟的数据没有及时的流入到窗口中</font>

flink还提供了一个兜底方案：侧输出流（SideOutputLateData）

```java
        //定义侧输出流标签
        OutputTag<SensorReading> outputTag = new OutputTag<SensorReading>("late SensorReading Data") {
        };

        SingleOutputStreamOperator<SensorReading> min = sensorReadingSingleOutputStreamOperator.keyBy(sensorReading -> sensorReading.getId())
                .window(TumblingEventTimeWindows.of(Time.seconds(5)))
                .allowedLateness(Time.seconds(1))
                .sideOutputLateData(outputTag)  // 兜底方案：将无法捕捉到的数据存放到侧输出流中
                .min("temperature");
        min.getSideOutput(outputTag).print("late data");  // 获取侧输出流
```

 

![flink数据流上的类型和操作](https://img-blog.csdnimg.cn/20190321213307414.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RkeHlncQ==,size_16,color_FFFFFF,t_70)





## Flink状态管理

流式计算分为无状态和有状态两种情况。无状态的计算观察每个独立事件，并根据最后一个事件输出结果。

例如：流处理应用程序从传感器接收温度读数，并在温度超过90度的时候发出报警。

有状态的计算则会基于多个事件输出结果。一下有一些例子：

- 所有类型的窗口。例如：计算过去一小时的平均温度，就是有状态计算（需要用到历史数据）
- 所有用于复杂时间处理的状态机。例如：若在一分钟内收到两个相差20度以上的温度读数，则发出告警信息，就也是有状态计算（需要用到前一个数据）。
- 流与流之间的所有关联操作，以及流与静态或动态之间的关联操作，都是有状态的计算。

无状态流处理分别接受数据记录，然后根据最新输入的数据生成输出数据。

有状态流处理会维护状态（根据每条输入记录进行更新），并基于最新输入的纪录和当前的状态值生成输出记录。

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg-blog.csdnimg.cn%2F20200424090646923.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h6MzcwMDU3NDQ4%2Csize_16%2Ccolor_FFFFFF%2Ct_70&refer=http%3A%2F%2Fimg-blog.csdnimg.cn&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1629907502&t=b5d7d2cb16d77ed59fe518b0ef958c77)

无状态流处理每次只转换一条输入记录，并且仅根据最新的输入记录输出结果。

有状态流处理维护所有已经处理的纪录状态，并根据每条新输入的纪录更新状态，因此输出记录反映的是综合考虑多个事件之后的结果。

尽管无状态的计算很重要，但是流处理对有状态的计算更感兴趣。事实上，正确地实现有状态的计算比实现无状态的计算难得多。

flink中的状态包括两种：

1. 算子状态（Operator State）
2. 键控状态（Keyed State）

### 算子状态（Operator State）

算子状态的作用范围限定为算子任务。这意味着同一并行任务所处理的所有数据都可以访问到相同的状态，状态对于同一任务而言是共享 的。算子状态不能由相同或不同算子的另一个任务访问。

![img](https://img1.baidu.com/it/u=402931042,4216055398&fm=15&fmt=auto&gp=0.jpg)

Flink为算子状态提供三种基本数据结构：

- 列表状态（List State）
  - 将状态表示为一组数据的列表（可用于持续的状态存储）
- 联合列表状态（Union list state）
  - 也将状态表示为数据的列表。它与常规列表状态的区别在于，在发生故障时，或者从保存点（savepoint）启动应用程序时如何恢复。
- 广播状态（BroadCast State）
  - 如果一个算子有多项任务，而它的每项任务状态又都相同，那么这种特殊情况最适合应用广播状态。



### 键控状态（Keyed State）

键控状态是根据输入数据流中定义的键（key）来维护和访问的。

Flink为每个键值维护一个状态实例，并将具有相同键的所有数据，都分区到同一个算子任务中，这个任务会维护和处理这个key对应的状态。

当任务处理一条数据时，它会自动将状态的访问范围限定为当前数据的key。

因此，具有相同key的所有数据都会访问相同的状态。Keyed State很类似于一个分布式的key-value map数据结构，只能用于KeyedStream（keyBy算子处理之后）。

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg-blog.csdnimg.cn%2F20210104153154601.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_Q2FwZV9zaXI%3D%2Csize_36%2Ccolor_FFFFFF%2Ct_30&refer=http%3A%2F%2Fimg-blog.csdnimg.cn&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1629908308&t=07382c840a0c0fe45d5c354392e04e09)

Flink的Keyed State支持以下数据类型：

- ValueState<T> 保存单个值，值的类型为T

  - ```java
    get操作：ValueState.value()
    set操作：ValueState.update(T value);
    ```

- ListState<T>保存一个列表，列表里的元素的数据类型为T

  - ```java
    ListState.add(T value);
    ListState.addAll(List<T> values);
    ListState.get() 返回 Iterable<T>;
    ListState.update(List<T> values);
    ```

- MapState<K,V> 保存Key-Value对儿

  - ```java
    MapState.get(UK key);
    MapState.put(UK key, UV value);
    MapState.contains(UK key);
    MapState.remove(UK key);
    ```

- ReducingState<T>

- AggregatingState<I,O>

`State.clear()可以清空状态`

上述这些状态可以在抽象类的富函数`Rich`中通过`getRuntimeContext().getState(new ValueStateDescriptor<Double>("状态名", Double.class));`来获取当前的监控状态。

**测试例子：**

利用KeyState，实现一个需求：检测传感器的温度值，如果连续的两个温度差值超过10度，就输出报警信息。

```java
import com.yulu.apitest.beans.SensorReading;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.state.ValueState;
import org.apache.flink.api.common.state.ValueStateDescriptor;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.util.Collector;
import org.apache.flink.util.OutputTag;

public class processDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStreamSource<String> source = env.socketTextStream("192.168.0.103", 7777);
        SingleOutputStreamOperator<SensorReading> sensorReadingDS = source.map(new MapFunction<String, SensorReading>() {
            @Override
            public SensorReading map(String s) throws Exception {
                String[] split = s.split(",");
                return new SensorReading(split[0], Long.parseLong(split[1]), Double.parseDouble(split[2]));
            }
        });

        /**
         * 小需求： 实现监控每个传感器连续10s中钟温度上升，输出报警信息
         */
        // 定义一个侧输出流标签
        OutputTag<SensorReading> outputTag = new OutputTag<SensorReading>("温度值非连续输出"){};
        SingleOutputStreamOperator<String> result = sensorReadingDS
                .keyBy(sensorReading -> sensorReading.getId())
                .process(new MyKeyedProcessFunction(5000L, outputTag));
        result.print();
        result.getSideOutput(outputTag).print("其他数据");

        env.execute();

    }

    static class MyKeyedProcessFunction extends KeyedProcessFunction<String, SensorReading, String> {

        private Long Interval;
        private ValueState<Long> timerTS; // 定时器时间
        private ValueState<Double> lastTempurate;
        private OutputTag<SensorReading> outputTag;

        public MyKeyedProcessFunction(Long interval, OutputTag<SensorReading> outputTag){
            this.Interval=interval;
            this.outputTag=outputTag;
        }

        @Override
        public void open(Configuration parameters) throws Exception {
            timerTS = getRuntimeContext().getState(new ValueStateDescriptor<Long>("timer-TS", Long.class));
            lastTempurate = getRuntimeContext().getState(new ValueStateDescriptor<Double>("lastTempurate", Double.class));
        }

        @Override
        public void close() throws Exception {
            timerTS.clear();
            lastTempurate.clear();
        }

        @Override
        public void processElement(SensorReading sensorReading, Context context, Collector<String> collector) throws Exception {
            Long timerts = timerTS.value();
            Double lasttemp = lastTempurate.value();
            // 当温度值上升，且定时器没有创建，则创建定时器
            if (lasttemp!=null  && sensorReading.getTemperature()>lasttemp && timerts==null){
                // 创建一个定时器， 时间到达某个时刻就触发一次操作
                Long ts = context.timerService().currentProcessingTime()+Interval;
                context.timerService().registerProcessingTimeTimer(ts);
                timerTS.update(ts);
            }
            // 如果温度值下降， 就删除定时器，状态归
            if (lasttemp!=null  && sensorReading.getTemperature()<=lasttemp && timerts !=null){
                // 根据传入的时间，删除指定的定时器
                context.timerService().deleteProcessingTimeTimer(timerts);
                timerTS.clear();
                context.output(this.outputTag, sensorReading);
            }
            // 更新状态

            lastTempurate.update(sensorReading.getTemperature());
        }

        @Override
        public void onTimer(long timestamp, OnTimerContext ctx, Collector<String> out) throws Exception {
            out.collect("传感器："+ctx.getCurrentKey()+"\t 温度值在："+this.Interval+"ms 温度值连续上升");
        }
    }
}
```



## Flink的容错机制

### 一致性检查点

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Foscimg.oschina.net%2Foscnet%2Fc15e5cf0-5f5a-48ac-ba14-b3d370a196cb.png&refer=http%3A%2F%2Foscimg.oschina.net&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1630159034&t=49399bac16c4c42293f8fa7cffe1f0d2)

- Flink故障恢复机制的核心，就是应用状态的一致性检查点
- 有状态流应用的一直检查点，其实就是所有任务的状态，在某个时间点的一份拷贝（一份快照）；这个时间点，<font color=red>应该是所有任务都恰好处理完一个相同的出入数据的时候。</font>

以上述图为例：如果当前source读取到5，下游任务也必须是都处理完5这个数据之后才做checkpoint。



### 从检查点恢复状态

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg2020.cnblogs.com%2Fblog%2F1824311%2F202009%2F1824311-20200908213604870-202910531.png&refer=http%3A%2F%2Fimg2020.cnblogs.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1630159603&t=9156f9309a498e83d6fe0a15fa6dc8e9)



- 在执行流应用程序期间，、flink会定期保存状态的一致检查点
- 如果发生故障，Flink将会使用最近的检查点来恢复应用程序的状态，并重新启动处理流程。

这就要数据源能够有**数据重放的功能**

![img](https://nimg.ws.126.net/?url=http%3A%2F%2Fdingyue.ws.126.net%2F2021%2F0727%2F1e1f12b7j00qwwn0g002ac000hs00umg.jpg&thumbnail=650x2147483647&quality=80&type=jpg)

- 遇到故障之后，第一步就是重启应用
- 第二步是从checkpoint中读取状态，将状态重置
- 从检查点重新启动应用程序后，其内部状态与检查点完成时的状态完全相同
- 第三步：开始消费并处理检查点到发生故障之间的所有数据
- 这种检查点的保存和恢复机制可以为应用程序状态提供“精确一次”（exactly-once）的一致性，因为所有算子都会保存检查点并回复其所有状态，这样一来所有的输入流就都会被重置到检查点完成时的位置。





### Flink检查点算法

- 一种简单的想法
  - 暂停应用，保存状态到检查点，再重新恢复应用
- Flink的改进实现
  - 基于Chandy-Lamport算法的分布式快照
  - 将检查点的保存和数据处理分离开，不暂停整个应用



#### 检查点分界线（Checkpoint Barrier）

- Flink的检查点算法用到了一种称为分界线（barrier）的特殊数据形式，用来把一条流上数据按照不同的检查点分开
- 分界线之前到来的数据导致的状态更改，都会被包含在当前分界线所属的检查点中；而基于分界线之后的数据导致的所有更改，就会被包含在之后的检查点中。



#### Flink检查点算法介绍

![在这里插入图片描述](https://www.icode9.com/i/ll/?i=20210111152147546.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_Q2FwZV9zaXI=,size_36,color_FFFFFF,t_30)



- 现在是一个两个输入流的应用程序，用并行的两个source任务来读取



![在这里插入图片描述](https://www.icode9.com/i/ll/?i=2021011115251435.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_Q2FwZV9zaXI=,size_36,color_FFFFFF,t_30)

- JobManager 会向每个Source任务发送一条带有新检查点ID的消息，通过这种方法来启动检查点
- 这个消息会传递给下游所有任务

![在这里插入图片描述](https://www.icode9.com/i/ll/?i=20210111153027619.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_Q2FwZV9zaXI=,size_36,color_FFFFFF,t_30)

- 数据源将它们的状态写入检查点，并发出一个检查点barrier，广播发送到下游任务。状态后端在状态存入检查点之后，会返回通知给source任务，source任务就会向JobManager确认检查点完成。

![在这里插入图片描述](https://www.icode9.com/i/ll/?i=20210111153426447.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_Q2FwZV9zaXI=,size_36,color_FFFFFF,t_30)

分界线对齐（barrier对齐）：barrier向下游传递，sum任务会等待所有输入分区的barrier到达。

对于barrier已经到达的分区，继续接收到的数据会被缓存起来；而barrier尚未到达的分区，数据会被正常处理。

![在这里插入图片描述](https://www.icode9.com/i/ll/?i=20210111153926877.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_Q2FwZV9zaXI=,size_36,color_FFFFFF,t_30)

当接收到所有分区的barrier时，任务就将其状态保存到状态后端的检查点中，然后将barrier继续向下游转发。



![在这里插入图片描述](https://www.icode9.com/i/ll/?i=2021011115420988.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_Q2FwZV9zaXI=,size_36,color_FFFFFF,t_30)

向下游转发barrier后，任务继续正常的数据处理。

![在这里插入图片描述](https://www.icode9.com/i/ll/?i=20210111154304922.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjY1MjU5Ng==,size_16,color_FFFFFF,t_70)

- sink任务向JobManager确认状态保存到checkpoint完毕
- 当所有任务都确认已成功将状态保存到检查点时，检查点就真正完成了。

每个任务都相应的状态的检查点，所有任务的状态checkpoint被状态后端state backend保存，当出现故障的时候，从此checkpoint就可以恢复任务。

上述蓝8，黄8就是source处理完蓝3和黄4的结果。

### 保存点（savepoints）

- Flink还提供了可以自定义的镜像保存功能，就是保存点（savepoints）
- 原则上，创建保存点使用的算法与检查点完全相同，因为保存点可以认为就是具有一些额外元数据的检查点
- Flink不会自动创建保存点，因此用户（或者外部调度程序）必须明确地触发创建操作
- 保存点是一个强大的功能。除了故障恢复外，保存点可以用于：有计划的手动备份，更新应用程序，版本迁移，暂停和重启应用，等等。



## 状态一致性

当在分布式系统中引入状态时，自然也引入了一致性问题。一致性实际上是“正确性级别”的另一种说法，也就是说在成功处理故障之后得到的结果，与没有发生然和故障时得到的结果相比，前者到底有多正确？举例来说：假设要对最近一小时登录的用户计数。在系统经理故障之后，计数结果是多少？如果有偏差，是有漏掉的计数还是重复计数？

### 一致性级别

在流处理中，一致性可以分3个级别：

- at-most-once：至多一次
  - 这其实是没有正确性保障的委婉说法——故障发生之后，计数结果可能丢失。
- at-least-once：至少一次
  - 这表示计数结果可能大于正确值，但绝对不会小于正确值。也就是说，计数程序在发生故障后可能多算，但是绝不会少算
- exactly-once
  - 这指的是系统保证在发生故障后得到的计数结果与正确值一致。



Flink的一个重大价值在于，它既保证了**exactly-once**，**也具有低延迟和高吞吐的处理能力**。

从根本上说，Flink通过使自身满足所有需求来避免权衡，它是业界的一次意义重大的技术飞跃。

### 一致性检查点（checkpoints）

- Flink使用了一种轻量级快照机制——检查点来保证exactly-once语义
- 有状态流应用的一致检查点，其实就是：所有任务的状态，在某个时间点的一份拷贝（一份快照）。而这个时间点，应该是所有任务都恰好处理完一个相同的输入数据的时候。
- 应用状态的一致检查点，是Flink故障恢复机制的核心



### 端到端（end-to-end）状态一致性

目前我们看到的一致性保证都是有流处理器实现的，也就是说都是在Flink流处理器内部保证的；而在真实应用中，流处理应用除了流处理器以外还包含了数据源（例如Kafka）和输出到持久化系统。

端到端的一致性保证，意味着结果的正确性贯穿了整个流处理应用的始终；每一个组件都保证了它自己的一致性，整个端到端的一致性级别取决于所有组件中一致性最弱的组件。具体可以划分如下：

- 内部保证——依赖checkpoints
- source端——**需要外部源可重设数据的读取位置**
- sink端——需要保证从故障恢复时，数据不会重复写入外部系统

而对于sink端，又有两种具体的实现方式：幂等性写入和事务性写入

- 幂等写入
  - 所谓幂等操作，是说一个操作，可以重复执行很多次，但只导致一次结果更改，也就是说，后面再重复执行就不起作用了。
- 事务写入
  - 需要构建事务来写入外部系统，构建的事务对应着checkpoint，等到checkpoint真正完成的时候，才把所有对应的结果写入sink系统中。
  - 具有原子性：一个事务中的一系列的操作要么全部成功，要么一个都不做。
  - 实现思想：构建的事务对应着checkpoint，等到checkpoint真正完成的时候，才把所有对应的结果写入sink系统中。
  - 实现方式
    - 预写日志
    - 两阶段提交

#### 预写日志

- 把结果数据先当成状态保存，然后在收到checkpoint完成的通知时，一次性写入sink系统
- 简单易于实现，由于数据提前在状态后端中做了缓存，所以无论什么sink系统，都能用这种方式一批搞定。
- DataStreamAPI提供了一个模板类：GenericWriteAheadSink，来实现这种事务性sink

缺点：一个checkpoint内的数据变成的批处理

#### 两阶段提交（2PC）

- 对于每个checkpoint，sink任务启动一个事务，并将接下来所有接收的数据添加到事务里
- 然后将这些数据写入外部sink系统，但不提交它们——这是只是“预提交“
- 当它收到checkpoint完成的通知时，它才正式提交事务，实现结果的真正写入
- 这种方式真正实现了exactly-once，它需要一个提供事务支持的外部sink系统。Flink提供了TwoPhaseCommitSinkFunction接口。

**2PC对外部sink系统的要求**

- 外部sink系统必须提供事务支持，或者sink任务必须能够模拟外部系统上的事务

- 在checkpoint的间隔期间里，必须能够开启一个事务并接受数据写入

- 在收到checkpoint完成的通知之前，事务必须是“等待提交”的状态。

  在故障恢复的情况下，这可能需要一些时间。如果这个时候sink系统关闭事务（例如超时），那么未提交的数据就会丢失

- sink任务必须能够在进程失败后恢复事务

- 提交事务必须是幂等操作

| sink\source       | 不可重置     | 可重置                                           |
| ----------------- | ------------ | ------------------------------------------------ |
| 任意(Any)         | At-most-once | At-least-once                                    |
| 幂等              | At-most-once | Exactly-once<br />（故障恢复时会出现暂时不一致） |
| 预写日志（WAL）   | At-most-once | At-least-once                                    |
| 两阶段提交（2PC） | At-most-once | Exactly-once                                     |



## Flink SQL 和 Table SQL

### 1. 整体介绍

#### 1.1 什么是Table API 和 Flink SQL



Flink本身是批流统一的处理框架，所以Table API和SQL，就是批流统一的上层处理API。

目前功能尚未完善，处于活跃的开发阶段。

Table API是一套内嵌在Java和Scala语言中的查询API，它允许我们以非常直观的方式，组合来自一些关系运算符的查询（比如select、filter和join）。而对于Flink SQL，就是直接可以在代码中写SQL，来实现一些查询（Query）操作。Flink的SQL支持，基于实现了SQL标准的Apache Calcite（Apache开源SQL解析工具）。

无论输入是批输入还是流式输入，在这两套API中，指定的查询都具有相同的语义，得到相同的结果。



#### 1.2 需要引入的依赖

Table API 和 SQL 需要引入的依赖有两个：planner和bridge。

```java
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-table-planner_2.11</artifactId>
    <version>1.10.0</version>
</dependency>
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-table-api-scala（java）-bridge_2.11</artifactId>
    <version>1.10.0</version>
</dependency>
```

**flink-table-planner**：planner计划器，是table API最主要的部分，提供了运行时环境和生成程序执行计划的planner；

**flink-table-api-scala-bridge**：bridge桥接器，主要负责table API和 DataStream/DataSet API的连接支持，按照语言分java和scala。

这里的两个依赖，是IDE环境下运行需要添加的；如果是生产环境，lib目录下默认已经有了planner，就只需要有bridge就可以了。

当然，如果想使用用户自定义函数，或是跟kafka做连接，需要有一个SQL client，这个包含在flink-table-common里。

#### 1.3 两种planner（old&blink）的区别

1. 批流统一：Blink将批处理作业，视为流式处理的特殊情况。所以，blink不支持表和DataSet之间的转换，批处理作业将不转换为DataSet应用程序，而是跟流处理一样，转换为DataStream程序来处理。

2. 因为批流统一，Blink planner也不支持BatchTableSource，而使用有界的StreamTableSource代替。

3. Blink planner只支持全新的目录，不支持已弃用的ExternalCatalog。

4. 旧planner和Blink planner的FilterableTableSource实现不兼容。旧的planner会把PlannerExpressions下推到filterableTableSource中，而blink planner则会把Expressions下推。
5. 基于字符串的键值配置选项仅适用于Blink planner。

6. PlannerConfig在两个planner中的实现不同。

7. Blink planner会将多个sink优化在一个DAG中（仅在TableEnvironment上受支持，而在StreamTableEnvironment上不受支持）。而旧planner的优化总是将每一个sink放在一个新的DAG中，其中所有DAG彼此独立。

8. 旧的planner不支持目录统计，而Blink planner支持。



### 2. API调用

#### 2.1 基本程序结构



Table API 和 SQL 的程序结构，与流式处理的程序结构类似；也可以近似地认为有这么几步：首先创建执行环境，然后定义source、transform和sink。

具体操作流程如下：

```java
val tableEnv = ...     // 创建表的执行环境

// 创建一张表，用于读取数据
tableEnv.connect(...).createTemporaryTable("inputTable")
// 注册一张表，用于把计算结果输出
tableEnv.connect(...).createTemporaryTable("outputTable")

// 通过 Table API 查询算子，得到一张结果表
val result = tableEnv.from("inputTable").select(...)
// 通过 SQL查询语句，得到一张结果表
val sqlResult  = tableEnv.sqlQuery("SELECT ... FROM inputTable ...")

// 将结果表写入输出表中
result.insertInto("outputTable")
```



#### 2.2 创建表环境

创建表环境最简单的方式，就是基于流处理执行环境，调用create方法直接创建：

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);
```

表环境（**TableEnvironment**）是flink中集成Table API & SQL的核心概念。它负责:

- 注册catalog

- 在内部 catalog 中注册表

- 执行 SQL 查询

-  注册用户自定义函数

- 将 DataStream 或 DataSet 转换为表

-  保存对 ExecutionEnvironment或者StreamExecutionEnvironment 的引用

在创建TableEnv的时候，可以多传入一个EnvironmentSettings或者TableConfig参数，可以用来配置TableEnvironment的一些特性。

#### 2.3 在Catalog中注册表

##### 2.3.1 表（Table）的概念

TableEnvironment可以注册目录Catalog，并可以基于Catalog注册表。它会维护一个Catalog-Table表之间的map。

表（Table）是由一个“标识符”来指定的，由3部分组成：Catalog名、数据库（database）名和对象名（表名）。如果没有指定目录或数据库，就使用当前的默认值。

表可以是常规的（Table，表），或者虚拟的（View，视图）。常规表（Table）一般可以用来描述外部数据，比如文件、数据库表或消息队列的数据，也可以直接从 DataStream转换而来。视图可以从现有的表中创建，通常是table API或者SQL查询的一个结果。

##### 2.3.2 连接到文件系统（CSV格式）

连接外部系统在Catalog中注册表，直接调用tableEnv.connect就可以，里面参数要传入一个ConnectorDescriptor，也就是connector描述器。对于文件系统的connector而言，flink内部已经提供了，FileSystem().

```java
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);
        tableEnv.connect(new FileSystem().path("E:\\DATA\\IdeaProject\\FlinkProject\\src\\main\\resources\\Files\\sensor"))
                .withFormat(new Csv())
                .withSchema(new Schema() // 指定schema
                .field("sensor_id", DataTypes.STRING())
                .field("timestamp", DataTypes.BIGINT())
                .field("tempara", DataTypes.DOUBLE())
                )
                .createTemporaryTable("sensor_table"); // 创建临时表
        Table table = tableEnv.sqlQuery("select * from sensor_table");
        table.getSchema();
        tableEnv.toAppendStream(table, Row.class).print()
```



##### 2.3.3 连接到kafka

kafka的连接器flink-kafka-connector中，1.10版本的已经提供了Table API的支持。我们可以在 connect方法中直接传入一个叫做Kafka的类，这就是kafka连接器的描述器ConnectorDescriptor。

```java
        tableEnv.connect(
                new Kafka()
                        .version("0.11") // 定义kafka的版本
                        .topic("sensor") // 定义主题
                        .property("zookeeper.connect", "localhost:2181")
                        .property("bootstrap.servers", "localhost:9092")
        )
                .withFormat(new Csv())
                .withSchema(new Schema()
                        .field("id", DataTypes.STRING())
                        .field("timestamp", DataTypes.BIGINT())
                        .field("temperature", DataTypes.DOUBLE())
                )

                .createTemporaryTable("kafka_table");
```

当然也可以连接到ElasticSearch、MySql、HBase、Hive等外部系统，实现方式基本上是类似的。

#### 2.4 表的查询

利用外部系统的连接器connector，我们可以读写数据，并在环境的Catalog中注册表。接下来就可以对表做查询转换了。

Flink给我们提供了两种查询方式：Table API和 SQL。

##### 2.4.1 Table API的调用

Table API是集成在Scala和Java语言内的查询API。与SQL不同，Table API的查询不会用字符串表示，而是在宿主语言中一步一步调用完成的。

Table API基于代表一张“表”的Table类，并提供一整套操作处理的方法API。这些方法会返回一个新的Table对象，这个对象就表示对输入表应用转换操作的结果。有些关系型转换操作，可以由多个方法调用组成，构成链式调用结构。例如table.select(…).filter(…)，其中select（…）表示选择表中指定的字段，filter(…)表示筛选条件。

代码中的实现如下：

```java
Table table = tableEnv.from("inputTable")
table.select("id, temperator")
    .filter("id='sensor_1'")


```



##### 2.4.2 SQL 查询

Flink的SQL集成，基于的是ApacheCalcite，它实现了SQL标准。在Flink中，用常规字符串来定义SQL查询语句。SQL 查询的结果，是一个新的 Table。

代码实现如下：

```java
tableEnv.sqlQuery("select id, temperature from inputTable where id='sensor_1'");
```



#### 2.5 DataStream 转换成表

Flink允许我们把Table和DataStream做转换：我们可以基于一个DataStream，先流式地读取数据源，然后map成样例类，再把它转成Table。Table的列字段（column fields），就是样例类里的字段，这样就不用再麻烦地定义schema了。

**tableEnv.fromDataStream("table_name", PoJos)**

```java
 StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        SingleOutputStreamOperator<SensorReading> sensorReadingDS = env.socketTextStream("192.168.0.103", 7777)
                .map(s -> {
                    String[] fields = s.split(",");
                    return new SensorReading(fields[0], Long.parseLong(fields[1]), Double.parseDouble(fields[2]));
                });

        StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);
        Table dataTable = tableEnv.fromDataStream(sensorReadingDS);
        tableEnv.createTemporaryView("sensor", dataTable);
        Table resultTable = tableEnv.sqlQuery("select * from sensor");
        DataStream<Row> rowDataStream = tableEnv.toAppendStream(resultTable, Row.class);
        rowDataStream.print();
        env.execute();
```

#### 2.6 创建临时视图

创建临时视图的第一种方式，就是直接从DataStream转换而来。同样，可以直接对应字段转换；也可以在转换的时候，指定相应的字段。

```java
tableEnv.createTemporaryView("sensorView",dataStream);

tableEnv.createTemporaryView("sensorView",dataStream, "id","temperature","timestamp as ts");
```

#### 2.7 输出表

##### 2.7.1 输出到文件

```java
table.insertInto("outputTable");
```



##### 2.7.2 更新模式（update Mode）

在流处理过程中，表的处理并不像传统定义的那样简单。

对于流式查询（Streaming Queries），需要声明如何在（动态）表和外部连接器之间执行转换。与外部系统交换的消息类型，由**更新模式**（update mode）指定。

Flink Table API中的更新模式有以下三种：

###### 1. 追加模式（Append Mode）

在追加模式下，表（动态表）和外部连接器只交换插入（Insert）消息。 

###### 2. 撤回模式（RetractMode）

在撤回模式下，表和外部连接器交换的是：添加（Add）和撤回（Retract）消息。

- 插入（insert）会被编码为添加消息
- 删除（Delete）则编码为撤回消息
- 更新（Update）则会编码为，已更新行（上一行）的撤回想消息，和更新行（新行）的添加消息。

在此模式下，不能定义key，这一点跟upsert模式完全不同。

###### 3. Upsert（更新插入）模式

在Upsert模式下，动态表和外部连接器交换Upsert和Delete消息

这个模式需要一个唯一的key，通过这个key可以传递更新消息。为了正确应用消息，外部连接器需要知道这个唯一key的属性。

- 插入（Insert）和更新（Update）都被编码为Upsert消息；

-  删除（Delete）编码为Delete信息。

这种模式和Retract模式的主要区别在于，Update操作是用单个消息编码的，所以效率会更高。



#### 2.8 将表转换成DataStream



将表转换为DataStream或DataSet时，需要指定生成的数据类型，即要将表的每一行转换成的数据类型。通常，最方便的转换类型就是Row。当然，因为结果的所有字段类型都是明确的，我们也经常会用元组类型来表示。

 

表作为流式查询的结果，是动态更新的。所以，将这种动态查询转换成的数据流，同样需要对表的更新操作进行编码，进而有不同的转换模式。

Table API中表到DataStream有两种模式：

-  追加模式（Append Mode）

  用于表只会被插入（Insert）操作更改的场景。

-  撤回模式（Retract Mode）

  用于任何场景。有些类似于更新模式中Retract模式，它只有Insert和Delete两类操作。

得到的数据会增加一个Boolean类型的标识位（返回的第一个字段），用它来表示到底是新增的数据（Insert），还是被删除的数据（老数据， Delete）。

```java
DataStream<Row> rowDataStream = tableEnv.toAppendStream(resultTable, Row.class);
```

所以，没有经过groupby之类聚合操作，可以直接用 toAppendStream 来转换；而如果经过了聚合，有更新操作，一般就必须用 toRetractDstream。

#### 2.9 Query 的解释和执行

Table API提供了一种机制来解释（Explain）计算表的逻辑和优化查询计划。这是通过TableEnvironment.explain（table）方法或TableEnvironment.explain（）方法完成的。

explain方法会返回一个字符串，描述三个计划：

-  未优化的逻辑查询计划

-  优化后的逻辑查询计划

-  实际执行计划

```java
tableEnv.explain(resultTable)
```

Query的解释和执行过程，老planner和blink planner大体是一致的，又有所不同。整体来讲，Query都会表示成一个逻辑查询计划，然后分两步解释：

1. 优化查询计划

2. 解释成 DataStream 或者 DataSet程序

而Blink版本是批流统一的，所以所有的Query，只会被解释成DataStream程序；另外在批处理环境TableEnvironment下，Blink版本要到tableEnv.execute()执行调用才开始解释。

### 3. 流处理中的特殊概念

Table API和SQL，本质上还是基于关系型表的操作方式；而关系型表、关系代数，以及SQL本身，一般是有界的，更适合批处理的场景。这就导致在进行流处理的过程中，理解会稍微复杂一些，需要引入一些特殊概念。



#### 3.1 流处理和关系代数（表，及SQL）的区别

![image-20210801231913955](C:\Users\LuYu\AppData\Roaming\Typora\typora-user-images\image-20210801231913955.png)

可以看到，其实关系代数（主要就是指关系型数据库中的表）和SQL，主要就是针对批处理的，这和流处理有天生的隔阂。

#### 3.2 动态表（Dynamic Tables）

因为流处理面对的数据，是连续不断的，这和我们熟悉的关系型数据库中保存的“表”完全不同。所以，如果我们把流数据转换成Table，然后执行类似于table的select操作，结果就不是一成不变的，而是随着新数据的到来，会不停更新。

我们可以随着新数据的到来，不停地在之前的基础上更新结果。这样得到的表，在Flink Table API概念里，就叫做“**动态表**”（Dynamic Tables）。

动态表是Flink对流数据的Table API和SQL支持的核心概念。与表示批处理数据的静态表不同，动态表是随时间变化的。动态表可以像静态的批处理表一样进行查询，查询一个动态表会产生持续查询（Continuous Query）。连续查询永远不会终止，并会生成另一个动态表。查询（Query）会不断更新其动态结果表，以反映其动态输入表上的更改。



#### 3.3 流式持续查询的过程

![image-20210801232116759](C:\Users\LuYu\AppData\Roaming\Typora\typora-user-images\image-20210801232116759.png)

流式持续查询的过程为：

1. 流被转换为动态表
2. 对动态表计算连续查询，生成的新的动态表
3. 生成的动态表被转换回流









#### 3.4 时间特性

基于时间的操作（比如Table API和SQL中窗口操作），需要定义相关的时间语义和时间数据来源的信息。所以，Table可以提供一个逻辑上的时间字段，用于在表处理程序中，指示时间和访问相应的时间戳。

时间属性，可以是每个表schema的一部分。一旦定义了时间属性，它就可以作为一个字段引用，并且可以在基于时间的操作中使用。

时间属性的行为类似于常规时间戳，可以访问，并且进行计算。

##### 3.4.1 处理时间（Processing Time）





##### 3.4.2 事件时间

事件时间语义，允许表处理程序根据每个记录中包含的时间生成结果。这样即使在有乱序事件或者延迟事件时，也可以获得正确的结果。

为了处理无序事件，并区分流中的准时和迟到事件；Flink需要从事件数据中，提取时间戳，并用来推进事件时间的进展（watermark）。





