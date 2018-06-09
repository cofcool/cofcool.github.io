AbstractNativeSessionManager
```java
// session有效期，由globalSessionTimeout控制
public Session start(SessionContext context) {
    Session session = createSession(context);
    applyGlobalSessionTimeout(session);
    onStart(session, context);
    notifyStart(session);
    //Don't expose the EIS-tier Session object to the client-tier:
    return createExposedSession(session, context);
}

protected void applyGlobalSessionTimeout(Session session) {
    session.setTimeout(getGlobalSessionTimeout());
    onChange(session);
}
```
