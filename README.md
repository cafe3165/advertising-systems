# advertising-systems
## 第2章 广告系统概览与准备工作

#### 广告系统介绍

1. 广告投放系统

   既然是广告系统，一定得有广告数据，数据当然是由广告主或代理商投放，那么，也就需要有个投放广告的平台，这就是广告投放系统

2. 广告检索系统

   媒体方对广告系统发起请求，广告系统能够检索符合要求的广告数据，这就是广告检索系统的核心功能

#### 完整的广告系统又需要包含哪些子系统呢？

1. 曝光监测系统

   监测广告数据的曝光记录

2. 报表系统

   构建广告数据报表，比如广告A在地域B中一共曝光了多少次，主要时OLAP的过程

3. 扣费系统

   广告的每一次曝光都是需要扣费的，且这个系统里面负责了将广告数据置位的功能

#### 思考

如果让你做一个广告系统，你会考虑到哪些点（技术选型、架构、实现难点、功能接口等等）？

## 第3章 广告系统骨架开发

#### Maven 基础知识

1. Maven常用命令

   ```
   mvn -v 
   mvn compile
   mvn test 
   mvn package
   mvn clean 
   mvn install
   ```

2. 

#### Maven 相关特性

1. 传递依赖和排除依赖

2. 依赖冲突

   1. 短路优先
   2. 声明优先

3. 多模块项目/聚合

   1. 父模块pom文件的配置：packaging类型必须是pom

      ```xml
          <groupId>com.imooc.ad</groupId>
          <artifactId>imooc-ad</artifactId>
      	<packaging>pom</packaging>
          <version>1.0-SNAPSHOT</version>
      ```

   2. 聚合子模块：使用modules标签

      ```xml
          <modules>
              <module>ad-eureka</module>
              <module>ad-gateway</module>
              <module>imooc-ad-service</module>
          </modules>
      ```

   3. 父模块统一管理依赖包：使用dependencyManagement标签

      ```xml
          <dependencyManagement>
              <dependencies>
                  <dependency>
                      <groupId>org.springframework.cloud</groupId>
                      <artifactId>spring-cloud-dependencies</artifactId>
                      <version>${spring-cloud.version}</version>
                      <type>pom</type>
                      <scope>import</scope>
                  </dependency>
              </dependencies>
          </dependencyManagement>
      ```

   4. 子模块需要在pom中声明父模块：使用parent标签

      ```xml
          <parent>
              <artifactId>imooc-ad</artifactId>
              <groupId>com.imooc.ad</groupId>
              <version>1.0-SNAPSHOT</version>
          </parent>
      ```



#### 多节点Eureka Server 的部署

1. hosts文件

   ```
   127.0.0.1 server1
   127.0.0.1 server2
   127.0.0.1 server3
   ```

2. 使用不同的配置文件打包

   ```shell
   mvn clean package -Dmaven.test.skip=true -U
   cd target 
   java -jar ad-eureka-1.0-SNAPSHOT.jar --spring.profiles.active=server1
   java -jar ad-eureka-1.0-SNAPSHOT.jar --spring.profiles.active=server2
   java -jar ad-eureka-1.0-SNAPSHOT.jar --spring.profiles.active=server3
   ```


#### Eureka Server的高可用

1. 思考
   1. Eureka Server维护了系统中的服务的元信息，这些元信息包含什么你知道吗？
   2. 元信息又是怎么存储的呢

#### 微服务架构及网关组件介绍

1. 微服务架构及其应用场景
   1. 微服务架构的两种方式
      1. 点对点的方式：服务之间直接调用，每个微服务都开放Rest API，并调用其他微服务的接口
      2. API网关方式：业务接口通过API网关暴露，是所有客户端接口的唯一入口。微服务之间的通信也通过API网关
2. Zuul的介绍
   1. 思考：微服务系统中往往包含多个功能不通过的子系统或微服务，那么，外部应用怎样去访问各种各样的微服务呢？这也是Zuul所要解决的一个主要问题
   2. 在微服务架构中，后端服务往往不直接开放给调用端，而是通过一个服务网关根据请求的url，路由到响应的服务，即实现请求转发
   3. Zuul提供了服务网关的功能，可以实现负载均衡、反向代理、动态路由、请求转发等功能。Zuul大部分功能都是通过过滤器实现的，Zuul中定义了四种标准的过滤器类型，同时，还支持自定义过滤器
      1. pre：在请求被路由之前调用
      2. route：在路由请求时被调用
      3. post：在route和error过滤器之后被调用
      4. error：处理请求时发生错误时被调用
3. 问题
   1. Zuul的功能大部分都是由过滤器实现的，你还可以定义怎样的过滤器实现你想要的功能呢？
   2. 如果要给我们的系统接入用户模块（用户和权限），放在网关里面做合适吗?

## 第4章 微服务通用模块开发

#### 通用模块设计的思想与实现的功能

1. 设计思想
   1. 通用的代码，配置不应该散落在各个业务模块中，不利于维护与更新
   2. 一个大的系统，响应对象需要统一外层格式
   3. 各种业务设计与实现，可能会抛出各种各样的异常，异常信息的收集也应该做到统一
2. 实现的功能
   1. 统一响应处理
   2. 统一异常处理
      1. 不直接展示错误，对用户友好
      2. 异常分类，便于排查问题，Debug
      3. 降低业务代码中对异常处理的耦合
   3. 统一配置
3. 问题
   1. 为什么响应对象需要统一格式呢？
   2. 除了通用的AdException，你可能还会设计哪些自定义的异常类呢？这样设计的理由是什么呢？