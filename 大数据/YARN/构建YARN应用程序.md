# 构建YARN应用程序

参考：https://github.com/hortonworks/simple-yarn-app 来构建一个 YARN 应用。

首先创建一个 Maven 程序。

在 pom.xml 中加入以下依赖：

```xml
   <dependencies>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-yarn-client</artifactId>
            <version>3.2.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>3.2.1</version>
        </dependency>
    </dependencies>
```

