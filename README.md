# 数据同步工具CanalClientEx
------
  典型应用场景：

> * 跨机房的整个数据库实时备份；
> * 数据库表实时变动监控与通知（比如通知刷新缓存，表结构修改发送到邮件组等）；
> * 数据库表不完全一样的实时同步：比如需定义字段映射，或生成一些辅助字段（比如拼音码，简拼，简单计算映射等）；

  本项目基于**canal 1.0.24**最新版基础上修改，扩展了服务端和客户端的一些功能，主要包含：
> * 客户端：数据库镜像备份（解决扩机房的数据同步问题）；
> * 客户端：数据库表的映射同步（如自动生成拼音码，简拼，字段计算等）；
> * 客户端：数据库指定表的数据变动通知到Java类；
> * 客户端：支持数据库表MySQL->MS SQL的同步；
> * 服务端：position位置同步到Redis集群（以便在File情况下当服务器硬盘坏掉的情况下无法再同步）
> * 服务端：同步异常发预警邮件；
> * 服务端：修复bat文件无法在win10运行的问题；

[客户端1.0.24下载](https://github.com/kongshanxuelin/canalClientEx/files/1375074/canal.canalClientEx-1.0.24.zip)   [服务端1.0.24下载](https://github.com/kongshanxuelin/canalClientEx/files/1375087/canal.deployer-1.0.24.tar.gz)，点击加入QQ群讨论：[![QQ](http://pub.idqqimg.com/wpa/images/group.png)](https://jq.qq.com/?_wv=1027&k=5onpjJC)

## 开始使用

* 确保MySQL5.5+，并启动了binlog；
* 下载canal.deployer-1.0.24.tar.gz(服务端）,canal.canalClientEx-1.0.24.tar.gz(客户端）
* 解压并配置：按需增加同步实例，在conf目录下添加多个文件夹即可，并配置每个目录下的的instance.properties，如example目录下配置canal.instance.mysql.slaveId(不可重复），数据库信息：canal.instance.master.address，canal.instance.dbUsername，canal.instance.dbPassword；
* 运行
    * Windows：服务端（启动bin\startup.bat即可），客户端（启动bin\startup.bat即可）
    * Linux：确保bin目录下的sh文件有执行权限，服务端（启动bin\startup.sh即可），客户端（启动bin\startup.sh即可）

## 服务端增强

* 编译：
mvn clean install -Dmaven.test.skip -Denv=release

* 在canal.properties中加入以下属性会将position信息同步到Redis集群：
redis.server=192.168.1.170:7000,192.168.1.170:7001,192.168.1.170:7002,192.168.1.215:7000,192.168.1.215:7001,192.168.1.215:7002

* 新增出错时的邮件预警

## 客户端增强
* 支持数据库镜像备份,config.xml中节点的配置信息如下：
```xml
    <node name="test" desc="test mirror db">
        <canal-server-mode>simple</canal-server-mode>  
		<canal-server-ip>127.0.0.1</canal-server-ip>
		<canal-server-port>PORT</canal-server-port>
    	<canal-server-inst>INSTNAME</canal-server-inst>
		<!-- 是否有效 -->
		<active>true</active>
		<!-- 同步到库的JDBC配置 -->
		<db-url><![CDATA[jdbc:mysql://IP:PORT/test?useUnicode=true&characterEncoding=UTF-8]]></db-url>
		<db-driver>com.mysql.jdbc.Driver</db-driver>
		<db-username>USER</db-username>
		<db-password>PASSWORD</db-password>
		<db-schema>test</db-schema>
	</node>
```
* 支持表数据合并同步，支持字段映射定义，支持同步预警等：
```xml
	<node name="test2" desc="test table mapping">
			<canal-server-mode>simple</canal-server-mode>  
			<canal-server-ip>127.0.0.1</canal-server-ip>
			<canal-server-port>PORT</canal-server-port>
    	    <canal-server-inst>INSTNAME</canal-server-inst>
			<!-- 是否有效 -->
			<active>true</active>
			<!-- 同步到库的JDBC配置 -->
			<db-url><![CDATA[jdbc:mysql://IP:PORT/test?useUnicode=true&characterEncoding=UTF-8]]></db-url>
			<db-driver>com.mysql.jdbc.Driver</db-driver>
			<db-username>USER</db-username>
			<db-password>PASSWORD</db-password>
			<tables>
				<table source-scheme-name="test" source-name="test" dest-name="test_dest" ddlSync='true' rule="AND" dest-name-pri="df" >
					<fields>
						<field name="df" text="df" />
						<field name="dff" type="py" text="dff"></field>
						<field name="c2" type="el" text="df*2"/>
					</fields>
				</table>
			</tables>
			<alarms>
				<alarm stype="ddl" type="email">
					<title>邮件标题#{tableName}表的字段发生了变化</title>
					<body>#{tableName}表的字段发生了变化,执行的SQL:#{sql}</body>
					<sendTo>xxx@xxx.com</sendTo>
				</alarm>
				<alarm stype="exception" type="email">
					<title>同步数据#{source-scheme-name}.#{source-name}发生异常提醒</title>
					<body>#{body}</body>
					<sendTo>xxx@xxx.com</sendTo>
				</alarm>
			</alarms>
	</node>
```
* 支持自定义类的同步侦听
```xml
	<node name="test3" desc="sync_gjk_eform">
		<canal-server-mode>simple</canal-server-mode>  
		<canal-server-ip>IP</canal-server-ip>
		<canal-server-port>PORT</canal-server-port>
		<canal-server-inst>INSTNAME</canal-server-inst>
		<active>true</active>
		<db-url>jdbc:mysql://IP:PORT/db_cdb?useUnicode=true&amp;characterEncoding=UTF-8</db-url>
		<db-driver>com.mysql.jdbc.Driver</db-driver>
		<db-username>USER</db-username>
		<db-password>PASSWORD</db-password>
		<db-schema>xxxx</db-schema>
		<db-trigger>com.xxx.yyy.TCustomerTableTrigger</db-trigger>
	</node>
```