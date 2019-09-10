---
title: 07_Dubbo_环境搭建_管理控制台
date: 2019-09-09 20:55:02
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 07_Dubbo\_环境搭建_管理控制台

> [dubbo-admin的github网址](https://github.com/apache/dubbo-admin/tree/master)

这里用到的是master分支的，切记！！！

点开[网址](https://github.com/apache/dubbo-admin/tree/master)，下载整个项目。

1. 解压后打开dubbo-admin文件夹，修改src/main/resource目录下的application.properties文件

   ```properties
   #
   # Licensed to the Apache Software Foundation (ASF) under one or more
   # contributor license agreements.  See the NOTICE file distributed with
   # this work for additional information regarding copyright ownership.
   # The ASF licenses this file to You under the Apache License, Version 2.0
   # (the "License"); you may not use this file except in compliance with
   # the License.  You may obtain a copy of the License at
   #
   #     http://www.apache.org/licenses/LICENSE-2.0
   #
   # Unless required by applicable law or agreed to in writing, software
   # distributed under the License is distributed on an "AS IS" BASIS,
   # WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   # See the License for the specific language governing permissions and
   # limitations under the License.
   #
   
   server.port=7001
   spring.velocity.cache=false
   spring.velocity.charset=UTF-8
   spring.velocity.layout-url=/templates/default.vm
   spring.messages.fallback-to-system-locale=false
   spring.messages.basename=i18n/message
   spring.root.password=root
   spring.guest.password=root
   
   dubbo.registry.address=zookeeper://127.0.0.1:2181
   ```

2. 在dubbo-admin下运行cmd.exe

3. 使用mvn clean package命令打包

4. 把target里面的jar放到一个指定目录，然后用java -jar的命令运行该jar

5. 从application.properties就可以知道，访问该项目，是本地的7001端口，账号密码皆为root