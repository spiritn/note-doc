# 1. Spring boot介绍

Spring Boot帮助您创建可以立即运行，独立的、基于Spring的生产级应用程序。我们对Spring平台和第三方库有自己的看法，这样您就可以以最少的麻烦上手。大多数Spring Boot应用程序只需要很少的Spring配置。

您可以使用Spring Boot来创建Java应用程序，可以通过使用`java -jar`或更传统的war包来启动。我们还提供了一个运行 "spring脚本 "的命令行工具。

我们的主要目标是

- 为所有的Spring开发提供更快、更广泛的入门体验。

- 开箱即用，但当需求与默认值相左时，我们会迅速退出。

- 提供一系列大类项目常见的非功能特性（如嵌入式服务器、安全、度量、健康检查和外部化配置）。

- 完全不需要生成代码，不需要XML配置。

`spring-boot-starter-parent`是一个特殊的starter，提供了有用的maven默认配置，也提供了一些常见依赖的版本管理，使得你可以在添加一些依赖时省略版本声明。


Auto-configuration是为了与 "Starters "很好地配合，但这两个概念并不直接挂钩。你可以自由选择 starters 以外的 jar 依赖。Spring Boot仍然会尽力自动配置你的应用程序。

spring-boot-maven-plugin插件可以帮助我们重新打包jar，打包为可执行jar又称为fat jars，它包含了你的代码和代码运行所需的其他依赖jar。

# 2. 系统要求

Maven    3.3+

Gradle     6

Tomcat9.0    4.0 

# 3. springboot介绍
Spring Boot使您可以轻松创建独立的、基于Spring的生产级应用程序，可以 "直接运行"。

我们从Spring平台和第三方库的角度出发，让您以最小的代价上手。大多数Spring Boot应用程序只需要最少的Spring配置。

## Features：

- 提供可选的starter依赖，简化项目构建
- 提供Spring和第三方库的默认自动配置，简化spring应用的开发
- 内置tomcat，jetty等启动容器，jar包即可启动
- 提供生产常用的特性，像metrics, health checks
- 不需要代码生成和xml配置



文档地址：[https://docs.spring.io/spring-boot/docs/current/reference/html/index.html](https://docs.spring.io/spring-boot/docs/current/reference/html/index.html)

![overview](images/overview.png)

