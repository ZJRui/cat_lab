实验一、CAT本地部署
======

## 注意！

本实验文档仅供学习参考！大众点评[CAT](https://github.com/dianping/cat)近期(2018下半年)更新比较频繁，已经升级到3.0，增加了不少新特性，建议大家后续跟进采用CAT新版本，生产级服务器部署请直接建议参考其[官方文档](https://github.com/dianping/cat)。

CAT 3.0的两个注意点：

* mysql请用6.x或7.x版本，暂不要用8.x版本，会有兼容性问题。
* Java应用配置方式和2.0不同，参考[java客户端使用](https://github.com/dianping/cat/tree/master/lib/java)的Initialization部分。

### 实验步骤

参考文档：https://blog.csdn.net/lqzkcx3/article/details/80626701
https://github.com/unidal/cat2/blob/master/script/Dianping%20CAT%20%E5%AE%89%E8%A3%85%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3.md <br>

https://github.com/unidal/cat2/blob/master/script/client.xml

官方部署文档：https://github.com/dianping/cat/blob/801f0b7b358814f8176dd2c47e25a85a9c83e6bc/integration/springMVC-AOP/CAT%E9%83%A8%E7%BD%B2.txt


#### 1. 下载CAT源码

```
git clone https://github.com/dianping/cat.git
```

#### 2. 构建cat war包

```
mvn clean install -DskipTests
```

如果构建过程中出现错误，可以直接在网上下载cat-home.war ，构建的目的就是为了得到war包部署到tomcat中
将`{CAT_SRC}/cat-home/target/cat-alpha-2.0.0.war`重命名为`cat.war`

#### 3. 创建数据库

在mysql中创建`cat`数据库，导入脚本`{CAT_SRC}/script/Cat.sql`

#### 4. 拷贝并更新配置文件

在CAT运行盘建`data\appdatas\cat`目录，将`{CAT_SRC}/script/`下列配置文件：

*  CAT客户端配置文件client.xml
*  CAT服务器数据源配置文件datasources.xml
*  CAT服务器配置文件server.xml

都复制到`data\appdatas\cat`目录下：


本地配置案例如下：

CAT客户端配置文件client.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--Cat 服务器本身也使用了Cat埋点，因此Cat服务器cat.war在Tomcat部署启动的过程中也会查找client.xml文件，然后通过client.xml文件中的server主动连接cat服务器-->
<!--IDea中的项目作为Cat客户端，在启动的时候也会查找client.xml文件寻找Cat服务器-->
<config mode="client" xmlns:xsi="http://www.w3.org/2001/XMLSchema" xsi:noNamespaceSchemaLocation="config.xsd">
	<servers>
		<!-- Local mode for development -->
		<!--port指定cat客户端和cat服务器传输日志的端口，http-port指定cat服务器提供web界面查看日志的端口-->
		  <!-- port : 配置服务端（cat-home）对外TCP协议开启端口，固定值为2280; -->
          <!-- http-port : 配置服务端（cat-home）对外HTTP协议开启端口, 如：tomcat默认是8080端口，若未指定，默认为8080端口; -->
		<server ip="192.168.7.70" port="2280" http-port="8080" />
		<!-- If under production environment, put actual server address as list. -->
		<!-- 
			<server ip="192.168.7.71" port="2280" /> 
			<server ip="192.168.7.72" port="2280" /> 
		-->
	</servers>
</config>
```
注意在client.xml文件中还可以配置一个domain标签，client.xml文件中的domain 表示命名空间，每一个Cat客户端都有一个client.xml配置文件，cat客户端读取client.xml文件中的server 得到catServer地址，同时使用domain作为自己的命名空间，一般client的domain都使用application.name作为domain.客户端发送个cat服务器的消息中会带上domain属性，区分消息。
比如<domain id="mobile-zuul-gateway" enable="true", max-message-size="1000">
	
------------------------------

CAT服务器数据源配置文件datasources.xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<data-sources>
	<data-source id="cat">
		<maximum-pool-size>3</maximum-pool-size>
		<connection-timeout>1s</connection-timeout>
		<idle-timeout>10m</idle-timeout>
		<statement-cache-size>1000</statement-cache-size>
		<properties>
			<driver>com.mysql.jdbc.Driver</driver>
			<url><![CDATA[jdbc:mysql://127.0.0.1:3306/cat]]></url>
			<user>root</user>
			<password>mysql</password>
			<connectionProperties><![CDATA[useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&socketTimeout=120000]]></connectionProperties>
		</properties>
	</data-source>
	<data-source id="app">
		<maximum-pool-size>3</maximum-pool-size>
		<connection-timeout>1s</connection-timeout>
		<idle-timeout>10m</idle-timeout>
		<statement-cache-size>1000</statement-cache-size>
		<properties>
			<driver>com.mysql.jdbc.Driver</driver>
			<url><![CDATA[jdbc:mysql://127.0.0.1:3306/cat]]></url>
			<user>root</user>
			<password>mysql</password>
			<connectionProperties><![CDATA[useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&socketTimeout=120000]]></connectionProperties>
		</properties>
	</data-source>
</data-sources>

```

CAT服务器配置文件server.xml

```
<?xml version="1.0" encoding="utf-8"?>

<!-- Configuration for development environment-->
<config local-mode="false" hdfs-machine="false" job-machine="true" alert-machine="false">
	
	<storage  local-base-dir="/data/appdatas/cat/bucket/" max-hdfs-storage-time="15" local-report-storage-time="7" local-logivew-storage-time="7">
	
	</storage>
	 <!--
        console : 定义服务控制台信息
        remote-servers : 定义HTTP服务列表，（远程监听端同步更新服务端信息即取此值）    
    -->
	<console default-domain="Cat" show-cat-domain="true">
		<!--注意上面的default-domain 为cat，也可以是其他名字,因为我们部署到tomcat中的cat是cat.war，因此最终访问的时候是http://localhost:port/Cat  上面的default-domain就是Catserver的contextPath-->
		<remote-servers>127.0.0.1:8080</remote-servers>		
	</console>
		
</config>
```
cat服务器本身也使用了cat进行日志输出，所以cat服务器本身也是一个cat客户端，cat服务器作为客户端的时候使用的domain是cat，也就是使用cat标记日志为cat服务器产生的日志， 上面的console标签配置了一个cat控制台，控制台默认展示的domain是cat命名空间下日志的信息。

**注意： server.xml配置文件的内容 有几种形式，在官方文档中给出的示例 可以参考这个链接 https://github.com/dianping/cat/blob/801f0b7b35/cat-core/src/test/resources/com/dianping/cat/server/server.xml  ，上面的这个server.xml配置文件可能会导致 cat.war包部署在tomcat容器中启动tomcat容器时在cat服务器日志（路径为 H:\data\applogs\cat）中出现如下错误**

```
[04-11 16:50:10.702] [INFO] [ServerConfigManager] Loading configuration file(H:\data\appdatas\cat\server.xml) ...
[04-11 16:50:10.725] [ERROR] [ServerConfigManager] Unknown root element(config) found! 提示无法解析处理server.xml文件中的config标签，导致catserver使用了默认自己的server.xml
[04-11 16:50:10.763] [INFO] [ServerConfigManager] CAT server is running with hdfs,false
[04-11 16:50:10.763] [INFO] [ServerConfigManager] CAT server is running with alert,false
[04-11 16:50:10.763] [INFO] [ServerConfigManager] CAT server is running with job,false
[04-11 16:50:10.764] [INFO] [ServerConfigManager] <?xml version="1.0" encoding="utf-8"?>
<server id="192.168.3.4">
   <storage local-base-dir="target/bucket" max-hdfs-storage-time="15" local-report-storage-time="3" local-logivew-storage-time="7" har-mode="true" upload-thread="5">
   </storage>
   <consumer>
      <long-config default-url-threshold="1000" default-sql-threshold="100" default-service-threshold="50">
      </long-config>
   </consumer>
</server>
```

#### 5. 部署war包

1. 将`cat.war`部署到tomcat的webapps目录下，启动tomcat
2. 打开cat控制台http://localhost:8080/cat
3. 配置全局路由，右上角`配置`，使用`catadmin/admin` 或者admin/admin登录，左边导航`全局告警配置->客户端路由`

本地路由配置案例

```xml
<?xml version="1.0" encoding="utf-8"?>
<router-config backup-server="192.168.7.70" backup-server-port="2280">
   <default-server id="192.168.7.70" weight="1.0" port="2280" enable="true"/>
</router-config>
```

#### 6. 校验CAT正常

通过导航`State`查看CAT状态，正常时显示`CAT服务端正常`字样，且CAT服务器会对自身进行监控，可通过导航`Transaction`查看。

如果服务器启动有问题，则可以通过查看日志解决：

1. `{CAT_HOME_DISK}:\data\applogs\cat`下面的CAT服务器日志
2. `{TOMCAT_HOME}\logs`下面的tomcat日志

### 部署参考

[CAT@github](https://github.com/dianping/cat)
