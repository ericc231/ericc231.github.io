---
title: "將Spring Boot打包成Docker Image"
date: 2020-03-18 00:00:09 +0800
categories: ["軟體開發", "IT生涯"]
tags: ["camel", "Docker", "Docker Image", "featured", "Spring Boot"]
---

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-15-e4b88be58d8811.08.19.png?w=1024)

要將Sprint Boot打包後的軟體部署到Kubernates裡面，首先得先將它們打包成Docker Image．

以下範例為打包一個Sprint Boot，提供用camel建立的rest api，首先使用Maven archetype快速建立一個Spring Boot的Maven專案

mvn archetype:generate \  
-DarchetypeGroupId=org.apache.camel.archetypes \  
-DarchetypeArtifactId=camel-archetype-spring-boot \  
-DarchetypeVersion=3.1.0

輸入groupId等屬性

Define value for property 'groupId': com.wordpress.ericchen231  
Define value for property 'artifactId': spring-boot-demo1  
Define value for property 'version' 1.0-SNAPSHOT: : 1.0.0  
Define value for property 'package' com.wordpress.ericchen231: : com.wordpress.ericchen231.spring\_boot\_demo1  
[INFO] Using property: spring-boot-version = 1.4.3.RELEASE  
Confirm properties configuration:  
groupId: com.wordpress.ericchen231  
artifactId: spring-boot-demo1  
version: 1.0.0  
package: com.wordpress.ericchen231.spring\_boot\_demo1  
spring-boot-version: 1.4.3.RELEASE  
Y: : Y

這時會產生spring-boot-demo1資料夾，會產生以下的檔案

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-16-e4b88be58d8810.10.21-1.png?w=784)

test資料夾此時沒有要處理它，就不列出來了．

修改MySpringBootRouter.java，修改方式如下

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-16-e4b88be58d8810.13.30.png?w=1024)

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-17-e4b88ae58d8812.34.39.png?w=944)

接著調整pom.xml，指定Java版本

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-16-e4b88be58d8810.23.38.png?w=1024)

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-16-e4b88be58d8810.23.50.png?w=1024)

增加maven.compiler.target及maven.compiler.source屬性

因為會使用Servlet元件來提供Rest API功能，須增加camel-rest與camel-servlet的依賴

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-17-e4b88ae58d8812.36.45.png?w=952)

調整src/main/resources/application.properties，增加camel.component.servlet.mapping.contextPath=/api/\*，Spring Boot預設啟動的port是8080，可以透過server.port來改變

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-17-e4b88ae58d8812.39.10.png?w=1024)

再來進行打包，透過以下指令

mvn package

啟動Spring Boot測試是否正常

mvn spring-boot:run

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-17-e4b88ae58d8812.46.53.png?w=1024)

成功啟動的樣子

測試是否可運作，用瀏覽器打開http://localhost:8888/api/say/hello，會顯示錯誤，檢視原始碼就可以看到「Hello World」，測試完畢可以按ctrl + c結束Spring Boot．

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-17-e4b88ae58d8812.51.05-1.png?w=1024)

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-17-e4b88ae58d8812.51.16.png?w=920)

檢視原始碼就可以看到Hello World的文字

要打包Docker Image要先準備一個Dockerfile，用vim編輯

vim Dockerfile

輸入以下的內容

FROM openjdk:8-jdk-alpine  
RUN addgroup -S spring && adduser -S spring -G spring  
USER spring:spring  
ARG JAR\_FILE=target/\*.jar  
COPY ${JAR\_FILE} app.jar  
ENTRYPOINT ["java","-jar","/app.jar"]

接著就可以開始產生Docker Image，可以使用Docker指令來打包

docker build -t spring-boot-demo1 .

啟動Docker載入剛剛建立的image

docker run -p 8888:8888 -t spring-boot-demo1

啟動後，一樣用瀏覽器開啟http://localhost:8888/api/say/hello，應該會出現相同的測試結果．

[GitHub 原始碼](https://github.com/ericc231/spring-boot-demo1/ "https://github.com/ericc231/spring-boot-demo1/")
