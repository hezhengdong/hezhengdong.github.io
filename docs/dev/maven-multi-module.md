# Maven 多模块与依赖管理

> 一项侧重于实践的技术，如果自己隔段时间不使用，相关知识总是淡忘的很快。开发本项目的过程中，通过向 AI 不断提问，又复习了一遍 Maven 高级功能。为了防止未来再次遗忘，需要沉淀一篇文章。自己撰写文章需要花时间，而且不一定能写好。AI 拥有世界知识，Coding Agent 拥有所有上下文信息，于是自己让 AI 基于聊天记录，结合本项目，结合技术发展脉络，沉淀了本篇文章，供未来遗忘后回看。

## 一、没有 Maven 的时候——Jar Hell

Java 项目最早用 Ant。Ant 只管"编译 → 打包"，不管理依赖。开发者需要手动下载 jar 放到 `lib/` 目录，版本全凭人脑记。

后来 Maven 出现了，解决的第一个问题就是**依赖管理**：声明 `groupId:artifactId:version`，Maven 自动从仓库拉取。

但新问题随之而来：**一个项目有多个子模块时，每个模块各自声明依赖，版本号散落各处。** 改一个版本要改 N 个文件，漏改一处就跑出不一致。

于是诞生了两个机制：

---

## 二、继承（parent）——"版本号只写一次"

### 问题

```
pushhub-handler/pom.xml:    hutool 5.8.44
pushhub-ingress/pom.xml:    hutool 5.8.44
pushhub-something/pom.xml:  hutool 5.8.43   ← 漏改了，版本不一致
```

### 解决方案：parent 继承

子模块的 `<parent>` 指向同一个父 POM，父 POM 用 `<dependencyManagement>` 统一管理版本号。

```
spring-boot-starter-parent (Spring 官方提供的父 POM)
        ↑
    pushhub (根 POM，自己写的父 POM)
   ╱       ↑        ╲
handler   common   ingress
```

根 pom.xml 中的 `<dependencyManagement>`：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.8.44</version>   <!-- 版本号只在这里出现一次 -->
        </dependency>
    </dependencies>
</dependencyManagement>
```

子模块中声明依赖，**不写版本号**：

```xml
<dependencies>
    <dependency>
        <groupId>cn.hutool</groupId>
        <artifactId>hutool-all</artifactId>
        <!-- 没有 <version>，从父 POM 的 dependencyManagement 继承 -->
    </dependency>
</dependencies>
```

### 关键认知：dependencyManagement ≠ dependencies

| | dependencyManagement | dependencies |
|------|-----------|-------------|
| 作用 | 定义"版本目录" | 实际引入依赖 |
| 是否加到 classpath | 否 | 是 |
| 子模块可否不写 | —— | 可以，不写的从父 POM 取版本 |

打个比方：`dependencyManagement` 是图书馆的图书目录（告诉你有哪些书、什么版本），子模块的 `dependencies` 是你实际借走的书。目录本身不是你借的书。

### parent 只能有一个

Maven 规定一个模块只能有一个 `<parent>`。那如果既想要 Spring Boot 的版本管理，又想要自己项目的版本管理，怎么办？

继承是**传递的**：

```
spring-boot-starter-parent
        ↑
    pushhub（根）
        ↑
    pushhub-handler
```

pushhub-handler 的 parent 是 pushhub，pushhub 的 parent 是 spring-boot-starter-parent。handler 能同时拿到两边的版本信息——自己的爹给的，以及爷爷给的。

---

## 三、聚合（modules）——"一行命令构建全部"

### 问题

有了继承以后，三个模块要分别构建：

```bash
cd pushhub-common && mvn install
cd ../pushhub-handler && mvn install   # 依赖 common，必须先构建 common
cd ../pushhub-ingress && mvn install
```

顺序不能错，忘了 common 先构建就会报错。

### 解决方案：modules 聚合

父 POM 中声明：

```xml
<packaging>pom</packaging>
<modules>
    <module>pushhub-common</module>
    <module>pushhub-handler</module>
    <module>pushhub-ingress</module>
</modules>
```

然后在根目录一行命令：

```bash
mvn install
```

Maven 的 **reactor** 会自动按依赖关系排序构建。`pushhub-handler` 依赖 `pushhub-common`，reactor 保证 common 先编译。

### packaging=pom 是什么意思？

普通模块的 packaging 默认是 `jar`（产出 jar 包）。父 POM 的 packaging 是 `pom`——**不产出任何工件，只是一个管理节点**。`spring-boot-starter-parent` 的 packaging 也是 `pom`，和你项目的根 POM 是同一种东西。

---

## 四、Spring Boot 的父 POM——官方替你管好了

### 问题

Spring Boot 项目依赖几十个库：Jackson、Tomcat、Kafka Client、Hibernate Validator……如果让开发者自己管理版本号，不仅麻烦，还容易冲突——Jackson 的版本要和 Spring 匹配，Kafka Client 的版本要和 Broker 兼容。

### 解决方案：spring-boot-starter-parent

你在根 POM 里写了：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>4.0.6</version>
</parent>
```

这个 POM 文件塞了约 200 行的 `<dependencyManagement>`，定了所有 Spring Boot 官方集成的版本号。它还包含：

- 所有 Spring Boot starter 的版本号
- 所有 Spring Boot 依赖的第三方库版本号（Jackson、Tomcat、Kafka 等）
- Maven 插件的版本和配置（`spring-boot-maven-plugin` 等）
- 通用编译参数（Java 版本、编码等）

这本身就是 Spring Boot 依赖管理机制的一部分——**不是额外的东西，而是 Spring Boot 的基石**。

### 所以你在子模块里写依赖几乎不用管版本号

```xml
<!-- handler/pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-kafka</artifactId>
    <!-- 版本号？spring-boot-starter-parent 已经定好了，不用写 -->
</dependency>
```

这就是 Spring Boot "开箱即用"的一个侧面——不只是 starter 打包了依赖，更是父 POM 替你管理了版本兼容性。

### 加新依赖时的判断流程

1. **这个库被 Spring Boot 管理了吗？** → 查 [Spring Boot 官方依赖版本表](https://docs.spring.io/spring-boot/appendix/dependency-versions/index.html)，如果在列，直接在子模块声明，不写版本。
2. **不在 Spring Boot 管理范围内？** → 先在根 pom.xml 的 `<dependencyManagement>` 里登记版本号，再在需要的子模块声明依赖（不写版本）。
3. **只在单个子模块用、未来也不打算共享？** → 直接在子模块写完整 GAV，没有必要强行上浮到父 POM。

---

## 五、总结

Maven 多模块体系解决的是两个独立的问题：

| 机制 | 写法 | 解决的问题 |
|------|------|-----------|
| 继承 | `<parent>` | 版本号、配置只写一次，子模块自动继承 |
| 聚合 | `<modules>` | 根目录一行命令按顺序构建全部模块 |
| 版本目录 | `<dependencyManagement>` | 定义版本号，不实际引入，子模块按需取用 |

Spring Boot 在此基础上提供了一层标准化的父 POM，让开发者不需要操心几十个库的版本兼容——这是 Spring Boot "约定优于配置"哲学在构建层面的体现。
