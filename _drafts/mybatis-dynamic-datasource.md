# Spring Mybatis 实现动态数据源

```xml
 <bean id="dataSource" class="com.zx.tk.common.data.DynamicDataSource">
    <property name="targetDataSources">
        <map>
            <entry key="dataSource1" value-ref="dataSource1"/>
        </map>
    </property>
    <property name="defaultTargetDataSource" ref="dataSource1"/>
</bean>
```
AbstractRoutingDataSource
```java
/**
 * Determine the current lookup key. This will typically be
 * implemented to check a thread-bound transaction context.
 * <p>Allows for arbitrary keys. The returned key needs
 * to match the stored lookup key type, as resolved by the
 * {@link #resolveSpecifiedLookupKey} method.
 */
protected abstract Object determineCurrentLookupKey();
```
