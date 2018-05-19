# Hadoop分布式文件系统HDFS
## HDFS前言
### 设计思想
    分而治之：将大文件、大批量文件，分布式存放在大量服务器上，以便于采取分而治之的方式对海量数据进行运算分析
    
### 在大数据系统中作用
    为各类分布式运算框架（如：mapreduce，spark，tez，……）提供数据存储服务
**重点概念：文件切块，副本存放，元数据**

## HDFS的概念和特性
    首先，它是一个文件系统，用于存储文件，通过统一的命名空间——目录树来定位文件。
    其次，它是分布式的，由很多服务器联合起来实现其功能，集群中的服务器有各自的角色。

### 重要特性如下：
1. HDFS中的文件在物理上是分块存储（block），块的大小可以通过配置参数( dfs.blocksize)来规定，默认大小在hadoop2.x版本中是128M，老版本中是64M

2. HDFS文件系统会给客户端提供一个统一的抽象目录树，客户端通过路径来访问文件，形如：hdfs://namenode:port/dir-a/dir-b/dir-c/file.data

3. 目录结构及文件分块信息(元数据)的管理由namenode节点承担 ——namenode是HDFS集群主节点，负责维护整个hdfs文件系统的目录树，以及每一个路径（文件）所对应的block块信息(block的id，及所在的datanode服务器)

4. 文件的各个block的存储管理由datanode节点承担 ---- datanode是HDFS集群从节点，每一个block都可以在多个datanode上存储多个副本（副本数量也可以通过参数设置dfs.replication）

5. HDFS是设计成适应一次写入，多次读出的场景，且不支持文件的修改
(注：适合用来做数据分析，并不适合用来做网盘应用，因为，不便修改，延迟大，网络开销大，成本太高)

## HDFS基本操作
### HDFS的shell(命令行客户端)操作
    命令行客户端支持的命令参数
```shell
        $hadoop fs -ls /
        
        [-appendToFile <localsrc> ... <dst>]
        [-cat [-ignoreCrc] <src> ...]
        [-checksum <src> ...]
        [-chgrp [-R] GROUP PATH...]
        [-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
        [-chown [-R] [OWNER][:[GROUP]] PATH...]
        [-copyFromLocal [-f] [-p] <localsrc> ... <dst>]
        [-copyToLocal [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
        [-count [-q] <path> ...]
        [-cp [-f] [-p] <src> ... <dst>]
        [-createSnapshot <snapshotDir> [<snapshotName>]]
        [-deleteSnapshot <snapshotDir> <snapshotName>]
        [-df [-h] [<path> ...]]
        [-du [-s] [-h] <path> ...]
        [-expunge]
        [-get [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
        [-getfacl [-R] <path>]
        [-getmerge [-nl] <src> <localdst>]
        [-help [cmd ...]]
        [-ls [-d] [-h] [-R] [<path> ...]]
        [-mkdir [-p] <path> ...]
        [-moveFromLocal <localsrc> ... <dst>]
        [-moveToLocal <src> <localdst>]
        [-mv <src> ... <dst>]
        [-put [-f] [-p] <localsrc> ... <dst>]
        [-renameSnapshot <snapshotDir> <oldName> <newName>]
        [-rm [-f] [-r|-R] [-skipTrash] <src> ...]
        [-rmdir [--ignore-fail-on-non-empty] <dir> ...]
        [-setfacl [-R] [{-b|-k} {-m|-x <acl_spec>} <path>]|[--set <acl_spec> <path>]]
        [-setrep [-R] [-w] <rep> <path> ...]
        [-stat [format] <path> ...]
        [-tail [-f] <file>]
        [-test -[defsz] <path>]
        [-text [-ignoreCrc] <src> ...]
        [-touchz <path> ...]
        [-usage [cmd ...]]
```



### 常用命令参数介绍
```shell
-help             
功能：输出这个命令参数手册
-ls                  
功能：显示目录信息
示例： hadoop fs -ls hdfs://hadoop-server01:9000/
备注：这些参数中，所有的hdfs路径都可以简写
hadoop fs -ls /   等同于上一条命令的效果

-mkdir              
功能：在hdfs上创建目录
示例：hadoop fs  -mkdir  -p  /aaa/bbb/cc/dd

-moveFromLocal            
功能：从本地剪切粘贴到hdfs
示例：hadoop  fs  - moveFromLocal  /home/hadoop/a.txt  /aaa/bbb/cc/dd

-moveToLocal              
功能：从hdfs剪切粘贴到本地
示例：hadoop  fs  - moveToLocal   /aaa/bbb/cc/dd  /home/hadoop/a.txt 

--appendToFile  
功能：追加一个文件到已经存在的文件末尾
示例：hadoop  fs  -appendToFile  ./hello.txt 
hdfs://hadoop-server01:9000/hello.txt
可以简写为：
hadoop  fs  -appendToFile  ./hello.txt  /hello.txt

-cat  
功能：显示文件内容  
示例：hadoop fs -cat  /hello.txt

-tail                 
功能：显示一个文件的末尾
示例：hadoop  fs  -tail  /weblog/access_log.1

-text                  
功能：以字符形式打印一个文件的内容
示例：hadoop  fs  -text  /weblog/access_log.1

-chgrp 
-chmod
-chown
功能：linux文件系统中的用法一样，对文件所属权限
示例：
hadoop  fs  -chmod  666  /hello.txt
hadoop  fs  -chown  someuser:somegrp   /hello.txt

-copyFromLocal    
功能：从本地文件系统中拷贝文件到hdfs路径去
示例：hadoop  fs  -copyFromLocal  ./jdk.tar.gz  /aaa/

-copyToLocal      
功能：从hdfs拷贝到本地
示例：hadoop fs -copyToLocal /aaa/jdk.tar.gz

-cp              
功能：从hdfs的一个路径拷贝hdfs的另一个路径
示例： hadoop  fs  -cp  /aaa/jdk.tar.gz  /bbb/jdk.tar.gz.2

-mv                     
功能：在hdfs目录中移动文件
示例： hadoop  fs  -mv  /aaa/jdk.tar.gz  /

-get              
功能：等同于copyToLocal，就是从hdfs下载文件到本地
示例：hadoop fs -get  /aaa/jdk.tar.gz

-getmerge             
功能：合并下载多个文件
示例：比如hdfs的目录 /aaa/下有多个文件:log.1, log.2,log.3,...
hadoop fs -getmerge /aaa/log.* ./log.sum

-put                
功能：等同于copyFromLocal
示例：hadoop  fs  -put  /aaa/jdk.tar.gz  /bbb/jdk.tar.gz.2

-rm                
功能：删除文件或文件夹
示例：hadoop fs -rm -r /aaa/bbb/

-rmdir                 
功能：删除空目录
示例：hadoop  fs  -rmdir   /aaa/bbb/ccc
-df               
功能：统计文件系统的可用空间信息
示例：hadoop  fs  -df  -h  /

-count         
功能：统计一个指定目录下的文件节点数量
示例：hadoop fs -count /aaa/

-setrep                
功能：设置hdfs中文件的副本数量
示例：hadoop fs -setrep 3 /aaa/jdk.tar.gz

-du 
功能：统计文件夹的大小信息
示例：hadoop  fs  -du  -s  -h /aaa/*
```





## HDFS原理
### hdfs的工作机制
    工作机制的学习主要是为加深对分布式系统的理解，以及增强遇到各种问题时的分析解决能力，形成一定的集群运维能力
    很多不是真正理解hadoop技术体系的人会常常觉得HDFS可用于网盘类应用，但实际并非如此。要想将技术准确用在恰当的地方，必须对技术有深刻的理解
### 概述
1. HDFS集群分为两大角色：NameNode、DataNode
2. NameNode负责管理整个文件系统的元数据
3. DataNode 负责管理用户的文件数据块
4. 文件会按照固定的大小（blocksize）切成若干块后分布式存储在若干台datanode上
5. 每一个文件块可以有多个副本，并存放在不同的datanode上
6. Datanode会定期向Namenode汇报自身所保存的文件block信息，而namenode则会负责保持文件的副本数量
7. HDFS的内部工作机制对客户端保持透明，客户端请求访问HDFS都是通过向namenode申请来进行
### HDFS写数据流程
    客户端要向HDFS写数据，首先要跟namenode通信以确认可以写文件并获得接收文件block的datanode，然后，客户端按顺序将文件逐个block传递给相应datanode，并由接收到block的datanode负责向其他datanode复制block的副本
### 详细步骤图
![这里写图片描述](https://img-blog.csdn.net/20180422161135890?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQxNDI4NzQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 详细步骤解析
1. 根namenode通信请求上传文件，namenode检查目标文件是否已存在，父目录是否存在
2. namenode返回是否可以上传
3. client请求第一个 block该传输到哪些datanode服务器上
4. namenode返回3个datanode服务器ABC
5. client请求3台dn中的一台A上传数据（本质上是一个RPC调用，建立pipeline），A收到请求会继续调用B，然后B调用C，将真个pipeline建立完成，逐级返回客户端
6. client开始往A上传第一个block（先从磁盘读取数据放到一个本地内存缓存），以packet为单位，A收到一个packet就会传给B，B传给C；A每传一个packet会放入一个应答队列等待应答
7. 当一个block传输完成之后，client再次请求namenode上传第二个block的服务器。
### HDFS读数据流程
    客户端将要读取的文件路径发送给namenode，namenode获取文件的元信息（主要是block的存放位置信息）返回给客户端，客户端根据返回的信息找到相应datanode逐个获取文件的block并在客户端本地进行数据追加合并从而获得整个文件
### 详细步骤图
![这里写图片描述](https://img-blog.csdn.net/2018042216115591?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQxNDI4NzQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
### 详细步骤解析
1. 跟namenode通信查询元数据，找到文件块所在的datanode服务器
2. 挑选一台datanode（就近原则，然后随机）服务器，请求建立socket流
3. datanode开始发送数据（从磁盘里面读取数据放入流，以packet为单位来做校验）
4. 客户端以packet为单位接收，现在本地缓存，然后写入目标文件


### NAMENODE工作机制
学习目标：
    理解namenode的工作机制尤其是元数据管理机制，以增强对HDFS工作原理的理解，及培养hadoop集群运营中“性能调优”、“namenode”故障问题的分析解决能力

问题场景：
1. 集群启动后，可以查看文件，但是上传文件时报错，打开web页面可看到namenode正处于safemode状态，怎么处理？
2. Namenode服务器的磁盘故障导致namenode宕机，如何挽救集群及数据？
3. Namenode是否可以有多个？namenode内存要配置多大？namenode跟集群数据存储能力有关系吗？
4. 文件的blocksize究竟调大好还是调小好？
......
诸如此类问题的回答，都需要基于对namenode自身的工作原理的深刻理解

#### NameNode职责
>负责客户端请求的响应
元数据的管理（查询，修改）
#### 元数据管理
>namenode对数据的管理采用了三种存储形式：
1. 内存元数据(NameSystem)
2. 磁盘元数据镜像文件
3. 数据操作日志文件（可通过日志运算出元数据）
#### 元数据存储机制
1. 内存中有一份完整的元数据(内存meta data)
2. 磁盘有一个“准完整”的元数据镜像（fsimage）文件(在namenode的工作目录中)
3. 用于衔接内存metadata和持久化元数据镜像fsimage之间的操作日志（edits文件）注：当客户端对hdfs中的文件进行新增或者修改操作，操作记录首先被记入edits日志文件中，当客户端操作成功后，相应的元数据会更新到内存meta.data中
#### 元数据手动查看
可以通过hdfs的一个工具来查看edits中的信息
```shell
bin/hdfs oev -i edits -o edits.xml
bin/hdfs oiv -i fsimage_0000000000000000087 -p XML -o fsimage.xml
```
#### 元数据的checkpoint
    每隔一段时间，会由secondary namenode将namenode上积累的所有edits和一个最新的fsimage下载到本地，并加载到内存进行merge（这个过程称为checkpoint）

##### checkpoint的详细过程
![这里写图片描述](https://img-blog.csdn.net/20180422161241805?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQxNDI4NzQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

##### checkpoint操作的触发条件配置参数
```shell
dfs.namenode.checkpoint.check.period=60  #检查触发条件是否满足的频率，60秒
dfs.namenode.checkpoint.dir=file://${hadoop.tmp.dir}/dfs/namesecondary
#以上两个参数做checkpoint操作时，secondary namenode的本地工作目录
dfs.namenode.checkpoint.edits.dir=${dfs.namenode.checkpoint.dir}

dfs.namenode.checkpoint.max-retries=3  #最大重试次数
dfs.namenode.checkpoint.period=3600  #两次checkpoint之间的时间间隔3600秒
dfs.namenode.checkpoint.txns=1000000 #两次checkpoint之间最大的操作记录
checkpoint的附带作用
namenode和secondary namenode的工作目录存储结构完全相同，所以，当namenode故障退出需要重新恢复时，可以从secondary namenode的工作目录中将fsimage拷贝到namenode的工作目录，以恢复namenode的元数据
```

### DataNode的工作机制
问题场景：
1. 集群容量不够，怎么扩容？
2. 如果有一些datanode宕机，该怎么办？
3. datanode明明已启动，但是集群中的可用datanode列表中就是没有，怎么办？
......
以上这类问题的解答，有赖于对datanode工作机制的深刻理解
#### 概述
1. Datanode工作职责：
>存储管理用户的文件块数据
定期向namenode汇报自身所持有的block信息（通过心跳信息上报）
（这点很重要，因为，当集群中发生某些block副本失效时，集群如何恢复block初始副本数量的问题）
```xml
<property>
	<name>dfs.blockreport.intervalMsec</name>
	<value>3600000</value>
	<description>Determines block reporting interval in milliseconds.</description>
</property>
```
2. DataNode掉线判断时限参数
>datanode进程死亡或者网络故障造成datanode无法与namenode通信，namenode不会立即把该节点判定为死亡，要经过一段时间，这段时间暂称作超时时长。HDFS默认的超时时长为10分钟+30秒。如果定义超时时间为timeout，则超时时长的计算公式为：
	timeout  = 2 * heartbeat.recheck.interval + 10 * dfs.heartbeat.interval。
	而默认的heartbeat.recheck.interval 大小为5分钟，dfs.heartbeat.interval默认为3秒。
	需要注意的是hdfs-site.xml 配置文件中的heartbeat.recheck.interval的单位为毫秒，dfs.heartbeat.interval的单位为秒。所以，举个例子，如果heartbeat.recheck.interval设置为5000（毫秒），dfs.heartbeat.interval设置为3（秒，默认），则总的超时时间为40秒。
```shell
<property>
        <name>heartbeat.recheck.interval</name>
        <value>2000</value>
</property>
<property>
        <name>dfs.heartbeat.interval</name>
        <value>1</value>
</property>
```

#### 观察验证DATANODE功能
    上传一个文件，观察文件的block具体的物理存放情况：
在每一台datanode机器上的这个目录中能找到文件的切块：
```shell
/home/hadoop/app/hadoop-2.4.1/tmp/dfs/data/current/BP-193442119-192.168.2.120-1432457733977/current/finalized
```



## HDFS应用开发
### HDFS的java操作
    hdfs在生产应用中主要是客户端的开发，其核心步骤是从hdfs提供的api中构造一个HDFS的访问客户端对象，然后通过该客户端对象操作（增删改查）HDFS上的文件
### 搭建开发环境
1. 引入依赖
```shell
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-client</artifactId>
    <version>2.6.1</version>
</dependency>
```
>注：如需手动引入jar包，hdfs的jar包----hadoop的安装目录的share下

2. window下开发的说明
>建议在linux下进行hadoop应用的开发，不会存在兼容性问题。如在window上做客户端应用开发，需要设置以下环境：
A、在windows的某个目录下解压一个hadoop的安装包
B、将安装包下的lib和bin目录用对应windows版本平台编译的本地库替换
C、在window系统中配置HADOOP_HOME指向你解压的安装包
D、在windows系统的path变量中加入hadoop的bin目录

### 获取api中的客户端对象
    在java中操作hdfs，首先要获得一个客户端实例
```java
Configuration conf = new Configuration()
FileSystem fs = FileSystem.get(conf)
```
>而我们的操作目标是HDFS，所以获取到的fs对象应该是DistributedFileSystem的实例。
get方法是从何处判断具体实例化那种客户端类呢？
从conf中的一个参数 fs.defaultFS的配置值判断。

>如果我们的代码中没有指定fs.defaultFS，并且工程classpath下也没有给定相应的配置，conf中的默认值就来自于hadoop的jar包中的core-default.xml，默认值为： file:///，则获取的将不是一个DistributedFileSystem的实例，而是一个本地文件系统的客户端对象

### HDFS客户端操作数据代码示例：
#### 文件的增删改查
```java
public class HdfsClient {

	FileSystem fs = null;

	@Before
	public void init() throws Exception {

		// 构造一个配置参数对象，设置一个参数：我们要访问的hdfs的URI
		// 从而FileSystem.get()方法就知道应该是去构造一个访问hdfs文件系统的客户端，以及hdfs的访问地址
		// new Configuration();的时候，它就会去加载jar包中的hdfs-default.xml
		// 然后再加载classpath下的hdfs-site.xml
		Configuration conf = new Configuration();
		conf.set("fs.defaultFS", "hdfs://hdp-node01:9000");
		/**
		 * 参数优先级： 1、客户端代码中设置的值 2、classpath下的用户自定义配置文件 3、然后是服务器的默认配置
		 */
		conf.set("dfs.replication", "3");

		// 获取一个hdfs的访问客户端，根据参数，这个实例应该是DistributedFileSystem的实例
		// fs = FileSystem.get(conf);

		// 如果这样去获取，那conf里面就可以不要配"fs.defaultFS"参数，而且，这个客户端的身份标识已经是hadoop用户
		fs = FileSystem.get(new URI("hdfs://hdp-node01:9000"), conf, "hadoop");

	}

	/**
	 * 往hdfs上传文件
	 * 
	 * @throws Exception
	 */
	@Test
	public void testAddFileToHdfs() throws Exception {

		// 要上传的文件所在的本地路径
		Path src = new Path("g:/redis-recommend.zip");
		// 要上传到hdfs的目标路径
		Path dst = new Path("/aaa");
		fs.copyFromLocalFile(src, dst);
		fs.close();
	}

	/**
	 * 从hdfs中复制文件到本地文件系统
	 * 
	 * @throws IOException
	 * @throws IllegalArgumentException
	 */
	@Test
	public void testDownloadFileToLocal() throws IllegalArgumentException, IOException {
		fs.copyToLocalFile(new Path("/jdk-7u65-linux-i586.tar.gz"), new Path("d:/"));
		fs.close();
	}

	@Test
	public void testMkdirAndDeleteAndRename() throws IllegalArgumentException, IOException {

		// 创建目录
		fs.mkdirs(new Path("/a1/b1/c1"));

		// 删除文件夹 ，如果是非空文件夹，参数2必须给值true
		fs.delete(new Path("/aaa"), true);

		// 重命名文件或文件夹
		fs.rename(new Path("/a1"), new Path("/a2"));

	}

	/**
	 * 查看目录信息，只显示文件
	 * 
	 * @throws IOException
	 * @throws IllegalArgumentException
	 * @throws FileNotFoundException
	 */
	@Test
	public void testListFiles() throws FileNotFoundException, IllegalArgumentException, IOException {

		// 思考：为什么返回迭代器，而不是List之类的容器
		RemoteIterator<LocatedFileStatus> listFiles = fs.listFiles(new Path("/"), true);

		while (listFiles.hasNext()) {
			LocatedFileStatus fileStatus = listFiles.next();
			System.out.println(fileStatus.getPath().getName());
			System.out.println(fileStatus.getBlockSize());
			System.out.println(fileStatus.getPermission());
			System.out.println(fileStatus.getLen());
			BlockLocation[] blockLocations = fileStatus.getBlockLocations();
			for (BlockLocation bl : blockLocations) {
				System.out.println("block-length:" + bl.getLength() + "--" + "block-offset:" + bl.getOffset());
				String[] hosts = bl.getHosts();
				for (String host : hosts) {
					System.out.println(host);
				}
			}
			System.out.println("--------------为angelababy打印的分割线--------------");
		}
	}

	/**
	 * 查看文件及文件夹信息
	 * 
	 * @throws IOException
	 * @throws IllegalArgumentException
	 * @throws FileNotFoundException
	 */
	@Test
	public void testListAll() throws FileNotFoundException, IllegalArgumentException, IOException {

		FileStatus[] listStatus = fs.listStatus(new Path("/"));

		String flag = "d--             ";
		for (FileStatus fstatus : listStatus) {
			if (fstatus.isFile())  flag = "f--         ";
			System.out.println(flag + fstatus.getPath().getName());
		}
	}
}
```


#### 通过流的方式访问hdfs
```java
/**
 * 相对那些封装好的方法而言的更底层一些的操作方式
 * 上层那些mapreduce   spark等运算框架，去hdfs中获取数据的时候，就是调的这种底层的api
 * @author
 *
 */
public class StreamAccess {
	
	FileSystem fs = null;

	@Before
	public void init() throws Exception {

		Configuration conf = new Configuration();
		fs = FileSystem.get(new URI("hdfs://hdp-node01:9000"), conf, "hadoop");

	}
	
	
	
	@Test
	public void testDownLoadFileToLocal() throws IllegalArgumentException, IOException{
		
		//先获取一个文件的输入流----针对hdfs上的
		FSDataInputStream in = fs.open(new Path("/jdk-7u65-linux-i586.tar.gz"));
		
		//再构造一个文件的输出流----针对本地的
		FileOutputStream out = new FileOutputStream(new File("c:/jdk.tar.gz"));
		
		//再将输入流中数据传输到输出流
		IOUtils.copyBytes(in, out, 4096);
		
		
	}
	
	
	/**
	 * hdfs支持随机定位进行文件读取，而且可以方便地读取指定长度
	 * 用于上层分布式运算框架并发处理数据
	 * @throws IllegalArgumentException
	 * @throws IOException
	 */
	@Test
	public void testRandomAccess() throws IllegalArgumentException, IOException{
		//先获取一个文件的输入流----针对hdfs上的
		FSDataInputStream in = fs.open(new Path("/iloveyou.txt"));
		
		
		//可以将流的起始偏移量进行自定义
		in.seek(22);
		
		//再构造一个文件的输出流----针对本地的
		FileOutputStream out = new FileOutputStream(new File("c:/iloveyou.line.2.txt"));
		
		IOUtils.copyBytes(in,out,19L,true);
		
	}
	
	
	
	/**
	 * 显示hdfs上文件的内容
	 * @throws IOException 
	 * @throws IllegalArgumentException 
	 */
	@Test
	public void testCat() throws IllegalArgumentException, IOException{
		
		FSDataInputStream in = fs.open(new Path("/iloveyou.txt"));
		
		IOUtils.copyBytes(in, System.out, 1024);
	}
}
```

#### 场景编程
```java
/**在mapreduce 、spark等运算框架中，有一个核心思想就是将运算移往数据，或者说，就是要在并发计算中尽可能让运算本地化，这就需要获取数据所在位置的信息并进行相应范围读取
以下模拟实现：获取一个文件的所有block位置信息，然后读取指定block中的内容*/
	@Test
	public void testCat() throws IllegalArgumentException, IOException{
		
		FSDataInputStream in = fs.open(new Path("/weblog/input/access.log.10"));
		//拿到文件信息
		FileStatus[] listStatus = fs.listStatus(new Path("/weblog/input/access.log.10"));
		//获取这个文件的所有block的信息
		BlockLocation[] fileBlockLocations = fs.getFileBlockLocations(listStatus[0], 0L, listStatus[0].getLen());
		//第一个block的长度
		long length = fileBlockLocations[0].getLength();
		//第一个block的起始偏移量
		long offset = fileBlockLocations[0].getOffset();
		
		System.out.println(length);
		System.out.println(offset);
		
		//获取第一个block写入输出流
//		IOUtils.copyBytes(in, System.out, (int)length);
		byte[] b = new byte[4096];
		
		FileOutputStream os = new FileOutputStream(new File("d:/block0"));
		while(in.read(offset, b, 0, 4096)!=-1){
			os.write(b);
			offset += 4096;
			if(offset>=length) return;
		};
		os.flush();
		os.close();
		in.close();
	}
```

