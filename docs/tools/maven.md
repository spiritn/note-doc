# 1. 如何发布项目到私服

在pom文件里配置如下

```xml
<distributionManagement>
  <repository>
    <id>releases</id>
    <url>http://localhost:8081/nexus/content/repositories/releases</url>
  </repository>
 <snapshotRepository>
    <id>snapshots</id>
    <url>http://localhost:8081/nexus/content/repositories/snapshots</url>
  </snapshotRepository>
</distributionManagement>
```

在setting.conf配置如下

```xml
<server>  
  <id>releases</id>  
  <username>admin</username>  
  <password>admin123</password>  
</server>  
<server>  
  <id>snapshots</id>  
  <username>admin</username>  
  <password>admin123</password>  
</server>  
```

然后mvn deploy

# 2. 发布源码到仓库

```xml
<!-- 这个插件可以在发布jar包到仓库时将源码也上传到仓库-->
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-source-plugin</artifactId>
  <executions>
    <execution>
      <id>attach-sources</id>
      <goals>
        <goal>jar</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

# 基本概念

repository 简单点来说就是个仓库。maven里有两种仓库，本地仓库和远程仓库。远程仓库相当于公共的仓库，大家都能看到。本地仓库是你本地的一个山寨版，只有你看的到，主要起缓存作用。当你向仓库请求插件或依赖的时候，会先检查本地仓库里是否有。如果有则直接返回，否则会向远程仓库请求，并做缓存。你也可以把你做的东西上传到本地仓库给你本地自己用，或上传到远程仓库，供大家使用。



中央仓库[https://repo1.maven.org/maven2/](https://repo1.maven.org/maven2/)去请求插件和依赖包。



mirror就是镜像，主要提供一个方便地切换远程仓库地址的途径。

如想要利用阿里云的镜像实现加速访问，即可以给id为central的远程仓库做个镜像。以后向central这个仓库发的请求都会发到http://maven.aliyun.com/nexus/content/groups/public/而不是去中央仓库了。

```xml
<mirror>
  <id>alimaven</id>  
  <name>aliyun maven</name>  
  <url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
  <mirrorOf>central</mirrorOf>  
</mirror>
```

`<mirrorOf>central</mirrorOf>`里是要替代的仓库的id。如果填*，就会替代所有仓库。