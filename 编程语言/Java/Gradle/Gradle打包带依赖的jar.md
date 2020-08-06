# Gradle 打包带依赖的 Jar

build.gradle 如下：

```groovy
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.github.jengelman.gradle.plugins:shadow:5.2.0'
    }
}

apply plugin: 'com.github.johnrengelman.shadow'
apply plugin: 'java'

group 'org.example'
version '1.0-SNAPSHOT'

sourceCompatibility = "1.8"
targetCompatibility = "1.8"

repositories {
    mavenCentral()
}

dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
    implementation 'org.apache.spark:spark-core_2.12:2.4.5'
}

jar {
    manifest {
        attributes 'Implementation-Title': 'Application',
        'Main-Class': 'com.bbd.spark.Main'
    }
}

shadowJar {}
```

运行 `gradle shadowJar` 会在 build/libs 目录看到有一个带 -all 后缀的文件。