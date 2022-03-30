# THIS REPO IS DEPRECATED

**The templates in this repo have been accepted upstream into OpenAPI Generator and are no longer needed**

# Customized Quarkus Template Files For OpenAPI Generator

## Overview
For our team, we almost never create an application in Quarkus without using Hibernate+Panache and so we wanted to ensure that we could add
Hibernate/JPA annotations to our entity classes as part of the generated code from OpenAPI Generator. In this repository you will find our customized `pojo.mustache` which allows for insertion of any sort of Java annotation at the class, field, getter, or setter level using the following vendor extensions:

* `x-java-class-annotations`
  * An array of annotations (They must be fully qualified as this will not import) to be added above the class definition
* `x-java-field-annotations`
  * An array of annotations (They must be fully qualified as this will not import) to be added above each field definition
* `x-java-getter-annotations`
  * An array of annotations (They must be fully qualified as this will not import) to be added above each getter definition
* `x-java-setter-annotations`
  * An array of annotations (They must be fully qualified as this will not import) to be added above each setter definition

## OpenAPI Specification Example:

```
// ... SNIP
components:
  schemas:
    Todo:
      title: Todo
      description: ''
      required:
        - completed
        - title
        - id
      type: object
      properties:
        id:
          format: uuid
          type: string
          x-java-field-annotations:
            - '@javax.persistence.Id'
            - '@javax.persistence.GeneratedValue(generator = "UUID")'
            - '@javax.persistence.Column(name = "id", updatable = false, nullable = false)'
        title:
          type: string
        description:
          type: string
        completed:
          default: false
          type: boolean
        dueDate:
          format: date-time
          type: string
      example:
        id: '337a644e-0282-11ec-accf-db1e758df2be'
        title: My Todo Task
        description: More details here
        completed: false
        dueDate: '2020-10-29'
      x-java-class-annotations:
        - '@javax.persistence.Entity'
        - '@javax.persistence.Table(name = "todos")'
```

And the resulting class:

```java
// ... SNIP
@javax.annotation.Generated(value = "org.openapitools.codegen.languages.JavaJAXRSSpecServerCodegen", date = "2021-08-21T09:02:26.056235811-04:00[America/New_York]")
@javax.persistence.Entity
@javax.persistence.Table(name = "todos")
public class Todo implements Serializable {

    @javax.persistence.Id
    @javax.persistence.GeneratedValue(generator = "UUID")
    @javax.persistence.Column(name = "id", updatable = false, nullable = false)
    private UUID id;

    // ... SNIP
}
```

## Quickstart

Add the following snippets to your Maven POM file:

```xml
<!-- Adds the `src/gen/java` directory to the Maven source path -->
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>build-helper-maven-plugin</artifactId>
  <version>3.1.0</version>
  <executions>
    <execution>
      <phase>generate-sources</phase>
      <goals>
        <goal>add-source</goal>
      </goals>
      <configuration>
        <sources>
          <source>src/gen/java</source>
        </sources>
      </configuration>
    </execution>
  </executions>
</plugin>
<!-- Clones this repository into the `templates` directory -->
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-scm-plugin</artifactId>
  <version>1.11.2</version>
  <executions>
    <execution>
      <?m2e execute onConfiguration?>
      <phase>initialize</phase>
      <goals>
        <goal>checkout</goal>
      </goals>
      <configuration>
        <connectionUrl>scm:git:https://github.com/redhat-appdev-practice/openapi-generator-quarkus-templates.git
        </connectionUrl>
        <checkoutDirectory>templates</checkoutDirectory>
        <connectionType>connection</connectionType>
      </configuration>
    </execution>
  </executions>
</plugin>
<!-- Adds the generated sources directory to the list of directories to be cleaned -->
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-clean-plugin</artifactId>
  <configuration>
    <filesets>
      <fileset>
        <directory>${project.basedir}/src/gen</directory>
      </fileset>
      <fileset>
        <directory>${project.basedir}/target</directory>
      </fileset>
      <fileset>
        <directory>${project.basedir}/templates</directory>
      </fileset>
    </filesets>
  </configuration>
</plugin>
<!-- Configure's OpenAPI Generator to use the custom templates -->
<plugin>
  <artifactId>openapi-generator-maven-plugin</artifactId>
  <groupId>org.openapitools</groupId>
  <version>5.1.1</version>
  <executions>
    <execution>
      <phase>generate-sources</phase>
      <goals>
        <goal>generate</goal>
      </goals>
      <configuration>
        <generatorName>jaxrs-spec</generatorName>
        <library>quarkus</library>
        <groupId>${project.groupId}</groupId>
        <artifactId>${project.artifactId}</artifactId>
        <artifactVersion>${project.version}</artifactVersion>
        <output>${project.basedir}</output>
        <skipOverwrite>false</skipOverwrite>
        <inputSpec>${project.basedir}/src/main/resources/openapi.yml</inputSpec>
        <templateDirectory>${project.basedir}/templates</templateDirectory>
        <configOptions>
          <dateLibrary>java8</dateLibrary>
          <modelPackage>com.redhat.consulting.feedback360.models</modelPackage>
          <invokerPackage>com.redhat.consulting.feedback360</invokerPackage>
          <apiPackage>com.redhat.consulting.feedback360.api</apiPackage>
          <serializableModel>true</serializableModel>
          <sortModelPropertiesByRequiredFlag>true</sortModelPropertiesByRequiredFlag>
          <useTags>true</useTags>
          <java8>true</java8>
          <useSwaggerAnnotations>false</useSwaggerAnnotations>
          <interfaceOnly>true</interfaceOnly>
          <sourceFolder>src/gen/java</sourceFolder>
        </configOptions>
      </configuration>
    </execution>
  </executions>
</plugin>
<!-- Formats the generated code to be clean and well structured -->
<plugin>
  <groupId>net.revelc.code.formatter</groupId>
  <artifactId>formatter-maven-plugin</artifactId>
  <version>2.16.0</version>
  <executions>
    <execution>
      <phase>process-sources</phase>
      <goals>
        <goal>format</goal>
      </goals>
      <configuration>
        <directories>
          <directory>${project.basedir}/src/gen/java</directory>
          <directory>${project.build.sourceDirectory}</directory>
          <directory>${project.build.testSourceDirectory}</directory>
        </directories>
      </configuration>
    </execution>
  </executions>
</plugin>
```

## Even Quicker Start

Use the [openapi-quarkus-archetype](https://github.com/redhat-appdev-practice/openapi-quarkus-archetype)
