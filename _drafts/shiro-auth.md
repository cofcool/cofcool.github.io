# 使用Spring MVC + Shiro简单权限控制

```xml
<dependency>
  <groupId>org.apache.shiro</groupId>
  <artifactId>shiro-core</artifactId>
  <version>${shiro.version}</version>
</dependency>

<dependency>
  <groupId>org.apache.shiro</groupId>
  <artifactId>shiro-web</artifactId>
  <version>${shiro.version}</version>
</dependency>

<dependency>
  <groupId>org.apache.shiro</groupId>
  <artifactId>shiro-spring</artifactId>
  <version>${shiro.version}</version>
</dependency>

<!-- 如果使用缓存可添加shiro-ehcache -->
<dependency>
  <groupId>org.apache.shiro</groupId>
  <artifactId>shiro-ehcache</artifactId>
  <version>${shiro-ehcache.version}</version>
</dependency>
```

web.xml
```xml
<!--
web.xml will usually contain a DelegatingFilterProxy definition, with the specified filter-name corresponding to a bean name in Spring's root application context. All calls to the filter proxy will then be delegated to that bean in the Spring context, which is required to implement the standard Servlet Filter interface.

DelegatingFilterProxy会代理Spring上下文中实现Servlet Filter接口的类
-->
<filter>
  <filter-name>shiroFilter</filter-name>
  <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
  <filter-name>shiroFilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```
shiro.xml
```xml
ShiroFilterFactoryBean会把程序上下文中的Filter类缓存到filters属性中。
该类实现Spring的BeanPostProcessor和FactoryBean，通过postProcessBeforeInitialization方法来缓存filter，getObject方法获取SpringShiroFilter实例。
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
  <property name="securityManager" ref="securityManager"/>
  <property name="loginUrl" value="/auth/login" />
  <property name="unauthorizedUrl" value="/auth/unauth"/>
  <property name="filterChainDefinitions" ref="shiroUrls" />
</bean>

<bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
  <property name="securityManager" ref="securityManager" />
</bean>

<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
  <property name="sessionManager" ref="sessionManager"/>
  <property name="realm" ref="myRealm" />
  <property name="cacheManager" ref="shiroCacheManager"/>
</bean>

<bean id="sessionManager" class="org.apache.shiro.web.session.mgt.DefaultWebSessionManager">
  <property name="globalSessionTimeout" value="43200000"/>
</bean>


<bean class="com.zx.tk.sys.server.controller.shiro.MyCredentialsMatcher" id="myCredentialsMatcher"/>
<bean id="myRealm" class="com.zx.tk.sys.server.controller.shiro.AuthRealm" >
  <property name="credentialsMatcher" ref="myCredentialsMatcher" />
</bean>

<bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor" />

<!-- 开启shiro注解 -->
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"
	depends-on="lifecycleBeanPostProcessor">
  <property name="proxyTargetClass" value="true" />
</bean>

<util:map id="filters">
  <entry key="validateFilter" value-ref="validateFilter" />
  <!-- 重写登录授权过滤器 -->
  <entry key="authc" value-ref="authFilter" />
</util:map>
<bean id="validateFilter" class="com.zx.tk.sys.server.controller.shiro.ValidateFilter" />
<bean id="authFilter" class="com.zx.tk.sys.server.controller.shiro.AuthFilter" />
```
