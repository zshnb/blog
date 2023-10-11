---
title: Protobuf结合Gradle最佳实践
date: 2022-01-22 20:30
tags:
- Protobuf
- Gradle
---

# 背景
最近打算实践一下Spring Cloud微服务，完整做一个系统。此次打算全部服务采用Gradle构建，微服务之间通信协议采用Protobuf，因此在系统架构上有2种方案
<!--more-->
1. 微服务自己定义message，需要对外提供api的进行单独打包并发布
2. 所有message定义在独立的项目中打包并发布，所有微服务引用该jar包

# 实现
经过一番考虑后选择了第2种，使用这种方案在每个微服务中可以不需要单独定义对外发布模块，比较省事。
首先使用IDEA新建1个Gradle项目，然后编辑build.gradle
```groovy
// 定义protobuf插件
plugins {
    id 'java-library'
    id "com.google.protobuf" version "0.8.18"
    id 'maven-publish'
}
apply plugin: 'java'
apply plugin: 'com.google.protobuf'

group 'com.zshnb.mall'
version '0.0.1-SNAPSHOT'

repositories {
    mavenCentral()
}

compileJava {
    sourceCompatibility(JavaVersion.VERSION_11.toString())
    targetCompatibility(JavaVersion.VERSION_11.toString())
}

dependencies {
    implementation 'com.google.protobuf:protobuf-java:3.19.3'
    implementation 'com.google.protobuf:protobuf-gradle-plugin:0.8.18'
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.8.2'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.8.2'
}

sourceSets {
    main {
        proto {
        }
    }
}
protobuf {
    protoc {
        // 使用官方的protoc编译器
        artifact = 'com.google.protobuf:protoc:3.0.0'
    }
}

publishing {
    publications {
        mavenJava(MavenPublication) {
            from components.java
//            artifact file("build/libs/protoss-0.0.1-SNAPSHOT.jar") // 最初尝试的方法，但是发布的jar包无法使用
        }
    }
    repositories {
        maven {
            def releasesRepoUrl = 'http://localhost:8000/repository/maven-releases/'
            def snapshotsRepoUrl = 'http://localhost:8000/repository/maven-snapshots/'
            url = version.endsWith('SNAPSHOT') ? snapshotsRepoUrl : releasesRepoUrl
            name 'nexus'
            url url
            allowInsecureProtocol true
            credentials {
                username 'admin'
                password 'admin'
            }
        }
    }
}

test {
    useJUnitPlatform()
}
```

通过以上设置，运行`./gradle clean build publish`后会自动把proto编译为class文件，构建成jar包后发布到自己的nexus私服中。经过此次项目搭建，发现还是官方文档靠谱
网上的文档要么版本太旧，要么有错误。

最后，附上此次搭建过程中参考文档
- [gradle maven publish plugin](https://docs.gradle.org/current/userguide/publishing_maven.html)
- [gradle protobuf plugin](https://github.com/google/protobuf-gradle-plugin)
- [windows安装nexus](https://juejin.cn/post/6844903991600480269#heading-13)