# Spring 实现动态数据源

```xml
 <bean id="dataSource" class="net.cofcool.test.spring.data.DynamicDataSource">
    <property name="targetDataSources">
        <map>
            <entry key="dataSource1" value-ref="dataSource1"/>
            <entry key="dataSource2" value-ref="dataSource2"/>
        </map>
    </property>
    <property name="defaultTargetDataSource" ref="dataSource1"/>
</bean>
```
AbstractRoutingDataSource
```java
public abstract class AbstractRoutingDataSource extends AbstractDataSource implements InitializingBean {

    @Nullable
    private Map<Object, Object> targetDataSources;

    public Connection getConnection() throws SQLException {
    	return determineTargetDataSource().getConnection();
    }

    protected DataSource determineTargetDataSource() {
        Assert.notNull(this.resolvedDataSources, "DataSource router not initialized");
        Object lookupKey = determineCurrentLookupKey();
        DataSource dataSource = this.resolvedDataSources.get(lookupKey);
        if (dataSource == null && (this.lenientFallback || lookupKey == null)) {
        	dataSource = this.resolvedDefaultDataSource;
        }
        if (dataSource == null) {
        	throw new IllegalStateException("Cannot determine target DataSource for lookup key [" + lookupKey + "]");
        }
        return dataSource;
    }

    // 子类实现数据源切换逻辑
    protected abstract Object determineCurrentLookupKey();

    ...
}
```
