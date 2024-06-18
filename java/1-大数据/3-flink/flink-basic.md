

# Flink-Basic

## 1. 概述

### 1.1. 是什么

Flink是一个框架和分布式处理引擎，用于对无界和有界数据流进行有状态计算

Flink 的核心目标是“有数据流上的有状态计算”。

- 有界数据：有定义流的开始和结束，可以等所有数据都到齐才计算，可以对数据进行排序。
- 无界数据：有定义流的开始，没有定义流的结束。无休止的产生数据。数据必须立即处理，持续处理，不能等到所有数据都到了才处理。

状态：

- 把流处理需要的额外数据保存成一个**“状态”**，然后针对这条数据进行处理，并更新状态，这就是“有状态的流处理”
- 状态在内存中：速度快，可靠性差
- 状态在分布式系统中：可靠性高、速度慢。

### 1.2. 特点

目标：低延迟、高吞吐、结果的准确性和良好的容错率

- 高吞吐、低延迟
- 结果准确性：提供 **事件时间** 和 **处理时间**语义，对于乱序事件流，事件时间语义仍然能提供一致且准确的结果。
- 精确一次的状态一致性保护
- 可以连接到常用的外部系统，如db、kafka、hdfs等
- 高可用

### 1.3.Flink VS Spark Streaming

|              | **Flink**          | **Streaming**                        |
| ------------ | ------------------ | ------------------------------------ |
| **计算模型** | 流计算             | 微批处理                             |
| **时间语义** | 事件时间、处理时间 | 处理时间                             |
| **窗口**     | 多、灵活           | 少、不灵活（窗口必须是批次的整数倍） |
| **状态**     | 有                 | 没有                                 |
| **流式SQL**  | 有                 | 没有                                 |

### 1.4.分层API

![](./pic/0001.png)

- 越顶层越抽象、表达含义越简明、使用越方便
- 越底层越具体、表达能力越丰富、使用越灵活



## 2.部署

### 2.1.集群角色

Flink 提交作业和执行任务，需要几个关键的组件：

- 客户端：代码由客户端获取并做转换，之后提交给JobManager
- JobManager：Flink集群的“管事人”，对作业进行中央调度管理；获取到要执行的作业后，会进一步处理转换，然后分发给众多的TaskManager
- TaskManager：真正“干活的人”，负责数据的处理操作

![](./pic/0002.png)

### 2.2.集群搭建

#### 2.2.1. 集群启动

##### 2.2.1.1. 集群规划

| **节点服务器** | **hadoop102**                 | **hadoop103** | **hadoop104** |
| -------------- | ----------------------------- | ------------- | ------------- |
| **角色**       | JobManager  <br />TaskManager | TaskManager   | TaskManager   |

##### 2.2.1.2.下载并解压安装包

下载安装包flink-1.17.0-bin-scala_2.12.tgz，并解压

##### 2.2.1.3.修改进群配置

（1）进入conf路径，修改flink-conf.yaml文件，指定hadoop102节点服务器为JobManager 

修改如下内容：

```properties
$ vim flink-conf.yaml
# JobManager节点地址.
jobmanager.rpc.address: hadoop102
jobmanager.bind-host: 0.0.0.0
rest.address: hadoop102
rest.bind-address: 0.0.0.0
# TaskManager节点地址.需要配置为当前机器名
taskmanager.bind-host: 0.0.0.0
taskmanager.host: hadoop102
```

（2）修改workers文件，指定hadoop102、hadoop103和hadoop104为TaskManager

```properties
$ vim workers
hadoop102
hadoop103
hadoop104
```

（3）修改masters文件

```properties
$ vim masters
hadoop102:8081
```

（4）另外，在flink-conf.yaml文件中还可以对集群中的JobManager和TaskManager组件进行优化配置，主要配置项如下：

- **jobmanager.memory.process.size**：对JobManager进程可使用到的全部内存进行配置，包括JVM元空间和其他开销，默认为1600M，可以根据集群规模进行适当调整。
- **taskmanager.memory.process.size**：对TaskManager进程可使用到的全部内存进行配置，包括JVM元空间和其他开销，默认为1728M，可以根据集群规模进行适当调整。
- **taskmanager.numberOfTaskSlots**：对每个TaskManager能够分配的Slot数量进行配置，默认为1，可根据TaskManager所在的机器能够提供给Flink的CPU数量决定。所谓Slot就是TaskManager中具体运行一个任务所分配的计算资源。
- **parallelism.default**：Flink任务执行的并行度，默认为1。优先级低于代码中进行的并行度配置和任务提交时使用参数指定的并行度数量。

（5）对主节点修改后，分发到从节点，然后进行以下修改

```properties
$ vim flink-conf.yaml
# TaskManager节点地址.需要配置为当前机器名
taskmanager.host: hadoop103


# TaskManager节点地址.需要配置为当前机器名
taskmanager.host: hadoop104
```

（6）启动集群

在主节点hadoop102上执行 start-cluster.sh启动Flink集群 `bin/start-cluster.sh`

（7）访问 WebUI

可以访问http://hadoop102:8081对flink集群和任务进行监控管理

![](./pic/0003.png)

这里可以明显看到，当前集群的TaskManager数量为3；由于默认每个TaskManager的Slot数量为1，所以总Slot数和可用Slot数都为3。

#### 2.2.2.向集群提交作业

##### 2.2.2.1.环境准备

##### 2.2.2.2.程序打包

maven依赖添加shade插件配置，然后使用命令进行打包

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.2.4</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <artifactSet>
                            <excludes>
                                <exclude>com.google.code.findbugs:jsr305</exclude>
                                <exclude>org.slf4j:*</exclude>
                                <exclude>log4j:*</exclude>
                            </excludes>
                        </artifactSet>
                        <filters>
                            <filter>
                                <!-- Do not copy the signatures in the META-INF folder.
                                Otherwise, this might cause SecurityExceptions when using the JAR. -->
                                <artifact>*:*</artifact>
                                <excludes>
                                    <exclude>META-INF/*.SF</exclude>
                                    <exclude>META-INF/*.DSA</exclude>
                                    <exclude>META-INF/*.RSA</exclude>
                                </excludes>
                            </filter>
                        </filters>
                        <transformers combine.children="append">
                            <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer">
                            </transformer>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

##### 2.2.2.3.在WebUI上提交



##### 2.2.2.4.命令行提交

（1）启动集群

（2）将Flink程序运行jar包上传到服务器上

（3）使用命令`bin/flink run -m hadoop102:8081 -c com.atguigu.wc.SocketStreamWordCount path/to/jar`。这里的参数 `-m`指定了提交到的JobManager，`-c`指定了入口类。

### 2.3.部署模式

在一些应用场景中，对于集群资源分配和占用的方式，可能会有特定的需求。Flink为各种场景提供了不同的部署模式，主要有以下三种：会话模式（Session Mode）、单作业模式（Per-Job Mode）、应用模式（Application Mode）。

它们的区别主要在于：集群的生命周期以及资源的分配方式；以及应用的main方法到底在哪里执行——客户端（Client）还是JobManager。

#### 2.3.1.会话模式（Session Mode）

启动一个集群，保持一个会话，在整个会话中通过客户端提交作业。

集群启动时所有的资源都已经确定，所以所有提交的作业会竞争集群中的资源。

会话模式比较适用于单个规模小，执行时间短的大量作业。

![](./pic/0004.png)

#### 2.3.2.单作业模式（Pre-Job mode）

会话模式因为资源共享会导致狠毒哦问题，所以为了更好的隔离资源，可以考虑为每一个提交的作业启动一个集群——单作业模式。

![](./pic/0005.png)

作业完成后，集群就会关闭，所有资源也会被释放。在生产环境中更加稳定，所以是实际应用首选的模式。

Flink本身无法这样运行，所以但作业模式一般需要**借助一些资源管理框架来启动集群**，比如Yarn、K8S。

#### 2.3.3.应用模式（Application Mode）

上述两种模式下，应用代码都是在客户端上执行，然后由客户端提交给JobManager。但是这种方式需要占用大量网络带宽，并且会加重客户端所在节点的资源消耗。

解决办法就是不需要客户端，直接把应用提交给JobManager上运行。即为每一个提交的应用单独启动了一个集群，整个集群只为执行这一个应用而存在，执行完成后关闭这个集群。

![](./pic/0006.png)

### 2.4.Standalone运行模式

#### 2.4.1.会话模式部署

提前启动集群，并通过Web页面客户端提交任务（可以多个任务，但是集群资源固定）。

#### 2.4.2.单作业模式部署

Flink的Standalone集群并不支持单作业模式部署。因为单作业模式需要借助一些资源管理平台。

#### 2.4.3.应用模式部署

### 2.5.Yarn运行模式



#### 2.5.1.相关准备和配置

#### 2.5.2.会话部署模式

#### 2.5.3.单作业部署模式

#### 2.5.4.应用部署模式

### 2.6.K8S运行模式（了解）

### 2.7.历史服务器

## 3.运行时架构

### 3.1.系统架构

YARN上部署的过程是：客户端把Flink应用提交给Yarn的`ResourceManager`，Yarn的`ResourceManager`会向Yarn的`NodeManager`申请容器。在这些容器上，Flink会部署`JobManager`和`TaskManager`的实例，从而启动集群。Flink会根据运行在`JobManger`上的作业所需要的`Slot`数量动态分配`TaskManager`资源。

![](./pic/0007.png)

#### 3.1.1. 作业管理器（JobManager）

JobManager是一个Flink集群中任务管理和调度的核心，是控制应用执行的主进程。也就是说，每个应用都应该被唯一的JobManager所控制执行。

JobManger又包含3个不同的组件。

**（1）JobMaster**

JobMaster是JobManager中最核心的组件，负责处理单独的作业（Job）。所以JobMaster和具体的Job是一一对应的，多个Job可以同时运行在一个Flink集群中, 每个Job都有一个自己的JobMaster。

**（2）资源管理器（ResourceManager）**

ResourceManager主要负责资源的分配和管理，在Flink 集群中只有一个。所谓“资源”，主要是指TaskManager的任务槽（task slots）。任务槽就是Flink集群中的资源调配单元，包含了机器用来执行计算的一组CPU和内存资源。每一个任务（Task）都需要分配到一个slot上执行。

这里注意要把Flink内置的ResourceManager和其他资源管理平台（比如YARN）的ResourceManager区分开。

**（3）分发器（Dispatcher）**

Dispatcher主要负责提供一个REST接口，用来提交应用，并且负责为每一个新提交的作业启动一个新的JobMaster 组件。Dispatcher也会启动一个Web UI，用来方便地展示和监控作业执行的信息。Dispatcher在架构中并不是必需的，在不同的部署模式下可能会被忽略掉。

#### 3.1.2.任务管理器（TaskManager）

TaskManager是Flink中的工作进程，数据流的具体计算就是它来做的。Flink集群中必须至少有一个TaskManager；每一个TaskManager都包含了一定数量的任务槽（task slots）。Slot是资源调度的最小单位，slot的数量限制了TaskManager能够并行处理的任务数量。

启动之后，TaskManager会向资源管理器注册它的slots；收到资源管理器的指令后，TaskManager就会将一个或者多个槽位提供给JobMaster调用，JobMaster就可以分配任务来执行了。

在执行过程中，TaskManager可以缓冲数据，还可以跟其他运行同一应用的TaskManager交换数据。

### 3.2.核心概念

#### 3.2.1.并行度

**（1）并行子任务和并行度**

当要处理的数据量非常大时，我们可以把一个算子操作，“复制”多份到多个节点，数据来了之后就可以到其中任意一个执行。这样一来，一个算子任务就被拆分成了多个并行的**“子任务”（subtasks）**，再将它们分发到不同节点，就真正实现了并行计算。

在Flink执行过程中，每一个算子（operator）可以包含一个或多个子任务（operator subtask），这些子任务在不同的线程、不同的物理机或不同的容器中完全独立地执行。

**一个特定算子的子任务（subtask）的个数被称之为其并行度（parallelism）**。这样，包含并行子任务的数据流，就是并行数据流，它需要多个分区（stream partition）来分配并行任务。一般情况下，一个流程序的并行度，可以认为就是其所有算子中最大的并行度。

**（2）并行度的设置**

**代码中设置**：

- 可以很简单地在**算子**后跟着调用`setParallelism()`方法，表示设置当前算子的并行度
- 直接调用执行**环境**的`setParallelism()`方法，全局设定并行度。`env.setParallelism(2);`

**提交时设置**：

- 在使用flink run命令提交应用时，可以增加-p参数来指定当前应用程序执行的并行度，它的作用类似于执行环境的全局设置：`bin/flink run –p 2 –c MainClass path/to/jar`
- 在WebUI提交作业时设置

**配置文件中设置**：

在 flink-conf.yaml 文件中更改默认并行度：

`parallelism.default: 2`

**（3）并行度优先级**

代码(算子>全局) > 命令 > 配置文件

#### 3.2.2.算子链

##### 3.2.2.1.算子之间的数据传输

**（1）一对一（One-to-One，forwarding）**

这种模式下，数据流维护着分区以及元素的顺序。

map、filter、flatMap等算子都是这种one-to-one的对应关系。

**（2）重分区（Redsitribution）**

在这种模式下，数据流的分区会发生改变。每一个算子的子任务，会根据数据传输的策略，把数据发送到不同的下游目标任务。这些传输方式都会引起重分区的过程，这一过程类似于Spark中的shuffle。

##### 3.2.2.2.合并算子链

并行度相同的一对一（one to one）算子操作，可以直接链接在一起形成一个“大”的任务（task），这样原来的算子就成为了真正任务里的一部分。途中source和map合并在一起。

![](pic/0008.png)

Flink默认会按照算子链的原则进行链接合并，如果我们想要禁止合并或者自行定义，也可以在代码中对算子做一些特定的设置：

```java
// 禁用算子链
.map(word -> Tuple2.of(word, 1L)).disableChaining();

// 从当前算子开始新链
.map(word -> Tuple2.of(word, 1L)).startNewChain()
```

#### 3.2.3.任务槽

##### 3.2.3.1. Slot

很显然，TaskManager的计算资源是有限的，并行的任务越多，每个线程的资源就会越少。那一个TaskManager到底能并行处理多少个任务呢？为了控制并发量，我们需要在TaskManager上对每个任务运行所占用的资源做出明确的划分，这就是所谓的**任务槽（task slots）**。

**每个任务槽（task slot）其实表示了TaskManager拥有计算资源的一个固定大小的子集。这些资源就是用来独立执行一个子任务的**

![](pic/0009.png)

##### 3.2.3.2.slot数量配置

在 conf/flink-conf.yaml 文件中，配置项为：

```properties
taskmanager.numberOfTaskSlots: 8  # 默认是1
```

**需要注意的是，slot目前仅仅用来隔离内存，不会涉及CPU的隔离。**在具体应用时，可以将slot数量配置为机器的CPU核心数，尽量避免不同任务之间对CPU的竞争。这也是开发环境默认并行度设为机器CPU数量的原因。

##### 3.2.3.3.任务对slot的共享

在同一个作业中，不同任务节点（算子）的并行子任务 可以放到同一个slot上执行。

优点：允许我们保存完整的作业管道，如下图第一个slot。

![](pic/0010.png)

Flink默认是允许slot共享的，如果希望某个算子对应的任务完全独占一个slot，或者只有某一部分算子共享slot，我们也可以通过设置“slot共享组”手动指定：

```java
.map(word -> Tuple2.of(word, 1L)).slotSharingGroup("1");
```

这样，只有属于同一个slot共享组的子任务，才会开启slot共享；不同组之间的任务是完全隔离的，必须分配到不同的slot上。在这种场景下，总共需要的slot数量，就是各个slot共享组最大并行度的总和。

#### 3.2.4.并行度和任务槽的关系

- 任务槽是静态的概念，是指TaskManager具有的并发执行能力，可以通过参数`taskmanager.numberOfTaskSlots`进行配置；

- 并行度是动态概念，也就是TaskManager运行程序时实际使用的并发能力，可以通过参数`parallelism.default`进行配置。

**整个流处理程序的并行度，就应该是所有算子并行度中最大的那个，这代表了运行程序需要的slot数量。**

### 3.3.作业提交流程

#### 3.3.1.Standalone模式提交流程

#### 3.3.2.逻辑流图/作业图/执行图/物理流图

#### 3.3.3.Yarn模式作业提交流程

## 4.DataStream API

### 4.1. 执行环境

我们在提交作业执行计算时，首先必须获取当前Flink的运行环境，从而建立起与Flink框架之间的联系。

#### 4.1.1.创建执行环境

在代码中创建执行环境的方式，就是调用`StreamExecutionEnvironment`这个类的静态方法:具体有三种：

**（1）getExecutionEnvironment**：根据当前运行的方式，自行决定该返回什么样的运行环境

最简单的方式，就是直接调用`getExecutionEnvironment`方法。它会根据当前运行的上下文直接得到正确的结果：

- 如果程序是独立运行的，就返回一个本地执行环境；
- 如果是创建了jar包，然后从命令行调用它并提交到集群执行，那么就返回集群的执行环境。

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
```

**（2）createLocalEnvironment**：这个方法返回一个本地执行环境

可以在调用时传入一个参数，指定默认的并行度；如果不传入，则默认并行度就是本地的CPU核心数。

```java
StreamExecutionEnvironment localEnv = StreamExecutionEnvironment.createLocalEnvironment();
```

**（3）createRemoteEnvironment**：这个方法返回集群执行环境。

需要在调用时指定JobManager的主机名和端口号，并指定要在集群中运行的Jar包。

```java
StreamExecutionEnvironment remoteEnv = StreamExecutionEnvironment
  		.createRemoteEnvironment(
    		"host",                   // JobManager主机名
    		1234,                     // JobManager进程端口号
   			"path/to/jarFile.jar"  // 提交给JobManager的JAR包
		); 
```

#### 4.1.2.执行模式

从Flink 1.12开始，官方推荐的做法是直接使用DataStream API，在提交任务时通过将执行模式设为BATCH来进行批处理。

DataStream API执行模式包括：

- 流执行模式（Streaming）：默认
- 批执行模式（Batch）：专门用于批处理的执行模式。
- 自动模式（AutoMatic）：在这种模式下，将由程序根据输入数据源是否有界，来自动选择执行模式。

指定执行模式为批处理的方法：

- 通过命令行配置：`bin/flink run -Dexecution.runtime-mode=BATCH ...`或者提交作业时增加`execution.runtime-mode`参数，指定值为`BATCH`
- 通过代码配置：`env.setRuntimeMode(RuntimeExecutionMode.BATCH);`

实际应用中一般不会在代码中配置，而是使用命令行，这样更加灵活。

#### 4.1.3.触发程序执行

需要注意的是，写完输出（sink）操作并不代表程序已经结束。**因为当main()方法被调用时，其实只是定义了作业的每个执行操作，然后添加到数据流图中**；这时并没有真正处理数据——因为数据可能还没来。Flink是由事件驱动的，只有等到数据到来，才会触发真正的计算，这也被称为“延迟执行”或“懒执行”。

所以我们需要显式地调用执行环境的execute()方法，来触发程序执行。execute()方法将一直等待作业完成，然后返回一个执行结果（JobExecutionResult）。

### 4.2.源算子Source

Flink一般将数据的输入来源称为数据源（data source），而读取数据的算子就是源算子（source operator）。

![](pic/0011.png)

在Flink 1.12以前，添加Source的方法是调用执行环境的addSource()

```java
DataStream<String> stream = env.addSource(...);
```

从Flink1.12开始，主要使用流批统一的新Source架构：

```java
DataStreamSource<String> stream = env.fromSource(…)
```

#### 4.2.1. 准备一个POJO类

```java
// lombok
public class WaterSensor {
    public String id;
    public Long ts;
    public Integer vc;
}
```

#### 4.2.2.从集合中读取数据

```java
env.fromCollection(...)
```

#### 4.2.3.从文件中读取数据

添加文件连接器依赖：

```xml
<dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-files</artifactId>
            <version>${flink.version}</version>
</dependency>
```

代码如下：

```java
public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        FileSource<String> fileSource = FileSource.forRecordStreamFormat(new TextLineInputFormat(), new Path("input/word.txt")).build();
        env.fromSource(fileSource,WatermarkStrategy.noWatermarks(),"file").print();
        env.execute();
}
```

- 参数可以是目录，也可以是文件；还可以从HDFS目录下读取，使用路径`hdfs://...`；
- 路径可以是相对路径，也可以是绝对路径；
- 相对路径是从系统属性user.dir获取路径：idea下是project的根目录，standalone模式下是集群节点根目录；

#### 4.2.4.从Socket

一般用于测试

```java
DataStream<String> stream = env.socketTextStream("localhost", 7777);
```

#### 4.2.5.从Kafka

Flink官方提供了连接工具flink-connector-kafka，直接帮我们实现了一个消费者FlinkKafkaConsumer，它就是用来读取Kafka数据的SourceFunction。

maven依赖：

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka</artifactId>
    <version>${flink.version}</version>
</dependency>
```

demo如下：

```java
public class SourceKafka {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        KafkaSource<String> kafkaSource = KafkaSource.<String>builder()
            .setBootstrapServers("hadoop102:9092")
            .setTopics("topic_1")
            .setGroupId("atguigu")
            .setStartingOffsets(OffsetsInitializer.latest())
            .setValueOnlyDeserializer(new SimpleStringSchema()) 
            .build();
        DataStreamSource<String> stream = env.fromSource(kafkaSource, WatermarkStrategy.noWatermarks(), "kafka-source");
        stream.print("Kafka");
        env.execute();
    }
}
```



#### 4.2.6.从数据生成器读取数据

Flink从1.11开始提供了一个内置的DataGen 连接器，主要是用于生成一些随机数，用于在没有数据源的时候，进行流任务的测试以及性能测试等。

maven依赖：

```xml
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-datagen</artifactId>
            <version>${flink.version}</version>
        </dependency>
```

demo如下：

```java
public class DataGeneratorDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        DataGeneratorSource<String> dataGeneratorSource =
                new DataGeneratorSource<>(
                        new GeneratorFunction<Long, String>() {
                            @Override
                            public String map(Long value) throws Exception {
                                return "Number:"+value;
                            }
                        },
                        Long.MAX_VALUE,
                        RateLimiterStrategy.perSecond(10),
                        Types.STRING
                );
        env.fromSource(dataGeneratorSource, WatermarkStrategy.noWatermarks(), "datagenerator")
                .print();
        env.execute();
    }
}
```

#### 4.2.7.Flink支持的数据类型

**（1）Flink类型系统**

Flink使用“类型信息”（**TypeInformation**）来统一表示数据类型。TypeInformation类是Flink中所有类型描述符的基类。它涵盖了类型的一些基本属性，并为每个数据类型生成特定的序列化器、反序列化器和比较器。

**（2）Flink支持的数据类型**

对于常见的Java和Scala数据类型，Flink都是支持的。Flink在内部，Flink对支持不同的类型进行了划分，这些类型可以在Types工具类中找到：

**基本类型**

所有Java基本类型及其包装类，再加上Void、String、Date、BigDecimal和BigInteger。

**数组类型**

包括基本类型数组（PRIMITIVE_ARRAY）和对象数组（OBJECT_ARRAY）。

**复合数据类型**

- Java元组类型（TUPLE）：这是Flink内置的元组类型，是Java API的一部分。最多25个字段，也就是从Tuple0~Tuple25，不支持空字段。
- Scala 样例类及Scala元组：不支持空字段。
- 行类型（ROW）：可以认为是具有任意个字段的元组，并支持空字段。
- POJO：Flink自定义的类似于Java bean模式的类。

**辅助类型**

Option、Either、List、Map等。

**泛型类型（GENERIC）**

Flink支持所有的Java类和Scala类。不过如果没有按照上面POJO类型的要求来定义，就会被Flink当作泛型类来处理。Flink会把泛型类型当作黑盒，无法获取它们内部的属性；它们也不是由Flink本身序列化的，而是由Kryo序列化的。

在这些类型中，元组类型和POJO类型最为灵活，因为它们支持创建复杂类型。而相比之下，POJO还支持在键（key）的定义中直接使用字段名，这会让我们的代码可读性大大增加。所以，在项目实践中，往往会将流处理程序中的元素类型定为Flink的POJO类型。

Flink对POJO类型的要求如下：

- 类是公有（public）的
-  有一个无参的构造方法
-  所有属性都是公有（public）的
-  所有属性的类型都是可以序列化的

**（3）类型提示（Type Hints）**

Flink还具有一个类型提取系统，可以分析函数的输入和返回类型，自动获取类型信息，从而获得对应的序列化器和反序列化器。

由于Java中泛型擦除的存在，在某些特殊情况下（比如Lambda表达式中），自动提取的信息是不够精细。为了解决这类问题，Java API提供了专门的“类型提示”（type hints）。

```java
.map(word -> Tuple2.of(word, 1L))
.returns(Types.TUPLE(Types.STRING, Types.LONG)); // 避免泛型擦除
```

Flink还专门提供了TypeHint类，它可以捕获泛型的类型信息，并且一直记录下来，为运行时提供足够的信息。我们同样可以通过.returns()方法，明确地指定转换之后的DataStream里元素的类型。

```java
returns(new TypeHint<Tuple2<Integer, SomeType>>(){})  // 避免泛型擦除
```

### 4.3.转换算子Transformation

转换算子：将一个或多个DataStream转换为新的DataStream

#### 4.3.1. 基本转换算子

##### 4.3.1.1.映射Map

一个“一一映射”，消费一个元素就产出一个元素。

只需基于DataStream调用map()方法就可以进行转换处理。

```java
public class TransMap {
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<WaterSensor> stream = env.fromElements(
                new WaterSensor("sensor_1", 1, 1),
                new WaterSensor("sensor_2", 2, 2)
        );

        // 方式一：传入匿名类，实现MapFunction
        stream.map(new MapFunction<WaterSensor, String>() {
            @Override
            public String map(WaterSensor e) throws Exception {
                return e.id;
            }
        }).print();

        // 方式二：传入MapFunction的实现类
        // stream.map(new UserMap()).print();

        env.execute();
    }

    public static class UserMap implements MapFunction<WaterSensor, String> {
        @Override
        public String map(WaterSensor e) throws Exception {
            return e.id;
        }
    }
}
```

##### 4.3.1.2.过滤fliter

filter转换操作，顾名思义是对数据流执行一个过滤，通过一个布尔条件表达式设置过滤条件，对于每一个流内元素进行判断，若为true则元素正常输出，若为false则元素被过滤掉。

```java
public class TransFilter {
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<WaterSensor> stream = env.fromElements(
                
new WaterSensor("sensor_1", 1, 1),
new WaterSensor("sensor_1", 2, 2),
new WaterSensor("sensor_2", 2, 2),
new WaterSensor("sensor_3", 3, 3)
        );

        // 方式一：传入匿名类实现FilterFunction
        stream.filter(new FilterFunction<WaterSensor>() {
            @Override
            public boolean filter(WaterSensor e) throws Exception {
                return e.id.equals("sensor_1");
            }
        }).print();

        // 方式二：传入FilterFunction实现类
        // stream.filter(new UserFilter()).print();
        
        env.execute();
    }
    public static class UserFilter implements FilterFunction<WaterSensor> {
        @Override
        public boolean filter(WaterSensor e) throws Exception {
            return e.id.equals("sensor_1");
        }
    }
}
```

##### 4.3.1.3.扁平映射（flatmap）

消费一个元素，可以产生0到多个元素。flatMap可以认为是“扁平化”（flatten）和“映射”（map）两步操作的结合，也就是先按照某种规则对数据进行打散拆分，再对拆分后的元素做转换处理。

```java
public class TransFlatmap {
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<WaterSensor> stream = env.fromElements(
                
new WaterSensor("sensor_1", 1, 1),
new WaterSensor("sensor_1", 2, 2),
new WaterSensor("sensor_2", 2, 2),
new WaterSensor("sensor_3", 3, 3)

        );

        stream.flatMap(new MyFlatMap()).print();

        env.execute();
    }

    public static class MyFlatMap implements FlatMapFunction<WaterSensor, String> {

        @Override
        public void flatMap(WaterSensor value, Collector<String> out) throws Exception {

            if (value.id.equals("sensor_1")) {
                out.collect(String.valueOf(value.vc));
            } else if (value.id.equals("sensor_2")) {
                out.collect(String.valueOf(value.ts));
                out.collect(String.valueOf(value.vc));
            }
        }
    }
} 
```

#### 4.3.2. 聚合算子（Aggregate）

##### 4.3.2.1.按键分区（keyBy）

**在Flink中，要做聚合，需要先进行分区**；这个操作就是通过`keyBy`来完成的。`keyBy`是聚合前必须要用到的一个算子。keyBy通过指定键（key），可以将一条流从逻辑上划分成不同的分区（partitions）。

```java
public class TransKeyBy {
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<WaterSensor> stream = env.fromElements(
new WaterSensor("sensor_1", 1, 1),
new WaterSensor("sensor_1", 2, 2),
new WaterSensor("sensor_2", 2, 2),
new WaterSensor("sensor_3", 3, 3)
        );

        // 方式一：使用Lambda表达式
        KeyedStream<WaterSensor, String> keyedStream = stream.keyBy(e -> e.id);

        // 方式二：使用匿名类实现KeySelector
        KeyedStream<WaterSensor, String> keyedStream1 = stream.keyBy(new KeySelector<WaterSensor, String>() {
            @Override
            public String getKey(WaterSensor e) throws Exception {
                return e.id;
            }
        });

        env.execute();
    }
}
```

`keyBy`得到的结果将不再是DataStream，而是会将DataStream转换为`KeyedStream`。`KeyedStream`可以认为是“分区流”或者“键控流”，它是对DataStream按照key的一个逻辑分区，所以泛型有两个类型：除去当前流中的元素类型外，还需要指定key的类型。

##### 4.3.2.2.简单聚合

Flink 内置了几种最简单的聚合API：

- `sum()`：在输入流上，对指定的字段做叠加求和的操作。
- `min()`：在输入流上，对指定的字段求最小值。
- `max()`：求最大值
- `minBy`：与`min()`类似，在输入流上针对指定字段求最小值。`minBy()`会返回包含字段最小值的整条数据。
- `maxBy`：返回指定字段中最大的整条数据

Flink 指定字段的方式有两种： 指定位置 ， 指定名称。**元组类型**的数据，可以使用这两种方式来指定字段，f0、f1、f2、…来命名。POJO类型只能通过名称来指定。

```java
public class TransAggregation {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<WaterSensor> stream = env.fromElements(
new WaterSensor("sensor_1", 1, 1),
new WaterSensor("sensor_1", 2, 2),
new WaterSensor("sensor_2", 2, 2),
new WaterSensor("sensor_3", 3, 3)
        );

        stream.keyBy(e -> e.id).max("vc");    // 指定字段名称

        env.execute();
    }
}
```

- keyBy和聚合是成对出现的，先分区、后聚合，得到的依然是一个DataStream。而且经过简单聚合之后的数据流，元素的数据类型保持不变。
- 一个聚合算子，会为每一个key保存一个聚合的值，在Flink中我们把它叫作“状态”（state）。

##### 4.3.2.3.归约聚合（Reduce）

`reduce`可以对已有的数据进行归约处理，把每一个新输入的数据和当前已经归约出来的值，再做一个聚合计算。调用KeyedStream的reduce方法时，需要传入一个参数，实现`ReduceFunction`接口

```java
public interface ReduceFunction<T> extends Function, Serializable {
    T reduce(T value1, T value2) throws Exception;
}
```

`ReduceFunction`接口里需要实现reduce()方法，这个方法接收两个输入事件，经过转换处理之后输出一个相同类型的事件。在流处理的底层实现过程中，实际上是将中间“合并的结果”作为任务的一个状态保存起来的；之后每来一个新的数据，就和之前的聚合状态进一步做归约。

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

env
   .socketTextStream("hadoop102", 7777)
   .map(new WaterSensorMapFunction())
   .keyBy(WaterSensor::getId)
   .reduce(new ReduceFunction<WaterSensor>()
   {
       @Override
       public WaterSensor reduce(WaterSensor value1, WaterSensor value2) throws Exception {
           System.out.println("Demo7_Reduce.reduce");

           int maxVc = Math.max(value1.getVc(), value2.getVc());
           //实现max(vc)的效果  取最大值，其他字段以当前组的第一个为主
           //value1.setVc(maxVc);
           //实现maxBy(vc)的效果  取当前最大值的所有字段
           if (value1.getVc() > value2.getVc()){
               value1.setVc(maxVc);
               return value1;
           }else {
               value2.setVc(maxVc);
               return value2;
           }
       }
   })
   .print();
env.execute();
```

`reduce`同简单聚合算子一样，也要针对每一个key保存状态。因为状态不会清空，所以我们需要将`reduce`算子作用在一个有限key的流上。

#### 4.3.3.用户自定义函数（UDF）

##### 4.3.3.1.函数类

Flink暴露了所有UDF函数的接口，具体实现方式为接口或者抽象类，例如MapFunction、FilterFunction、ReduceFunction等。

##### 4.3.3.2.富函数类

“富函数类”也是DataStream API提供的一个函数类的接口，所有的Flink函数类都有其Rich版本。富函数类一般是以抽象类的形式出现的。例如：RichMapFunction、RichFilterFunction、RichReduceFunction等。

与常规函数类的不同主要在于，富函数类可以获取运行环境的上下文，并拥有一些生命周期方法，所以可以实现更复杂的功能。

Rich Function有生命周期的概念。典型的生命周期方法有：

- `open`方法：初始化方法，也就是会开启一个算子的生命周期。当一个算子的实际工作方法被调用之前，`open`方法会首先被调用
- `close`方法：l 是生命周期中的最后一个调用的方法，类似于结束方法。一般用来做一些清理工作。

**需要注意的是**，这里的生命周期方法，对于一个并行子任务来说只会调用一次；而对应的，实际工作方法，例如RichMapFunction中的map()，在每条数据到来后都会触发一次调用。

#### 4.3.4.物理分区算子（**Physical Partitioning**）

##### 4.3.4.1. 随机分区（Shuffle）

最简单的重分区方式就是直接“洗牌”。通过调用`DataStream`的`.shuffle()`方法，将数据随机地分配到下游算子的并行任务中去。

随机分区服从均匀分布（uniform distribution），所以可以把流中的数据随机打乱，均匀地传递到下游任务分区。

![](pic/0012.png)

##### 4.3.4.2.轮训分区（Round-Robin）

轮询，简单来说就是“发牌”，按照先后顺序将数据做依次分发。通过调用`DataStream`的`.rebalance()`方法，就可以实现轮询重分区。

![](pic/0013.png)

##### 4.3.4.3.重缩放分区（Rescale）

重缩放分区和轮询分区非常相似。当调用`rescale()`方法时，其实底层也是使用Round-Robin算法进行轮询，但是只会将数据轮询发送到下游并行任务的一部分中。`rescale`的做法是分成小团体，发牌人只给自己团体内的所有人轮流发牌。

![](pic/0014.png)

##### 4.3.4.4.广播（Broadcast）

这种方式其实不应该叫做“重分区”，因为经过广播之后，数据会在不同的分区都保留一份，可能进行重复处理。可以通过调用DataStream的broadcast()方法，将输入数据复制并发送到下游算子的所有并行任务中去。

`stream.broadcast()`

##### 4.3.4.5.全局分区（Global）

全局分区也是一种特殊的分区方式。这种做法非常极端，通过调用.global()方法，会将所有的输入流数据都发送到下游算子的第一个并行子任务中去。这就相当于强行让下游任务并行度变成了1，所以使用这个操作需要非常谨慎，可能对程序造成很大的压力。

##### 4.3.4.6.自定义分区（Custom）

通过实现接口`Partitioner<String> `来实现自定义分区器，可以通过使用 `DataStream.partitionCustom()`方法来自定义分区策略。

#### 4.3.5.分流

所谓“分流”，就是将一条数据流拆分成完全独立的两条、甚至多条流。也就是基于一个DataStream，定义一些筛选条件，将符合条件的数据拣选出来放到对应的流里。

![](pic/0015.png)

##### 4.3.5.1. 简单实现

其实根据条件筛选数据的需求，本身非常容易实现：只要针对同一条流多次独立调用`.filter()`方法进行筛选，就可以得到拆分之后的流了。但是这种方法会将原始数据流复制多份。

##### 4.3.5.2.使用侧输出流

简单来说，只需要调用上下文ctx的.output()方法，就可以输出任意类型的数据了。而侧输出流的标记和提取，都离不开一个“输出标签”（OutputTag），指定了侧输出流的id和类型。

```java
public class SplitStreamByOutputTag {    
public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        SingleOutputStreamOperator<WaterSensor> ds = env.socketTextStream("hadoop102", 7777)
              .map(new WaterSensorMapFunction());


        OutputTag<WaterSensor> s1 = new OutputTag<>("s1", Types.POJO(WaterSensor.class)){};
        OutputTag<WaterSensor> s2 = new OutputTag<>("s2", Types.POJO(WaterSensor.class)){};
       //返回的都是主流
        SingleOutputStreamOperator<WaterSensor> ds1 = ds.process(new ProcessFunction<WaterSensor, WaterSensor>()
        {
            @Override
            public void processElement(WaterSensor value, Context ctx, Collector<WaterSensor> out) throws Exception {

                if ("s1".equals(value.getId())) {
                    ctx.output(s1, value);
                } else if ("s2".equals(value.getId())) {
                    ctx.output(s2, value);
                } else {
                    //主流
                    out.collect(value);
                }

            }
        });

        ds1.print("主流，非s1,s2的传感器");
        SideOutputDataStream<WaterSensor> s1DS = ds1.getSideOutput(s1);
        SideOutputDataStream<WaterSensor> s2DS = ds1.getSideOutput(s2);

        s1DS.printToErr("s1");
        s2DS.printToErr("s2");
        
        env.execute();
 
}
}
```

#### 4.3.6.合流

##### 4.3.6.1.联合（union）

最简单的合流操作，就是直接将多条流合在一起，叫作流的“联合”（union）。联合操作要求必须流中的数据类型必须相同，合并之后的新流会包括所有流中的元素，数据类型不变。

![](pic/0016.png)

基于DataStream直接调用.union()方法，传入其他DataStream作为参数，就可以实现流的联合了；得到的依然是一个DataStream。`stream1.union(stream2, stream3, ...)`

```java
public class UnionExample {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(1);

        DataStreamSource<Integer> ds1 = env.fromElements(1, 2, 3);
        DataStreamSource<Integer> ds2 = env.fromElements(2, 2, 3);
        DataStreamSource<String> ds3 = env.fromElements("2", "2", "3");

        ds1.union(ds2,ds3.map(Integer::valueOf))
           .print();

        env.execute();
    }
}
```



##### 4.3.6.1.连接（connect）

连接操作允许流的数据类型不同。得到的是一个“连接流”，连接流可以看成是两条流的“统一”，被放在了同一个流中；事实上内部仍然保持各自的数据形式不变，彼此之间是相互独立的。

此外还需要定义一个“同处理（co-process）”转换操作，用来说明对于不同来源、类型的数据，怎样分别处理转换，得到统一的输出类型。

```java
public class ConnectDemo {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

//        DataStreamSource<Integer> source1 = env.fromElements(1, 2, 3);
//        DataStreamSource<String> source2 = env.fromElements("a", "b", "c");

        SingleOutputStreamOperator<Integer> source1 = env
                .socketTextStream("hadoop102", 7777)
                .map(i -> Integer.parseInt(i));

        DataStreamSource<String> source2 = env.socketTextStream("hadoop102", 8888);

        /**
         * TODO 使用 connect 合流
         * 1、一次只能连接 2条流
         * 2、流的数据类型可以不一样
         * 3、 连接后可以调用 map、flatmap、process来处理，但是各处理各的
         */
        ConnectedStreams<Integer, String> connect = source1.connect(source2);

        SingleOutputStreamOperator<String> result = connect.map(new CoMapFunction<Integer, String, String>() {
            @Override
            public String map1(Integer value) throws Exception {
                return "来源于数字流:" + value.toString();
            }

            @Override
            public String map2(String value) throws Exception {
                return "来源于字母流:" + value;
            }
        });

        result.print();

        env.execute();    }
}
```

**CoMapFunction**

`ConnectedStreams`有两个类型参数，分别表示内部包含的两条流各自的数据类型,因此调用`.map()`方法时传入的不再是一个简单的`MapFunction`，而是一个`CoMapFunction`，表示分别对两条流中的数据执行map操作。需要实现的方法也非常直白：.map1()就是对第一条流中数据的map操作，.map2()则是针对第二条流。

**CoProcessFunction**

与`CoMapFunction`类似，如果是调用.map()就需要传入一个`CoMapFunction`，需要实现`map1()`、`map2()`两个方法；而调用.process()时，传入的则是一个`CoProcessFunction`。它需要实现的就是`processElement1()`、`processElement2()`两个方法，在每个数据到来时，会根据来源的流调用其中的一个方法进行处理。

**KeyBy**

ConnectedStreams也可以直接调用.keyBy()进行按键分区的操作，得到的还是一个ConnectedStreams：

`connectedStreams.keyBy(keySelector1, keySelector2);`

```java
public class ConnectKeybyDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(2);

        DataStreamSource<Tuple2<Integer, String>> source1 = env.fromElements(
                Tuple2.of(1, "a1"),
                Tuple2.of(1, "a2"),
                Tuple2.of(2, "b"),
                Tuple2.of(3, "c")
        );
        DataStreamSource<Tuple3<Integer, String, Integer>> source2 = env.fromElements(
                Tuple3.of(1, "aa1", 1),
                Tuple3.of(1, "aa2", 2),
                Tuple3.of(2, "bb", 1),
                Tuple3.of(3, "cc", 1)
        );

        ConnectedStreams<Tuple2<Integer, String>, Tuple3<Integer, String, Integer>> connect = source1.connect(source2);

        // 多并行度下，需要根据 关联条件 进行keyby，才能保证key相同的数据到一起去，才能匹配上
        ConnectedStreams<Tuple2<Integer, String>, Tuple3<Integer, String, Integer>> connectKey = connect.keyBy(s1 -> s1.f0, s2 -> s2.f0);

        SingleOutputStreamOperator<String> result = connectKey.process(
                new CoProcessFunction<Tuple2<Integer, String>, Tuple3<Integer, String, Integer>, String>() {
                    // 定义 HashMap，缓存来过的数据，key=id，value=list<数据>
                    Map<Integer, List<Tuple2<Integer, String>>> s1Cache = new HashMap<>();
                    Map<Integer, List<Tuple3<Integer, String, Integer>>> s2Cache = new HashMap<>();

                    @Override
                    public void processElement1(Tuple2<Integer, String> value, Context ctx, Collector<String> out) throws Exception {
                        Integer id = value.f0;
                        // TODO 1.来过的s1数据，都存起来
                        if (!s1Cache.containsKey(id)) {
                            // 1.1 第一条数据，初始化 value的list，放入 hashmap
                            List<Tuple2<Integer, String>> s1Values = new ArrayList<>();
                            s1Values.add(value);
                            s1Cache.put(id, s1Values);
                        } else {
                            // 1.2 不是第一条，直接添加到 list中
                            s1Cache.get(id).add(value);
                        }

                        //TODO 2.根据id，查找s2的数据，只输出 匹配上 的数据
                        if (s2Cache.containsKey(id)) {
                            for (Tuple3<Integer, String, Integer> s2Element : s2Cache.get(id)) {
                                out.collect("s1:" + value + "<--------->s2:" + s2Element);
                            }
                        }
                    }

                    @Override
                    public void processElement2(Tuple3<Integer, String, Integer> value, Context ctx, Collector<String> out) throws Exception {
                        Integer id = value.f0;
                        // TODO 1.来过的s2数据，都存起来
                        if (!s2Cache.containsKey(id)) {
                            // 1.1 第一条数据，初始化 value的list，放入 hashmap
                            List<Tuple3<Integer, String, Integer>> s2Values = new ArrayList<>();
                            s2Values.add(value);
                            s2Cache.put(id, s2Values);
                        } else {
                            // 1.2 不是第一条，直接添加到 list中
                            s2Cache.get(id).add(value);
                        }

                        //TODO 2.根据id，查找s1的数据，只输出 匹配上 的数据
                        if (s1Cache.containsKey(id)) {
                            for (Tuple2<Integer, String> s1Element : s1Cache.get(id)) {
                                out.collect("s1:" + s1Element + "<--------->s2:" + value);
                            }
                        }
                    }
                });

        result.print();

        env.execute();
    }
}
```



### 4.4.输出算子Sink

#### 4.4.1.输出到外部系统

Flink的DataStream API专门提供了向外部写入数据的方法：`addSink`。与`addSource`类似，`addSink`方法对应着一个“Sink”算子，主要就是用来实现与外部系统连接、并将数据提交写入的；Flink程序中所有对外的输出操作，一般都是利用Sink算子完成的。

`addSink`方法同样需要传入一个参数，实现的是SinkFunction接口。在这个接口中只需要重写一个方法`invoke()`，用来将指定的值写入到外部系统中。这个方法在每条数据记录到来时都会调用。

Flink1.12之前是通过`stream.addSink()`方法，Flink 1.12之后通过`stream.sinkTo()`方法。

我们可以看到，像Kafka之类流式系统，Flink提供了完美对接，source/sink两端都能连接，可读可写；而对于Elasticsearch、JDBC等数据存储系统，则只提供了输出写入的sink连接器。

#### 4.4.2.输出到文件

Flink专门提供了一个流式文件系统的连接器：FileSink，为批处理和流处理提供了一个统一的Sink，它可以将分区文件写入Flink支持的文件系统。

FileSink支持行编码（Row-encoded）和批量编码（Bulk-encoded）格式。这两种不同的方式都有各自的构建器（`builder`），可以直接调用FileSink的静态方法：

- 行编码： `FileSink.forRowFormat`（basePath，rowEncoder）。

- 批量编码：` FileSink.forBulkFormat`（basePath，bulkWriterFactory）。

#### 4.4.3.输出到Kafka

步骤：

（1）添加Kafka 连接器依赖；由于我们已经测试过从Kafka数据源读取数据，连接器相关依赖已经引入，这里就不重复介绍了。

（2）启动Kafka集群

（3）编写输出到Kafka的示例代码



#### 4.4.4.输出到MySQL（未完成）

#### 4.4.5.自定义Sink输出（未完成）



## 5.Flink中的时间和窗口

### 5.1.窗口

#### 5.1.1. 概念

Flink想要更加方便高效地处理无界流，一种方式就是将无限数据切割成有限的“数据块”进行处理，这就是所谓的“窗口”（Window）。

在Flink中，窗口其实并不是一个“框”，而是一个“桶“”。**窗口将流切割成有限大小的多个“存储桶（bucket）“**，当到达窗口结束时间时，就对每个桶中收集的数据进行计算处理。

**Flink中的窗口是动态创建的**——当有落在这个窗口区间范围的数据达到时，才创建对应的窗口

![](pic/0017.png)



#### 5.1.2. 分类

**按照驱动类型分类**：

- 时间窗口（time window）：以时间点来定义窗口的开始和结束，所以截取除的就是某一时间段的数据。“定点发车”
- 计数窗口（count window）：基于元素个数来截取数据。“人齐就发车”。

**按照窗口分配数据的规则分类**：

- 滚动窗口（tumbling window）：窗口之间没有重叠，没有间隔。窗口有固定大小。每个数据只会分配到一个窗口。
- 滑动窗口（sliding window）：窗口之间可以有重叠。窗口有固定大小。多一个配置项“滑动步长”。每个数据不一定只会分配到一个窗口。
- 会话窗口（session window）：基于会话来对数据进行分组。最重要的参数是“会话超时时间”，也就是两个窗口之间的最小距离。窗口长度不固定，窗口之间一定不会重叠，至少会留下间隔。
- 全局窗口（global window）：全局有效，将相同的key的所有数据分配到同一个窗口中；窗口没有结束的时候，默认不会触发计算。

#### 5.1.3. API概览

##### 5.1.3.1. 按键分区和非按键分区

**按键分区窗口（Keyed Windows）**

经过按键分区keyBy操作后，数据流会按照key被分为多条逻辑流（logical streams），这就是KeyedStream。基于KeyedStream进行窗口操作时，**窗口计算会在多个并行子任务上同时执行。相同key的数据会被发送到同一个并行子任务，而窗口操作会基于每个key进行单独的处理。**

```java
// 先keyby 再 调用window
stream.keyBy(...)
       .window(...)
```

**非按键分区（Non-Keyed Windows）**

如果没有进行keyBy，那么原始的DataStream就不会分成多条逻辑流。这时窗口逻辑只能在一个任务（task）上执行，就相当于并行度变成了1。代码：

```java
stream.windowAll(...)
```

非按键分区的窗口操作，手动调大窗口算子的并行度也是无效的，`windowAll`本身就是一个非并行的操作。

##### 5.1.3.2.窗口API 的调用

窗口操作主要有两个部分：窗口分配器（Window Assigners）和窗口函数（Window Functions）。

```java
stream.keyBy(<key selector>)
       .window(<window assigner>)
       .aggregate(<window function>)
```

- `.window()`方法需要传入一个窗口分配器，它指明了窗口的类型
- `.aggregate()`方法传入一个窗口函数作为参数，它用来定义窗口具体的处理逻辑。窗口函数调用方法不止这一种。

#### 5.1.4.窗口分配器

**窗口分配器其实就是在指定窗口的类型**

方法：

- 直接调用`.window()`方法，传入一个`WindowAssigner`参数，返回一个`WindowedStream`
- 不需要keyBy，则调用`.windowAll()`方法，参数同上，返回的是`AllWindowedStream`

##### 5.1.4.1.时间窗口

时间窗口是最常用的窗口类型，又可以细分为滚动、滑动和会话三种。

**滚动处理时间窗口**：

窗口分配器由类`TumblingProcessingTimeWindows`提供,需要调用它的静态方法`.of()`，入参表示窗口的大小。滑动窗口同样可以追加第二个参数，用于指定窗口起始点的偏移量，用法与滚动窗口完全一致。

```java
stream.keyBy(...)
       .window(TumblingProcessingTimeWindows.of(Time.seconds(5)))
       .aggregate(...)
```

**滑动处理时间窗口**:

窗口分配器由类`SlidingProcessingTimeWindows`提供，同样需要调用它的静态方法`.of()`，需要传入两个参数`size`和`slide`，分别表示窗口大小和滑动步长。滑动窗口同样可以追加第三个参数，用于指定窗口起始点的偏移量，用法与滚动窗口完全一致。

```java
stream.keyBy(...)
       .window(SlidingProcessingTimeWindows.of(Time.seconds(10)，Time.seconds(5)))
       .aggregate(...)
```

**处理时间会话窗口**

窗口分配器由类`ProcessingTimeSessionWindows`提供，需要调用它的静态方法`.withGap()`或者`.withDynamicGap()`。

这里`.withGap()`方法需要传入一个Time类型的参数size，表示会话的超时时间，也就是最小间隔session gap。

```java
stream.keyBy(...)
       .window(ProcessingTimeSessionWindows.withGap(Time.seconds(10)))
       .aggregate(...)
```

**滚动事件时间窗口**

窗口分配器由类`TumblingEventTimeWindows`提供，用法与滚动处理事件窗口完全一致。

```java
stream.keyBy(...)
       .window(TumblingEventTimeWindows.of(Time.seconds(5)))
       .aggregate(...)
```

**滑动事件时间窗口**

窗口分配器由类`SlidingEventTimeWindows`提供，用法与滑动处理事件窗口完全一致。

```java
stream.keyBy(...)
       .window(SlidingEventTimeWindows.of(Time.seconds(10)，Time.seconds(5)))
       .aggregate(...)
```

**事件时间会话窗口**

窗口分配器由类`EventTimeSessionWindows`提供，用法与处理事件会话窗口完全一致。

```java
stream.keyBy(...)
       .window(EventTimeSessionWindows.withGap(Time.seconds(10)))
       .aggregate(...)
```

##### 5.1.4.2.计数窗口

计数窗口概念非常简单，本身底层是基于全局窗口（Global Window）实现的。在Flink中直接调用`stream.countWindow()`方法。

**滚动计数窗口**

滚动计数窗口只需要传入一个长整型的参数size，表示窗口的大小。

```java
stream.keyBy(...)
       .countWindow(10)
```

**滑动计数窗口**

与滚动计数窗口类似，不过需要在.countWindow()调用时传入两个参数：size和slide，前者表示窗口大小，后者表示滑动步长。

```java
stream.keyBy(...)
       .countWindow(10，3)
```

##### 5.1.4.3.全局窗口

全局窗口是计数窗口的底层实现，一般在需要自定义窗口时使用。它的定义同样是直接调用.window()，分配器由`GlobalWindows`类提供。

```java
stream.keyBy(...)
       .window(GlobalWindows.create());
```

需要注意使用全局窗口，必须自行定义触发器才能实现窗口计算，否则起不到任何作用。

#### 5.1.5.窗口函数

窗口将数据收集起来，而窗口函数将收集的数据进行计算。

##### 5.1.5.1.增量聚合函数（ReduceFunction / AggregateFunction）

我们可以每来一个数据就在之前结果上聚合一次，这就是“增量聚合”。

典型的增量聚合函数有两个：ReduceFunction和AggregateFunction。

**归约函数 ReduceFunction**

聚合状态的类型、输出结果的类型都必须和输入数据类型一样。

```java
public class WindowReduceDemo {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        env
                .socketTextStream("hadoop102", 7777)
                .map(new WaterSensorMapFunction())
                .keyBy(r -> r.getId())
                // 设置滚动事件时间窗口
                .window(TumblingProcessingTimeWindows.of(Time.seconds(10)))
                .reduce(new ReduceFunction<WaterSensor>() {

                    @Override
                    public WaterSensor reduce(WaterSensor value1, WaterSensor value2) throws Exception {
                        System.out.println("调用reduce方法，之前的结果:"+value1 + ",现在来的数据:"+value2);
                        return new WaterSensor(value1.getId(), System.currentTimeMillis(),value1.getVc()+value2.getVc());
                    }
                })
                .print();

        env.execute();
    }
}
```



**聚合函数 AggregateFunction**

`AggregateFunction`可以看作是`ReduceFunction`的通用版本，这里有三种类型：输入类型（IN）、累加器类型（ACC）和输出类型（OUT）。输入类型IN就是输入流中元素的数据类型；累加器类型ACC则是我们进行聚合的中间状态类型；而输出类型当然就是最终计算结果的类型了。

接口中有四个方法：

- `createAccumulator()`：创建一个累加器，这就是为聚合创建了一个初始状态，每个聚合任务只会调用一次。

- `add()`：将输入的元素添加到累加器中。

- `getResult()`：从累加器中提取聚合的输出结果。

- `merge()`：合并两个累加器，并将合并后的状态作为一个累加器返回。

`AggregateFunction`的工作原理是：首先调用`createAccumulator()`为任务初始化一个状态（累加器）；而后每来一个数据就调用一次`add()`方法，对数据进行聚合，得到的结果保存在状态中；等到了窗口需要输出时，再调用`getResult()`方法得到计算结果。

```java
public class WindowAggregateDemo {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);


        SingleOutputStreamOperator<WaterSensor> sensorDS = env
                .socketTextStream("hadoop102", 7777)
                .map(new WaterSensorMapFunction());


        KeyedStream<WaterSensor, String> sensorKS = sensorDS.keyBy(sensor -> sensor.getId());

        // 1. 窗口分配器
        WindowedStream<WaterSensor, String, TimeWindow> sensorWS = sensorKS.window(TumblingProcessingTimeWindows.of(Time.seconds(10)));

        SingleOutputStreamOperator<String> aggregate = sensorWS
                .aggregate(
                        new AggregateFunction<WaterSensor, Integer, String>() {
                            @Override
                            public Integer createAccumulator() {
                                System.out.println("创建累加器");
                                return 0;
                            }

                            @Override
                            public Integer add(WaterSensor value, Integer accumulator) {
                                System.out.println("调用add方法,value="+value);
                                return accumulator + value.getVc();
                            }

                            @Override
                            public String getResult(Integer accumulator) {
                                System.out.println("调用getResult方法");
                                return accumulator.toString();
                            }

                            @Override
                            public Integer merge(Integer a, Integer b) {
                                System.out.println("调用merge方法");
                                return null;
                            }
                        }
                );
        
        aggregate.print();

        env.execute();
    }
}
```

Flink也为窗口的聚合提供了一系列预定义的简单聚合方法，可以直接基于`WindowedStream`调用。主要包括`.sum()/max()/maxBy()/min()/minBy()`，与`KeyedStream`的简单聚合非常相似。它们的底层，其实都是通过`AggregateFunction`来实现的。

##### 5.1.5.2.全窗口函数

全窗口函数需要先收集窗口中的数据，并在内部缓存起来，等到窗口要输出结果的时候再取出数据进行计算。

在Flink中，全窗口函数有两种：WindowFunction和ProcessWindowFunction。

**窗口函数（WindowFunction）**

这个类中可以获取到包含窗口所有数据的可迭代集合（Iterable），还可以拿到窗口（Window）本身的信息。

可以基于WindowedStream调用.apply()方法，传入一个WindowFunction的实现类

```java
stream
    .keyBy(<key selector>)
    .window(<window assigner>)
    .apply(new MyWindowFunction());
```

**处理窗口函数（ProcessWindowFunction）**

`ProcessWindowFunction`可以拿到窗口中的所有数据之外，还可以获取到一个“上下文对象”（Context）。Context对象非常强大，不仅能够获取窗口信息，还可以访问当前的时间和状态信息。这里的时间就包括了处理时间（processing time）和事件时间水位线（event time watermark）。

事实上，`ProcessWindowFunction`是Flink底层API——处理函数（process function）中的一员，关于处理函数我们会在后续章节展开讲解。

```java
public class WindowProcessDemo {
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);


        SingleOutputStreamOperator<WaterSensor> sensorDS = env
                .socketTextStream("hadoop102", 7777)
                .map(new WaterSensorMapFunction());

        KeyedStream<WaterSensor, String> sensorKS = sensorDS.keyBy(sensor -> sensor.getId());

        // 1. 窗口分配器
        WindowedStream<WaterSensor, String, TimeWindow> sensorWS = sensorKS.window(TumblingProcessingTimeWindows.of(Time.seconds(10)));

        SingleOutputStreamOperator<String> process = sensorWS
                .process(
                        new ProcessWindowFunction<WaterSensor, String, String, TimeWindow>() {
                            @Override
                            public void process(String s, Context context, Iterable<WaterSensor> elements, Collector<String> out) throws Exception {
                                long count = elements.spliterator().estimateSize();
                                long windowStartTs = context.window().getStart();
                                long windowEndTs = context.window().getEnd();
                                String windowStart = DateFormatUtils.format(windowStartTs, "yyyy-MM-dd HH:mm:ss.SSS");
                                String windowEnd = DateFormatUtils.format(windowEndTs, "yyyy-MM-dd HH:mm:ss.SSS");

                                out.collect("key=" + s + "的窗口[" + windowStart + "," + windowEnd + ")包含" + count + "条数据===>" + elements.toString());
                            }
                        }
                );

        process.print();

        env.execute();
    }
}
```

##### 5.1.5.3.增量聚合和全窗口函数的结合

在实际应用中，我们往往希望兼具这两者的优点，把它们结合在一起使用。Flink的Window API就给我们实现了这样的用法。

我们之前在调用WindowedStream的.reduce()和.aggregate()方法时，只是简单地直接传入了一个ReduceFunction或AggregateFunction进行增量聚合，此外还可以传入第二个参数：一个全窗口函数，可以是WindowFunction或者ProcessWindowFunction。

```java
// ReduceFunction与WindowFunction结合
public <R> SingleOutputStreamOperator<R> reduce(
        ReduceFunction<T> reduceFunction，WindowFunction<T，R，K，W> function) 

// ReduceFunction与ProcessWindowFunction结合
public <R> SingleOutputStreamOperator<R> reduce(
        ReduceFunction<T> reduceFunction，ProcessWindowFunction<T，R，K，W> function)

// AggregateFunction与WindowFunction结合
public <ACC，V，R> SingleOutputStreamOperator<R> aggregate(
        AggregateFunction<T，ACC，V> aggFunction，WindowFunction<V，R，K，W> windowFunction)

// AggregateFunction与ProcessWindowFunction结合
public <ACC，V，R> SingleOutputStreamOperator<R> aggregate(
        AggregateFunction<T，ACC，V> aggFunction,
        ProcessWindowFunction<V，R，K，W> windowFunction)
```

**处理机制**是：基于第一个参数（增量聚合函数）来处理窗口数据，每来一个数据就做一次聚合；等到窗口需要触发计算时，则调用第二个参数（全窗口函数）的处理逻辑输出结果。

```java
public class CombineWindowDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.addSource(new SelfDefineSource(true))
                .keyBy(Event::getUrl)
                .window(TumblingEventTimeWindows.of(Time.seconds(10)))
                .aggregate(new UrlViewCountIncrFunc(), new UrlViewAllFunc())
                .print();

        env.execute();
    }

    /**
     * 增量 聚合
     */
    private static class UrlViewCountIncrFunc implements AggregateFunction<Event, Long, Long> {

        @Override
        public Long createAccumulator() {
            return 0L;
        }

        @Override
        public Long add(Event value, Long accumulator) {
            return accumulator + 1;
        }

        @Override
        public Long getResult(Long accumulator) {
            return accumulator;
        }

        @Override
        public Long merge(Long a, Long b) {
            return null;
        }
    }

    private static class UrlViewAllFunc extends ProcessWindowFunction<Long, UrlViewCount, String, TimeWindow> {
        @Override
        public void process(String s, ProcessWindowFunction<Long, UrlViewCount, String, TimeWindow>.Context context,
                            Iterable<Long> elements, Collector<UrlViewCount> out) throws Exception {
            out.collect(new UrlViewCount(s, elements.iterator().next(),
                    CommonUtils.getDate(context.window().getStart()),
                    CommonUtils.getDate(context.window().getEnd()))
            );
        }
    }
}
```

#### 5.1.6.其他API

##### 5.1.6.1.触发器（Trigger）

触发器主要是用来控制窗口什么时候触发计算。所谓的“触发计算”，本质上就是执行窗口函数，所以可以认为是计算得到结果并输出的过程。

```java
stream.keyBy(...)
       .window(...)
       .trigger(new MyTrigger())
```

##### 5.1.6.2.移除器（Evictor）

移除器主要用来定义移除某些数据的逻辑。基于WindowedStream调用`.evictor()`方法，就可以传入一个自定义的移除器（Evictor）。`Evictor`是一个接口，不同的窗口类型都有各自预实现的移除器。

```java
stream.keyBy(...)
       .window(...)
```

### 5.2.时间语义

![](pic/0018.png)

**事件时间**：一个数据产生的时间

**处理时间**：数据真正被处理的时刻

从Flink1.12版本开始，Flink已经将事件时间作为默认的时间语义了。

### 5.3.水位线（Watermark）

在窗口的处理过程中，可以基于数据的时间戳，自定义一个**“逻辑时钟”**，其时间的进展靠着新到数据的时间戳来推动。

#### 5.3.1. What

在Flink中，用来衡量事件时间进展的标记，就被称作**“水位线”（Watermark）**。

水位线可以看作一条特殊的数据记录，它是插入到数据流中的一个标记点，主要内容就是一个时间戳，用来指示当前的事件时间。

**水位线插入流中的位置，就应该是在某个数据到来之后；**这样就可以从这个数据中提取时间戳，作为当前水位线的时间戳了。

##### 5.3.1.1. 有序流中的水位线

数据按照生成的先后顺序进入流中。

##### 5.3.1.2.乱序流中的水位线

**乱序+小数据量**

靠数据驱动，每来一个数据就提取它的时间戳、插入一个水位线。乱序情况下插入新的水位线，要先判断一下时间戳是否比之前的大，否则就不再生成新的水位线。**即只有数据的时间戳比当前水位线大，才插入水位线，才能推动管时钟前进。**

![](pic/0019.png)

**乱序+大数据量**

大量数据同时到来，如果考虑效率，可以周期性的生成水位线。这时需要保存之前所有数据的最大时间戳。

![](pic/0020.png)

**乱序+迟到数据**

为了让窗口能够正确收集到迟到数据，可以让窗口等上一段时间，比如2秒。窗口必须等到结束时间+2秒才会触发处理。

![](pic/0021.png)

**水位线的特性：**

- 是插入到数据流中的一个标记，可以认为是一个特殊的数据
- 主要内容是一个时间戳，表示当前事件时间的进展
- 基于数据的时间戳产生
- 单调递增
- 可以设置延迟，正确处理乱序数据
- `Watermart(t)`表示当前流中事件时间已经达到了时间戳`t`，这代表`t`之前的所有数据都到齐了，之后流中不会出现时间戳`<t`的数据

#### 5.3.2. 水位线和窗口的工作原理

窗口的定义是包含起始时间，不包含结束时间，即左闭右开的区间`[start_time, end_time)`。

在Flink中，窗口不是一个“框”，而是一个**”桶“**。窗口可以把流切割成有限大小的多个**“存储桶（bucket）”**。每个数据都会分发到对应的桶中，当到达窗口结束时间时，就会对桶中的数据进行计算处理。

![](pic/0022.png)

- 窗口是动态创建：有落在窗口的数据到达时才创建。
- 到达窗口结束时间时，窗口就触发计算并关闭，事实上“触发计算”和“窗口关闭”两个行为也可以分开。

#### 5.3.3.生成水位线

##### 5.3.3.1.生成总体原则

乱序数据：需要考虑准确性和时延。

##### 5.3.3.2.生成策略

在Flink中，调用流的`.assignTimestampsAndWatermarks()`方法为流数据分配时间戳，生成水位线。`

```java
DataStream<Event> stream = env.addSource(new ClickSource());

DataStream<Event> withTimestampsAndWatermarks = 
stream.assignTimestampsAndWatermarks(<watermark strategy>);
```

`WatermarkStrategy`作为参数，这就是所谓的“水位线生成策略”。`WatermarkStrategy`是一个接口，该接口中包含了一个“时间戳分配器”`TimestampAssigner`和一个“水位线生成器”`WatermarkGenerator`。

```java
public interface WatermarkStrategy<T> 
    extends TimestampAssignerSupplier<T>,
            WatermarkGeneratorSupplier<T>{

    // 负责从流中数据元素的某个字段中提取时间戳，并分配给元素。时间戳的分配是生成水位线的基础。
    @Override
    TimestampAssigner<T> createTimestampAssigner(TimestampAssignerSupplier.Context context);

    // 主要负责按照既定的方式，基于时间戳生成水位线
    @Override
    WatermarkGenerator<T> createWatermarkGenerator(WatermarkGeneratorSupplier.Context context);
}
```



##### 5.3.3.3.Flink内置水位线

**有序流的内置水位线设置**：调用`WatermarkStrategy.forMonotonousTimestamps()`方法

```java
public class WatermarkMonoDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        SingleOutputStreamOperator<WaterSensor> sensorDS = env
                .socketTextStream("hadoop102", 7777)
                .map(new WaterSensorMapFunction());

        // TODO 1.定义Watermark策略
        WatermarkStrategy<WaterSensor> watermarkStrategy = WatermarkStrategy
                // 1.1 指定watermark生成：升序的watermark，没有等待时间
                .<WaterSensor>forMonotonousTimestamps()
                // 1.2 指定 时间戳分配器，从数据中提取
                .withTimestampAssigner(new SerializableTimestampAssigner<WaterSensor>() {
                    @Override
                    public long extractTimestamp(WaterSensor element, long recordTimestamp) {
                        // 返回的时间戳，要 毫秒
                        System.out.println("数据=" + element + ",recordTs=" + recordTimestamp);
                        return element.getTs() * 1000L;
                    }
                });

        // TODO 2. 指定 watermark策略
        SingleOutputStreamOperator<WaterSensor> sensorDSwithWatermark = sensorDS.assignTimestampsAndWatermarks(watermarkStrategy);


        sensorDSwithWatermark.keyBy(sensor -> sensor.getId())
                // TODO 3.使用 事件时间语义 的窗口
                .window(TumblingEventTimeWindows.of(Time.seconds(10)))
                .process(
                        new ProcessWindowFunction<WaterSensor, String, String, TimeWindow>() {

                            @Override
                            public void process(String s, Context context, Iterable<WaterSensor> elements, Collector<String> out) throws Exception {
                                long startTs = context.window().getStart();
                                long endTs = context.window().getEnd();
                                String windowStart = DateFormatUtils.format(startTs, "yyyy-MM-dd HH:mm:ss.SSS");
                                String windowEnd = DateFormatUtils.format(endTs, "yyyy-MM-dd HH:mm:ss.SSS");

                                long count = elements.spliterator().estimateSize();

                                out.collect("key=" + s + "的窗口[" + windowStart + "," + windowEnd + ")包含" + count + "条数据===>" + elements.toString());
                            }
                        }
                )
                .print();

        env.execute();
    }
}
```



**无序流的内置水位线设置**:调用`WatermarkStrategy.forBoundedOutOfOrderness()`,这个方法需要传入一个`maxOutOfOrderness`参数，表示“最大乱序程度”，它表示数据流中乱序数据时间戳的最大差值,就是窗口等待时间；

```java
public class WatermarkOutOfOrdernessDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);


        SingleOutputStreamOperator<WaterSensor> sensorDS = env
                .socketTextStream("hadoop102", 7777)
                .map(new WaterSensorMapFunction());


        // TODO 1.定义Watermark策略
        WatermarkStrategy<WaterSensor> watermarkStrategy = WatermarkStrategy
                // 1.1 指定watermark生成：乱序的，等待3s
                .<WaterSensor>forBoundedOutOfOrderness(Duration.ofSeconds(3))
                // 1.2 指定 时间戳分配器，从数据中提取
                .withTimestampAssigner(
                        (element, recordTimestamp) -> {
                            // 返回的时间戳，要 毫秒
                            System.out.println("数据=" + element + ",recordTs=" + recordTimestamp);
                            return element.getTs() * 1000L;
                        });

        // TODO 2. 指定 watermark策略
        SingleOutputStreamOperator<WaterSensor> sensorDSwithWatermark = sensorDS.assignTimestampsAndWatermarks(watermarkStrategy);


        sensorDSwithWatermark.keyBy(sensor -> sensor.getId())
                // TODO 3.使用 事件时间语义 的窗口
                .window(TumblingEventTimeWindows.of(Time.seconds(10)))
                .process(
                        new ProcessWindowFunction<WaterSensor, String, String, TimeWindow>() {

                            @Override
                            public void process(String s, Context context, Iterable<WaterSensor> elements, Collector<String> out) throws Exception {
                                long startTs = context.window().getStart();
                                long endTs = context.window().getEnd();
                                String windowStart = DateFormatUtils.format(startTs, "yyyy-MM-dd HH:mm:ss.SSS");
                                String windowEnd = DateFormatUtils.format(endTs, "yyyy-MM-dd HH:mm:ss.SSS");

                                long count = elements.spliterator().estimateSize();

                                out.collect("key=" + s + "的窗口[" + windowStart + "," + windowEnd + ")包含" + count + "条数据===>" + elements.toString());
                            }
                        }
                )
                .print();

        env.execute();
    }
}
```

##### 5.3.3.4.自定义水位线生成器

**周期性WM生成器**：周期性生成器一般是通过onEvent()观察判断输入的事件，而在onPeriodicEmit()里发出水位线。

```java
// 自定义水位线的产生
public class CustomPeriodicWatermarkExample {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env
                .addSource(new ClickSource())
                .assignTimestampsAndWatermarks(new CustomWatermarkStrategy())
                .print();

        env.execute();
    }

    public static class CustomWatermarkStrategy implements WatermarkStrategy<Event> {

        @Override
        public TimestampAssigner<Event> createTimestampAssigner(TimestampAssignerSupplier.Context context) {

            return new SerializableTimestampAssigner<Event>() {

                @Override
                public long extractTimestamp(Event element，long recordTimestamp) {
                    return element.timestamp; // 告诉程序数据源里的时间戳是哪一个字段
                }
            };
        }

        @Override
        public WatermarkGenerator<Event> createWatermarkGenerator(WatermarkGeneratorSupplier.Context context) {
            return new CustomBoundedOutOfOrdernessGenerator();
        }
    }

    public static class CustomBoundedOutOfOrdernessGenerator implements WatermarkGenerator<Event> {

        private Long delayTime = 5000L; // 延迟时间
        private Long maxTs = -Long.MAX_VALUE + delayTime + 1L; // 观察到的最大时间戳

        @Override
        public void onEvent(Event event，long eventTimestamp，WatermarkOutput output) {
            // 每来一条数据就调用一次
            maxTs = Math.max(event.timestamp，maxTs); // 更新最大时间戳
        }

        @Override
        public void onPeriodicEmit(WatermarkOutput output) {
            // 发射水位线，默认200ms调用一次
            output.emitWatermark(new Watermark(maxTs - delayTime - 1L));
        }
    }
}
```

在`onPeriodicEmit()`里调用`output.emitWatermark()`，就可以发出水位线了；这个方法由系统框架周期性地调用，默认200ms一次。

如果想修改默认周期时间，可以通过下面方法修改。例如：修改为400ms

`env.getConfig().setAutoWatermarkInterval(400L)`;

**断点式水位线生成器（Punctuated Generator）**

断点式生成器会不停地检测`onEvent()`中的事件，当发现带有水位线信息的事件时，就立即发出水位线。我们把发射水位线的逻辑写在`onEvent`方法当中即可。

**在数据源中发送水位线**

我们也可以在自定义的数据源中抽取事件时间，然后发送水位线。这里要注意的是，在自定义数据源中发送了水位线以后，就不能再在程序中使用assignTimestampsAndWatermarks方法来生成水位线了。

```java
env.fromSource(
kafkaSource, WatermarkStrategy.forBoundedOutOfOrderness(Duration.ofSeconds(3)), "kafkasource"
)
```

#### 5.3.4.水位线传递

![](pic/0023.png)

在流处理中，上游任务处理完水位线、时钟改变之后，要把当前的水位线再次发出，广播给所有的下游子任务。**而当一个任务接收到多个上游并行任务传递来的水位线时，应该以最小的那个作为当前任务的事件时钟。**

```java
public class WatermarkIdlenessDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(2);


        // 自定义分区器：数据%分区数，只输入奇数，都只会去往map的一个子任务
        SingleOutputStreamOperator<Integer> socketDS = env
                .socketTextStream("hadoop102", 7777)
                .partitionCustom(new MyPartitioner(), r -> r)
                .map(r -> Integer.parseInt(r))
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<Integer>forMonotonousTimestamps()
                                .withTimestampAssigner((r, ts) -> r * 1000L)
                                .withIdleness(Duration.ofSeconds(5))  //空闲等待5s
                );


        // 分成两组： 奇数一组，偶数一组 ， 开10s的事件时间滚动窗口
        socketDS
                .keyBy(r -> r % 2)
                .window(TumblingEventTimeWindows.of(Time.seconds(10)))
                .process(new ProcessWindowFunction<Integer, String, Integer, TimeWindow>() {
                    @Override
                    public void process(Integer integer, Context context, Iterable<Integer> elements, Collector<String> out) throws Exception {
                        long startTs = context.window().getStart();
                        long endTs = context.window().getEnd();
                        String windowStart = DateFormatUtils.format(startTs, "yyyy-MM-dd HH:mm:ss.SSS");
                        String windowEnd = DateFormatUtils.format(endTs, "yyyy-MM-dd HH:mm:ss.SSS");

                        long count = elements.spliterator().estimateSize();

                        out.collect("key=" + integer + "的窗口[" + windowStart + "," + windowEnd + ")包含" + count + "条数据===>" + elements.toString());

                    }
                })
                .print();


        env.execute();
    }
}
```



#### 5.3.5.迟到数据的处理

##### 5.3.5.1.推迟水印推进

在水印产生时，设置一个乱序容忍度，推迟系统时间的推进，保证窗口计算被延迟执行，为乱序的数据争取更多的时间进入窗口。

```java
WatermarkStrategy.forBoundedOutOfOrderness(Duration.ofSeconds(10));
```

##### 5.3.5.2.设置窗口延迟关闭

 Flink的窗口，也允许迟到数据。当触发了窗口计算后，会先计算当前的结果，但是此时并不会关闭窗口。

以后每来一条迟到数据，就触发一次这条数据所在窗口计算(增量计算)。直到wartermark 超过了窗口结束时间+推迟时间，此时窗口会真正关闭。

```java
.window(TumblingEventTimeWindows.of(Time.seconds(5)))
.allowedLateness(Time.seconds(3))
```

##### 5.3.5.3.用侧流接收迟到的数据

在上述中添加``，使用测流接收迟到数据。

```java
.windowAll(TumblingEventTimeWindows.of(Time.seconds(5)))
.allowedLateness(Time.seconds(3))
.sideOutputLateData(lateWS)
```

```java
public class WaterMarkLateDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);


        SingleOutputStreamOperator<WaterSensor> sensorDS = env
                .socketTextStream("hadoop102", 7777)
                .map(new WaterSensorMapFunction());

        WatermarkStrategy<WaterSensor> watermarkStrategy = WatermarkStrategy
                .<WaterSensor>forBoundedOutOfOrderness(Duration.ofSeconds(3))
                .withTimestampAssigner((element, recordTimestamp) -> element.getTs() * 1000L);

        SingleOutputStreamOperator<WaterSensor> sensorDSwithWatermark = sensorDS.assignTimestampsAndWatermarks(watermarkStrategy);


        OutputTag<WaterSensor> lateTag = new OutputTag<>("late-data", Types.POJO(WaterSensor.class));

        SingleOutputStreamOperator<String> process = sensorDSwithWatermark.keyBy(sensor -> sensor.getId())
                .window(TumblingEventTimeWindows.of(Time.seconds(10)))
                .allowedLateness(Time.seconds(2)) // 推迟2s关窗
                .sideOutputLateData(lateTag) // 关窗后的迟到数据，放入侧输出流
                .process(
                        new ProcessWindowFunction<WaterSensor, String, String, TimeWindow>() {

                            @Override
                            public void process(String s, Context context, Iterable<WaterSensor> elements, Collector<String> out) throws Exception {
                                long startTs = context.window().getStart();
                                long endTs = context.window().getEnd();
                                String windowStart = DateFormatUtils.format(startTs, "yyyy-MM-dd HH:mm:ss.SSS");
                                String windowEnd = DateFormatUtils.format(endTs, "yyyy-MM-dd HH:mm:ss.SSS");

                                long count = elements.spliterator().estimateSize();

                                out.collect("key=" + s + "的窗口[" + windowStart + "," + windowEnd + ")包含" + count + "条数据===>" + elements.toString());
                            }
                        }
                );


        process.print();
        // 从主流获取侧输出流，打印
        process.getSideOutput(lateTag).printToErr("关窗后的迟到数据");

        env.execute();
    }

}
```



### 5.4.基于时间的合流——双流联结（join）

Flink中两条流的`connect`操作，就可以通过`keyBy`指定键进行分组后合并，实现了类似于SQL中的join操作；

如果我们希望统计固定时间内两条流数据的匹配情况，那就需要自定义来实现——其实这完全可以用窗口（window）来表示。为了更方便地实现基于时间的合流操作，Flink的DataStrema API提供了内置的join算子。

#### 5.4.1.窗口联结（window join）

Flink为基于一段时间的双流合并专门提供了一个窗口联结算子，可以定义时间窗口，并将两条流中共享一个公共键（key）的数据放在窗口中进行配对处理。

窗口联结在代码中的实现，首先需要调用`DataStream`的`.join()`方法来合并两条流，得到一个`JoinedStreams`；接着通过`.where()`和`.equalTo()`方法指定两条流中联结的`key`；然后通过`.window()`开窗口，并调用`.apply()`传入联结窗口函数进行处理计算。通用调用形式如下：

```java
stream1.join(stream2)
        .where(<KeySelector>)
        .equalTo(<KeySelector>)
        .window(<WindowAssigner>)
        .apply(<JoinFunction>)
```

`.where()`的参数是键选择器（`KeySelector`），用来指定第一条流中的`key`；而`.equalTo()`传入的`KeySelector`则指定了第二条流中的key。两者相同的元素，如果在同一窗口中，就可以匹配起来，并通过一个“联结函数”（`JoinFunction`）进行处理了。

`.window()`传入的就是窗口分配器，之前讲到的三种时间窗口都可以用在这里：滚动窗口（tumbling window）、滑动窗口（sliding window）和会话窗口（session window）。

调用`.apply()`可以看作实现了一个特殊的窗口函数。注意这里只能调用`.apply()`，没有其他替代的方法。

传入的`JoinFunction`也是一个函数类接口，使用时需要实现内部的`.join()`方法。这个方法有两个参数，分别表示两条流中成对匹配的数据。

而Flink中的`window join`，同样类似于`inner join`。也就是说，最后处理输出的，只有两条流中数据按key配对成功的那些；如果某个窗口中一条流的数据没有任何另一条流的数据匹配，那么就不会调用`JoinFunction`的`.join()`方法，也就没有任何输出了。

**demo**:

```java
public class WindowJoinDemo {
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        SingleOutputStreamOperator<Tuple2<String, Integer>> ds1 = env
                .fromElements(
                        Tuple2.of("a", 1),
                        Tuple2.of("a", 2),
                        Tuple2.of("b", 3),
                        Tuple2.of("c", 4)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<Tuple2<String, Integer>>forMonotonousTimestamps()
                                .withTimestampAssigner((value, ts) -> value.f1 * 1000L)
                );


        SingleOutputStreamOperator<Tuple3<String, Integer,Integer>> ds2 = env
                .fromElements(
                        Tuple3.of("a", 1,1),
                        Tuple3.of("a", 11,1),
                        Tuple3.of("b", 2,1),
                        Tuple3.of("b", 12,1),
                        Tuple3.of("c", 14,1),
                        Tuple3.of("d", 15,1)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<Tuple3<String, Integer,Integer>>forMonotonousTimestamps()
                                .withTimestampAssigner((value, ts) -> value.f1 * 1000L)
                );

        // TODO window join
        // 1. 落在同一个时间窗口范围内才能匹配
        // 2. 根据keyby的key，来进行匹配关联
        // 3. 只能拿到匹配上的数据，类似有固定时间范围的inner join
        DataStream<String> join = ds1.join(ds2)
                .where(r1 -> r1.f0)  // ds1的keyby
                .equalTo(r2 -> r2.f0) // ds2的keyby
                .window(TumblingEventTimeWindows.of(Time.seconds(10)))
                .apply(new JoinFunction<Tuple2<String, Integer>, Tuple3<String, Integer, Integer>, String>() {
                    /**
                     * 关联上的数据，调用join方法
                     * @param first  ds1的数据
                     * @param second ds2的数据
                     * @return
                     * @throws Exception
                     */
                    @Override
                    public String join(Tuple2<String, Integer> first, Tuple3<String, Integer, Integer> second) throws Exception {
                        return first + "<----->" + second;
                    }
                });

        join.print();

        env.execute();
    }
}
```



#### 5.4.2.间隔联结（interval join）

**间隔联结的思路**就是针对一条流的每个数据，开辟出其时间戳前后的一段时间间隔，看这期间是否有来自另一条流的数据匹配。

**原理**：给定两个时间点，分别叫作间隔的“上界”（`upperBound`）和“下界”（`lowerBound`）；于是对于一条流（不妨叫作A）中的任意一个数据元素a，就可以开辟一段时间间隔：`[a.timestamp + lowerBound, a.timestamp + upperBound]`,即以a的时间戳为中心，下至下界点、上至上界点的一个闭区间：我们就把这段时间作为可以匹配另一条流数据的“窗口”范围。所以对于另一条流（不妨叫B）中的数据元素b，如果它的时间戳落在了这个区间范围内，a和b就可以成功配对，进而进行计算输出结果。所以匹配的条件为：

`a.timestamp + lowerBound <= b.timestamp <= a.timestamp + upperBound`

![](pic/0024.png)

所以我们可以看到，间隔联结同样是一种内连接（`inner join`）。与窗口联结不同的是，`interval join`做匹配的时间段是基于流中数据的，所以并不确定；而且流B中的数据可以不只在一个区间内被匹配。

**API调用**：

基于`KeyedStream`的联结（`join`）操作。

`DataStream`在`keyBy`得到`KeyedStream`之后，可以调用`.intervalJoin()`来合并两条流，传入的参数同样是一个`KeyedStream`，两者的key类型应该一致；得到的是一个`IntervalJoin`类型。

先通过`.between()`方法指定间隔的上下界，再调用`.process()`方法，定义对匹配数据对的处理操作。调用`.process()`需要传入一个处理函数，这是处理函数家族的最后一员：“处理联结函数”`ProcessJoinFunction`。

```java
stream1
    .keyBy(<KeySelector>)
    .intervalJoin(stream2.keyBy(<KeySelector>))
    .between(Time.milliseconds(-2), Time.milliseconds(1))
    .process (new ProcessJoinFunction<Integer, Integer, String(){

        @Override
        public void processElement(Integer left, Integer right, Context ctx, Collector<String> out) {
            out.collect(left + "," + right);
        }
    });
```

抽象类`ProcessJoinFunction`就像是`ProcessFunction`和`JoinFunction`的结合，内部同样有一个抽象方法`.processElement()`。与其他处理函数不同的是，它多了一个参数，这自然是因为有来自两条流的数据。参数中`left`指的就是第一条流中的数据，`right`则是第二条流中与它匹配的数据。每当检测到一组匹配，就会调用这里的`.processElement()`方法，经处理转换之后输出结果。

**Demo**

```java
public class IntervalJoinDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        SingleOutputStreamOperator<Tuple2<String, Integer>> ds1 = env
                .fromElements(
                        Tuple2.of("a", 1),
                        Tuple2.of("a", 2),
                        Tuple2.of("b", 3),
                        Tuple2.of("c", 4)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<Tuple2<String, Integer>>forMonotonousTimestamps()
                                .withTimestampAssigner((value, ts) -> value.f1 * 1000L)
                );


        SingleOutputStreamOperator<Tuple3<String, Integer, Integer>> ds2 = env
                .fromElements(
                        Tuple3.of("a", 1, 1),
                        Tuple3.of("a", 11, 1),
                        Tuple3.of("b", 2, 1),
                        Tuple3.of("b", 12, 1),
                        Tuple3.of("c", 14, 1),
                        Tuple3.of("d", 15, 1)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<Tuple3<String, Integer, Integer>>forMonotonousTimestamps()
                                .withTimestampAssigner((value, ts) -> value.f1 * 1000L)
                );

        // TODO interval join
        //1. 分别做keyby，key其实就是关联条件
        KeyedStream<Tuple2<String, Integer>, String> ks1 = ds1.keyBy(r1 -> r1.f0);
        KeyedStream<Tuple3<String, Integer, Integer>, String> ks2 = ds2.keyBy(r2 -> r2.f0);

        //2. 调用 interval join
        ks1.intervalJoin(ks2)
                .between(Time.seconds(-2), Time.seconds(2))
                .process(
                        new ProcessJoinFunction<Tuple2<String, Integer>, Tuple3<String, Integer, Integer>, String>() {
                            /**
                             * 两条流的数据匹配上，才会调用这个方法
                             * @param left  ks1的数据
                             * @param right ks2的数据
                             * @param ctx   上下文
                             * @param out   采集器
                             * @throws Exception
                             */
                            @Override
                            public void processElement(Tuple2<String, Integer> left, Tuple3<String, Integer, Integer> right, Context ctx, Collector<String> out) throws Exception {
                                // 进入这个方法，是关联上的数据
                                out.collect(left + "<------>" + right);
                            }
                        })
                .print();



        env.execute();
    }
}
```

## 6.处理函数

流处理API，无论是基本的转换、聚合，还是更为复杂的窗口操作，其实都是基于DataStream进行转换的，所以可以统称为DataStream API。

在Flink更底层，我们可以不定义任何具体的算子（比如`map`，`filter`，或者`window`），而只是提炼出一个**统一的“处理”（`process`）操作**——它是所有转换算子的一个概括性的表达，可以自定义处理逻辑，所以这一层接口就被叫作“处理函数”（`process function`）。

![](pic/0001.png)

### 6.1.基本处理函数 ProcessFunction

#### 6.1.1. 处理函数功能和使用

处理函数提供了一个“定时服务”（TimerService），我们可以通过它访问流中的事件（event）、时间戳（timestamp）、水位线（watermark），甚至可以注册“定时事件”。而且处理函数继承了`AbstractRichFunction`抽象类，所以拥有**富函数类**的所有特性，同样可以访问状态（state）和其他运行时信息。处理函数还可以直接将数据输出到侧输出流（side output）中。

处理函数的使用与基本的转换操作类似，只需要直接基于`DataStream`调用`.process()`方法就可以了。方法需要传入一个`ProcessFunction`作为参数，用来定义处理逻辑。



#### 6.1.2.ProcessFunction解析

```java
public abstract class ProcessFunction<I, O> extends AbstractRichFunction {

    ...
    public abstract void processElement(I value, Context ctx, Collector<O> out) throws Exception;

    public void onTimer(long timestamp, OnTimerContext ctx, Collector<O> out) throws Exception {}
    ...

}
```

抽象类`ProcessFunction`继承了`AbstractRichFunction`，有两个泛型类型参数：`I`表示`Input`，也就是输入的数据类型；`O`表示`Output`，也就是处理完成之后输出的数据类型。

内部单独定义了两个方法：一个是必须要实现的抽象方法`.processElement()`；另一个是非抽象方法`.onTimer()`。

**抽象方法`.processElement()`**

- 用于“处理元素”，定义了处理的核心逻辑。这个方法对于流中的每个元素都会调用一次.

- 参数包括三个：输入数据值`value`，上下文`ctx`，以及“收集器”（Collector）`out`。方法没有返回值，处理之后的输出数据是通过收集器out来定义的。
  - `value`：当前**流中的输入元素**，也就是正在处理的数据，类型与流中数据类型一致。
  - `ctx`：类型是`ProcessFunction`中定义的内部抽象类Context，**表示当前运行的上下文**，可以获取到当前的时间戳，并提供了用于查询时间和注册定时器的“定时服务”（TimerService），以及可以将数据发送到“侧输出流”（side output）的方法`.output()`。
  - `out`：“收集器”（类型为Collector）**，用于返回输出数据**。使用方式与`flatMap`算子中的收集器完全一样，直接调用`out.collect()`方法就可以向下游发出一个数据。这个方法可以多次调用，也可以不调用。

`ProcessFunction`可以轻松实现`flatMap、map、filter`这样的基本转换功能；而通过富函数提供的获取上下文方法`.getRuntimeContext()`，也可以自定义状态（state）进行处理，这也就能实现聚合操作的功能了。

**非抽象方法`.onTimer()`**

方法只有在注册好的定时器触发的时候才会调用，而定时器是通过“定时服务”TimerService来注册的。

本质上是一个基于时间的“回调”（callback）方法，通过时间的进展来触发；在事件时间语义下就是由水位线（watermark）来触发了。

定时方法.onTimer()也有三个参数：时间戳（timestamp），上下文（ctx），以及收集器（out）

- timestamp是指设定好的触发时间，事件时间语义下当然就是水位线了

既然有`.onTimer()`方法做定时触发，我们用`ProcessFunction`也可以自定义数据按照时间分组、定时触发计算输出结果；这其实就实现了窗口（window）的功能。

**注意：在Flink中，只有“按键分区流”`KeyedStream`才支持设置定时器的操作。**

#### 6.1.3.处理函数分类

对于不同类型的流，其实都可以直接调用`.process()`方法进行自定义处理，这时传入的参数就都叫作处理函数。

Flink提供了8个不同的处理函数：

- `ProcessFunction` 最基本的处理函数，基于`DataStream`直接调用`.process()`时作为参数传入。
- `KeyedProcessFunction`对流按键分区后的处理函数，基于`KeyedStream`调用`.process()`时作为参数传入。要想使用定时器，比如基于`KeyedStream`。
- `ProcessWindowFunction`
- `ProcessAllWindowFunction`
- `CoProcessFunction`：合并（connect）两条流之后的处理函数，基于ConnectedStreams调用.process()时作为参数传入。
- `ProcessJoinFunction`：间隔连接（interval join）两条流之后的处理函数，基于IntervalJoined调用.process()时作为参数传入。
- `BroadcastProcessFunction`：广播连接流处理函数，基于`BroadcastConnectedStream`调用.process()时作为参数传入。这里的“广播连接流”`BroadcastConnectedStream`，是一个未keyBy的普通DataStream与一个广播流（BroadcastStream）做连接（conncet）之后的产物。
- `KeyedBroadcastProcessFunction`：按键分区的广播连接流处理函数，同样是基于BroadcastConnectedStream调用.process()时作为参数传入。

### 6.2.按键分区处理函数 KeyedProcessFunction

#### 6.2.1.定时器（Timer）和定时服务（TimerService）

在`.onTimer()`方法中可以实现定时处理的逻辑，而它能触发的前提，就是之前曾经注册过定时器、并且现在已经到了触发时间。注册定时器的功能，是通过上下文中提供的“定时服务”来实现的。

定时服务与当前运行的环境有关。`ProcessFunction`的上下文（Context）中提供了`.timerService()`方法，可以直接返回一个`TimerService`对象。`TimerService`是基础服务接口，包含六个方法：

```java
// 获取当前的处理时间
long currentProcessingTime();

// 获取当前的水位线（事件时间）
long currentWatermark();

// 注册处理时间定时器，当处理时间超过time时触发
void registerProcessingTimeTimer(long time);

// 注册事件时间定时器，当水位线超过time时触发
void registerEventTimeTimer(long time);

// 删除触发时间为time的处理时间定时器
void deleteProcessingTimeTimer(long time);

// 删除触发时间为time的处理时间定时器
void deleteEventTimeTimer(long time);
```

六个方法可以分成两大类：基于处理时间和基于事件时间。而对应的操作主要有三个：获取当前时间，注册定时器，以及删除定时器。

**尽管处理函数中都可以直接访问TimerService，不过只有基于KeyedStream的处理函数，才能去调用注册和删除定时器的方法；未作按键分区的DataStream不支持定时器操作，只能获取当前时间。**

`TimerService`会以键（key）和时间戳为标准，对定时器进行去重；也就是说对于每个key和时间戳，最多只有一个定时器，如果注册了多次，`onTimer()`方法也将只被调用一次。

#### 6.2.2.KeyedProcessFunction案例

```java
public class KeyedProcessTimerDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);


        SingleOutputStreamOperator<WaterSensor> sensorDS = env
                .socketTextStream("hadoop102", 7777)
                .map(new WaterSensorMapFunction())
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<WaterSensor>forBoundedOutOfOrderness(Duration.ofSeconds(3))
                                .withTimestampAssigner((element, ts) -> element.getTs() * 1000L)
                );


        KeyedStream<WaterSensor, String> sensorKS = sensorDS.keyBy(sensor -> sensor.getId());

        // TODO Process:keyed
        SingleOutputStreamOperator<String> process = sensorKS.process(
                new KeyedProcessFunction<String, WaterSensor, String>() {
                    /**
                     * 来一条数据调用一次
                     * @param value
                     * @param ctx
                     * @param out
                     * @throws Exception
                     */
                    @Override
                    public void processElement(WaterSensor value, Context ctx, Collector<String> out) throws Exception {
                        //获取当前数据的key
                        String currentKey = ctx.getCurrentKey();

                        // TODO 1.定时器注册
                        TimerService timerService = ctx.timerService();

                        // 1、事件时间的案例
                        Long currentEventTime = ctx.timestamp(); // 数据中提取出来的事件时间
                        timerService.registerEventTimeTimer(5000L);
                        System.out.println("当前key=" + currentKey + ",当前时间=" + currentEventTime + ",注册了一个5s的定时器");

                        // 2、处理时间的案例
//                        long currentTs = timerService.currentProcessingTime();
//                        timerService.registerProcessingTimeTimer(currentTs + 5000L);
//                        System.out.println("当前key=" + currentKey + ",当前时间=" + currentTs + ",注册了一个5s后的定时器");


                        // 3、获取 process的 当前watermark
//                        long currentWatermark = timerService.currentWatermark();
//                        System.out.println("当前数据=" + value + ",当前watermark=" + currentWatermark);



                        // 注册定时器： 处理时间、事件时间
//                        timerService.registerProcessingTimeTimer();
//                        timerService.registerEventTimeTimer();
                        // 删除定时器： 处理时间、事件时间
//                        timerService.deleteEventTimeTimer();
//                        timerService.deleteProcessingTimeTimer();

                        // 获取当前时间进展： 处理时间-当前系统时间，  事件时间-当前watermark
//                        long currentTs = timerService.currentProcessingTime();
//                        long wm = timerService.currentWatermark();
                    }


                    /**
                     * TODO 2.时间进展到定时器注册的时间，调用该方法
                     * @param timestamp 当前时间进展，就是定时器被触发时的时间
                     * @param ctx       上下文
                     * @param out       采集器
                     * @throws Exception
                     */
                    @Override
                    public void onTimer(long timestamp, OnTimerContext ctx, Collector<String> out) throws Exception {
                        super.onTimer(timestamp, ctx, out);
                        String currentKey = ctx.getCurrentKey();

                        System.out.println("key=" + currentKey + "现在时间是" + timestamp + "定时器触发");
                    }
                }
        );

        process.print();

        env.execute();
    }

}
```



### 6.3.窗口处理函数 ProcessWindowFunction

#### 6.3.1.窗口处理函数的使用

窗口处理函数`ProcessWindowFunction`的使用与其他窗口函数类似，也是基于`WindowedStream`直接调用方法就可以，只不过这时调用的是`.process()`。

```java
stream.keyBy( t -> t.f0 )
        .window( TumblingEventTimeWindows.of(Time.seconds(10)) )
        .process(new MyProcessWindowFunction())
```

#### 6.3.2.ProcessWindowFunction解析

`ProcessWindowFunction`既是处理函数又是全窗口函数。

```java
public abstract class ProcessWindowFunction<IN, OUT, KEY, W extends Window> extends AbstractRichFunction {
    ...

    public abstract void process(
            KEY key, Context context, Iterable<IN> elements, Collector<OUT> out) throws Exception;

    public void clear(Context context) throws Exception {}

    public abstract class Context implements java.io.Serializable {...}
}
```

`ProcessWindowFunction`依然是一个继承了`AbstractRichFunction`的抽象类，它有四个类型参数：

- IN：input，数据流中窗口任务的输入数据类型。
- OUT：output，窗口任务进行计算之后的输出数据类型。
- KEY：数据中键key的类型。
- W：窗口的类型，是Window的子类型。一般情况下我们定义时间窗口，W就是TimeWindow。

`ProcessWindowFunction`里面处理数据的核心方法`.process()`。方法包含四个参数。

- key：窗口做统计计算基于的键，也就是之前keyBy用来分区的字段。
- context：当前窗口进行计算的上下文，它的类型就是`ProcessWindowFunction`内部定义的抽象类Context。
- elements：窗口收集到用来计算的所有数据，这是一个可迭代的集合类型。
- out：用来发送数据输出计算结果的收集器，类型为Collector。

内部类`Context `的代码：

```java
    public abstract class Context implements java.io.Serializable {
        /** Returns the window that is being evaluated. */
        public abstract W window();

        /** Returns the current processing time. */
        public abstract long currentProcessingTime();

        /** Returns the current event-time watermark. */
        public abstract long currentWatermark();

        /**
         * State accessor for per-key and per-window state.
         *
         * <p><b>NOTE:</b>If you use per-window state you have to ensure that you clean it up by
         * implementing {@link ProcessWindowFunction#clear(Context)}.
         */
        public abstract KeyedStateStore windowState();

        /** State accessor for per-key global state. */
        public abstract KeyedStateStore globalState();

        /**
         * Emits a record to the side output identified by the {@link OutputTag}.
         *
         * @param outputTag the {@code OutputTag} that identifies the side output to emit to.
         * @param value The record to emit.
         */
        public abstract <X> void output(OutputTag<X> outputTag, X value);
    }
```

除了可以通过`.output()`方法定义侧输出流不变外，其他部分都有所变化。这里不再持有TimerService对象，只能通过`currentProcessingTime()`和`currentWatermark()`来获取当前时间，所以失去了设置定时器的功能；另外由于当前不是只处理一个数据，所以也不再提供`.timestamp()`方法。

与此同时，也增加了一些获取其他信息的方法：比如可以通过`.window()`直接获取到当前的窗口对象，也可以通过`.windowState()`和`.globalState()`获取到当前自定义的窗口状态和全局状态。注意这里的“窗口状态”是自定义的，不包括窗口本身已经有的状态，针对当前key、当前窗口有效；而“全局状态”同样是自定义的状态，针对当前key的所有窗口有效。



`ProcessWindowFunction`中除了`.process()`方法外，并没有`.onTimer()`方法，而是多出了一个.clear()方法。从名字就可以看出，这主要是方便我们进行窗口的清理工作。如果我们自定义了窗口状态，那么必须在`.clear()`方法中进行显式地清除，避免内存溢出。



至于另一种窗口处理函数`ProcessAllWindowFunction`，它的用法非常类似。区别在于它基于的是`AllWindowedStream`，相当于对没有`keyBy`的数据流直接开窗并调用`.process()`方法：

```java
stream.windowAll( TumblingEventTimeWindows.of(Time.seconds(10)) )
    .process(new MyProcessAllWindowFunction())
```



### 6.4.案例TopN

**案例需求：**实时统计一段时间内的出现次数最多的水位。



#### 6.4.1.使用 ProcessAllWindowFunction

**思路一：**一种最简单的想法是，我们干脆不区分不同水位，而是将所有访问数据都收集起来，统一进行统计计算。所以可以不做`keyBy`，直接基于`DataStream`开窗，然后使用全窗口函数`ProcessAllWindowFunction`来进行处理。

```java
public class ProcessAllWindowTopNDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);


        SingleOutputStreamOperator<WaterSensor> sensorDS = env
                .socketTextStream("hadoop102", 7777)
                .map(new WaterSensorMapFunction())
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<WaterSensor>forBoundedOutOfOrderness(Duration.ofSeconds(3))
                                .withTimestampAssigner((element, ts) -> element.getTs() * 1000L)
                );


        // 最近10秒= 窗口长度， 每5秒输出 = 滑动步长
        // TODO 思路一： 所有数据到一起， 用hashmap存， key=vc，value=count值
        sensorDS.windowAll(SlidingEventTimeWindows.of(Time.seconds(10), Time.seconds(5)))
                .process(new MyTopNPAWF())
                .print();


        env.execute();
    }


    public static class MyTopNPAWF extends ProcessAllWindowFunction<WaterSensor, String, TimeWindow> {

        @Override
        public void process(Context context, Iterable<WaterSensor> elements, Collector<String> out) throws Exception {
            // 定义一个hashmap用来存，key=vc，value=count值
            Map<Integer, Integer> vcCountMap = new HashMap<>();
            // 1.遍历数据, 统计 各个vc出现的次数
            for (WaterSensor element : elements) {
                Integer vc = element.getVc();
                if (vcCountMap.containsKey(vc)) {
                    // 1.1 key存在，不是这个key的第一条数据，直接累加
                    vcCountMap.put(vc, vcCountMap.get(vc) + 1);
                } else {
                    // 1.2 key不存在，初始化
                    vcCountMap.put(vc, 1);
                }
            }

            // 2.对 count值进行排序: 利用List来实现排序
            List<Tuple2<Integer, Integer>> datas = new ArrayList<>();
            for (Integer vc : vcCountMap.keySet()) {
                datas.add(Tuple2.of(vc, vcCountMap.get(vc)));
            }
            // 对List进行排序，根据count值 降序
            datas.sort(new Comparator<Tuple2<Integer, Integer>>() {
                @Override
                public int compare(Tuple2<Integer, Integer> o1, Tuple2<Integer, Integer> o2) {
                    // 降序， 后 减 前
                    return o2.f1 - o1.f1;
                }
            });

            // 3.取出 count最大的2个 vc
            StringBuilder outStr = new StringBuilder();

            outStr.append("================================\n");
            // 遍历 排序后的 List，取出前2个， 考虑可能List不够2个的情况  ==》 List中元素的个数 和 2 取最小值
            for (int i = 0; i < Math.min(2, datas.size()); i++) {
                Tuple2<Integer, Integer> vcCount = datas.get(i);
                outStr.append("Top" + (i + 1) + "\n");
                outStr.append("vc=" + vcCount.f0 + "\n");
                outStr.append("count=" + vcCount.f1 + "\n");
                outStr.append("窗口结束时间=" + DateFormatUtils.format(context.window().getEnd(), "yyyy-MM-dd HH:mm:ss.SSS") + "\n");
                outStr.append("================================\n");
            }

            out.collect(outStr.toString());

        }
    }

}
```

#### 6.4.2.使用KeyedProcessFunction

在 6.4.1. 的基础上做两个优化：

- 一是对数据进行按键分区，分别统计vc的出现次数；
- 二是进行增量聚合，得到结果最后再做排序输出。

```java
public class KeyedProcessFunctionTopNDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);


        SingleOutputStreamOperator<WaterSensor> sensorDS = env
                .socketTextStream("hadoop102", 7777)
                .map(new WaterSensorMapFunction())
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<WaterSensor>forBoundedOutOfOrderness(Duration.ofSeconds(3))
                                .withTimestampAssigner((element, ts) -> element.getTs() * 1000L)
                );


        // 最近10秒= 窗口长度， 每5秒输出 = 滑动步长
        /**
         * TODO 思路二： 使用 KeyedProcessFunction实现
         * 1、按照vc做keyby，开窗，分别count
         *    ==》 增量聚合，计算 count
         *    ==》 全窗口，对计算结果 count值封装 ，  带上 窗口结束时间的 标签
         *          ==》 为了让同一个窗口时间范围的计算结果到一起去
         *
         * 2、对同一个窗口范围的count值进行处理： 排序、取前N个
         *    =》 按照 windowEnd做keyby
         *    =》 使用process， 来一条调用一次，需要先存，分开存，用HashMap,key=windowEnd,value=List
         *      =》 使用定时器，对 存起来的结果 进行 排序、取前N个
         */

        // 1. 按照 vc 分组、开窗、聚合（增量计算+全量打标签）
        //  开窗聚合后，就是普通的流，没有了窗口信息，需要自己打上窗口的标记 windowEnd
        SingleOutputStreamOperator<Tuple3<Integer, Integer, Long>> windowAgg = sensorDS.keyBy(sensor -> sensor.getVc())
                .window(SlidingEventTimeWindows.of(Time.seconds(10), Time.seconds(5)))
                .aggregate(
                        new VcCountAgg(),
                        new WindowResult()
                );


        // 2. 按照窗口标签（窗口结束时间）keyby，保证同一个窗口时间范围的结果，到一起去。排序、取TopN
        windowAgg.keyBy(r -> r.f2)
                .process(new TopN(2))
                .print();


        env.execute();
    }


    public static class VcCountAgg implements AggregateFunction<WaterSensor, Integer, Integer> {

        @Override
        public Integer createAccumulator() {
            return 0;
        }

        @Override
        public Integer add(WaterSensor value, Integer accumulator) {
            return accumulator + 1;
        }

        @Override
        public Integer getResult(Integer accumulator) {
            return accumulator;
        }

        @Override
        public Integer merge(Integer a, Integer b) {
            return null;
        }
    }


    /**
     * 泛型如下：
     * 第一个：输入类型 = 增量函数的输出  count值，Integer
     * 第二个：输出类型 = Tuple3(vc，count，windowEnd) ,带上 窗口结束时间 的标签
     * 第三个：key类型 ， vc，Integer
     * 第四个：窗口类型
     */
    public static class WindowResult extends ProcessWindowFunction<Integer, Tuple3<Integer, Integer, Long>, Integer, TimeWindow> {

        @Override
        public void process(Integer key, Context context, Iterable<Integer> elements, Collector<Tuple3<Integer, Integer, Long>> out) throws Exception {
            // 迭代器里面只有一条数据，next一次即可
            Integer count = elements.iterator().next();
            long windowEnd = context.window().getEnd();
            out.collect(Tuple3.of(key, count, windowEnd));
        }
    }


    public static class TopN extends KeyedProcessFunction<Long, Tuple3<Integer, Integer, Long>, String> {
        // 存不同窗口的 统计结果，key=windowEnd，value=list数据
        private Map<Long, List<Tuple3<Integer, Integer, Long>>> dataListMap;
        // 要取的Top数量
        private int threshold;

        public TopN(int threshold) {
            this.threshold = threshold;
            dataListMap = new HashMap<>();
        }

        @Override
        public void processElement(Tuple3<Integer, Integer, Long> value, Context ctx, Collector<String> out) throws Exception {
            // 进入这个方法，只是一条数据，要排序，得到齐才行 ===》 存起来，不同窗口分开存
            // 1. 存到HashMap中
            Long windowEnd = value.f2;
            if (dataListMap.containsKey(windowEnd)) {
                // 1.1 包含vc，不是该vc的第一条，直接添加到List中
                List<Tuple3<Integer, Integer, Long>> dataList = dataListMap.get(windowEnd);
                dataList.add(value);
            } else {
                // 1.1 不包含vc，是该vc的第一条，需要初始化list
                List<Tuple3<Integer, Integer, Long>> dataList = new ArrayList<>();
                dataList.add(value);
                dataListMap.put(windowEnd, dataList);
            }

            // 2. 注册一个定时器， windowEnd+1ms即可（
            // 同一个窗口范围，应该同时输出，只不过是一条一条调用processElement方法，只需要延迟1ms即可
            ctx.timerService().registerEventTimeTimer(windowEnd + 1);

        }


        @Override
        public void onTimer(long timestamp, OnTimerContext ctx, Collector<String> out) throws Exception {
            super.onTimer(timestamp, ctx, out);
            // 定时器触发，同一个窗口范围的计算结果攒齐了，开始 排序、取TopN
            Long windowEnd = ctx.getCurrentKey();
            // 1. 排序
            List<Tuple3<Integer, Integer, Long>> dataList = dataListMap.get(windowEnd);
            dataList.sort(new Comparator<Tuple3<Integer, Integer, Long>>() {
                @Override
                public int compare(Tuple3<Integer, Integer, Long> o1, Tuple3<Integer, Integer, Long> o2) {
                    // 降序， 后 减 前
                    return o2.f1 - o1.f1;
                }
            });


            // 2. 取TopN
            StringBuilder outStr = new StringBuilder();

            outStr.append("================================\n");
            // 遍历 排序后的 List，取出前 threshold 个， 考虑可能List不够2个的情况  ==》 List中元素的个数 和 2 取最小值
            for (int i = 0; i < Math.min(threshold, dataList.size()); i++) {
                Tuple3<Integer, Integer, Long> vcCount = dataList.get(i);
                outStr.append("Top" + (i + 1) + "\n");
                outStr.append("vc=" + vcCount.f0 + "\n");
                outStr.append("count=" + vcCount.f1 + "\n");
                outStr.append("窗口结束时间=" + vcCount.f2 + "\n");
                outStr.append("================================\n");
            }

            // 用完的List，及时清理，节省资源
            dataList.clear();

            out.collect(outStr.toString());
        }
    }
}
```



### 6.5.侧输出流

处理函数还有另外一个特有功能，就是将自定义的数据放入“侧输出流”（side output）输出。

我们之前讲到的绝大多数转换算子，输出的都是单一流，流里的数据类型只能有一种。而侧输出流可以认为是“主流”上分叉出的“支流”，所以可以由一条流产生出多条流，而且这些流中的数据类型还可以不一样。利用这个功能可以很容易地实现**“分流”**操作。

具体应用时，只要在处理函数的`.processElement()`或者`.onTimer()`方法中，调用上下文的`.output()`方法就可以了。如果想要获取这个侧输出流，可以基于处理之后的DataStream直接调用`.getSideOutput()`方法，传入对应的OutputTag。

```java
DataStream<Integer> stream = env.fromSource(...);

// 在外部先将OutputTag声明出来：
OutputTag<String> outputTag = new OutputTag<String>("side-output") {};

SingleOutputStreamOperator<Long> longStream = stream.process(new ProcessFunction<Integer, Long>() {

    @Override
    public void processElement( Integer value, Context ctx, Collector<Integer> out) throws Exception {

      // 转换成Long，输出到主流中
      out.collect(Long.valueOf(value));
      
      // 转换成String，输出到侧输出流中
      ctx.output(outputTag, "side-output: " + String.valueOf(value));
    }
});

// 获取这个侧输出流
DataStream<String> stringStream = longStream.getSideOutput(outputTag);
```

## 7.状态管理

### 7.1.Flink中的状态

#### 7.1.1.状态概述

Flink中的任务可以分为**无状态**和**有状态**两种。

无状态算子任务只需要观察每个独立事件，根据当前的输入数据直接转换输出结果，例如`map,fliter,flatMap`属于无状态算子。

有状态算子任务除了当前数据之外，还需要一些其他数据来得到计算结果。此**“其他数据”即状态（state）**，聚合算子、窗口算子属于有状态算子。

有状态算子任务的一般处理流程如下图：

![](pic/0025.png)

#### 7.1.2.状态分类

**托管状态（Managed State）和原始状态（Raw State）**

- 托管状态（更常用）：由Flink 统一管理，状态的存储访问、故障恢复和重组等一系列问题都由Flink实现，开发只需要调接口。
- 原始状态：自定义的，相当于就是开辟了一块内存，需要自己管理，实现状态的序列化和故障恢复。

**算子状态（Operator State）和按键分区状态（Keyed State）**

一个算子任务一般有多个并行子任务执行，子任务会占据不同的任务槽（slot），slot之间是隔离的。

Flink能管理的状态在并行任务间是无法共享的，每个状态只能针对当前子任务的实例有效。

而很多有状态的操作（比如聚合、窗口）都是要先做keyBy进行按键分区的。按键分区之后，任务所进行的所有计算都应该只针对当前key有效，所以状态也应该按照key彼此隔离。基于此，可以将托管状态分为两类：算子状态和按键分区状态。

- 算子状态：作用范围限定为当前的算子任务实例，对当前并行子任务实例有效。算子状态也可以用在所有算子之上。

  ![](pic/0026.png)

- 按键分区状态：状态是根据输入流中定义的key来维护和访问的，只能定义在KeyedStream中。

  ![](pic/0027.png)

另外，也可以通过富函数类（Rich Function）来自定义Keyed State，所以只要提供了富函数类接口的算子，也都可以使用Keyed State。所以即使是map、filter这样无状态的基本转换算子，我们也可以通过富函数类给它们“追加”Keyed State。比如RichMapFunction、RichFilterFunction。在富函数中，我们可以调用.getRuntimeContext()获取当前的运行时上下文（RuntimeContext），进而获取到访问状态的句柄；这种富函数中自定义的状态也是Keyed State。从这个角度讲，Flink中所有的算子都可以是有状态的。

无论是Keyed State还是Operator State，它们都是在本地实例上维护的，也就是说每个并行子任务维护着对应的状态，算子的子任务之间状态不共享。

### 7.2.按键分区状态 （Keyed State）



### 7.3.算子状态（Operator State）



### 7.4.状态后端（State Backends）



## Flink SQL



