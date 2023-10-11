---
title: Gradle7多模块项目配置
date: 2022-03-24 13:50
tags:
- Java
- Gradle
---

# 背景
最近自己的项目全部使用Gradle构建，但是在使用的过程中踩了不少坑，因此打算把遇到的坑全部记录下来，方便自己回顾的同时，也能帮助其他人。

# 实现
这篇文章主要记录如何使用Gradle配置多模块项目。因为自己的项目采用微服务架构，因此每个服务除了本身业务逻辑模块，还会有对外提供的api模块，于是我把它们拆成了2个模块
api和service。
<!--more-->
在根目录下的build.gradle的内容如下
```groovy
plugins {
	id 'org.springframework.boot' version '2.6.3'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
}

// 应用所有子module
subprojects {
	apply plugin: 'java'
	apply plugin: 'io.spring.dependency-management'
	apply plugin: 'org.springframework.boot'

	jar {
		enabled = true
		archiveClassifier.set('')
	}

	ext {
		set('springCloudVersion', "2021.0.0")
	}

	compileJava {
		sourceCompatibility(JavaVersion.VERSION_11.toString())
		targetCompatibility(JavaVersion.VERSION_11.toString())
	}

	dependencies {
		implementation ''
	}

	dependencyManagement {
		imports {
			mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
		}
	}
}

// 应用到所有module，包括根module
allprojects {
	group = ''
	version = '0.0.1-SNAPSHOT'

	repositories {
		maven {
			allowInsecureProtocol true
			url 'http://xxx:8000/repository/maven-releases/'
		}
		maven {
			allowInsecureProtocol true
			url 'http://xxx:8000/repository/maven-snapshots/'
		}
		mavenCentral()
	}
}
```
api module的build.gradle如下
```groovy
// 因为需要对外发布jar包，所以需要这2个plugin
plugins {
    id 'java-library'
    id 'maven-publish'
}

// api jar包没有main类
bootJar {
    enabled = false
}
// 具体发布nexus配置
publishing {
    publications {
        mavenJava(MavenPublication) {
            from components.java
        }
    }
    repositories {
        maven {
            def releasesRepoUrl = 'http://xxx:8000/repository/maven-releases/'
            def snapshotsRepoUrl = 'http://xxx:8000/repository/maven-snapshots/'
            url = version.endsWith('SNAPSHOT') ? snapshotsRepoUrl : releasesRepoUrl
            name 'nexus'
            url url
            allowInsecureProtocol true
            credentials {
                username ''
                password ''
            }
        }
    }
}

```
service module的build.gradle如下
```groovy
// 比较简单，都是定义了一些依赖
dependencies {
    implementation 'cn.dev33:sa-token-dao-redis-jackson:1.29.0'
    implementation 'cn.dev33:sa-token-spring-boot-starter:1.29.0'
    implementation 'org.apache.commons:commons-pool2'
    runtimeOnly 'mysql:mysql-connector-java'
    annotationProcessor 'org.mapstruct:mapstruct-processor:1.4.2.Final'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testAnnotationProcessor 'org.mapstruct:mapstruct-processor:1.4.2.Final'
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.8.2'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.8.2'
}

test {
    useJUnitPlatform()
}

repositories {
    mavenCentral()
}
```
