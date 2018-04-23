# HADOOP集群搭建与使用初步
## HADOOP集群搭建 
### 集群简介
    HADOOP集群具体来说包含两个集群：HDFS集群和YARN集群，两者逻辑上分离，但物理上常在一起
>HDFS集群：负责海量数据的存储，集群中的角色主要有 NameNode / DataNode
YARN集群：负责海量数据运算时的资源调度，集群中的角色主要有ResourceManager /NodeManager

本集群搭建案例，以5节点为例进行搭建，角色分配如下：

>hdp-node-01   NameNode   SecondaryNameNode  
 hdp-node-02   ResourceManager             
 hdp-node-03   DataNode   NodeManager  
 hdp-node-04   DataNode   NodeManager  
 hdp-node-05   DataNode   NodeManager  

部署图如下：

![这里写图片描述](https://img-blog.csdn.net/20180415235843551?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQxNDI4NzQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 服务器准备
本案例使用虚拟机服务器来搭建HADOOP集群，所用软件及版本：
1. Vmware 11.0
2. Centos  6.5  64bit

### 网络环境准备
1. 采用NAT方式联网
2. 网关地址：192.168.33.1
3. 3个服务器节点IP地址：192.168.33.101、192.168.33.102、192.168.33.103
4. 子网掩码：255.255.255.0
### 服务器系统设置
1. 添加HADOOP用户
2. 为HADOOP用户分配sudoer权限
3. 同步时间
4. 设置主机名
>hdp-node-01
hdp-node-02
hdp-node-03
5. 配置内网域名映射：
192.168.33.101  :   hdp-node-01
192.168.33.102  :   hdp-node-02
192.168.33.103  :   hdp-node-03
6. 配置ssh免密登陆
7. 配置防火墙

### JDK环境安装
1. 上传jdk安装包
2. 规划安装目录  /home/hadoop/apps/jdk_1.7.65
3. 解压安装包
4. 配置环境变量 /etc/profile


### HADOOP安装部署
1. 上传HADOOP安装包
2. 规划安装目录  /home/hadoop/apps/hadoop-2.6.1
3. 解压安装包
4. 修改配置文件  $HADOOP_HOME/etc/hadoop/

最简化配置如下：
```
vi  hadoop-env.sh
# The java implementation to use.
export JAVA_HOME=/home/hadoop/apps/jdk1.7.0_51
vi  core-site.xml
```
```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hdp-node-01:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/HADOOP/apps/hadoop-2.6.1/tmp</value>
    </property>
</configuration>
```
```
vi  hdfs-site.xml
```
```xml
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/home/hadoop/data/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/home/hadoop/data/data</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    <property>
        <name>dfs.secondary.http.address</name>
        <value>hdp-node-01:50090</value>
    </property>
</configuration>
```
```
vi  mapred-site.xml
```
```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```
```
vi  yarn-site.xml
```
```xml
<configuration>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop01</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```
```
vi  salves
```
```
hdp-node-01
hdp-node-02
hdp-node-03
```

###  启动集群
初始化HDFS
```
bin/hadoop  namenode  -format
```
启动HDFS
```
sbin/start-dfs.sh
```
启动YARN
```
sbin/start-yarn.sh
```
### 测试
1. 上传文件到HDFS
从本地上传一个文本文件到hdfs的/wordcount/input目录下
```
[HADOOP@hdp-node-01 ~]$ HADOOP fs -mkdir -p /wordcount/input
[HADOOP@hdp-node-01 ~]$ HADOOP fs -put /home/HADOOP/somewords.txt  /wordcount/input
```
2. 运行一个mapreduce程序
在HADOOP安装目录下，运行一个示例mr程序
```
cd $HADOOP_HOME/share/hadoop/mapreduce/
hadoop jar mapredcue-example-2.6.1.jar wordcount /wordcount/input  /wordcount/output 
```




## 集群使用初步
### HDFS使用
1. 查看集群状态
```
hdfs  dfsadmin  –report 
```
![这里写图片描述](https://img-blog.csdn.net/20180415235926404?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQxNDI4NzQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

可以看出，集群共有3个datanode可用,也可打开web控制台查看HDFS集群信息，在浏览器打开http://hdp-node-01:50070/

![这里写图片描述](https://img-blog.csdn.net/20180416000025106?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQxNDI4NzQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

2. 上传文件到HDFS
查看HDFS中的目录信息
```
hadoop  fs  –ls  /
```
![这里写图片描述](https://img-blog.csdn.net/20180416000047722?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQxNDI4NzQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

3. 上传文件
```
hadoop  fs  -put  ./ scala-2.10.6.tgz  to  /
```
![这里写图片描述](https://img-blog.csdn.net/20180416000106866?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQxNDI4NzQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

4. 从HDFS下载文件
```
hadoop  fs  -get  /yarn-site.xml
```
![这里写图片描述](https://img-blog.csdn.net/20180416000126207?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQxNDI4NzQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
### MAPREDUCE使用
>mapreduce是hadoop中的分布式运算编程框架，只要按照其编程规范，只需要编写少量的业务逻辑代码即可实现一个强大的海量数据并发处理程序
#### Demo开发——wordcount
##### 1. 需求
从大量（比如T级别）文本文件中，统计出每一个单词出现的总次数
##### 2. mapreduce实现思路
+ Map阶段：
    + 从HDFS的源数据文件中逐行读取数据
    + 将每一行数据切分出单词
    + 为每一个单词构造一个键值对(单词，1)
    + 将键值对发送给reduce

+ Reduce阶段：
    + 接收map阶段输出的单词键值对
    + 将相同单词的键值对汇聚成一组
    + 对每一组，遍历组中的所有“值”，累加求和，即得到每一个单词的总次数将(单词，总次数)输出到HDFS的文件中


##### 3. 具体编码实现
定义一个mapper类
```java
//首先要定义四个泛型的类型
//keyin:  LongWritable    valuein: Text
//keyout: Text            valueout:IntWritable

public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable>{
	//map方法的生命周期：  框架每传一行数据就被调用一次
	//key :  这一行的起始点在文件中的偏移量
	//value: 这一行的内容
	@Override
	protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
		//拿到一行数据转换为string
		String line = value.toString();
		//将这一行切分出各个单词
		String[] words = line.split(" ");
		//遍历数组，输出<单词，1>
		for(String word:words){
			context.write(new Text(word), new IntWritable(1));
		}
	}
}
```
定义一个reducer类
```java
	//生命周期：框架每传递进来一个kv 组，reduce方法被调用一次
	@Override
	protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
		//定义一个计数器
		int count = 0;
		//遍历这一组kv的所有v，累加到count中
		for(IntWritable value:values){
			count += value.get();
		}
		context.write(key, new IntWritable(count));
	}
}
```
定义一个主类，用来描述job并提交job
```java
public class WordCountRunner {
	//把业务逻辑相关的信息（哪个是mapper，哪个是reducer，要处理的数据在哪里，输出的结果放哪里。。。。。。）描述成一个job对象
	//把这个描述好的job提交给集群去运行
	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		Job wcjob = Job.getInstance(conf);
		//指定我这个job所在的jar包
//		wcjob.setJar("/home/hadoop/wordcount.jar");
		wcjob.setJarByClass(WordCountRunner.class);
		
		wcjob.setMapperClass(WordCountMapper.class);
		wcjob.setReducerClass(WordCountReducer.class);
		//设置我们的业务逻辑Mapper类的输出key和value的数据类型
		wcjob.setMapOutputKeyClass(Text.class);
		wcjob.setMapOutputValueClass(IntWritable.class);
		//设置我们的业务逻辑Reducer类的输出key和value的数据类型
		wcjob.setOutputKeyClass(Text.class);
		wcjob.setOutputValueClass(IntWritable.class);
		
		//指定要处理的数据所在的位置
		FileInputFormat.setInputPaths(wcjob, "hdfs://hdp-server01:9000/wordcount/data/big.txt");
		//指定处理完成之后的结果所保存的位置
		FileOutputFormat.setOutputPath(wcjob, new Path("hdfs://hdp-server01:9000/wordcount/output/"));
		
		//向yarn集群提交这个job
		boolean res = wcjob.waitForCompletion(true);
		System.exit(res?0:1);
	}
```



#### 程序打包运行
##### 1. 将程序打包
##### 2. 准备输入数据
```
vi  /home/hadoop/test.txt
```
```
Hello tom
Hello jim
Hello ketty
Hello world
Ketty tom
```
在hdfs上创建输入数据文件夹：
```
hadoop   fs  mkdir  -p  /wordcount/input
```
将words.txt上传到hdfs上
```
hadoop  fs  –put  /home/hadoop/words.txt  /wordcount/input
```
![这里写图片描述](https://img-blog.csdn.net/20180416000158622?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQxNDI4NzQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
##### 3. 将程序jar包上传到集群的任意一台服务器上
##### 4. 使用命令启动执行wordcount程序jar包
```
$ hadoop jar wordcount.jar cn.itcast.bigdata.mrsimple.WordCountDriver /wordcount/input /wordcount/out
```
![这里写图片描述](https://img-blog.csdn.net/20180416000214554?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQxNDI4NzQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
##### 5. 查看执行结果
```
$ hadoop fs –cat /wordcount/out/part-r-00000
```
![这里写图片描述](https://img-blog.csdn.net/2018041600052842?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQxNDI4NzQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



