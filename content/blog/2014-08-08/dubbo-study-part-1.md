+++
title = 'Dubbo分布式服务框架'
subtitle = 'Dubbo distribution service framework'
brief = ''
date = 2014-08-08
categories = ['Java', 'Dubbo', 'Engineering']
series = []
tags = ['Dubbo', 'Java', 'RMI']
type = 'blog'
+++

## 1. 分布式远程方法调用(RMI)介绍

### 1.1. 技术背景

随着这个计算机在各个领域中的应用的不断深入，在用户纬度、数据量纬度、描述深度、广度方面均越来越大，因而从整体计算量上来讲，存在单一主机，垂直结构无法满足大量计算的问题，
且问题级别逐渐升级，亟待解决。分布式计算技术将逐渐膨胀升级的问题用分化的方式进行处理，因此在各类大型/大规模项目中分布式计算有着较为广泛的应用。Dubbo就是这个领域中新兴的一种实用技术。

- - -

### 1.2. 远程方法调用(RMI/RPC)

以下是常见的几种RMI技术

| RMI/RPC service    |
| :------------------ |
| SOAP web service |
| REST service |
| CORBA(Common Object Request Broker Architecture) |
| Any database |

- - -

### 1.3. 分布式计算技术

以下是与分布式计算相关的几种概念

| Around distribution calc |
|--------------------------|
| 网格计算 Grid computing |
| 负载均衡 Load blance |
| 失效转移 Failover |
| 数据库切分 |

- - -

## 2. 远程方法调用(RMI)与I/O的关系

### 2.1. 远程方法调用(RMI)通用模型

![](../images/1.jpg)

* 服务器在哪？ 如何找到服务器？
* 请求和回复中包含哪些内容？ 

- - -

### 2.2. 网络I/O与本地I/O的区别

#### 话题讨论

- - -

## 3. Dubbo服务开发框架

### 3.1. Dubbo服务的结构

![](../images/dubbo-architecture.jpg)

- - -

### 3.2. 如何使用Dubbo服务

#### 概念简述

本地一般性的Spring服务声明如下：

##### [声明定义]

###### Std loacal:

```xml
<bean id=“xxxService” class=“com.xxx.XxxServiceImpl” />

<bean id=“xxxAction” class=“com.xxx.XxxAction”>
    <property name=“xxxService” ref=“xxxService” />
</bean>
```

- - -

Dubbo服务在Spring环境中的声明：

###### Provider:

```xml
<bean id=“xxxService” class=“com.xxx.XxxServiceImpl” /> <!-- 和本地服务一样实现远程服务 -->

<dubbo:service interface=“com.xxx.XxxService” ref=“xxxService” /> <!-- 增加暴露远程服务配置 -->
```

- - -

##### [引用调用]

###### Consumer:

```xml
<dubbo:reference id=“xxxService” interface=“com.xxx.XxxService” /> <!-- 增加引用远程服务配置 -->

<bean id=“xxxAction” class=“com.xxx.XxxAction”> <!-- 和本地服务一样使用远程服务 -->
    <property name=“xxxService” ref=“xxxService” />
</bean>
```

- - -

#### 在工程框架中使用（该工程骨架还有待完善）

该工程框架基于Apache manve建立，通过maven命令可自动生成可执行程序包，配合CI系统可实现自动部署（VPN环境有限范围支持）。

- - -

本章描述基于以下条件:
* Apache maven环境已经安装，并能够正常使用。
* Eclipse中的Apache maven插件于shell中的Apache maven使用同一份可执行程序、相同的仓库配置。



##### [安装工程骨架]

###### 从github.com/sarins下载示例工程，[工程连接](https://github.com/sarins/distributed_service)。

```bash
# goto you work folder.
git clone https://github.com/sarins/distributed_service.git distributed_service
```

###### 根据以下命令，将工程骨架安装至Apache maven的本地仓库中，已备后续步骤使用。（基于默认的Apache maven设置）

```bash
cd distributed_service

mvn archetype:create-from-project -Darchetype.filteredExtensions=java

[INFO] Scanning for projects...
[WARNING]
[WARNING] Some problems were encountered while building the effective model for com.huatek.unicorn:vimgr-biz:jar:1.0
[WARNING] 'build.plugins.plugin.version' for org.apache.maven.plugins:maven-javadoc-plugin is missing. @ com.huatek.unicorn:vimgr:1.0, /source/git_repo/java_fm/distributed_service/pom.xml, line 308, column 12
[WARNING]
[WARNING] Some problems were encountered while building the effective model for com.huatek.unicorn:vimgr-service-client:jar:1.0
[WARNING] 'build.plugins.plugin.version' for org.apache.maven.plugins:maven-javadoc-plugin is missing. @ com.huatek.unicorn:vimgr:1.0, /source/git_repo/java_fm/distributed_service/pom.xml, line 308, column 12
[WARNING]
[WARNING] Some problems were encountered while building the effective model for com.huatek.unicorn:vimgr-service-impl:jar:1.0
[WARNING] 'build.plugins.plugin.version' for org.apache.maven.plugins:maven-javadoc-plugin is missing. @ com.huatek.unicorn:vimgr:1.0, /source/git_repo/java_fm/distributed_service/pom.xml, line 308, column 12
[WARNING]
[WARNING] Some problems were encountered while building the effective model for com.huatek.unicorn:vimgr:pom:1.0
[WARNING] 'build.plugins.plugin.version' for org.apache.maven.plugins:maven-javadoc-plugin is missing. @ line 308, column 12
[WARNING]
[WARNING] It is highly recommended to fix these problems because they threaten the stability of your build.
[WARNING]
[WARNING] For this reason, future Maven versions might no longer support building such malformed projects.
[WARNING]
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Build Order:
[INFO]
[INFO] vimgr
[INFO] vimgr-biz
[INFO] vimgr-service-client
[INFO] vimgr-service-impl
Downloading: http://192.168.129.188:19999/nexus/content/groups/public/org/apache/maven/plugins/maven-javadoc-plugin/maven-metadata.xml
[WARNING] Could not transfer metadata org.apache.maven.plugins:maven-javadoc-plugin/maven-metadata.xml from/to nexus (http://192.168.129.188:19999/nexus/content/groups/public): Not authorized , ReasonPhrase:Unauthorized.
Downloading: http://192.168.129.188:19999/nexus/content/groups/public/org/apache/maven/plugins/maven-metadata.xml
Downloading: http://192.168.129.188:19999/nexus/content/groups/public/org/codehaus/mojo/maven-metadata.xml
[WARNING] Could not transfer metadata org.apache.maven.plugins/maven-metadata.xml from/to nexus (http://192.168.129.188:19999/nexus/content/groups/public): Not authorized , ReasonPhrase:Unauthorized.
[WARNING] Could not transfer metadata org.codehaus.mojo/maven-metadata.xml from/to nexus (http://192.168.129.188:19999/nexus/content/groups/public): Not authorized , ReasonPhrase:Unauthorized.
Downloading: http://192.168.129.188:19999/nexus/content/groups/public/org/apache/maven/plugins/maven-archetype-plugin/maven-metadata.xml
[WARNING] Could not transfer metadata org.apache.maven.plugins:maven-archetype-plugin/maven-metadata.xml from/to nexus (http://192.168.129.188:19999/nexus/content/groups/public): Not authorized , ReasonPhrase:Unauthorized.
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building vimgr 1.0
[INFO] ------------------------------------------------------------------------
[WARNING] Failure to transfer org.apache.maven.plugins/maven-metadata.xml from http://192.168.129.188:19999/nexus/content/groups/public was cached in the local repository, resolution will not be reattempted until the update interval of nexus has elapsed or updates are forced. Original error: Could not transfer metadata org.apache.maven.plugins/maven-metadata.xml from/to nexus (http://192.168.129.188:19999/nexus/content/groups/public): Not authorized , ReasonPhrase:Unauthorized.
[WARNING] Failure to transfer org.codehaus.mojo/maven-metadata.xml from http://192.168.129.188:19999/nexus/content/groups/public was cached in the local repository, resolution will not be reattempted until the update interval of nexus has elapsed or updates are forced. Original error: Could not transfer metadata org.codehaus.mojo/maven-metadata.xml from/to nexus (http://192.168.129.188:19999/nexus/content/groups/public): Not authorized , ReasonPhrase:Unauthorized.
[WARNING] Failure to transfer org.apache.maven.plugins:maven-archetype-plugin/maven-metadata.xml from http://192.168.129.188:19999/nexus/content/groups/public was cached in the local repository, resolution will not be reattempted until the update interval of nexus has elapsed or updates are forced. Original error: Could not transfer metadata org.apache.maven.plugins:maven-archetype-plugin/maven-metadata.xml from/to nexus (http://192.168.129.188:19999/nexus/content/groups/public): Not authorized , ReasonPhrase:Unauthorized.
[WARNING] Failure to transfer org.apache.maven.plugins:maven-javadoc-plugin/maven-metadata.xml from http://192.168.129.188:19999/nexus/content/groups/public was cached in the local repository, resolution will not be reattempted until the update interval of nexus has elapsed or updates are forced. Original error: Could not transfer metadata org.apache.maven.plugins:maven-javadoc-plugin/maven-metadata.xml from/to nexus (http://192.168.129.188:19999/nexus/content/groups/public): Not authorized , ReasonPhrase:Unauthorized.
[WARNING] Failure to transfer org.apache.maven.plugins:maven-javadoc-plugin/maven-metadata.xml from http://192.168.129.188:19999/nexus/content/groups/public was cached in the local repository, resolution will not be reattempted until the update interval of nexus has elapsed or updates are forced. Original error: Could not transfer metadata org.apache.maven.plugins:maven-javadoc-plugin/maven-metadata.xml from/to nexus (http://192.168.129.188:19999/nexus/content/groups/public): Not authorized , ReasonPhrase:Unauthorized.
[WARNING] Failure to transfer org.apache.maven.plugins:maven-javadoc-plugin/maven-metadata.xml from http://192.168.129.188:19999/nexus/content/groups/public was cached in the local repository, resolution will not be reattempted until the update interval of nexus has elapsed or updates are forced. Original error: Could not transfer metadata org.apache.maven.plugins:maven-javadoc-plugin/maven-metadata.xml from/to nexus (http://192.168.129.188:19999/nexus/content/groups/public): Not authorized , ReasonPhrase:Unauthorized.
[INFO] 
[INFO] >>> maven-archetype-plugin:2.2:create-from-project (default-cli) @ vimgr >>>
[INFO] 
[INFO] <<< maven-archetype-plugin:2.2:create-from-project (default-cli) @ vimgr <<<
[INFO] 
[INFO] --- maven-archetype-plugin:2.2:create-from-project (default-cli) @ vimgr ---
[INFO] Setting default groupId: com.huatek.unicorn
[INFO] Setting default artifactId: vimgr
[INFO] Setting default version: 1.0
[INFO] Setting default package: com.huatek.unicorn
Scanned 3 filtered files in 3 files: filters/filter-dev.properties, filters/filter-production.properties, filters/filter-test.properties
Scanned 0 filtered files in 1 files: 
Scanned 9 filtered files in 9 files: src/main/java/com/huatek/unicorn/continer/ReloadableMain.java, src/main/java/com/huatek/unicorn/brandauth/dao/ApplyDAO.java, src/main/java/com/huatek/unicorn/brandauth/service/ApplyService.java, src/main/java/com/huatek/unicorn/brandauth/service/impl/ApplyServiceImpl.java, src/main/java/com/huatek/unicorn/brandauth/model/ApplyBaseInfo.java, src/main/java/com/huatek/unicorn/brandauth/domain/Apply.java, src/main/java/com/huatek/unicorn/util/Json.java, src/main/java/com/huatek/unicorn/util/IpUtil.java, src/main/java/com/huatek/unicorn/util/IdentityTool.java
Scanned 7 filtered files in 7 files: src/main/resources/spring/spring-common.xml, src/main/resources/spring/spring-datasource.xml, src/main/resources/spring/spring-transaction.xml, src/main/resources/spring/spring-log.xml, src/main/resources/spring/spring-external.xml, src/main/resources/log4j.xml, src/main/resources/env.properties
Scanned 1 filtered files in 1 files: src/main/java/com/huatek/unicorn/brandauth/service/ApplyServiceClient.java
Scanned 1 filtered files in 1 files: src/main/java/com/huatek/unicorn/brandauth/service/ApplyServiceClientImpl.java
Scanned 4 filtered files in 11 files: src/main/resources/spring_service/spring-dubbo.xml, src/main/resources/spring_service/brandauth/spring-brandauth-provider.xml, src/main/conf/dubbo.properties, src/main/assemble/service-impl-jar-with-dependency.xml
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building vimgr-archetype 1.0
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ vimgr-archetype ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] Copying 38 resources
[INFO]
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ vimgr-archetype ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] Copying 2 resources
[INFO] 
[INFO] --- maven-archetype-plugin:2.2:jar (default-jar) @ vimgr-archetype ---
[INFO] Building archetype jar: /source/git_repo/java_fm/distributed_service/target/generated-sources/archetype/target/vimgr-archetype-1.0
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1.314s
[INFO] Finished at: Tue Aug 12 17:16:18 CST 2014
[INFO] Final Memory: 11M/104M
[INFO] ------------------------------------------------------------------------
[INFO] Archetype created in /source/git_repo/java_fm/distributed_service/target/generated-sources/archetype
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] vimgr ............................................. SUCCESS [3.804s]
[INFO] vimgr-biz ......................................... SKIPPED
[INFO] vimgr-service-client .............................. SKIPPED
[INFO] vimgr-service-impl ................................ SKIPPED
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 4.971s
[INFO] Finished at: Tue Aug 12 17:16:18 CST 2014
[INFO] Final Memory: 19M/173M
[INFO] ------------------------------------------------------------------------

cd target/generated-sources/archetype/

mvn install

[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building vimgr-archetype 1.0
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ vimgr-archetype ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] Copying 38 resources
[INFO]
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ vimgr-archetype ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] Copying 2 resources
[INFO]
[INFO] --- maven-archetype-plugin:2.2:jar (default-jar) @ vimgr-archetype ---
[INFO] Building archetype jar: /source/git_repo/java_fm/distributed_service/target/generated-sources/archetype/target/vimgr-archetype-1.0
[INFO]
[INFO] --- maven-archetype-plugin:2.2:integration-test (default-integration-test) @ vimgr-archetype ---
[INFO] Processing Archetype IT project: basic
[INFO] ----------------------------------------------------------------------------
[INFO] Using following parameters for creating project from Archetype: vimgr-archetype:1.0
[INFO] ----------------------------------------------------------------------------
[INFO] Parameter: groupId, Value: archetype.it
[INFO] Parameter: artifactId, Value: basic
[INFO] Parameter: version, Value: 0.1-SNAPSHOT
[INFO] Parameter: package, Value: it.pkg
[INFO] Parameter: packageInPathFormat, Value: it/pkg
[INFO] Parameter: version, Value: 0.1-SNAPSHOT
[INFO] Parameter: package, Value: it.pkg
[INFO] Parameter: groupId, Value: archetype.it
[INFO] Parameter: artifactId, Value: basic
[INFO] Parent element not overwritten in /source/git_repo/java_fm/distributed_service/target/generated-sources/archetype/target/test-classes/projects/basic/project/basic/basic-biz/pom.xml
[INFO] Parent element not overwritten in /source/git_repo/java_fm/distributed_service/target/generated-sources/archetype/target/test-classes/projects/basic/project/basic/basic-service-client/pom.xml
[INFO] Parent element not overwritten in /source/git_repo/java_fm/distributed_service/target/generated-sources/archetype/target/test-classes/projects/basic/project/basic/basic-service-impl/pom.xml
[INFO] project created from Archetype in dir: /source/git_repo/java_fm/distributed_service/target/generated-sources/archetype/target/test-classes/projects/basic/project/basic
[INFO] No post-archetype-generation goals to invoke.
[INFO]
[INFO] --- maven-install-plugin:2.5.1:install (default-install) @ vimgr-archetype ---
[INFO] Installing /source/git_repo/java_fm/distributed_service/target/generated-sources/archetype/target/vimgr-archetype-1.0.jar to /opt/devtools/MavenRepositories/Maven/com/huatek/unicorn/vimgr-archetype/1.0/vimgr-archetype-1.0.jar
[INFO] Installing /source/git_repo/java_fm/distributed_service/target/generated-sources/archetype/pom.xml to /opt/devtools/MavenRepositories/Maven/com/huatek/unicorn/vimgr-archetype/1.0/vimgr-archetype-1.0.pom
[INFO]
[INFO] --- maven-archetype-plugin:2.2:update-local-catalog (default-update-local-catalog) @ vimgr-archetype ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1.614s
[INFO] Finished at: Tue Aug 12 17:18:21 CST 2014
[INFO] Final Memory: 11M/107M
[INFO] ------------------------------------------------------------------------
```

###### 非默认的Apache maven设置时，需要特别处理。

```bash
# 假设Apache maven的默认仓库位置被更改至”/opt/xx/MavenRepo/Maven“时。

# 生成archetype的目录文件，使用命令后会在仓库目录生成名为“archetype-catalog.xml”的目录文件。

mvn archetype:crawl

[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building vimgr-archetype 1.0
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-archetype-plugin:2.2:crawl (default-cli) @ vimgr-archetype ---
repository /opt/devtools/MavenRepositories/Maven
catalogFile null
[INFO] Scanning /opt/devtools/MavenRepositories/Maven/org/sonatype/plexus/plexus-build-api/0.0.4/plexus-build-api-0.0.4.jar
...
[INFO] Scanning /opt/devtools/MavenRepositories/Maven/commons-collections/commons-collections/3.2.1/commons-collections-3.2.1.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1.084s
[INFO] Finished at: Tue Aug 12 16:55:38 CST 2014
[INFO] Final Memory: 7M/104M
[INFO] ----------------------------------------------------------------------
```

- - -

##### [根据工程骨架建立项目]

###### 根据以下命令，使用上步建立的工程骨架建立项目（基于默认的Apache maven设置）。

```bash
# goto new_project_base_folder

mvn archetype:generate -DarchetypeCatalog=local

[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building Maven Stub Project (No POM) 1
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] >>> maven-archetype-plugin:2.2:generate (default-cli) @ standalone-pom >>>
[INFO]
[INFO] <<< maven-archetype-plugin:2.2:generate (default-cli) @ standalone-pom <<<
[INFO]
[INFO] --- maven-archetype-plugin:2.2:generate (default-cli) @ standalone-pom ---
[INFO] Generating project in Interactive mode
[INFO] No archetype defined. Using maven-archetype-quickstart (org.apache.maven.archetypes:maven-archetype-quickstart:1.0)
Choose archetype:
1: file:///opt/devtools/MavenRepositories/Maven/archetype-catalog.xml -> com.huatek.unicorn:vimgr-archetype (vimgr)
Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains): 1
# 选择对应archetype，这里我们选1。
# 以下4行均为交互式的输入： “com.rst”，“testproject”，“1.0”，“null”

Define value for property 'groupId': : com.rst
Define value for property 'artifactId': : testproject
Define value for property 'version':  1.0-SNAPSHOT: : 1.0
Define value for property 'package':  com.rst: :
Confirm properties configuration:
groupId: com.rst
artifactId: testproject
version: 1.0
package: com.rst
 Y: : Y
[INFO] ----------------------------------------------------------------------------
[INFO] Using following parameters for creating project from Archetype: vimgr-archetype:1.0
[INFO] ----------------------------------------------------------------------------
[INFO] Parameter: groupId, Value: com.rst
[INFO] Parameter: artifactId, Value: testproject
[INFO] Parameter: version, Value: 1.0
[INFO] Parameter: package, Value: com.rst
[INFO] Parameter: packageInPathFormat, Value: com/rst
[INFO] Parameter: package, Value: com.rst
[INFO] Parameter: version, Value: 1.0
[INFO] Parameter: groupId, Value: com.rst
[INFO] Parameter: artifactId, Value: testproject
[INFO] Parent element not overwritten in /source/git_repo/java_fm/demo_service/testproject/testproject-biz/pom.xml
[INFO] Parent element not overwritten in /source/git_repo/java_fm/demo_service/testproject/testproject-service-client/pom.xml
[INFO] Parent element not overwritten in /source/git_repo/java_fm/demo_service/testproject/testproject-service-impl/pom.xml
[INFO] project created from Archetype in dir: /source/git_repo/java_fm/demo_service/testproject
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 37.884s
[INFO] Finished at: Tue Aug 12 17:22:13 CST 2014
[INFO] Final Memory: 12M/104M
[INFO] ------------------------------------------------------------------------

# 查看以下建立好的工程
ls

testproject

ls testproject/

filters  testproject-biz             testproject-service-impl
pom.xml  testproject-service-client
```

###### 非默认的Apache maven设置时，需要特别处理。

```bash
# 假设Apache maven的默认仓库位置被更改至”/opt/xx/MavenRepo/Maven“时。
# 生成archetype的目录文件，使用命令后会在仓库目录生成名为“archetype-catalog.xml”的目录文件。
# 以下命令明确的指定archetype的仓库文件路径

mvn archetype:generate -DarchetypeCatalog='file:///opt/xx/MavenRepo/Maven/archetype-catalog.xml'

[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building Maven Stub Project (No POM) 1
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] >>> maven-archetype-plugin:2.2:generate (default-cli) @ standalone-pom >>>
[INFO]
[INFO] <<< maven-archetype-plugin:2.2:generate (default-cli) @ standalone-pom <<<
[INFO]
[INFO] --- maven-archetype-plugin:2.2:generate (default-cli) @ standalone-pom ---
[INFO] Generating project in Interactive mode
[INFO] No archetype defined. Using maven-archetype-quickstart (org.apache.maven.archetypes:maven-archetype-quickstart:1.0)
Choose archetype:
1: file:///opt/xx/MavenRepo/Maven/archetype-catalog.xml -> com.huatek.unicorn:vimgr-archetype (vimgr)
Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains):

# choose it...
```

- - -

###### 打开Eclipse，导入新建立的Apache maven工程。

###### [Step-1]选择导入工程类型为“Existing Maven Projects”

![](../images/e_imp_maven_project.png)

###### [Step-2]选择导入工程的根文件夹，Eclipse会自动扫描出工程结构，如图所示

![](../images/e_imp_maven_project_2.png)

###### [Step-3]选取Maven插件与Eclipse的映射关联

![](../images/e_imp_maven_project_3.png)

###### [Step-4]工程导入成功后应如下图所示

![](../images/e_imp_maven_project_finished.png)

###### [注意]可能需要手工导入ojdbc文件到本地maven库中。

```bash
# 命令中的导入的ojdbc7.jar需要从oracle网站手工下载。
# -Dversion=12.1.0.1 声明该jar文件的版本号。

mvn install:install-file -DgroupId=com.oracle -DartifactId=ojdbc7 -Dversion=12.1.0.1 -Dpackaging=jar -Dfile=ojdbc7.jar -DgeneratePom=true

```

- - -

##### [工程骨架简介]

###### 组成结构

```
# 工程骨架由以下4个部分组成:

[p-name] 父工程：模块名称、模块公共定义，模块依赖约束等公共定义部分。
[p-name-biz] 模块业务逻辑实现：该部分具体实现本模块的业务逻辑。
[p-name-service-client] 模块服务接口定义：该部分定义本模块对外服务的接口，也将作为接口依赖提供给其他引用该模块的模块使用。
[p-name-service-impl] 模块服务实现：本部分实现服务接口，并作为独立进程将发布至运行环境中。
```

###### [p-name]

```
# 目录结构如下：

p-name                        工程根文件夹
|-- filters                   配置过滤器文件夹
|-- p-name-biz                业务逻辑子工程
|-- p-name-service-client     服务接口子工程
|-- p-name-service-impl       服务实现子工程
|-- pom.xml                   Apache maven 配置文件

父工程主要包含两个部分，分别是“pom.xml”于“filters”文件夹。
“pom.xml”中定义了该工程的依赖管理、宏、工程属性、编译器版本等公共信息。
“filers”目录中定义该工程在不同环境下打包时的不同配置项，并将这些文件的引用定义于“pom.xml”中，默认的有开发、测试以及生产环境三种不同的配置，工程在打包时默认会采用开发环境的配置。
```

###### [p-name-biz]

```
# 目录结构如下：

p-name-biz
|-- src
|---|-- main
|---|---|-- java        java文件
|---|---|-- resources   各类配置文件

该子工程负责在该模块工程中实现业务逻辑，即包含了数据访问（但建议不包含对其他远程服务的引用，这样的引用因在服务接口实现中完成，从而避免分布式事务的繁琐于相互制约。），该部分在访问数据持久化、文件等系统资源是应做到事务闭环、业务闭环，即该部分向外部提供的服务均应具备足够的原子性，继而达到系统服务之间的解偶。
该子工程默认使用spring作为管理容器（后续可根据业务适应性进行调整），该部分关于spring、以及其他引用的类库的配置均应在内部闭环，不应再依赖其他环境配置。
```

###### [p-name-service-client]

```
# 目录结构如下：

p-name-service-client
|-- src
|---|-- main
|---|---|-- java         java源代码

该子工程仅定义服务接口（java接口形式）。
```

###### [p-name-service-impl]

```
# 目录结构如下：

p-name-service-impl
|-- src
|---|-- main
|---|---|-- assemble      Apache maven打包装配配置文件
|---|---|-- bin           bash与windows shell启动脚本
|---|---|-- conf          dubbo服务配置文件
|---|---|-- java          java源代码
|---|---|-- resources     各类配置文件

该工程打包后需要具备独立进程启动能力，使用spring作为管理容器，引用业务逻辑子工程、服务接口子工程，最后打包为tar格式，通过命令行脚本启动进程。
该子工程中需要将业务实现的模型于内部服务，有机的组合继而向外部系统提供RMI调用服务，该子工程中可以引用其他远程服务（RMI），单依旧需要保证本服务的闭环，不可将事务建立于远程调用间。

```

- - -

##### [业务逻辑编写]


- - -


##### [服务配置]

###### Spring配置原则，Spring配置文件结构如下：

![](../images/e_P_biz_struct.png)

```
spring-common.xml 通用、公共组件配置。
spring-datasource.xml 数据源配置。
spring-external.xml 外部组件引用配置。
spring-log.xml 日志配置。
spring-transaction.xml 事务相关配置。

# 以上配置均使用一般的spring配置方式，这里不再做进一步的说明。
```

###### 过滤器适配型配置

```xml
# 在filters目录中的配置文件*.properties，根据文件名称适配不同的运行环境，打包程序时会根据传入的运行环境参数对配置文件中的宏执行宏替换，从而达到动态改变配置的目的。

# 以下是dubbo于datasource的动态适配配置示例：

#dubbo
dubbo.registry.svr_xxx=192.168.129.188:2181
dubbo.registry.svr_yyy=192.168.129.189:2181,192.168.129.199:2181
dubbo.port=-1

#jdbc configuration
jdbc.driver=oracle.jdbc.driver.OracleDriver
jdbc.url=jdbc:oracle:thin:@192.168.128.89:1521/cqpetro
jdbc.username=xbb
jdbc.password=xbb876

# 以下是宏引用的配置示例：

<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
<property name="driverClassName" value="${jdbc.driver}" />
<property name="url" value="${jdbc.url}" />
<property name="username" value="${jdbc.username}" />
<property name="password" value="${jdbc.password}" />
</bean>

```

###### dubbo服务配置

```xml
# 服务接口暴露配置在spring配置文件中完成。

# 通过先将需暴露的dubbo配置成一个spring服务，然后通过dubbo的spring扩展配置标签，将本地服务映射到dubbo服务上。以下是一个简单的示例：

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://code.alibabatech.com/schema/dubbo
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd
        ">
	<!-- 使用dubbo协议暴露服务 -->
	<bean id="applyServiceClient"
		class="com.rst.xbb.brandauth.service.ApplyServiceClientImpl">
	</bean>
	<dubbo:service interface="com.rst.xbb.brandauth.service.ApplyServiceClient"
		owner="${app.admin}" ref="applyServiceClient" version="1.0" protocol="dubbo"
		delay="-1" registry="dubboRegistryXxx" />
</beans>

```

- - -

##### [打包&发布，运行]

```bash
# 本工程的打包使用Apache maven。

mvn package -Pdev -Dmaven.test.skip=true

# 打包后生成 p-name-service-impl/target/p-name-service-impl-$version-package.tar

# 解压打包后的文件
tar -xf p-name-service-impl-$version-package.tar

# 解压后得到与文件同名的目录 p-name-service-impl-$version

p-name-service-impl-$version
|-- bin
|-- lib
|-- conf

# 进入p-name-service-impl-$version/bin，运行对应平台的脚本启动服务进程。

cd p-name-service-impl-$version/bin

# *nix bash 平台
./start.sh

# Windows 平台
start.bat

```

- - -

#### 调试

由于Dubbo服务启动后为独立的java进程，调试需要建立在java远程调试的方式上。

##### [启动进程]

将以下启动参数加入被启动的java命令中:

```bash
-Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,address=8000,server=y,suspend=n
```

##### [附着调试]

以Eclipse为例，以下介绍如何完成远程调试，该调试方法适用于大多数java程序的远程调试。

```bash
此处假设被调式进程宿主主机IP为：111.111.111.111, 端口为：8000
```

###### 启动Eclipse，打开Debug配置界面，并选择“Remote Java Application”， 然后点击“New”新建一个远程调式配置。

![](../images/e_select_debug_rmt_app.png)

###### 打开远程调试配置界面，选择“Connect”选项卡，填写基本信息。

![](../images/e_type_debug_info.png)

###### 选择“Source”选项卡，选配置目源码攫取路径。

![](../images/e_select_debug_src.png)

###### 选择“Connect”选项卡，查看配置信息并点击“Apply”保存，点击“Debug”启动附着调试（这里假设被调试程序已经于前边步骤启动完毕）

![](../images/e_type_debug_info.png)




## 工具与资源

>[1] [Alibaba dubbo service framework](https://github.com/alibaba/dubbo)

>[2] [Eclipse](http://www.eclipse.org)

>[3] [Java](http://java.sun.com)

>[4] [Apache maven](http://maven.apache.org)

>[5] [Linux any LSB release](http://kernel.org)

