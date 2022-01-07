# Spring Boot Web 项目文档自动生成

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-ui</artifactId>
    <version>1.5.12</version>
</dependency>
```

enunciate.xml
```xml
<enunciate xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:noNamespaceSchemaLocation="https://enunciate.webcohesion.com/schemas/enunciate-2.9.0.xsd">
    <application root="/rest" />
    <api-classes>
        <include pattern="com.eversec.saas.app.common.domain"/>
        <include pattern="com.eversec.saas.app.common.param"/>
        <include pattern="com.eversec.saas.app.common.constant"/>
        <include pattern="com.eversec.saas.app.dataserver.**"/>
        <include pattern="com.eversec.saas.app.engine.**"/>
    </api-classes>
    <modules>
        <gwt-json-overlay disabled="true"/>
        <c-xml-client disabled="true"/>
        <csharp-xml-client disabled="true"/>
        <idl disabled="true"/>
        <java-json-client disabled="true"/>
        <javascript-client disabled="true"/>
        <java-xml-client disabled="true"/>
        <jaxb disabled="true"/>
        <jackson disabled="false"/>
        <gwt-json-overlay disabled="true"/>
        <jaxws disabled="true"/>
        <jaxrs disabled="true"/>
        <php-xml-client disabled="true"/>
        <php-json-client disabled="true"/>
        <ruby-json-client disabled="true"/>
        <obj-c-xml-client disabled="true"/>
    </modules>
</enunciate>
```

```xml
<plugin>
    <groupId>com.webcohesion.enunciate</groupId>
    <artifactId>enunciate-maven-plugin</artifactId>
    <version>2.13.3</version>
    <executions>
        <execution>
            <id>docs</id>
            <goals>
                <goal>docs</goal>
            </goals>
            <configuration>
                <configFile>${project.basedir}/../enunciate.xml</configFile>
                <docsDir>${project.build.directory}</docsDir>
                <sourcepath-includes>
                    <sourcepath-include>
                        <groupId>com.eversec.saas.app</groupId>
                        <artifactId>app-data-common</artifactId>
                    </sourcepath-include>
                </sourcepath-includes>
                <sources>
                    <source>${project.build.directory}/generated-sources/delombok</source>
                    <source>${project.basedir}/../app-data-common/target/generated-sources/delombok</source>
                </sources>
            </configuration>
        </execution>
    </executions>
</plugin>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-source-plugin</artifactId>
    <executions>
        <execution>
            <id>attach-sources</id>
            <phase>package</phase>
            <goals>
                <goal>jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>
<plugin>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok-maven-plugin</artifactId>
    <version>${lombok-plugin.version}</version>
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>delombok</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <encoding>utf-8</encoding>
        <addOutputDirectory>false</addOutputDirectory>
        <sourceDirectory>src/main/java</sourceDirectory>
    </configuration>
</plugin>

```

```java
@Bean
public OpenApiCustomiser openApiCustomiser(ObjectMapper objectMapper) {
    return openApi -> {
        try {
            String path;
            if (DataSyncVersion.getVersion().equals(DataSyncVersion.UNKNOWN)) {
                // IDE run
                return;
            } else {
                // start.sh run from zip file
                path = System.getProperty("user.dir") + File.separator + "swagger.json";
            }
            Map<String, ?> api = objectMapper.readValue(ResourceUtils.getFile(path), LinkedHashMap.class);
            Map<String, Map<String, ?>> definitions = (Map<String, Map<String, ?>>) api.get("definitions");
            Map<String, Schema> schemaMap = new LinkedHashMap<>();
            definitions.forEach((k, v) -> {
                schemaMap.put(k, new Schema()
                    .title((String) v.get("title"))
                    .type((String) v.get("type"))
                    .properties((Map<String, Schema>) v.get("properties"))
                    .example(v.get("example"))
                    .description((String) v.get("description"))
                );
            });
            openApi.getComponents().setSchemas(schemaMap);
        } catch (IOException e) {
            log.error("OpenApi configure error", e);
        }
    };
}
```