Spring HTTP invoker

HttpInvokerRequestExecutor
```java
public interface HttpInvokerRequestExecutor {

	RemoteInvocationResult executeRequest(HttpInvokerClientConfiguration config, RemoteInvocation invocation)
			throws Exception;

}
```



HttpInvokerServiceExporter

----
client

HttpInvokerClientConfiguration
```java
public interface HttpInvokerClientConfiguration {

	String getServiceUrl();

	@Nullable
	String getCodebaseUrl();

}
```

HttpInvokerProxyFactoryBean