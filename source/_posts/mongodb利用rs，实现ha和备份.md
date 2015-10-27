title: mongodb利用rs，实现ha和备份
date: 2013-10-01 15:14
categories: database 
---
mongodb最简单的部署方式，是单节点部署，但是单节点部署有些问题，第一是无法做HA，如果该mongod down掉，或者部署的server down掉，应用就无法工作；第二是不利于备份，因为在备份的时候，会给mongod额外的负担，有可能影响业务；第三是无法做读写分离。所以在生产环境下，应该优先考虑集群部署
<!--more-->

# 概述

mongod支持的集群部署方式有3种：

1、master-slave

2、replica set

3、sharding

master-slave可以解决备份的问题，但是无法透明地HA，所以也不大好；sharding是mongo的一个亮点特性，可以自动分片。但是根据我的测试结果，在单集合达到2000万条数据的门槛之后，sharding才开始体现出性能优势，而sharding的数据是分布式的，所以备份会比较复杂，而且也需要更多的服务器，不利于成本。所以最后我考虑，还是使用replica set方式的集群部署比较合适。可以解决HA，备份，读写分离的问题

基本上采用官网推荐的这种TOPO：

![](http://img.blog.csdn.net/20131001142634968?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

理想情况下，最好是3个mongod实例分别部署在单独的server上，这样就需要3台server；出于成本考虑，也可以把2台secondary部署在同一台server上，primary由于要处理读写请求（读写暂不分离），需要很多内存，并且考虑HA因素，所以primary是需要保证独占一台server比较好，这样就一共需要2台server

# 配置集群

## 以replset方式启动mongod

也可以用命令行启动，但是不利于管理，所以建议采用--config参数来启动，配置文件如下：

```
port=2222 
bind_ip=127.0.0.1 
dbpath=/home/zhengshengdong/mongo_dbs/db1 
fork=true 
replSet=rs0 
logpath=/home/zhengshengdong/mongo_log/log1.txt 
logappend=true journal=true
```

./mongod --config /home/zhengshengdong/mongo_confs/mongo1.conf

然后如法炮制，启动另外2个mongod实例

## 初始化replica set，并添加secondary

用./mongo --port 2222连上准备作为primary的mongod实例，然后依次执行以下命令

```
rs.initiate()
rs.conf()
rs.add("host1:port")
rs.add("host2:port")
```

## 检查

在primary实例上执行

```
rs.status()
```
应该能看到类似下图的效果

![](http://img.blog.csdn.net/20131001144337187?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# 验证HA场景

用java driver写了以下代码来做验证

```
public static void main(String[] args) throws UnknownHostException {

		ScheduledExecutorService executor = Executors
				.newSingleThreadScheduledExecutor();

		final MongoClient client = MongoConnectionHelper.getClient();
		client.setWriteConcern(WriteConcern.REPLICA_ACKNOWLEDGED);

		Runnable task = new Runnable() {

			private int i = 0;

			@Override
			public void run() {
				DB db = client.getDB("yilos");
				DBCollection collection = db.getCollection("test");
				DBObject o = new BasicDBObject("name", "MongoDB" + i).append(
						"count", i);
				try {
					collection.insert(o);
				} catch (Exception exc) {
					exc.printStackTrace();
				}

				i++;
			}
		};

		executor.scheduleWithFixedDelay(task, 1, 1, TimeUnit.SECONDS);

	}
```
每隔1秒往集群中写入一条数据，然后手动把primary shutdown，观察发现jvm console给出提示：

警告: Primary switching from zhengshengdong-K43SA/127.0.0.1:2222 to mongodb.kyfxbl.net/127.0.0.1:4444

这条消息说明，mongo自动处理了primary的切换，对于应用来说是透明的。然后查看mongo中的记录，发现在切换完成以后，写入操作确实继续了

![](http://img.blog.csdn.net/20131001145402625?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

上图可以看到，在2926和2931之间，正在进行primary倒换，在完成之后，应用可以继续写入数据

但是明显可以看到，中间2927,2928,2929,2930这4条数据丢失了（mongo的primary倒换大约需要3-5秒时间），对于业务来说，虽然时间不长，但是如果因此丢失了业务数据，也是不能接受的

接下来再仔细看下这段时间内代码抛出的异常：

```
com.mongodb.MongoException$Network: Write operation to server mongodb.kyfxbl.net/127.0.0.1:4444 failed on database yilos 
at com.mongodb.DBTCPConnector.say(DBTCPConnector.java:153) 
at com.mongodb.DBTCPConnector.say(DBTCPConnector.java:115) 
at com.mongodb.DBApiLayer$MyCollection.insert(DBApiLayer.java:248) 
at com.mongodb.DBApiLayer$MyCollection.insert(DBApiLayer.java:204) 
at com.mongodb.DBCollection.insert(DBCollection.java:76) 
at com.mongodb.DBCollection.insert(DBCollection.java:60) 
at com.mongodb.DBCollection.insert(DBCollection.java:105) 
at com.yilos.mongo.HATester$1.run(HATester.java:36) 
at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:441) 
at java.util.concurrent.FutureTask$Sync.innerRunAndReset(FutureTask.java:317) 
at java.util.concurrent.FutureTask.runAndReset(FutureTask.java:150) 
at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$101(ScheduledThreadPoolExecutor.java:98) 
at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.runPeriodic(ScheduledThreadPoolExecutor.java:180) 
at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:204) 
at java.util.concurrent.ThreadPoolExecutor$Worker.runTask(ThreadPoolExecutor.java:886) 
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:908) 
at java.lang.Thread.run(Thread.java:662) Caused by: java.net.SocketException: Broken pipe 
at java.net.SocketOutputStream.socketWrite0(Native Method) 
at java.net.SocketOutputStream.socketWrite(SocketOutputStream.java:92) 
at java.net.SocketOutputStream.write(SocketOutputStream.java:136) 
at org.bson.io.PoolOutputBuffer.pipe(PoolOutputBuffer.java:129) 
at com.mongodb.OutMessage.pipe(OutMessage.java:236) 
at com.mongodb.DBPort.go(DBPort.java:133) 
at com.mongodb.DBPort.go(DBPort.java:106) 
at com.mongodb.DBPort.findOne(DBPort.java:162) 
at com.mongodb.DBPort.runCommand(DBPort.java:170) 
at com.mongodb.DBTCPConnector._checkWriteError(DBTCPConnector.java:100) 
at com.mongodb.DBTCPConnector.say(DBTCPConnector.java:142) 
... 16 more
```

可以发现，实际上这并不是mongo的问题，而是这段测试代码的BUG引发的：

```
Runnable task = new Runnable() {

			private int i = 0;

			@Override
			public void run() {
				DB db = client.getDB("yilos");
				DBCollection collection = db.getCollection("test");
				DBObject o = new BasicDBObject("name", "MongoDB" + i).append(
						"count", i);
				try {
					collection.insert(o);
				} catch (Exception exc) {
					exc.printStackTrace();
				}

				i++;
			}
		};
```
问题就出在这里，try语句中捕获到了MongoException，但是我写的这段代码并没有进行处理，只是简单地打印出异常，然后把i自增后，进入下一次循环。

从中也可以判断出，mongo提供的java driver，并不会把失败的write操作暂存在队列中，稍后重试；而是抛弃这次写操作，抛出异常让客户端自行处理。我觉得这个设计也是没问题的，但是客户端一定要进行处理才行。关键是要处理mongo exception，以及设置较高级别的write concern。replica set本身的HA机制是可行的

# 备份方案

采用2级备份：

第一层是集群部署提供的天然备份，由于存在2个secondary node，会和primary始终保持同步，因此在任何时候，集群都有2份完整的数据镜像副本

第二层则是使用mongo提供的mongodump和fsync工具，通过手工执行或者脚本的方式，在业务负载低（凌晨3点）的时候，获取关键collection的每日副本。备份采集也在secondary上执行，不给primary额外的压力

采集的备份，保存到本地或者其他的server，避免server的存储损坏，造成数据和备份全部丢失

同时，在启动mongod的时候，打开journal参数，这样在极端情况下（上述2种备份都失败），还可以通过oplog进行手工恢复