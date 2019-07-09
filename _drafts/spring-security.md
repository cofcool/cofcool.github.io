# Spring Security 笔记

`EnableGlobalAuthentication`允许配置`AuthenticationManagerBuilder`

`GlobalMethodSecurityConfiguration`可对方法进行拦截处理鉴权等逻辑, `EnableGlobalMethodSecurity`注解启用。

`AuthenticationConfiguration` 暴露`AuthenticationManager`相关API

`FilterComparator`，默认启用的"filter"

重写`UsernamePasswordAuthenticationFilter`类来实现自定义逻辑，若JSON请求等

自定义 `org.springframework.security.web.authentication.AuthenticationFailureHandler`处理权限验证失败等情况,
`org.springframework.security.web.authentication.AuthenticationSuccessHandler`处理成功情况

`AbstractSecurityInterceptor`, 检查对象的调用安全

`PermissionEvaluator`

`ExceptionTranslationFilter`处理`AccessDeniedException`和`AuthenticationException`, 通过`AuthenticationEntryPoint`处理

权限控制, `AccessDecisionManager`, 处理权限验证等, 管理`AccessDecisionVoter`列表，通过它来进行角色验证。`AbstractInterceptUrlConfigurer`, `AbstractSecurityInterceptor`

`SecurityMetadataSource` 存储用户的 `ConfigAttribute` 信息, 可用来查询用户权限等

![Voting Decision Manager](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/images/access-decision-voting.png)

![Security interceptors and the "secure object" model](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/images/security-interception.png)

![After Invocation Implementation](https://docs.spring.io/spring-security/site/docs/5.2.0.M3/reference/htmlsingle/images/after-invocation.png)

`ProviderManager`实现`AuthenticationManager`接口, 负责管理`AuthenticationProvider`实例

```plantuml
filter-> AuthenticationManager
AuthenticationManager -> AuthenticationProvider
AuthenticationProvider -> AuthenticationProvider: supports(toekn)
AuthenticationProvider -> AuthenticationProvider: authenticate(toekn)
AuthenticationProvider -> AuthenticationManager
AuthenticationManager -> filter
```

`ConcurrentSessionControlAuthenticationStrategy` 阻止一个账号多处登录, `ConcurrentSessionFilter`

匿名访问: `AnonymousAuthenticationFilter`, `AuthenticatedVoter`

WebSocket: `AbstractSecurityWebSocketMessageBrokerConfigurer·`

`表达式`:

<div class="section"><div class="titlepage"><div><div><h3 class="title"><a name="overview" href="#overview"></a>11.3.1&nbsp;Overview</h3></div></div></div>
<p>Spring Security uses Spring EL for expression support and you should look at how that works if you are interested in understanding the topic in more depth.
Expressions are evaluated with a "root object" as part of the evaluation context.
Spring Security uses specific classes for web and method security as the root object, in order to provide built-in expressions and access to values such as the current principal.</p>
<div class="section"><div class="titlepage"><div><div><h4 class="title"><a name="el-common-built-in" href="#el-common-built-in"></a>Common Built-In Expressions</h4></div></div></div>
<p>The base class for expression root objects is <code class="literal">SecurityExpressionRoot</code>.
This provides some common expressions which are available in both web and method security.</p>
<div class="table"><a name="common-expressions" href="#common-expressions"></a><p class="title"><b>Table&nbsp;11.1.&nbsp;Common built-in expressions</b></p><div class="table-contents">
<table summary="Common built-in expressions" style="border-collapse: collapse;border-top: 0.5pt solid ; border-bottom: 0.5pt solid ; border-left: 0.5pt solid ; border-right: 0.5pt solid ; "><colgroup><col class="col_1"><col class="col_2"></colgroup><thead><tr><th style="border-right: 0.5pt solid ; border-bottom: 0.5pt solid ; " valign="top" align="left">Expression</th><th style="border-bottom: 0.5pt solid ; " valign="top" align="left">Description</th></tr></thead><tbody><tr><td style="border-right: 0.5pt solid ; border-bottom: 0.5pt solid ; " valign="top" align="left"><p><code class="literal">hasRole([role])</code></p></td><td style="border-bottom: 0.5pt solid ; " valign="top" align="left"><p>Returns <code class="literal">true</code> if the current principal has the specified role.
By default if the supplied role does not start with 'ROLE_' it will be added.
This can be customized by modifying the <code class="literal">defaultRolePrefix</code> on <code class="literal">DefaultWebSecurityExpressionHandler</code>.</p></td></tr><tr><td style="border-right: 0.5pt solid ; border-bottom: 0.5pt solid ; " valign="top" align="left"><p><code class="literal">hasAnyRole([role1,role2])</code></p></td><td style="border-bottom: 0.5pt solid ; " valign="top" align="left"><p>Returns <code class="literal">true</code> if the current principal has any of the supplied roles (given as a comma-separated list of strings).
By default if the supplied role does not start with 'ROLE_' it will be added.
This can be customized by modifying the <code class="literal">defaultRolePrefix</code> on <code class="literal">DefaultWebSecurityExpressionHandler</code>.</p></td></tr><tr><td style="border-right: 0.5pt solid ; border-bottom: 0.5pt solid ; " valign="top" align="left"><p><code class="literal">hasAuthority([authority])</code></p></td><td style="border-bottom: 0.5pt solid ; " valign="top" align="left"><p>Returns <code class="literal">true</code> if the current principal has the specified authority.</p></td></tr><tr><td style="border-right: 0.5pt solid ; border-bottom: 0.5pt solid ; " valign="top" align="left"><p><code class="literal">hasAnyAuthority([authority1,authority2])</code></p></td><td style="border-bottom: 0.5pt solid ; " valign="top" align="left"><p>Returns <code class="literal">true</code> if the current principal has any of the supplied authorities (given as a comma-separated list of strings)</p></td></tr><tr><td style="border-right: 0.5pt solid ; border-bottom: 0.5pt solid ; " valign="top" align="left"><p><code class="literal">principal</code></p></td><td style="border-bottom: 0.5pt solid ; " valign="top" align="left"><p>Allows direct access to the principal object representing the current user</p></td></tr><tr><td style="border-right: 0.5pt solid ; border-bottom: 0.5pt solid ; " valign="top" align="left"><p><code class="literal">authentication</code></p></td><td style="border-bottom: 0.5pt solid ; " valign="top" align="left"><p>Allows direct access to the current <code class="literal">Authentication</code> object obtained from the <code class="literal">SecurityContext</code></p></td></tr><tr><td style="border-right: 0.5pt solid ; border-bottom: 0.5pt solid ; " valign="top" align="left"><p><code class="literal">permitAll</code></p></td><td style="border-bottom: 0.5pt solid ; " valign="top" align="left"><p>Always evaluates to <code class="literal">true</code></p></td></tr><tr><td style="border-right: 0.5pt solid ; border-bottom: 0.5pt solid ; " valign="top" align="left"><p><code class="literal">denyAll</code></p></td><td style="border-bottom: 0.5pt solid ; " valign="top" align="left"><p>Always evaluates to <code class="literal">false</code></p></td></tr><tr><td style="border-right: 0.5pt solid ; border-bottom: 0.5pt solid ; " valign="top" align="left"><p><code class="literal">isAnonymous()</code></p></td><td style="border-bottom: 0.5pt solid ; " valign="top" align="left"><p>Returns <code class="literal">true</code> if the current principal is an anonymous user</p></td></tr><tr><td style="border-right: 0.5pt solid ; border-bottom: 0.5pt solid ; " valign="top" align="left"><p><code class="literal">isRememberMe()</code></p></td><td style="border-bottom: 0.5pt solid ; " valign="top" align="left"><p>Returns <code class="literal">true</code> if the current principal is a remember-me user</p></td></tr><tr><td style="border-right: 0.5pt solid ; border-bottom: 0.5pt solid ; " valign="top" align="left"><p><code class="literal">isAuthenticated()</code></p></td><td style="border-bottom: 0.5pt solid ; " valign="top" align="left"><p>Returns <code class="literal">true</code> if the user is not anonymous</p></td></tr><tr><td style="border-right: 0.5pt solid ; border-bottom: 0.5pt solid ; " valign="top" align="left"><p><code class="literal">isFullyAuthenticated()</code></p></td><td style="border-bottom: 0.5pt solid ; " valign="top" align="left"><p>Returns <code class="literal">true</code> if the user is not an anonymous or a remember-me user</p></td></tr><tr><td style="border-right: 0.5pt solid ; border-bottom: 0.5pt solid ; " valign="top" align="left"><p><code class="literal">hasPermission(Object target, Object permission)</code></p></td><td style="border-bottom: 0.5pt solid ; " valign="top" align="left"><p>Returns <code class="literal">true</code> if the user has access to the provided target for the given permission.
For example, <code class="literal">hasPermission(domainObject, 'read')</code></p></td></tr><tr><td style="border-right: 0.5pt solid ; " valign="top" align="left"><p><code class="literal">hasPermission(Object targetId, String targetType, Object permission)</code></p></td><td style="" valign="top" align="left"><p>Returns <code class="literal">true</code> if the user has access to the provided target for the given permission.
For example, <code class="literal">hasPermission(1, 'com.example.domain.Message', 'read')</code></p></td></tr></tbody></table>
</div></div><br class="table-break">
</div>
</div>

OAuth:

`HttpSecurity.oauth2Login()`


`@AuthenticationPrincipal` 注入用户数据

```java
@RequestMapping("/messages/inbox")
public ModelAndView findMessagesForUser(@AuthenticationPrincipal CustomUser customUser) {

    // .. find messages for this user and return them ...
}
```