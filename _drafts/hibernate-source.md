# Spring Boot JPA自动配置分析

```java
@Configuration
@ConditionalOnClass({ LocalContainerEntityManagerFactoryBean.class, EntityManager.class })
@Conditional(HibernateEntityManagerCondition.class)
@EnableConfigurationProperties(JpaProperties.class)
@AutoConfigureAfter({ DataSourceAutoConfiguration.class })
@Import(HibernateJpaConfiguration.class)
public class HibernateJpaAutoConfiguration {

}
```

```java
@EnableConfigurationProperties(JpaProperties.class)
@Import(DataSourceInitializedPublisher.Registrar.class)
public abstract class JpaBaseConfiguration implements BeanFactoryAware {
    @Bean
	@ConditionalOnMissingBean
	public PlatformTransactionManager transactionManager() {
		JpaTransactionManager transactionManager = new JpaTransactionManager();
		if (this.transactionManagerCustomizers != null) {
			this.transactionManagerCustomizers.customize(transactionManager);
		}
		return transactionManager;
	}

	@Bean
	@ConditionalOnMissingBean
	public JpaVendorAdapter jpaVendorAdapter() {
		AbstractJpaVendorAdapter adapter = createJpaVendorAdapter();
		adapter.setShowSql(this.properties.isShowSql());
		adapter.setDatabase(this.properties.determineDatabase(this.dataSource));
		adapter.setDatabasePlatform(this.properties.getDatabasePlatform());
		adapter.setGenerateDdl(this.properties.isGenerateDdl());
		return adapter;
	}

	@Bean
	@ConditionalOnMissingBean
	public EntityManagerFactoryBuilder entityManagerFactoryBuilder(
			JpaVendorAdapter jpaVendorAdapter,
			ObjectProvider<PersistenceUnitManager> persistenceUnitManager) {
		EntityManagerFactoryBuilder builder = new EntityManagerFactoryBuilder(
				jpaVendorAdapter, this.properties.getProperties(),
				persistenceUnitManager.getIfAvailable());
		builder.setCallback(getVendorCallback());
		return builder;
	}

	@Bean
	@Primary
	@ConditionalOnMissingBean({ LocalContainerEntityManagerFactoryBean.class,
			EntityManagerFactory.class })
	public LocalContainerEntityManagerFactoryBean entityManagerFactory(
			EntityManagerFactoryBuilder factoryBuilder) {
		Map<String, Object> vendorProperties = getVendorProperties();
		customizeVendorProperties(vendorProperties);
		return factoryBuilder.dataSource(this.dataSource).packages(getPackagesToScan())
				.properties(vendorProperties).mappingResources(getMappingResources())
				.jta(isJta()).build();
	}
}
```

```java
@ConfigurationProperties(prefix = "spring.jpa")
public class JpaProperties {

	/**
	 * Additional native properties to set on the JPA provider.
	 */
	private Map<String, String> properties = new HashMap<>();

	/**
	 * Mapping resources (equivalent to "mapping-file" entries in persistence.xml).
	 */
	private final List<String> mappingResources = new ArrayList<>();

	/**
	 * Name of the target database to operate on, auto-detected by default. Can be
	 * alternatively set using the "Database" enum.
	 */
	private String databasePlatform;

	/**
	 * Target database to operate on, auto-detected by default. Can be alternatively set
	 * using the "databasePlatform" property.
	 */
	private Database database;

	/**
	 * Whether to initialize the schema on startup.
	 */
	private boolean generateDdl = false;

	/**
	 * Whether to enable logging of SQL statements.
	 */
	private boolean showSql = false;

	/**
	 * Register OpenEntityManagerInViewInterceptor. Binds a JPA EntityManager to the
	 * thread for the entire processing of the request.
	 */
	private Boolean openInView;

	private Hibernate hibernate = new Hibernate();

     ...

    public static class Hibernate {

		private static final String USE_NEW_ID_GENERATOR_MAPPINGS = "hibernate.id."
				+ "new_generator_mappings";

		/**
		 * DDL mode. This is actually a shortcut for the "hibernate.hbm2ddl.auto"
		 * property. Defaults to "create-drop" when using an embedded database and no
		 * schema manager was detected. Otherwise, defaults to "none".
		 */
		private String ddlAuto;

		/**
		 * Whether to use Hibernate's newer IdentifierGenerator for AUTO, TABLE and
		 * SEQUENCE. This is actually a shortcut for the
		 * "hibernate.id.new_generator_mappings" property. When not specified will default
		 * to "true".
		 */
		private Boolean useNewIdGeneratorMappings;

		private final Naming naming = new Naming();

		public String getDdlAuto() {
			return this.ddlAuto;
		}

		public void setDdlAuto(String ddlAuto) {
			this.ddlAuto = ddlAuto;
		}

		public Boolean isUseNewIdGeneratorMappings() {
			return this.useNewIdGeneratorMappings;
		}

		public void setUseNewIdGeneratorMappings(Boolean useNewIdGeneratorMappings) {
			this.useNewIdGeneratorMappings = useNewIdGeneratorMappings;
		}

		public Naming getNaming() {
			return this.naming;
		}

		private Map<String, Object> getAdditionalProperties(Map<String, String> existing,
				HibernateSettings settings) {
			Map<String, Object> result = new HashMap<>(existing);
			applyNewIdGeneratorMappings(result);
			getNaming().applyNamingStrategies(result);
			String ddlAuto = determineDdlAuto(existing, settings::getDdlAuto);
			if (StringUtils.hasText(ddlAuto) && !"none".equals(ddlAuto)) {
				result.put("hibernate.hbm2ddl.auto", ddlAuto);
			}
			else {
				result.remove("hibernate.hbm2ddl.auto");
			}
			Collection<HibernatePropertiesCustomizer> customizers = settings
					.getHibernatePropertiesCustomizers();
			if (!ObjectUtils.isEmpty(customizers)) {
                // 调用自定义方法
				customizers.forEach((customizer) -> customizer.customize(result));
			}
			return result;
		}

		private void applyNewIdGeneratorMappings(Map<String, Object> result) {
			if (this.useNewIdGeneratorMappings != null) {
				result.put(USE_NEW_ID_GENERATOR_MAPPINGS,
						this.useNewIdGeneratorMappings.toString());
			}
			else if (!result.containsKey(USE_NEW_ID_GENERATOR_MAPPINGS)) {
				result.put(USE_NEW_ID_GENERATOR_MAPPINGS, "true");
			}
		}

		private String determineDdlAuto(Map<String, String> existing,
				Supplier<String> defaultDdlAuto) {
			String ddlAuto = existing.get("hibernate.hbm2ddl.auto");
			if (ddlAuto != null) {
				return ddlAuto;
			}
			return (this.ddlAuto != null ? this.ddlAuto : defaultDdlAuto.get());
		}

	}

	public static class Naming {

		private static final String DEFAULT_PHYSICAL_STRATEGY = "org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy";

		private static final String DEFAULT_IMPLICIT_STRATEGY = "org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy";

		/**
		 * Fully qualified name of the implicit naming strategy.
		 */
		private String implicitStrategy;

		/**
		 * Fully qualified name of the physical naming strategy.
		 */
		private String physicalStrategy;

		public String getImplicitStrategy() {
			return this.implicitStrategy;
		}

		public void setImplicitStrategy(String implicitStrategy) {
			this.implicitStrategy = implicitStrategy;
		}

		public String getPhysicalStrategy() {
			return this.physicalStrategy;
		}

		public void setPhysicalStrategy(String physicalStrategy) {
			this.physicalStrategy = physicalStrategy;
		}

		private void applyNamingStrategies(Map<String, Object> properties) {
			applyNamingStrategy(properties, "hibernate.implicit_naming_strategy",
					this.implicitStrategy, DEFAULT_IMPLICIT_STRATEGY);
			applyNamingStrategy(properties, "hibernate.physical_naming_strategy",
					this.physicalStrategy, DEFAULT_PHYSICAL_STRATEGY);
		}

		private void applyNamingStrategy(Map<String, Object> properties, String key,
				Object strategy, Object defaultStrategy) {
			if (strategy != null) {
				properties.put(key, strategy);
			}
			else if (defaultStrategy != null && !properties.containsKey(key)) {
				properties.put(key, defaultStrategy);
			}
		}

	}
}
```

JpaConfig
```java
@Bean
public LocalContainerEntityManagerFactoryBean entityManagerFactory(DataSource dataSource) {
    LocalContainerEntityManagerFactoryBean entityManagerFactoryBean = new LocalContainerEntityManagerFactoryBean();
    entityManagerFactoryBean.setPackagesToScan("com.xingdata.server.control.service");

    HibernateJpaVendorAdapter jpaVendorAdapter = new HibernateJpaVendorAdapter();
    jpaVendorAdapter.setDatabasePlatform("org.hibernate.dialect.MySQL57Dialect");
    entityManagerFactoryBean.setJpaVendorAdapter(jpaVendorAdapter);
    // 定义packagesToScan, mappingResources其中之一即可
    // DefaultPersistenceUnitManager负责扫描，可参考readPersistenceUnitInfos方法
    entityManagerFactoryBean.setDataSource(dataSource);
    entityManagerFactoryBean.setPackagesToScan("com.xingdata.server.api");

    return entityManagerFactoryBean;
}

```

LocalContainerEntityManagerFactoryBean
```java
```