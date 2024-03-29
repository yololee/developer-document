### 编译插件

> 指定jdk1.8版本

#### Maven全局配置JDK版本方式

修改`${MAVEN_HOME}\conf\settings.xml`

```xml 
<profiles>
    <!-- 全局配置项目JDK版本 -->
    <profile>
        <id>jdk1.8</id>
        <activation>
            <activeByDefault>true</activeByDefault>
            <jdk>1.8</jdk>
        </activation>
        <properties>
            <maven.compiler.source>1.8</maven.compiler.source>
            <maven.compiler.target>1.8</maven.compiler.target>
            <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
        </properties>
    </profile>
</profiles>
```

#### 单独项目配置

如果没有上面的全局配置，我们可以通过maven的编译插件来实现

`pom.xml`

```xml
<!-- 编译插件 -->
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <encoding>UTF-8</encoding>
                <compilerArgs>
                    <arg>-parameters</arg>
                </compilerArgs>
            </configuration>
        </plugin>
    </plugins>
</build>
```