---
layout: post
category : Tech
title : Shiro 登陆原理解析以及配置多个Realm
tags : [java, shiro]
excerpt: Apache Shiro是一个强大且易用的Java安全框架,执行身份验证、授权、密码学和会话管理，内置了众多的基础角色和权限验证功能。它通过Realm来执行具体授权的验证，如账号查询，密码验证等。
---
{% include JB/setup %}

Apache Shiro是一个强大且易用的Java安全框架,执行身份验证、授权、密码学和会话管理，内置了众多的基础角色和权限验证功能。它通过Realm来执行具体授权的验证，如账号查询，密码验证等。

我们来看看基本的账号密码验证体系，抽象类`AuthenticatingRealm`实现了账号密码的登陆验证，由账号授权和密码验证组成。一个AuthenticatingRealm实例的创建如下：

**spring-shiro.xml**
```xml
<bean class="net.cofcool.test.shiro.controller.shiro.MyCredentialsMatcher" id="myCredentialsMatcher"/>
<bean id="myRealm" class="net.cofcool.test.shiro.controller.shiro.AuthRealm" >
	<pruserty name="credentialsMatcher" ref="myCredentialsMatcher" />
</bean>
```

**AuthRealm**
```java
@Slf4j
public class AuthRealm extends AuthorizingRealm {

    @Resource
    private UserService userService;

    /**
     * 权限验证
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
       return null;
    }


    /**
     * 登录验证
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authcToken) throws AuthenticationException {
        CaptchaUsernamePasswordToken token = (CaptchaUsernamePasswordToken) authcToken;

        String username = token.getUsername();

        String realCaptcha = (String) SecurityUtils.getSubject().getSession().getAttribute(LoginConstant.CAPTCHA_CODE_KEY);
        String captcha = token.getCaptcha();

        if (StringFunction.isEmpty(captcha) || StringFunction.isEmpty(captcha)) {
            throw new CaptchaException("验证码不能为空");
        }
        if (!captcha.equalsIgnoreCase(realCaptcha)) {
            throw new CaptchaException("验证码错误");
        }

        User user = userService.login(username, null);

        if (user == null) {
            throw new UserNotExistException("用户不存在");
        }

        return new SimpleAuthenticationInfo(user, token, this.getName());
    }
}
```

**CaptchaUsernamePasswordToken**
```java
public class CaptchaUsernamePasswordToken extends UsernamePasswordToken {

    private String username;
    private char[] password;
    private String captcha;

    @Override
    public String getUsername() {
        return username;
    }

    @Override
    public void setUsername(String username) {
        this.username = username;
    }

    @Override
    public char[] getPassword() {
        return password;
    }

    @Override
    public void setPassword(char[] password) {
        this.password = password;
    }

    public String getCaptcha() {
        return captcha;
    }

    public void setCaptcha(String captcha) {
        this.captcha = captcha;
    }

    public CaptchaUsernamePasswordToken(String username, char[] password, String captcha) {
        this.username = username;
        this.password = password;
        this.captcha = captcha;

    }
}
```

**MyCredentialsMatcher**
```java
public class MyCredentialsMatcher implements CredentialsMatcher {

    @Override
    public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {
        if (!(token instanceof CaptchaUsernamePasswordToken)) {
            return false;
        }

        CaptchaUsernamePasswordToken userToken = (CaptchaUsernamePasswordToken) token;
        User user = (User) info.getPrincipals().getPrimaryPrincipal();

        return new String(userToken.getPassword()).equals(user.getLoginPwd());
    }
}

```

以上代码即可创建一个拥有登陆账号密码验证功能的Realm。

下面我们来看看Shrio是如何登陆的。

Shiro通过`Subject`实例的`void login(AuthenticationToken token) throws AuthenticationException`方法进行登陆。`DelegatingSubject`实现了login方法。

```java
public void login(AuthenticationToken token) throws AuthenticationException {
    clearRunAsIdentitiesInternal();

    // 执行securityManager的登陆
    Subject subject = securityManager.login(this, token);

    ...
}
```

**securityManager**
```xml
<!-- DefaultWebSecurityManager 为默认的SecurityManager -->
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager" />
```

DefaultWebSecurityManager继承DefaultSecurityManager，DefaultSecurityManager的`login`方法如下:
```java
public Subject login(Subject subject, AuthenticationToken token) throws AuthenticationException {
        AuthenticationInfo info;
    try {
        // 进行授权
        info = authenticate(token);
    } catch (AuthenticationException ae) {
        try {
            onFailedLogin(token, ae, subject);
        } catch (Exception e) {
            if (log.isInfoEnabled()) {
                log.info("onFailedLogin method threw an " +
                        "exception.  Logging and propagating original AuthenticationException.", e);
            }
        }
        throw ae; //propagate
    }

    Subject loggedIn = createSubject(token, info, subject);

    onSuccessfulLogin(token, info, loggedIn);

    return loggedIn;
}

public AuthenticationInfo authenticate(AuthenticationToken token) throws AuthenticationException {
    // 调用authenticator的authenticate方法
    // 本对象的authenticator为ModularRealmAuthenticator实例
    return this.authenticator.authenticate(token);
}
```

**ModularRealmAuthenticator**，继承`AbstractAuthenticator`，AbstractAuthenticator定义了authenticate方法，在该方法中调用抽象方法`doAuthenticate(AuthenticationToken token)`。
```java
// AbstractAuthenticator
public final AuthenticationInfo authenticate(AuthenticationToken token) throws AuthenticationException {

    if (token == null) {
        throw new IllegalArgumentException("Method argumet (authentication token) cannot be null.");
    }

    log.trace("Authentication attempt received for token [{}]", token);

    AuthenticationInfo info;
    try {
        // 调用抽象方法 doAuthenticate
        info = doAuthenticate(token);
        if (info == null) {
            String msg = "No account information found for authentication token [" + token + "] by this " +
                    "Authenticator instance.  Please check that it is configured correctly.";
            throw new AuthenticationException(msg);
        }
    } catch (Throwable t) {
        AuthenticationException ae = null;
        if (t instanceof AuthenticationException) {
            ae = (AuthenticationException) t;
        }
        if (ae == null) {
            //Exception thrown was not an expected AuthenticationException.  Therefore it is probably a little more
            //severe or unexpected.  So, wrap in an AuthenticationException, log to warn, and propagate:
            String msg = "Authentication failed for token submission [" + token + "].  Possible unexpected " +
                    "error? (Typical or expected login exceptions should extend from AuthenticationException).";
            ae = new AuthenticationException(msg, t);
        }
        try {
            notifyFailure(token, ae);
        } catch (Throwable t2) {
            if (log.isWarnEnabled()) {
                String msg = "Unable to send notification for failed authentication attempt - listener error?.  " +
                        "Please check your AuthenticationListener implementation(s).  Logging sending exception " +
                        "and propagating original AuthenticationException instead...";
                log.warn(msg, t2);
            }
        }


        throw ae;
    }

    log.debug("Authentication successful for token [{}].  Returned account [{}]", token, info);

    notifySuccess(token, info);

    return info;
}

// ModularRealmAuthenticator
protected AuthenticationInfo doAuthenticate(AuthenticationToken authenticationToken) throws AuthenticationException {
    assertRealmsConfigured();
    Collection<Realm> realms = getRealms();
    if (realms.size() == 1) {
        return doSingleRealmAuthentication(realms.iterator().next(), authenticationToken);
    } else {
        return doMultiRealmAuthentication(realms, authenticationToken);
    }
}
```
当定义了一个Realm是调用`doSingleRealmAuthentication`方法,多个Realm时调用`doMultiRealmAuthentication`方法。在这里我们看看doMultiRealmAuthentication是如何实现的。
```java
protected AuthenticationInfo doMultiRealmAuthentication(Collection<Realm> realms, AuthenticationToken token) {

    AuthenticationStrategy strategy = getAuthenticationStrategy();

    AuthenticationInfo aggregate = strategy.beforeAllAttempts(realms, token);

    if (log.isTraceEnabled()) {
        log.trace("Iterating through {} realms for PAM authentication", realms.size());
    }

    // 遍历定义的Realm
    for (Realm realm : realms) {

        aggregate = strategy.beforeAttempt(realm, token, aggregate);

        if (realm.supports(token)) {

            log.trace("Attempting to authenticate token [{}] using realm [{}]", token, realm);

            AuthenticationInfo info = null;
            Throwable t = null;
            try {
                // 调用Realm实例的getAuthenticationInfo，进行登陆授权
                // 默认由AuthenticatingRealm实现
                info = realm.getAuthenticationInfo(token);
            } catch (Throwable throwable) {
                t = throwable;
                if (log.isDebugEnabled()) {
                    String msg = "Realm [" + realm + "] threw an exception during a multi-realm authentication attempt:";
                    log.debug(msg, t);
                }
            }

            aggregate = strategy.afterAttempt(realm, token, info, aggregate, t);

        } else {
            log.debug("Realm [{}] does not support token {}.  Skipping realm.", realm, token);
        }
    }

    aggregate = strategy.afterAllAttempts(token, aggregate);

    return aggregate;
}
```

**AuthenticatingRealm**
```java
public final AuthenticationInfo getAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {

    AuthenticationInfo info = getCachedAuthenticationInfo(token);
    if (info == null) {
        // 调用抽象方法doGetAuthenticationInfo
        info = doGetAuthenticationInfo(token);
        log.debug("Looked up AuthenticationInfo [{}] from doGetAuthenticationInfo", info);
        if (token != null && info != null) {
            cacheAuthenticationInfoIfPossible(token, info);
        }
    } else {
        log.debug("Using cached authentication info [{}] to perform credentials matching.", info);
    }

    if (info != null) {
        // 进行密码验证
        assertCredentialsMatch(token, info);
    } else {
        log.debug("No AuthenticationInfo found for submitted AuthenticationToken [{}].  Returning null.", token);
    }

    return info;
}
```

以上就是执行登陆需要的操作。

我们重点看看`ModularRealmAuthenticator`的`doMultiRealmAuthentication`的授权代码部分：
```java
for (Realm realm : realms) {

    aggregate = strategy.beforeAttempt(realm, token, aggregate);

    if (realm.supports(token)) {

        log.trace("Attempting to authenticate token [{}] using realm [{}]", token, realm);

        AuthenticationInfo info = null;
        Throwable t = null;
        try {
            info = realm.getAuthenticationInfo(token);
        } catch (Throwable throwable) {
            t = throwable;
            if (log.isDebugEnabled()) {
                String msg = "Realm [" + realm + "] threw an exception during a multi-realm authentication attempt:";
                log.debug(msg, t);
            }
        }

        aggregate = strategy.afterAttempt(realm, token, info, aggregate, t);

    } else {
        log.debug("Realm [{}] does not support token {}.  Skipping realm.", realm, token);
    }
}
```
该方法遍历定义的多个Realm，执行每个Realm的授权验证方法。但是在验证时抛出指定异常的话，ModularRealmAuthenticator并不会把异常抛出，因为在执行`strategy.afterAllAttempts(token, aggregate)`时会判断有没有正确的授权，如果没有的话则抛出AuthenticationException异常，因为strategy的默认实例为AtLeastOneSuccessfulStrategy对象，要求至少有一个授权通过。

因此如果我们想要定义多个Realm的话，需要重写`AbstractAuthenticationStrategy`的`afterAllAttempts`方法和`ModularRealmAuthenticator`的`doMultiRealmAuthentication`方法，避免在登陆授权时我们自定义的异常没有抛出。

示例代码:
**spring-shiro.xml**
```xml
<bean class="net.cofcool.test.shiro.shiro.AuthenticationStrategy" id="authenticationStrategy" />
<bean class="net.cofcool.test.shiro.shiro.RealmAuthenticator" id="realmAuthenticator" >
    <property name="authenticationStrategy" ref="authenticationStrategy"/>
    <!-- ⚠️注意: 如果有多个Realm的话，通过以下代码定义 -->
    <property name="realms" ref="myRealms" />
</bean>
<util:list id="myRealms">
    <ref bean="myRealm1" />
    <ref bean="myRealm2" />
</util:list>
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <property name="authenticator" ref="realmAuthenticator" />
</bean>
```

**RealmAuthenticator**
```java
@Override
    protected AuthenticationInfo doMultiRealmAuthentication(Collection<Realm> realms,
        AuthenticationToken token) {
    AuthenticationStrategy strategy = getAuthenticationStrategy();

    AuthenticationInfo aggregate = strategy.beforeAllAttempts(realms, token);

    if (log.isTraceEnabled()) {
        log.trace("Iterating through {} realms for PAM authentication", realms.size());
    }

    for (Realm realm : realms) {

        aggregate = strategy.beforeAttempt(realm, token, aggregate);

        if (realm.supports(token)) {

            log.trace("Attempting to authenticate token [{}] using realm [{}]", token, realm);

            AuthenticationInfo info = null;
            Throwable t = null;
            try {
                info = realm.getAuthenticationInfo(token);
            } catch (Throwable throwable) {
                t = throwable;
                if (log.isDebugEnabled()) {
                    String msg = "Realm [" + realm + "] threw an exception during a multi-realm authentication attempt:";
                    log.debug(msg, t);
                }

                // 在此处抛出自定义异常
                if (t instanceof RuntimeException) {
                    throw (RuntimeException)t;
                }
            }

            aggregate = strategy.afterAttempt(realm, token, info, aggregate, t);

        } else {
            log.debug("Realm [{}] does not support token {}.  Skipping realm.", realm, token);
        }
    }

    aggregate = strategy.afterAllAttempts(token, aggregate);

    return aggregate;
}
```

**AuthenticationStrategy**
```java
public class WxAuthenticationStrategy extends AbstractAuthenticationStrategy {

    @Override
    public AuthenticationInfo afterAllAttempts(AuthenticationToken token,
        AuthenticationInfo aggregate) throws AuthenticationException {
        return super.afterAllAttempts(token, aggregate);
    }
}
```

除了以上方法外，也可重写`AbstractAuthenticationStrategy`的`afterAttempt`和`afterAllAttempts`来实现，这样就不需要重写`ModularRealmAuthenticator`的`doMultiRealmAuthentication`方法。
