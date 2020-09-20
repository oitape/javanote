- 命令
    - mvn compile  编译
    - mvn clean  清理
    - mvn test  测试
    - mvn package 打包
    - mvn instal  打包并copy到本地仓库

- 启动Web项目Tomcat插件配置
    - <build>内部进行配置plugins
    ```xml
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

- 启动Web项目jetty插件配置(最新的Tomcat插件是7，不支持websocket)
  ```xml
  <build>
    <finalName>websocket</finalName>
    <plugins>
      <plugin>
          <groupId>org.eclipse.jetty</groupId>
          <artifactId>jetty-maven-plugin</artifactId>
          <version>9.4.31.v20200723</version>
      </plugin>
    </plugins>
  </build>
  ```
  - Add Configuation
    ![](/assets/iShot2020-09-20上午10.05.06.png)
  

- 打包的同时生成源代码jar包插件
    ```xml
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-source-plugin</artifactId>
        <version>2.2.1</version>
        <executions>
          <execution>
            <goals>
              <goal>jar-no-fork</goal>
            </goals>
            <phase>verify</phase>
          </execution>
        </executions>
      </plugin>
    ```
- JDK编译插件
  ```xml
  <plugin>
    <groupId>org.apache.maven.plugins</groupId> 
    <artifactId>maven-compiler-plugin</artifactId> 
    <version>3.8.0</version>
    <configuration> 
      <source>1.8</source> 
      <target>1.8</target> 
      <encoding>UTF-8</encoding>
    </configuration>
  </plugin>
  ```
- 私服
  - 私服的好处
    - 缓存Maven中央仓库的jar包，这样不需要每次本地仓库没有jar包而不必到中央仓库下载，而是到私服下载。
    - 解决公司无法上网，而又要连接中央仓库的问题。只要连接私服，确保私服可以连接到中央仓库
    - 方便公司内部不同团队或者项目共享jar包，需要共享jar包，可以上传到私服，通过私服共享
  - 安装
    - Nexus下载地址:https://www.sonatype.com/nexus-repository-oss
    - 解压启动服务 nexus.exe /run (启动成功后不要关闭命令行窗口)
    - 安装服务 nexus.exe /install (重新使用新的命令行安装，可能出现权限不足，请以管理员身份运行命令行) 
    - 启动服务 net start nexus
    - 停止服务 net stop nexus
    - 访问页面 http://localhost:8081/
    - 初始登录用户名和密码:admin/admin123
  - 使用私服
    - 配置settings.xml
      - 添加镜像配置，将所有访问外网仓库的请求指向私服
      ```xml
         <mirror>
          <id>nexus</id>
          <mirrorOf>*</mirrorOf>           
          <url>http://localhost:8081/repository/maven-public/</url>
        </mirror>
      ```
      - 添加profiles，所有请求均通过镜像
      ```xml
      <profile> 
        <id>nexus</id> 
        <repositories>
          <repository>
            <id>central</id> 
            <url>https://repo1.maven.org/maven2</url> 
            <releases>
              <enabled>true</enabled>
            </releases> 
            <snapshots>
              <enabled>true</enabled>
            </snapshots>
          </repository> 
        </repositories> 
        <pluginRepositories>
          <pluginRepository>
            <id>central</id> 
            <url>https://repo1.maven.org/maven2</url> 
            <releases>
              <enabled>true</enabled>
            </releases> 
            <snapshots>
              <enabled>true</enabled>
            </snapshots>
          </pluginRepository> 
        </pluginRepositories>
      </profile>
      ```
      - 发布到私服配置
      ```xml
      <distributionManagement>
        <repository>
            <id>deployment</id>
            <name>Internal Releases</name>
            <url>http://nexus.wosai-inc.com:8081/nexus/content/repositories/releases/</url>
        </repository>
        <snapshotRepository>
            <id>deployment</id>
            <name>Internal Snapshots</name>
            <url>http://nexus.wosai-inc.com:8081/nexus/content/repositories/snapshots/</url>
        </snapshotRepository>
      </distributionManagement>
      ```
      - 配置连接私服权限
      ```xml
      <servers>
        <server>
            <id>deployment</id>
            <username>deployment</username>
            <password>deployment123</password>
        </server>
        <server>
            <id>release</id>
            <username>admin</username>
            <password>admin123</password>
        </server>
      </servers>
      ```
      
    
  