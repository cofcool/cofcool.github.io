---
layout: post
category : Tech
title : Java笔记
tags : [java, notes]
---
{% include JB/setup %}

目录:

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1. 配置](#1-配置)
	* [1. Ubuntu安装Tomcat](#1-ubuntu安装tomcat)
	* [2.Centos安装mysql](#2centos安装mysql)
		* [1. 安装](#1-安装)
		* [2. 配置](#2-配置)
			* [1. 开启MySql远程连接](#1-开启mysql远程连接)
			* [2. 安装配置tomcat](#2-安装配置tomcat)
			* [3. centos6配置iptables](#3-centos6配置iptables)
			* [4. Tomcat 在linux下开机自启动](#4-tomcat-在linux下开机自启动)
			* [5. Apache映射到Tomcat](#5-apache映射到tomcat)
			* [6. https](#6-https)
* [2. Spring](#2-spring)
	* [1. IOC](#1-ioc)
		* [1. 注解](#1-注解)
	* [2. AOP](#2-aop)
		* [1. 配置](#1-配置-1)
	* [3. MVC](#3-mvc)
		* [1. 启用jackson解析JSON](#1-启用jackson解析json)
* [3. 常见问题](#3-常见问题)
	* [1. IDEA 2016 使用junit4](#1-idea-2016-使用junit4)
	* [2. tomcat无响应](#2-tomcat无响应)
		* [1. 数据库连接处于等待中](#1-数据库连接处于等待中)
		* [2. 连接数过多](#2-连接数过多)
	* [3. Maven项目（spring, mybatis...）在eclipse中可以运行，无法在Idea中运行](#3-maven项目spring-mybatis在eclipse中可以运行无法在idea中运行)
	* [4. 使用Idea时提示无法找到配置文件](#4-使用idea时提示无法找到配置文件)
	* [5. 引入第三方库](#5-引入第三方库)
	* [6. lib已存在但抛出java.lang.NoClassDefFoundError](#6-lib已存在但抛出javalangnoclassdeffounderror)
	* [7. jetty多实例部署](#7-jetty多实例部署)
	* [8. linux环境下调用so库](#8-linux环境下调用so库)
	* [9. Spring相关](#9-spring相关)

<!-- /code_chunk_output -->



环境:
* 主机: ubuntu 16.04 LTS
* 服务器: Tomcat8, Jetty 9.4
* VM: OpenJDK 8
* IntelliJ IDEA Ultimate

## 1. 配置

### 1. Ubuntu安装Tomcat

Ubuntu 默认把 Tomcat 分到2个目录, Eclipse 无法识别安装的 Tomcat。 解决方案：（注意下面路径有些有空格）

```
sudo ln -s /var/lib/tomcat7/conf /usr/share/tomcat7/conf
sudo ln -s /etc/tomcat7/policy.d/03catalina.policy /usr/share/tomcat7/conf/catalina.policy
sudo ln -s /var/log/tomcat7 /usr/share/tomcat7/log
sudo ln -s /var/lib/tomcat7/webapps /usr/share/tomcat7/
sudo chmod -R 777 /usr/share/tomcat7/conf
```

### 2.Centos安装mysql

#### 1. 安装
#### 2. 配置

##### 1. 开启MySql远程连接

`GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'youpassword' WITH GRANT OPTION;`

重载授权表：

`FLUSH PRIVILEGES;`

***NOTE:*** 如果是ubuntu的话需要更改配置文件，取消登录ip绑定。

##### 2. 安装配置tomcat

1. 准备Java环境： 安装JDK
2. 下载tomcat到指定的目录
3. 解压，执行
```
./bin/startup.sh
```
4. 启动成功之后，可查看8080端口是否被监控：
```
netstat -antp
```

***NOTE:*** 如果提示“org.apache.catalina.core.AprLifecycleListener.lifecycleEvent The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: /usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib”，可到apache官网[下载](http://apr.apache.org/download.cgi)<b>APR</b>来安装,重启即可。

##### 3. centos6配置iptables

1. 添加允许访问的端口
    ```
    /sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT
    /sbin/iptables -I INPUT -p tcp --dport 3306 -j ACCEPT
    /sbin/iptables -I INPUT -p tcp --dport 22 -j ACCEPT
    /sbin/iptables -I INPUT -p tcp --dport 8080 -j ACCEPT
    ```
2. 保存
    ```
    service iptables save
    ```
3. 重启服务
    ```
    service iptables restart
    ```

##### 4. Tomcat 在linux下开机自启动

1. 修改/etc/rc.d/rc.local
	```
	vim /etc/rc.d/rc.local
	```
2. 添加下面两行脚本
	```
	export JAVA_HOME=$JAVA_PATH$
	/usr/local/tomcat/bin/startup.sh start
	```
3. 修改rc.local文件为可执行
	```
	chmod +x  rc.local
	```
##### 5. Apache映射到Tomcat

1. 搭建Apache虚拟主机
	```
	<VirtualHost *:8080>
	    DocumentRoot /opt/tomcat7/webapps/rd
	    ServerName lb.test.com：8080
	</VirtualHost>
	```
2. 映射到Tomcat使用的8080端口

##### 6. https

1. 生成证书
	```
	keytool -genkey -v -alias tomcat -keyalg RSA -keystore tomcat.keystore -validity 36500
	# 导出cer证书
	keytool -keystore tomcat.keystore -export -alias tomcat -file tomcat.cer
	```
2. 配置https
	1. 拷贝第一步生成的tomcat.keystore文件到**${TOMCAT_HOME}/conf**目录下
	2. 编辑server.xml

	    ```
	    sudo vim ${TOMCAT_HOME}/conf/server.xml
	    # 内容如下
	    <Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true"
	               maxThreads="150" scheme="https" secure="true"
	               clientAuth="false" sslProtocol="TLS"
	               keystoreFile="${TOMCAT_HOME}/conf/tomcat.keystore" keystorePass="${PASSWD}"
	               truststoreFile="${TOMCAT_HOME}/conf/tomcat.keystore" truststorePass="${PASSWD}" />
	    ```
3. 编辑web.xml，http自动跳转为https
    ```
    sudo vim ${TOMCAT_HOME}/conf/web.xml
    # 内容如下
    <security-constraint>
       <web-resource-collection >
              <web-resource-name >SSL</web-resource-name>
              <url-pattern>/*</url-pattern>
       </web-resource-collection>
       <user-data-constraint>
              <transport-guarantee>CONFIDENTIAL</transport-guarantee>
       </user-data-constraint>
    </security-constraint>
    ```
 4. 对于沃通的证书，由于支持的加密解密方式不同，需要自己添加加密方式：
    ```
    # 错误提示
    ERR_SSL_VERSION_OR_CIPHER_MISMATCH
    # 解决办法
    # 在Connector节点下添加
    ciphers="TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256,
    				TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,
    				TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384,
    				TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA,
    				TLS_ECDHE_RSA_WITH_RC4_128_SHA,
    				TLS_RSA_WITH_AES_128_CBC_SHA256,
    				TLS_RSA_WITH_AES_128_CBC_SHA,
    				TLS_RSA_WITH_AES_256_CBC_SHA256,
    				TLS_RSA_WITH_AES_256_CBC_SHA,
    				SSL_RSA_WITH_RC4_128_SHA"
    ```
5. 沃通的证书使用`Oracle JDK`，使用OpenJDK时`https`无法通过验证

## 2. Spring

### 1. IOC

#### 1. 注解

1. 通过`@Autowired`获取`Bean`时需根据接口来引用变量, 因为自动注入策略默认为`byType`
2. 多个类实现同一接口，并通过`@Autowired`获取`Bean`，出现异常`expected single matching bean but found 2`，需通过实现类的注解指定该Bean的变量名，可通过`@Resource(name="xxx")`的形式来引用该实例,也可使用`@Qualifier`。
    ```
    // 实现类
    @Component("XXX")
    // 引用
    @Resource(name="XXX")
    ```
3. 使用`@Autowired`时，可通过`@Qualifier`设置为自动注入策略`byName`。
	```
	@Autowired
    @Qualifier("userServiceImpl")
    private UserService userService;
	```

### 2. AOP

#### 1. 配置
1. 启用
    在spring配置文件的scheme中引入
    ```
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd"
    ```
    启用`@Aspect`
    ```
    <aop:aspectj-autoproxy />
    ```

### 3. MVC

#### 1. 启用jackson解析JSON

配置:
```
<mvc:annotation-driven>
	<mvc:message-converters>
	    <bean class="org.springframework.http.converter.StringHttpMessageConverter">
	        <property name="supportedMediaTypes">
	            <list>
	                <value>text/html; charset=UTF-8</value>
	                <value>application/json;charset=UTF-8</value>
	            </list>
	        </property>
	    </bean>
	    <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
	        <property name="supportedMediaTypes">
	            <list>
	                <value>text/html; charset=UTF-8</value>
	                <value>application/json;charset=UTF-8</value>
	            </list>
	        </property>
	        <property name="prettyPrint">
	            <value>true</value>
	        </property>
	        <property name="objectMapper">
	            <bean class="com.fasterxml.jackson.databind.ObjectMapper">
	                <property name="serializationInclusion">
	                    <value>NON_NULL</value>
	                </property>
	            </bean>
	        </property>
	    </bean>
	</mvc:message-converters>
</mvc:annotation-driven>
```

在代码中使用`@ResponseBody`返回JSON。

## 3. 常见问题

### 1. IDEA 2016 使用junit4

**IDEA** 集成了junit4，因此进行测试时，只需添加junit的jar包到工程中即可。

1. 添加jar包

    ![img1]({{ site.url }}/public/upload/images/0067.png)

    ![img2]({{ site.url }}/public/upload/images/0068.png)

    ![img3]({{ site.url }}/public/upload/images/0069.png)

2. 新建类

    新建一个测试类，该类继承 `junit.framework.TestCase`,在需要测试的方法前添加`test`前缀，开始测试时该方法会自动运行。

    ```
    package net.cofcool.junit;


    import junit.framework.TestCase;

    /**
     * Created by CofCool on 4/6/16.
     */
    public class JunitTest extends TestCase {

        public void testAdd() {
            int result = Demo.add(2, 5);
            assertEquals(7, result);
        }

        public void testFail() {
            fail("test failure");
        }

        /*
        * do something after test
        * */
        @Override
        protected void tearDown() throws Exception {
            super.tearDown();
        }

        /*
        * do something before test
        * */
        @Override
        protected void setUp() throws Exception {
            super.setUp();
        }
    }

    ```
3. 添加测试配置：

    ![img4]({{ site.url }}/public/upload/images/0072.png)

    ![img5]({{ site.url }}/public/upload/images/0070.png)

    ![img6]({{ site.url }}/public/upload/images/0071.png)

    更多信息参考[junit](http://junit.org/junit4/)。

### 2. tomcat无响应

#### 1. 数据库连接处于等待中

#### 2. 连接数过多

......

### 3. Maven项目（spring, mybatis...）在eclipse中可以运行，无法在Idea中运行

IDEA的maven项目中，默认源代码目录下的xml等资源文件并不会在编译的时候一块打包进classes文件夹。为了保证mybatis的`XML`映射文件打包进`classes`文件夹中，可做如下修改：

**pom.xml**:
```
<build>
	<resources>
		<resource>
			<directory>src/main/java</directory>
			<includes>
				<include>**/*.xml</include>
			</includes>
		</resource>
	</resources>
</build>
```

### 4. 使用Idea时提示无法找到配置文件

Idea在编译打包时并没有把某些资源文件包含进去，因此需手动指定。

![]({{ site.url }}/public/upload/images/0140.png)

### 5. 引入第三方库

1. **添加文件**： 在WEB_INF目录下，新建**lib**目录，把需要的jar包添加到该目录中
  ![image]({{ site.url }}/public/upload/images/0073.png)
2. **添加为库**：在该jar包上右键点击**Add As Library..**，
  ![image]({{ site.url }}/public/upload/images/0074.png)

### 6. lib已存在但抛出java.lang.NoClassDefFoundError

环境:
* Idea
* Tomcat
* Java Web项目

开发过程中，虽然在`pom.xml`中添加了lib，并没有引入到发布环境的路径中, 虽然在编译时无错误,但在运行时会因为缺少必要的类库而抛出异常。

如下图所示，类库`lib`虽然包含在项目依赖库中，但是并不在`classes/WEB-INF/lib`目录中。
![]({{ site.url }}/public/upload/images/0142.png)

![]({{ site.url }}/public/upload/images/0143.png)

要想包含进去，只需在`Project Structure(Ctrl+Alt+Shift+S)`下的`Artifacts`菜单中把该lib引入`WEB-INF/lib`即可。

![]({{ site.url }}/public/upload/images/0144.png)

### 7. jetty多实例部署

从`jetty`的启动文件`jetty.sh`可以得到的基本信息:

1. 变量`NAME`的值为该脚本的文件名(不包含后缀)

	```
	NAME=$(echo $(basename $0) | sed -e 's/^[SK][0-9]*//' -e 's/\.sh$//')
	```
2. 通过`findDirectory`找到可写的目录,并在该目录下创建`jetty`文件夹, 最后得到的目录即为`JETTY_RUN`

	```
	if [ -z "$JETTY_RUN" ]
	then
	  JETTY_RUN=$(findDirectory -w /var/run /usr/var/run $JETTY_BASE /tmp)/jetty
	  [ -d "$JETTY_RUN" ] || mkdir $JETTY_RUN
	fi
	```
3. 每次jetty启动时会检测当前的`JETTY_RUN`目录时候包含文件名为`${NAME}.pid`的文本文件, 如存在,即不会创建新的jetty实例.

	```
	#####################################################
	# Find a pid and state file
	#####################################################
	if [ -z "$JETTY_PID" ]
	then
	  JETTY_PID="$JETTY_RUN/${NAME}.pid"
	fi

	if [ -z "$JETTY_STATE" ]
	then
	  JETTY_STATE=$JETTY_BASE/${NAME}.state
	fi
	```

综上, 可以的出结论: 只要保证每个jetty拥有唯一的`pid`文件即可.

解决:

1. 重命名`jetty.sh`文件
2. 保证jetty只对自己当前的`JETTY_BASE`目录拥有写权限, 没有对系统目录`/var/run, /usr/var/run, /tmp`的写权限, 可通过创建`jetty用户`指定权限来实现

### 8. linux环境下调用so库

通过`System.loadLibrary()`来引入so库时, so库的文件名不需要加`lib`前缀.在载入so库时,`public static String mapLibraryName(String libname)`会根据系统的特性来对so库的文件名进行处理, 生成适合特定系统的文件名,然后根据该名称去寻找so库.

### 9. Spring相关

1. 开发中经常需要捕获异常避免直接抛出给调用者，但是代码中充斥的try-catch异常的难看，可使用`ExceptionHandler`来统一捕捉异常。
2. `Spring MVC` action filter:

	![]({{ site.url }}/public/upload/images/0146.png)