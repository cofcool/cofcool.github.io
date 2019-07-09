# Hibernate 实体自动更新源码解析

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