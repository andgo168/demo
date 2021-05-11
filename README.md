### 一、前言
最近公司項目準備開始重構，框架選定為 Spring Boot ，本篇主要記錄了在 IDEA 中搭建 Spring Boot Maven 多模組專案的過程。

***
### 二、軟體及硬體環境
* macOS Sierra 10.12.6
* IntelliJ IDEA 2018.2
* JDK 1.8
* Maven 3.2.1
* Spring Boot 2.0.4

***
### 三、專案結構
* biz 層（業務邏輯層）
* dao 層（資料持久層）
* common 層（公用組件層）
* web 層（請求處理層）

> 注：biz 層依賴 dao 及 common 層， web 層依賴 biz 層

***
### 四、項目搭建
#### 4.1 創建父工程
① IDEA 主面板選擇功能表「Create New Project 」或者工具列選擇功能表「 File -> New -> Project... 」
![SpringBoot_1_1.png](http://blog.shoegaze.com/SpringBoot_1_1.png)
② 側邊欄選擇「 Spring Initializr 」，Initializr 默認選擇 Default ，然後點擊「 Next 」
![SpringBoot_1_2.png](http://blog.shoegaze.com/SpringBoot_1_2.png)
③ 修改 Group 、 Artifact 、 Package 輸入框中的值後點擊「 Next 」
![SpringBoot_1_3.png](http://blog.shoegaze.com/SpringBoot_1_3.png)
④ 這步暫時先不需要選擇，直接點「 Next 」
![SpringBoot_1_4.png](http://blog.shoegaze.com/SpringBoot_1_4.png)
⑤ 點擊「 Finish 」創建項目
![SpringBoot_1_5.png](http://blog.shoegaze.com/SpringBoot_1_5.png)
⑥ 最終得到的專案目錄結構如下
``` bash
|-- demo
  |-- .gitignore
  |-- mvnw
  |-- mvnw.cmd
  |-- pom.xml
  |-- .mvn
  |   |-- wrapper
  |       |-- maven-wrapper.jar
  |       |-- maven-wrapper.properties
  |-- src
      |-- main
      |   |-- java
      |   |   |-- com
      |   |       |-- example
      |   |           |-- demo
      |   |               |-- DemoApplication.java
      |   |-- resources
      |       |-- application.properties
      |-- test
          |-- java
              |-- com
                  |-- example
                      |-- demo
                          |-- DemoApplicationTests.java
```
⑦ 刪除無用的 .mvn 目錄、 src 目錄、 mvnw 及 mvnw.cmd 檔，最終只留 .gitignore 和 pom.xml
#### 4.2 創建子模組
① 選擇專案根目錄，右鍵呼出功能表，選擇「 New -> Module 」
![SpringBoot_1_6.png](http://blog.shoegaze.com/SpringBoot_1_6.png)
② 側邊欄選擇「 Maven 」，點擊「 Next 」
![SpringBoot_1_7.png](http://blog.shoegaze.com/SpringBoot_1_7.png)
③ 填寫 ArifactId ，點擊「 Next 」
![SpringBoot_1_8.png](http://blog.shoegaze.com/SpringBoot_1_8.png)
④ 修改 Module name 增加橫杠提升可讀性，點擊「 Finish 」
![SpringBoot_1_9.png](http://blog.shoegaze.com/SpringBoot_1_9.png)
⑤ 同理添加「 demo-dao 」、「 demo-common 」、「 demo-web 」子模組，最終得到專案目錄結構如下
``` bash
|-- demo
    |-- .gitignore
    |-- pom.xml
    |-- demo-biz
    |   |-- pom.xml
    |   |-- src
    |       |-- main
    |       |   |-- java
    |       |   |-- resources
    |       |-- test
    |           |-- java
    |-- demo-common
    |   |-- pom.xml
    |   |-- src
    |       |-- main
    |       |   |-- java
    |       |   |-- resources
    |       |-- test
    |           |-- java
    |-- demo-dao
    |   |-- pom.xml
    |   |-- src
    |       |-- main
    |       |   |-- java
    |       |   |-- resources
    |       |-- test
    |           |-- java
    |-- demo-web
      |-- pom.xml
      |-- src
          |-- main
          |   |-- java
          |   |-- resources
          |-- test
              |-- java
```
#### 4.3 整理父 pom 檔中的內容
① 刪除 dependencies 標籤及其中的 spring-boot-starter 和 spring-boot-starter-test 依賴，因為 Spring Boot 提供的父工程已包含，並且父 pom 原則上都是通過 dependencyManagement 標籤管理依賴包。
> 注：dependencyManagement 及 dependencies 的區別自行查閱文檔

② 刪除 build 標籤及其中的所有內容，spring-boot-maven-plugin 外掛程式作用是打一個可運行的包，多模組專案僅僅需要在入口類所在的模組添加打包外掛程式，這裡父模組不需要打包運行。而且該外掛程式已被包含在 Spring Boot 提供的父工程中，這裡刪掉即可。
③ 最後整理父 pom 檔中的其餘內容，按其代表含義歸類，整理結果如下：
``` xml
<!-- 基本資訊 -->
<modelVersion>4.0.0</modelVersion>
<packaging>pom</packaging>
<name>demo</name>
<description>Demo project for Spring Boot</description>

<!-- 專案說明：這裡作為聚合工程的父工程 -->
<groupId>com.example.demo</groupId>
<artifactId>demo</artifactId>
<version>0.0.1-SNAPSHOT</version>

<!-- 繼承說明：這裡繼承Spring Boot提供的父工程 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.2.RELEASE</version>
    <relativePath/>
</parent>

<!-- 模組說明：這裡聲明多個子模組 -->
<modules>
    <module>demo-biz</module>
    <module>demo-common</module>
    <module>demo-dao</module>
    <module>demo-web</module>
</modules>

<!-- 屬性說明 -->
<properties>
    <java.version>1.8</java.version>
    <demo.version>0.0.1-SNAPSHOT</demo.version>
</properties>
```
#### 4.4 簡易 HTTP 介面測試
準備工作都完成之後，通過一個簡易的 HTTP 介面測試專案是否正常運行。

① 首先在 demo-web 層創建 com.example.demo.web 包並添加入口類 DemoWebApplication.java
> 注：com.example.demo.web 為多級目錄結構並非單個目錄名

``` java
package com.example.demo.web;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @author linjian
 * @date 2019/1/15
 */
@SpringBootApplication
public class DemoWebApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoWebApplication.class, args);
    }
}
```
② 其次在 demo-web 層的 pom 檔中添加必要的依賴包
``` xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```
② 然後在 com.example.demo.web 包中添加 controller 目錄並新建一個 controller，添加 test 方法測試介面是否可以正常訪問。
``` java
package com.example.demo.web.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author linjian
 * @date 2019/1/15
 */
@RestController
@RequestMapping("demo")
public class DemoController {

    @GetMapping("test")
    public String test() {
        return "Hello World!";
    }
}
```
③ 最後運行 DemoWebApplication 類中的 main 方法啟動專案，默認埠為 8080，訪問 http://localhost:8080/demo/test 即可測試介面
![SpringBoot_1_10.png](http://blog.shoegaze.com/SpringBoot_1_10.png)
#### 4.5 配置模組間的依賴關係
通常 JAVA Web 專案會按照功能劃分不同模組，模組之間通過依賴關係進行協作，下面將完善模組之間的依賴關係。

① 首先在父 pom 檔中使用「 dependencyManagement 」標籤聲明所有子模組依賴
``` xml
<!-- 依賴管理：這裡統一管理依賴的版本號 -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.example.demo</groupId>
            <artifactId>demo-biz</artifactId>
            <version>${demo.version}</version>
        </dependency>
        <dependency>
            <groupId>com.example.demo</groupId>
            <artifactId>demo-common</artifactId>
            <version>${demo.version}</version>
        </dependency>
        <dependency>
            <groupId>com.example.demo</groupId>
            <artifactId>demo-dao</artifactId>
            <version>${demo.version}</version>
        </dependency>
        <dependency>
            <groupId>com.example.demo</groupId>
            <artifactId>demo-web</artifactId>
            <version>${demo.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```
> 注：${demo.version} 定義在 properties 標籤中

② 其次在 demo-biz 層中的 pom 檔中添加 demo-dao 及 demo-common 依賴
``` xml
<dependencies>
    <dependency>
        <groupId>com.example.demo</groupId>
        <artifactId>demo-common</artifactId>
    </dependency>
    <dependency>
        <groupId>com.example.demo</groupId>
        <artifactId>demo-dao</artifactId>
    </dependency>
</dependencies>
```
③ 之後在 demo-web 層中的 pom 檔中添加 demo-biz 依賴
``` xml
<dependencies>
    <dependency>
        <groupId>com.example.demo</groupId>
        <artifactId>demo-biz</artifactId>
    </dependency>
</dependencies>
```
#### 4.6 web 層調用 biz 層介面測試
模組依賴關係配置完成之後，通過 web 層 測試下 biz 層的介面是否可以正常調用。

① 首先在 demo-biz 層創建 com.example.demo.biz 包，添加 service 目錄並在其中創建 DemoService 介面類別及 impl 目錄（用於存放介面實現類）。
``` java
package com.example.demo.biz.service;

/**
 * @author linjian
 * @date 2019/1/15
 */
public interface DemoService {

    String test();
}
```

``` java
package com.example.demo.biz.service.impl;

import com.example.demo.biz.service.DemoService;
import org.springframework.stereotype.Service;

/**
 * @author linjian
 * @date 2019/1/15
 */
@Service
public class DemoServiceImpl implements DemoService {

    @Override
    public String test() {
        return "interface test";
    }
}
```
② DemoController 通過 @Autowired 注解注入  DemoService ，修改 DemoController 的 test 方法使之調用 DemoService 的 test 方法
``` java
@Autowired
private DemoService demoService;

@GetMapping("test")
public String test() {
    return demoService.test();
}
```
③ 再次運行 DemoWebApplication 類中的 main 方法啟動專案，發現如下報錯
``` bash
***************************
APPLICATION FAILED TO START
***************************

Description:

Field demoService in com.example.demo.web.controller.DemoController required a bean of type 'com.example.demo.biz.service.DemoService' that could not be found.

The injection point has the following annotations: - @org.springframework.beans.factory.annotation.Autowired(required=true)

Action:

Consider defining a bean of type 'com.example.demo.biz.service.DemoService' in your configuration.
```
`原因是找不到 DemoService 類`

④ 在 DemoWebApplication 入口類中增加包掃描，設置 @SpringBootApplication 注解中的 scanBasePackages 值為 com.example.demo
``` java
@SpringBootApplication(scanBasePackages = "com.example.demo")
```
⑤ 設置完後重新運行 main 方法，專案正常啟動，訪問  http://localhost:8080/demo/test 測試介面
![SpringBoot_1_11.png](http://blog.shoegaze.com/SpringBoot_1_11.png)
#### 4.7 集成 MyBatis
以上介面均是靜態的，不涉及資料庫操作，下面將集成 MyBatis 訪問資料庫中的資料。

① 首先父 pom 檔中聲明 mybatis-spring-boot-starter 及 lombok 依賴
``` xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.16.22</version>
</dependency>
```
② 其次在 demo-dao 層中的 pom 檔中添加上述依賴
``` xml
<dependencies>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>
```
③ 之後在 demo-dao 層創建 com.example.demo.dao 包，通過 mybatis-genertaor 工具生成 dao 層相關檔（ DO 、 Mapper 、 xml ），目錄結構如下
``` bash
|-- demo-dao
    |-- pom.xml
    |-- src
        |-- main
        |   |-- java
        |   |   |-- com
        |   |       |-- example
        |   |           |-- demo
        |   |               |-- dao
        |   |                   |-- entity
        |   |                   |   |-- UserDO.java
        |   |                   |-- mapper
        |   |                       |-- UserMapper.java
        |   |-- resources
        |       |-- mybatis
        |           |-- UserMapper.xml
        |-- test
            |-- java
```
④ 然後在 demo-web 層中的 resources 目錄 創建 applicatio.properties 檔並在其中添加 datasource 及 MyBatis 相關配置項
``` yml
spring.datasource.driverClassName = com.mysql.jdbc.Driver
spring.datasource.url = jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf-8
spring.datasource.username = test
spring.datasource.password = 123456

mybatis.mapper-locations = classpath:mybatis/*.xml
mybatis.type-aliases-package = com.example.demo.dao.entity
```
> 注：如果生成的 xml 在 dao 層 resources 目錄的子目錄中則 mybatis.mapper-locations 需設置為  classpath:mybatis/\*/\*.xml

⑤ DemoService 通過 @Autowired 注解注入 UserMapper ，修改 DemoService 的 test 方法使之調用 UserMapper 的 selectById 方法
``` java
@Autowired
private UserMapper userMapper;

@Override
public String test() {
    UserDO user = userMapper.selectById(1);
    return user.toString();
}
```
⑥ 再次運行 DemoWebApplication 類中的 main 方法啟動項目，出現如下報錯
``` bash
***************************
APPLICATION FAILED TO START
***************************

Description:

Field userMapper in com.example.demo.biz.service.impl.DemoServiceImpl required a bean of type 'com.example.demo.dao.mapper.business.UserMapper' that could not be found.

The injection point has the following annotations: - @org.springframework.beans.factory.annotation.Autowired(required=true)

Action:

Consider defining a bean of type 'com.example.demo.dao.mapper.business.UserMapper' in your configuration.
```
`原因是找不到 UserMapper 類`
⑦ 在 DemoWebApplication入口類中增加 dao 層包掃描，添加 @MapperScan 注解並設置其值為 com.example.demo.dao.mapper
``` bash
@MapperScan("com.example.demo.dao.mapper")
```
⑧ 設置完後重新運行 main 方法，專案正常啟動，訪問 http://localhost:8080/demo/test 測試介面
![SpringBoot_1_12.png](http://blog.shoegaze.com/SpringBoot_1_12.png)

***
### 五、外部 Tomcat 部署 war 包
外部 Tomcat 部署的話，就不能依賴於入口類的 main 函數了，而是要以類似於 web.xml 檔配置的方式來啟動 Spring應用上下文。
① 在入口類中繼承 SpringBootServletInitializer 並實現 configure 方法
``` java
public class DemoWebApplication extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(DemoWebApplication.class);
    }

    public static void main(String[] args) {
        SpringApplication.run(DemoWebApplication.class, args);
    }
}
```
② 之前在 demo-web 引入了 spring-boot-starter-web 的依賴，該依賴包包含內嵌的 Tomcat 容器，所以直接部署在外部 Tomcat 會衝突報錯。這裡在 demo-web 層中的 pom 文件中重定義 spring-boot-starter-tomcat 依賴包的「 scope 」即可解決該問題。
``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
```
③ 聲明 demo-web 層的打包方式及最終的包名
``` xml
<packaging>war</packaging>
...省略其餘部分...
<build>
    <finalName>demo</finalName>
</build>
```
④ 此時在 demo-web 層目錄執行「 mvn clean install 」即可打出一個名為 demo.war 的包。
### 六、Maven Profile 多環境打包
在日常開發中，通常不止一套環境，如開發環境、測試環境、預發環境、生成環境，而每個環境的配置項可能都不一樣，這就需要用到多環境打包來解決這個問題。

① 在 demo-web 層的 resources 目錄中新建 conf 目錄，再在其中按照環境創建相應目錄，這裡創建開發環境「 dev 」及測試環境「 test 」，再將原本的 application.properties 檔分別拷貝一份到兩個目錄中，根據環境修改其中的配置項，最後刪除原本的設定檔。得到目錄結構如下：
``` bash
|-- resources
    |-- conf
      |-- dev
      |   |-- application.properties
      |-- test
          |-- application.properties
```
② 往 demo-web 層的 pom 文件添加 profile 標籤
``` xml
<profiles>
    <profile>
        <id>dev</id>
        <properties>
            <profile.env>dev</profile.env>
        </properties>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>
    <profile>
        <id>test</id>
        <properties>
            <profile.env>test</profile.env>
        </properties>
    </profile>
</profiles>
```
> 注：其中 dev 為默認啟動的 profile ，如要增加其他環境按照上述步驟操作即可。

③ 設置打包時資源檔路徑
``` xml
<build>
    <finalName>demo</finalName>
    <resources>
        <resource>
            <directory>${basedir}/src/main/resources</directory>
            <excludes>
                <exclude>conf/**</exclude>
            </excludes>
        </resource>
        <resource>
            <directory>src/main/resources/conf/${profile.env}</directory>
        </resource>
    </resources>
</build>
```
> 注：${basedir} 為當前子模組的根目錄

④ 打包時通過「 P 」參數指定 profile
``` bash
mvn clean install -P test
```
***
### 七、自訂 archetype 範本
#### 7.1 什麼是 archetype 範本？
archetype 是一個 Maven 專案範本工具包，通過 archetype 我們可以快速搭建 Maven 項目。
![SpringBoot_1_13.png](http://blog.shoegaze.com/SpringBoot_1_13.png)
每個範本裡其實就是附帶不同的依賴和外掛程式。一般在公司私服裡都會有屬於本公司的一套 archetype 範本，裡面有著調試好的專案用到的依賴包和版本號。
#### 7.2 創建 archetype 範本
① cd 到專案根目錄（即父 pom 檔所在目錄）執行 mvn 命令，此時會在專案根目錄生成 target 目錄，其包含一個名為 generated-sources 的目錄
``` bash
mvn archetype:create-from-project
```
② 打開「 /target/generated-sources/archetype/src/main/resources/META-INF/maven/ 」目錄下的 archetype-metadata.xml 檔，從中清理一些不需要的檔，如 IDEA 的一些文件（.idea、.iml）等。
``` xml
<fileSet filtered="true" encoding="UTF-8">
    <directory>.idea/libraries</directory>
    <includes>
        <include>**/*.xml</include>
    </includes>
</fileSet>
<fileSet filtered="true" encoding="UTF-8">
    <directory>.idea/inspectionProfiles</directory>
    <includes>
        <include>**/*.xml</include>
    </includes>
</fileSet>
<fileSet filtered="true" encoding="UTF-8">
    <directory>.idea/artifacts</directory>
    <includes>
        <include>**/*.xml</include>
    </includes>
</fileSet>
<fileSet filtered="true" encoding="UTF-8">
    <directory>.idea</directory>
    <includes>
        <include>**/*.xml</include>
    </includes>
</fileSet>
```
③ 然後 cd target/generated-sources/archetype/，然後執行 install 命令，在本地倉庫的根目錄生成 archetype-catalog.xml 骨架設定檔
``` bash
mvn install
```
檔內容如下：
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<archetype-catalog xsi:schemaLocation="http://maven.apache.org/plugins/maven-archetype-plugin/archetype-catalog/1.0.0 http://maven.apache.org/xsd/archetype-catalog-1.0.0.xsd"
    xmlns="http://maven.apache.org/plugins/maven-archetype-plugin/archetype-catalog/1.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <archetypes>
        <archetype>
            <groupId>com.example.demo</groupId>
            <artifactId>demo-archetype</artifactId>
            <version>0.0.1-SNAPSHOT</version>
            <description>demo</description>
        </archetype>
    </archetypes>
</archetype-catalog>
```
#### 7.3 使用 archetype 範本
到本機的工作目錄執行 mvn archetype:generate -DarchetypeCatalog=local 從本地 archeType 範本中創建專案
``` bash
~/Workspace/JAVA $ mvn archetype:generate -DarchetypeCatalog=local
[INFO] Scanning for projects...
[INFO]
[INFO] Using the builder org.apache.maven.lifecycle.internal.builder.singlethreaded.SingleThreadedBuilder with a thread count of 1
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building Maven Stub Project (No POM) 1
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] >>> maven-archetype-plugin:3.0.1:generate (default-cli) @ standalone-pom >>>
[INFO]
[INFO] <<< maven-archetype-plugin:3.0.1:generate (default-cli) @ standalone-pom <<<
[INFO]
[INFO] --- maven-archetype-plugin:3.0.1:generate (default-cli) @ standalone-pom ---
[INFO] Generating project in Interactive mode
[INFO] No archetype defined. Using maven-archetype-quickstart (org.apache.maven.archetypes:maven-archetype-quickstart:1.0)
Choose archetype:
1: local -> com.example.demo:demo-archetype (demo)
Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains): : 1
Define value for property 'groupId': com.orz.test
Define value for property 'artifactId': test
Define value for property 'version' 1.0-SNAPSHOT: :
Define value for property 'package' com.orz.test: :
Confirm properties configuration:
groupId: com.orz.test
artifactId: test
version: 1.0-SNAPSHOT
package: com.orz.test
 Y: : y
[INFO] ----------------------------------------------------------------------------
[INFO] Using following parameters for creating project from Archetype: demo-archetype:0.0.1-SNAPSHOT
[INFO] ----------------------------------------------------------------------------
[INFO] Parameter: groupId, Value: com.orz.test
[INFO] Parameter: artifactId, Value: test
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Parameter: package, Value: com.orz.test
[INFO] Parameter: packageInPathFormat, Value: com/orz/test
[INFO] Parameter: package, Value: com.orz.test
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Parameter: groupId, Value: com.orz.test
[INFO] Parameter: artifactId, Value: test
[INFO] Parent element not overwritten in /Users/linjian/Workspace/JAVA/test/test-biz/pom.xml
[INFO] Parent element not overwritten in /Users/linjian/Workspace/JAVA/test/test-common/pom.xml
[INFO] Parent element not overwritten in /Users/linjian/Workspace/JAVA/test/test-dao/pom.xml
[INFO] Parent element not overwritten in /Users/linjian/Workspace/JAVA/test/test-web/pom.xml
[INFO] Project created from Archetype in dir: /Users/linjian/Workspace/JAVA/test
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 01:01 min
[INFO] Finished at: 2019-01-15T18:51:31+08:00
[INFO] Final Memory: 14M/155M
[INFO] ------------------------------------------------------------------------
```
上面羅列出了所有可用的範本，首先選擇使用哪個範本，這裡選擇 1 ，其次輸入「 groupId 」、「 articleId 」、「 version 」及「 package 」，然後輸入「 Y 」確認創建，最終專案創建成功。

### 八、結語
至此 Spring Boot Maven 多模組專案的搭建過程已經介紹完畢，後續會在此基礎上繼續集成一些中介軟體。
> 源碼：[https://github.com/SymonLin/demo](https://github.com/SymonLin/demo)
