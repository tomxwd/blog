---
2019-09-02 10:33:43

---

#

- 构建后操作
  - 增加构建后操作步骤
    - Deploy war/ear to a container
      - war/ear files：是以工作区间目录为基准，那么就是target/Apple-0.0.1-SNAPSHOT.war
      - Context path：指定上下文路径apple
      - Containers：添加容器
        - 添加tomcat用户名密码
        - tomcat的URL：http://localhost:8080

再次点立即构建，看tomcat的webapp目录下，发现会有war包且已经部署好项目在上面；

