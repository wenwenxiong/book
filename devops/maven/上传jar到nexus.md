maven服务为HTTP时，使用如下方式：

如你需要上传utils-1.0.jar包，首先需准备新建一个pom.xml文件，内容如下：
```
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.foo</groupId>
  <artifactId>utils</artifactId>
  <version>1</version>
</project>
```
pom.xml和utils-1.0.jar两个文件放到一块就行，接下来使用命令上传到nexus上面：
```
curl -v -u admin:admin123 --upload-file pom.xml http://localhost:8081/nexus/repository/maven-releases/org/foo/utils/1.0/utils-1.0.pom
```
上述命令上传pom.xml文件到nexus上，并改名为utils-1.0.pom，注意你的release路径和包放的路径
```
curl -v -u admin:admin123 --upload-file utils-1.0.jar http://localhost:8081/nexus/repository/maven-releases/org/foo/utils/1.0/utils-1.0.jar
```
上述命令上传jar到相同路径下面。

也可以访问nexus界面点击上传
