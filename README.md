# jdk-tomcat
jdk and tomcat install and config （version2）
服务器版本：Centos7.4 
jdk版本：jdk1.8.0_111
Tomcat版本：apache-tomcat-8.5.32

安装包放置路径：/opt/package
软件安装地址： /usr/setup

第一步： 安装jdk1.8.0_111
- 解压 
	tar zxf /opt/package/jdk-8u111-linux-x64.tar.gz -C /usr/setup
- 设置环境变量
	vi /etc/profile 添加以下内容：
	export JAVA_HOME=/usr/setup/jdk1.8.0_111
	export JRE_HOME=$JAVA_HOME/jre
	export CLASSPATH=./:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
	export PATH=$PATH:$JAVA_HOME/bin

- 使环境变量生效
	source /etc/profile

- 重启
	reboot
  
第二步：安装Tomcat8.5
- 解压文件
	$ tar zxf apache-tomcat-8.5.32.tar.gz -C /usr/setup/

- 创建用户，并将home目录放置到安装目录下面
	useradd -m -U -d /usr/setup/apache-tomcat-8.5.32 -s /bin/false tomcat

- 创建快捷方式
	ln -s /usr/setup/apache-tomcat-* /usr/setup/latestTomcat

- 改变文件所属组和用户为Tomcat
	chown -R tomcat: /usr/setup/apache-tomcat-*
	chown -R tomcat: /usr/setup/latestTomcat

- 可执行状态
	chmod +x /usr/setup/latestTomcat/bin/*.sh

- 创建/etc/systemd/system/tomcat.service文件	
	```添加以下内容
	[Unit]
	Description=Tomcat 8.5 servlet container
	After=network.target

	[Service]
	Type=forking

	User=tomcat
	Group=tomcat

	Environment="JAVA_HOME=/usr/setup/jdk1.8.0_111"

	Environment="CATALINA_BASE=/usr/setup/latestTomcat"
	Environment="CATALINA_HOME=/usr/setup/latestTomcat"
	Environment="CATALINA_PID=/usr/setup/latestTomcat/temp/tomcat.pid"

	ExecStart=/usr/setup/latestTomcat/bin/startup.sh
	ExecStop=/usr/setup/latestTomcat/bin/shutdown.sh

	[Install]
	WantedBy=multi-user.target
	```

- 让创建的服务生效，然后启动Tomcat
	systemctl daemon-reload
	systemctl start tomcat
	systemctl status tomcat

- 加入 automatically started at boot time
	systemctl enable tomcat

- 开放端口
	firewall-cmd --zone=public --permanent --add-port=8080/tcp
	firewall-cmd --reload

- 修改配置文件
	a. vi /usr/setup/latestTomcat/conf/tomcat-users.xml
	```添加以下内容
	<role rolename="manager-script"/>	
	<user username="tomcat" password="tomcat" roles="manager-script"/>
	```
	b. vi /usr/setup/latestTomcat/webapps/manager/META-INF/context.xml
<Context antiResourceLocking="false" privileged="true" >
	<!--  <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> --> 把这一行注释掉
	<Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
</Context>

注意：Tomcat启动慢的解决办法
第一种 :
通过修改Tomcat启动文件 -Djava.security.egd=file:/dev/urandom 通过修改JRE中的java.security文件 securerandom.source=file:/dev/urandom

第二种(推荐):
yum -y install rng-tools
##启动熵服务
systemctl start rngd
systemctl restart rngd
