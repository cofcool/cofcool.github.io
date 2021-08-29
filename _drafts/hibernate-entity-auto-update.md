# Hibernate 源码解析

DefaultFlushEntityEventListener.onFlushEntity

```java
/**
    * Flushes a single entity's state to the database, by scheduling
    * an update action, if necessary
    */
public void onFlushEntity(FlushEntityEvent event) throws HibernateException {
    final Object entity = event.getEntity();
    final EntityEntry entry = event.getEntityEntry();
    final EventSource session = event.getSession();
    final EntityPersister persister = entry.getPersister();
    final Status status = entry.getStatus();
    final Type[] types = persister.getPropertyTypes();

    final boolean mightBeDirty = entry.requiresDirtyCheck( entity );

    final Object[] values = getValues( entity, entry, mightBeDirty, session );

    event.setPropertyValues( values );

    //TODO: avoid this for non-new instances where mightBeDirty==false
    boolean substitute = wrapCollections( session, persister, types, values );

    if ( isUpdateNecessary( event, mightBeDirty ) ) {
        substitute = scheduleUpdate( event ) || substitute;
    }

    if ( status != Status.DELETED ) {
        // now update the object .. has to be outside the main if block above (because of collections)
        if ( substitute ) {
            persister.setPropertyValues( entity, values );
        }

        // Search for collections by reachability, updating their role.
        // We don't want to touch collections reachable from a deleted object
        if ( persister.hasCollections() ) {
            new FlushVisitor( session, entity ).processEntityPropertyValues( values, types );
        }
    }

}
```

主键生成策略:

`DefaultIdentifierGeneratorFactory`:

```java
public DefaultIdentifierGeneratorFactory() {
    register( "uuid2", UUIDGenerator.class );
    register( "guid", GUIDGenerator.class );			// can be done with UUIDGenerator + strategy
    register( "uuid", UUIDHexGenerator.class );			// "deprecated" for new use
    register( "uuid.hex", UUIDHexGenerator.class ); 	// uuid.hex is deprecated
    register( "assigned", Assigned.class );
    register( "identity", IdentityGenerator.class );
    register( "select", SelectGenerator.class );
    register( "sequence", SequenceStyleGenerator.class );
    register( "seqhilo", SequenceHiLoGenerator.class );
    register( "increment", IncrementGenerator.class );
    register( "foreign", ForeignGenerator.class );
    register( "sequence-identity", SequenceIdentityGenerator.class );
    register( "enhanced-sequence", SequenceStyleGenerator.class );
    register( "enhanced-table", TableGenerator.class );
}
```


保存数据:

`AbstractSaveEventListener.performSave`

```java
/**
 * Prepares the save call by checking the session caches for a pre-existing
 * entity and performing any lifecycle callbacks.
 *
 * @param entity The entity to be saved.
 * @param id The id by which to save the entity.
 * @param persister The entity's persister instance.
 * @param useIdentityColumn Is an identity column being used?
 * @param anything Generally cascade-specific information.
 * @param source The session from which the event originated.
 * @param requiresImmediateIdAccess does the event context require
 * access to the identifier immediately after execution of this method (if
 * not, post-insert style id generators may be postponed if we are outside
 * a transaction).
 *
 * @return The id used to save the entity; may be null depending on the
 *         type of id generator used and the requiresImmediateIdAccess value
 */
protected Serializable performSave(
        Object entity,
        Serializable id,
        EntityPersister persister,
        boolean useIdentityColumn,
        Object anything,
        EventSource source,
        boolean requiresImmediateIdAccess) {

    if ( LOG.isTraceEnabled() ) {
        LOG.tracev( "Saving {0}", MessageHelper.infoString( persister, id, source.getFactory() ) );
    }

    final EntityKey key;
    // 主键是否由数据库生成，否则由 Hibernate 生成
    if ( !useIdentityColumn ) {
        key = source.generateEntityKey( id, persister );
        final PersistenceContext persistenceContext = source.getPersistenceContextInternal();
        Object old = persistenceContext.getEntity( key );
        if ( old != null ) {
            if ( persistenceContext.getEntry( old ).getStatus() == Status.DELETED ) {
                source.forceFlush( persistenceContext.getEntry( old ) );
            }
            else {
                throw new NonUniqueObjectException( id, persister.getEntityName() );
            }
        }
        persister.setIdentifier( entity, id, source );
    }
    else {
        key = null;
    }

    if ( invokeSaveLifecycle( entity, persister, source ) ) {
        return id; //EARLY EXIT
    }

    return performSaveOrReplicate(
            entity,
            key,
            persister,
            useIdentityColumn,
            anything,
            source,
            requiresImmediateIdAccess
    );
}
```

`AbstractEntityPersister`, 持久化实体, `EntityLoader` 负责载入实体, `org.hibernate.loader.Loader#loadEntity(org.hibernate.engine.spi.SharedSessionContractImplementor, java.lang.Object, org.hibernate.type.Type, java.lang.Object, java.lang.String, java.io.Serializable, org.hibernate.persister.entity.EntityPersister, org.hibernate.LockOptions, java.lang.Boolean)` 方法执行具体操作, 

`org.hibernate.loader.entity.CacheEntityLoaderHelper` 负责多级缓存处理

`org.hibernate.loader.Loader#instanceNotYetLoaded` 创建实体