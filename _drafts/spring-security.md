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

![Voting Decision Manager](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/images/access-decision-voting.png)

![Security interceptors and the "secure object" model](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/images/security-interception.png)

`ProviderManager`实现`AuthenticationManager`接口, 负责管理`AuthenticationProvider`实例

```plantuml
filter-> AuthenticationManager
AuthenticationManager -> AuthenticationProvider
AuthenticationProvider -> AuthenticationProvider: supports(toekn)
AuthenticationProvider -> AuthenticationProvider: authenticate(toekn)
AuthenticationProvider -> AuthenticationManager
AuthenticationManager -> filter
```