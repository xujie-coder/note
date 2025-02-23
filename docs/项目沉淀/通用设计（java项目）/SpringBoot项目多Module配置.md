在 Spring Boot 项目中，配置多个模块可以帮助我们将应用分解成多个逻辑部分，每个部分都是一个独立的模块，这样可以提高代码的可管理性和可重用性。这里使用别人的项目作为参考，具体应用到自己的项目需要变更

```plain
.
├── nft-turbo-common
│   ├── pom.xml
│   ├── src
│   │   └── main
│   │       ├── java
│   │       └── resources
├── nft-turbo-business
│   ├── pom.xml
│   ├── src
│   │   └── main
│   │       ├── java
│   │       └── resources
├── nft-turbo-auth
│   └── pom.xml
├── nft-turbo-gateway
│   ├── pom.xml
│   ├── src
│   │   ├── main
│   │   │   └── resources
│   │   │       └── application.yml
├── pom.xml
```

这里面有4个 module，分别是nft-turbo-gateway、nft-turbo-auth、nft-turbo-business 以及nft-turbo-common。每一个 module 中都需要有一个自己的 pom 文件，并且在整个项目中还有一个总的 pom 文件。

父 pom 中需要配置如下：

```plain
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">

<modelVersion>4.0.0</modelVersion>
<groupId>com.xjtu</groupId>
<artifactId>NFTFast_Server</artifactId>
<version>1.0-SNAPSHOT</version>
<name>NFT FAST</name>
<packaging>pom</packaging>
<description>A NFT FAST</description>

<properties>
    <java.version>21</java.version>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>21</java.version>
    <spring-boot.version>3.2.2</spring-boot.version>
    <spring-cloud.version>2023.0.0</spring-cloud.version>
    <spring-cloud-alibaba.version>2023.0.1.2</spring-cloud-alibaba.version>
    <dubbo.version>3.2.10</dubbo.version>
</properties>

<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.2</version>
</parent>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>

        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>${spring-cloud-alibaba.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>6.1.3</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.30</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-base</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-auth</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-gateway</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-admin</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-notice</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-chain</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-cache</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-sa-token</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-limiter</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-web</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-datasource</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-es</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-rpc</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-mq</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-lock</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-file</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-sms</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-config</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-collection</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-goods</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-pay</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-order</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-user</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-trade</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-job</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.xjtu</groupId>
            <artifactId>NFTFast_Server-seata</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>${dubbo.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-registry-nacos</artifactId>
            <version>${dubbo.version}</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba.fastjson2</groupId>
            <artifactId>fastjson2</artifactId>
            <version>2.0.42</version>
        </dependency>

        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>32.1.3-jre</version>
        </dependency>

        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct</artifactId>
            <version>1.6.0.Beta1</version>
        </dependency>
    </dependencies>
</dependencyManagement>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.11.0</version>
            <configuration>
                <source>21</source>
                <target>21</target>
                <annotationProcessorPaths>
                    <path>
                        <groupId>org.projectlombok</groupId>
                        <artifactId>lombok</artifactId>
                        <version>1.18.30</version>
                    </path>
                    <path>
                        <groupId>org.mapstruct</groupId>
                        <artifactId>mapstruct-processor</artifactId>
                        <version>1.5.5.Final</version>
                    </path>
                </annotationProcessorPaths>
            </configuration>
        </plugin>

        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>

        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-resources-plugin</artifactId>
            <version>3.1.0</version>
            <configuration>
                <nonFilteredFileExtensions>
                    <nonFilteredFileExtension>p12</nonFilteredFileExtension>
                    <nonFilteredFileExtension>pem</nonFilteredFileExtension>
                </nonFilteredFileExtensions>
            </configuration>
        </plugin>
    </plugins>

    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>*/**</include>
            </includes>
            <filtering>true</filtering>
        </resource>
    </resources>

</build>
<repositories>
    <repository>
        <id>spring-milestones</id>
        <name>Spring Milestones</name>
        <url>https://repo.spring.io/milestone</url>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
</repositories>

<pluginRepositories>
    <pluginRepository>
        <id>spring-milestones</id>
        <name>Spring Milestones</name>
        <url>https://repo.spring.io/milestone</url>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </pluginRepository>
</pluginRepositories>

<modules>
    <module>NFTFast_Server-common</module>
    <module>NFTFast_Server-auth</module>
    <module>NFTFast_Server-gateway</module>
    <module>NFTFast_Server-business</module>
    <module>NFTFast_Server-admin</module>
</modules>
</project>

```



通过<modules></modules>标签来指定一共都包含哪些 module：

```plain
<modules>
  <module>nft-turbo-common</module>
  <module>nft-turbo-auth</module>
  <module>nft-turbo-gateway</module>
  <module>nft-turbo-business</module>
  <module>nft-turbo-admin</module>
</modules>
```



接下来，就在父 pom 的同级目录创建对应的 module的文件夹。并在 module 的目录中创建 pom 文件，如nft-turbo-business的 pom 内容如下：

```plain
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>
  <packaging>pom</packaging>
  <parent>
      <groupId>com.xjtu</groupId>
      <artifactId>NFTFast_Server</artifactId>
      <version>1.0-SNAPSHOT</version>
  </parent>

  <groupId>com.xjtu</groupId>
  <artifactId>NFTFast_Server-business</artifactId>
  <version>1.0-SNAPSHOT</version>
  <properties>
      <application.name>nffast-business</application.name>
      <maven.compiler.source>21</maven.compiler.source>
      <maven.compiler.target>21</maven.compiler.target>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <modules>
      <module>NFTFast_Server-notice</module>
      <module>NFTFast_Server-chain</module>
      <module>NFTFast_Server-collection</module>
      <module>NFTFast_Server-pay</module>
      <module>NFTFast_Server-app</module>
      <module>NFTFast_Server-order</module>
      <module>NFTFast_Server-user</module>
      <module>NFTFast_Server-trade</module>
      <module>NFTFast_Server-goods</module>
  </modules>
</project>

```



用parent指一下父包即可

```plain
<parent>
    <groupId>com.xjtu</groupId>
    <artifactId>NFTFast_Server</artifactId>
    <version>1.0-SNAPSHOT</version>
</parent>
```

