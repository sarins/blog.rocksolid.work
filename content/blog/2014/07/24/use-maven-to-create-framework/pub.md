+++
title = '使用Maven建立分布式工程骨架'
subtitle = 'Use Maven to create distributed project archetype'
brief = ''
date = 2014-07-24
categories = ['Engineering', 'Java', 'Arch']
series = []
tags = ['Maven', 'Java']
type = 'blog'
+++


<!-- # 使用Maven建立分布式工程骨架 -->

>[下载分布式工程框架 https://github.com/sarins/distributed_service](https://github.com/sarins/distributed_service)

>[Maven参考文档](http://maven.apache.org/archetype/maven-archetype-plugin/examples/create-multi-module-project.html)

- - -

## Shell commands as below

```bash
# Step 1
cd current_project_folder
# Step 2
mvn archetype:create-from-project -Darchetype.filteredExtensions=java
# Step 3
cd target/generated-sources/archetype/
# Step 4
mvn install
# Step 5
cd new_project_base_folder
# Step 6
mvn archetype:generate -DarchetypeCatalog=local
# [list local archetypes at there]
# choose archetype
```
