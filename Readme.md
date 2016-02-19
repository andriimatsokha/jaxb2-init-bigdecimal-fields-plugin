# Plugin: init-bigdecimal-fields

Plugin extends functionality of artifact
```xml
<dependency>
    <groupId>org.jvnet.jaxb2.maven2</groupId>
    <artifactId>maven-jaxb2-plugin</artifactId>
</dependency>
```

It processes all `java.math.BigDecimal` fields in classes generated by XJC and initializes them using provided customizations.

Status
------

[ ![Build Status](https://travis-ci.org/andriimatsokha/jaxb2-init-bigdecimal-fields-plugin.png?branch=master)](https://travis-ci.org/andriimatsokha/jaxb2-init-bigdecimal-fields-plugin)
[ ![Download](https://api.bintray.com/packages/andriimatsokha/maven/jaxb2-init-bigdecimal-fields-plugin/images/download.svg) ](https://bintray.com/andriimatsokha/maven/jaxb2-init-bigdecimal-fields-plugin/_latestVersion)

## Usage

1. Add this plugin to plugins section of **maven-jaxb2-plugin** in your pom.xml.
2. Enable it by providing `-Xinit-bigdecimal-fields` arg
3. Add customizations for `xs:decimal` fields in your `*.xsd` file
    * Add namespace used by plugin `xmlns:bd="http://init.bigdecimal"`
    * Enable processing of it's customizations `jaxb:extensionBindingPrefixes="bd"`

    **Example of `*.xsd` header:**
    ```xml
    <xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" elementFormDefault="qualified"
             xmlns:jaxb="http://java.sun.com/xml/ns/jaxb"
             jaxb:extensionBindingPrefixes="bd"
             jaxb:version="2.1"
             xmlns:bd="http://init.bigdecimal">
    ```
    * Add init-field customizations.

    **Example:**
    ```xml
    <xs:element name="amount" type="xs:decimal">
        <xs:annotation>
            <xs:appinfo>
                <bd:setStaticValue>ZERO</bd:setStaticValue>
                <bd:executeMethod>
                    <name>setScale</name>
                    <param type="int">2</param>
                    <param type="BigDecimal">ROUND_HALF_UP</param>
                </bd:executeMethod>
            </xs:appinfo>
        </xs:annotation>
    </xs:element>
    ```

#### Available customizations in `*.xsd` file:
Customizations are defined in `xmlns:bd="http://init.bigdecimal"` namespace and processed in the order, they are written in `*.xsd`.
1. **`<bd:setStaticValue>`** - initializes field with static value from `java.math.BigDecimal` class.<br>
For example, this customization:

    ```xml
    <xs:element name="amount" type="xs:decimal">
        <xs:annotation>
            <xs:appinfo>
                <bd:setStaticValue>ZERO</bd:setStaticValue>
            </xs:appinfo>
        </xs:annotation>
    </xs:element>
    ```

    will generate the following code:

    ```java
    BigDecimal amount = BigDecimal.ZERO;
    ```

2. **`<bd:executeMethod>`** - executes method on previously initialized value, so it must be placed after `<setStaticValue>` tag.<br>
    * `<name>` - method name
    * `<param type="">` - parameter that will be passed in method<br>
        Attribute `type` accepts values:
        * `int` - param will be qualified as java `int`
        * `BigDecimal` - param will be qualified as static field in `java.math.BigDecimal` class

    For example, this customization:
    ```xml
    <xs:element name="amount" type="xs:decimal">
        <xs:annotation>
            <xs:appinfo>
                <bd:setStaticValue>ZERO</bd:setStaticValue>
                <bd:executeMethod>
                    <name>setScale</name>
                    <param type="int">2</param>
                    <param type="BigDecimal">ROUND_HALF_UP</param>
                </bd:executeMethod>
            </xs:appinfo>
        </xs:annotation>
    </xs:element>
    ```

    will generate the following code:

    ```java
    BigDecimal amount = BigDecimal.ZERO.setScale(2, BigDecimal.ROUND_HALF_UP);
    ```


##### Example of config in pom.xml:

Below configuration assumes:
* `*.xsd` source files placed in `/src/main/resources/foo`
* generated classes will be placed in `${project.build.directory}/generated-sources/xjc-foo`
* classes will be placed in package `com.foo.bar.package`

```xml
<plugin>
    <groupId>org.jvnet.jaxb2.maven2</groupId>
    <artifactId>maven-jaxb2-plugin</artifactId>
    <version>0.13.1</version>
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>generate</goal>
            </goals>
            <configuration>
                <schemaLanguage>AUTODETECT</schemaLanguage>
                <episode>false</episode>
                <generatePackage>com.foo.bar.package</generatePackage>
                <generateDirectory>${project.build.directory}/generated-sources/xjc-foo</generateDirectory>
                <schemaDirectory>${basedir}/src/main/resources/foo</schemaDirectory>
                <schemaIncludes>
                    <include>**/*.xsd</include>
                </schemaIncludes>
            </configuration>
        </execution>
    </executions>
    <configuration>
        <args>
            <arg>-Xinit-bigdecimal-fields</arg>
        </args>
        <plugins>
            <plugin>
                <groupId>jaxb2-init-plugin</groupId>
                <artifactId>init-bigdecimal-fields</artifactId>
                <version>1.0-SNAPSHOT</version>
            </plugin>
        </plugins>
    </configuration>
</plugin>
```