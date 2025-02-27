# maven 多个仓库查询的优先级顺序

参考： https://zhuanlan.zhihu.com/p/623467597

## maven 仓库的优先级（由高到低）

1. 全局配置文件 ```settings.xml```，也就是 maven 安装目录下的 ```conf/settings.xml```
2. 用户级别的配置文件，默认是 ```${user.home}/.m2/settings.xml```
3. 本地的 pom.xml 文件
4. 当确定了要查询某个仓库时，maven 会先查询这个仓库有没有对应的镜像仓库，如果有的话，则转向去查镜像仓库，也就是会查当前仓库的镜像仓库，跳过对本仓库的检索
5. 如果同一个 pom 文件里面有多个激活的 profile，则靠后面激活的 profile 的优先级高
6. 针对 pom 文件，如果有激活的 profile，且 profile 里面配置了 repositories，则 profile 里面的 repositories 的仓库优先级比 project 标签下面的 repositories 的优先级高
7. **pom 文件中无论是 project 标签下面直接定义的 repositories，还是 profile 标签下面定义的 repositories，maven 都会按照其内部的 repository 的物理顺序查询，也就是自上而下查询**
8. 如果 settings.xml 中的 profile 的 id 和 pom 文件中的 profile 的 id 一样，则以 settings.xml 中的 profile 中配置的值为准
9. 如果同一个 pom 文件中有多个 profile 被激活，那么处于 profiles 内部靠后面生效的 profile 优先级比 profiles 中靠前的 profile 的优先级高

## 示例

假如有如下的全局配置文件 ```settings.xml```:

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.2.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.2.0 http://maven.apache.org/xsd/settings-1.2.0.xsd">

<localRepository>D:/programs/.m2/repository</localRepository>

  <servers>
    <server>
      <id>dev</id>
      <username>repouser</username>
      <password>repopwd</password>
    </server>
  </servers>

  <mirrors>
    <mirror>
      <id>nexus-aliyun</id>
      <mirrorOf>central</mirrorOf>
      <name>Nexus aliyun</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror> 
    <mirror>
      <id>dev-mirror</id>
      <mirrorOf>dev1</mirrorOf>
      <name>第二套开发仓库</name>
      <url>http://192.168.1.2/repository/devM</url>
    </mirror> 
  </mirrors>

  <profiles>
    
    <profile>
      <id>env-dev</id>
      <repositories>
        <repository>
          <id>dev5</id>
          <name>Repository for JDK 1.4 builds</name>
          <url>http://192.168.1.1/repository/dev5</url>
        </repository>
      </repositories>
    </profile>

    <profile>
      <id>env-test</id>
      <repositories>
        <repository>
          <id>test</id>
          <name>test</name>
          <url>http://192.168.1.1/repository/test</url>
        </repository>
      </repositories>
    </profile>

  </profiles>

  <activeProfiles>
    <activeProfile>env-dev</activeProfile>
  </activeProfiles>

</settings>
```

工程的 ```pom.xml``` 配置文件:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>test</artifactId>
    <version>${revision}</version>
    <packaging>pom</packaging>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <revision>1.0.0</revision>
    </properties>

    <repositories>
    <repository>
      <id>dev4</id>
      <name>dev4</name>
      <url>http://192.168.1.1/repository/dev4</url>
    </repository>
  </repositories>

  <profiles>
    <profile>
      <id>profile-1</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <repositories>
        <repository>
          <id>dev1</id>
          <name>dev1</name>
          <url>http://192.168.1.1/repository/dev1</url>
        </repository>
        <repository>
          <id>dev2</id>
          <name>dev2</name>            <url>http://192.168.1.1/repository/dev2</url>
        </repository>
      </repositories>
    </profile>
    <profile>
      <id>profile-2</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <repositories>
        <repository>
          <id>dev3</id>
          <name>dev3</name>
         <url>http://192.168.1.1/repository/dev3</url>
        </repository>
      </repositories>
    </profile>
  </profiles>

</project>
```

### settings.xml 和 pom.xml 都配置激活了各自的 profile

```pom.xml``` 文件默认激活了 profile-1 和 profile-2，```settings.xml```中默认激活了 env-dev。按照在同一文件的 profile 的生效顺序规则，```pom.xml``` 文件中的仓库使用顺序为:

```dev5->dev3->dev1->dev2->dev4->central（超级pom中定义的中央仓库）```

而由于在 ```setttings.xml``` 中为 dev1 和 central 配置了镜像仓库，所以最终仓库的优先查询顺序为:

```dev5->dev3->dev-mirror->dev2->dev4->nexus-aliyun```

### ```settings.xml``` 没有配置激活的 profile，pom.xml 中配置了激活的 profile

这种情况下，```settings.xml``` 中没有设置 activeProfiles，我们只需要考虑 ```pom.xml``` 文件中仓库的查询顺序，按照先前说的规则，则仓库使用顺序为:

```dev3->dev1->dev2->dev4->central（超级pom中定义的中央仓库）```

而由于在 ```setttings.xml``` 中为 dev1 和 central 配置了镜像仓库，所以最终仓库的优先查询顺序为：

```dev3->dev-mirror->dev2->dev4->nexus-aliyun```
