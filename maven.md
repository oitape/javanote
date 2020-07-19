- 命令
    - mvn compile  编译
    - mvn clean  清理
    - mvn test  测试
    - mvn package 打包
    - mvn instal  打包并copy到本地仓库
- Web项目配置
    - <build>内部进行配置plugins
    ```
    <plugins>
        <plugin>
          <groupId>org.apache.tomcat.maven</groupId>
          <artifactId>tomcat7-maven-plugin</artifactId>
          <version>2.1</version>
          <configuration>
            <port>8081</port>
            <server>tomcat7</server>
          </configuration>
        </plugin>
    </plugins>
    ```
    - Add Configuation
    ![](/assets/iShot2020-07-19下午05.36.05.png)

- 打包源代码插件
    ```java
    
    ```