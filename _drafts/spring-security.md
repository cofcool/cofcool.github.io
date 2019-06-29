# Spring Security 笔记

`EnableGlobalAuthentication`允许配置`AuthenticationManagerBuilder`

`GlobalMethodSecurityConfiguration`可对方法进行拦截处理鉴权等逻辑, `EnableGlobalMethodSecurity`注解启用。

`AuthenticationConfiguration` 暴露`AuthenticationManager`相关API

`FilterComparator`，默认启用的"filter"

重写`UsernamePasswordAuthenticationFilter`类来实现自定义逻辑，若JSON请求等

自定义 `org.springframework.security.web.authentication.AuthenticationFailureHandler`处理权限验证失败等情况,
`org.springframework.security.web.authentication.AuthenticationSuccessHandler`处理成功情况

`PermissionEvaluator`

权限控制, `AccessDecisionManager`

```plantuml
filter-> AuthenticationManager
AuthenticationManager -> AuthenticationProvider
AuthenticationProvider -> AuthenticationProvider: supports(toekn)
AuthenticationProvider -> AuthenticationProvider: authenticate(toekn)
AuthenticationProvider -> AuthenticationManager
AuthenticationManager -> filter
```