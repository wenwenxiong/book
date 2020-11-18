###maven使用私有仓库nexus
配置maven

在```$MAVEN_HOME/conf```下修改```settings.xml```
```
 <pluginGroups>
    <pluginGroup>org.sonatype.plugins</pluginGroup>
  </pluginGroups>

 <servers>
    <server>
      <id>nexus</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
  </servers>

<mirrors>
    <mirror>
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <url>http://localhost:8081/repository/maven-public/</url>
    </mirror>
  </mirrors>

<profiles>
<profile>
      <id>nexus</id>
      <repositories>
        <repository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </repository>
      </repositories>
     <pluginRepositories>
        <pluginRepository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile>

  </profiles>
  <activeProfiles>
    <activeProfile>nexus</activeProfile>
  </activeProfiles>
```
配置maven docker 镜像```maven:3.5-jdk-10```
把自定义的```settings.xml```文件映射到docker镜像内部。
```
COPY settings.xml /usr/share/maven/ref/
```
使用maven编译会首先从nexus私有仓库中下载依赖包，如果nexus私有仓库没有，会从配置的mirror中maven官方中央仓库central中下载，同时存入到nexus私有仓库。
