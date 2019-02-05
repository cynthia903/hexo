---
title: Maven 插件总结
date: 2018-06-09 22:23:03
updated:
tags: Java
---

Maven 本质上是一个插件框架，它的核心并不执行任何具体的构建任务，所有这些任务都交给插件来完成，例如编译源代码是由 `maven-compiler-plugin` 完成的。每个插件会有一个或者多个目标（goal），例如 `maven-compiler-plugin` 插件的 `compile` 目标用来编译位于 `src/main/java/` 目录下的主源码，`testCompile` 目标用来编译位于 `src/test/java/` 目录下的测试源码。

用户可以通过两种方式调用 Maven 插件目标：

1. 将插件目标与生命周期阶段（lifecycle phase）绑定，这样用户在命令行只是输入生命周期阶段而已，例如 Maven 默认将 `maven-compiler-plugin` 的 `compile` 目标与 `compile` 生命周期阶段绑定，因此命令 `mvn compile` 实际上是先定位到 `compile` 这一生命周期阶段，然后再根据绑定关系调用 `maven-compiler-plugin` 的 `compile` 目标。
2. 直接在命令行指定要执行的插件目标（goal），例如 `mvn archetype:generate` 就表示调用 `maven-archetype-plugin` 的 `generate` 目标，**这种带冒号的调用方式与生命周期无关**。

常用插件整理如下：

![Maven 常用插件](/img/java/maven/plugins.png)

# 核心插件

# 打包工具

# 其它工具

## maven-archetype-plugin

用于生成骨架，详见：[Maven 骨架快速搭建项目](/2018/12/08/maven-archetype/)。

## maven-dependency-plugin

用于分析项目依赖，例如通过 `mvn dependency:tree` 命令分析 Dubbo 默认依赖的第三方库：

```
[INFO] +- com.alibaba:dubbo:jar:2.5.9-SNAPSHOT:compile
[INFO] |  +- org.springframework:spring-context:jar:4.3.10.RELEASE:compile
[INFO] |  +- org.javassist:javassist:jar:3.21.0-GA:compile
[INFO] |  \- org.jboss.netty:netty:jar:3.2.5.Final:compile
```

# IDEA Maven 插件

最后来看下 IDEA Maven 插件提供的 Maven Projects tool window 功能：

![IDEA Maven Projects](/img/java/idea/maven_projects.png)

# 参考

http://maven.apache.org/plugins/index.html