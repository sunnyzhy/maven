# Maven

## Maven 仓库

仓库分类：本地仓库和远程仓库。Maven根据坐标寻找构件的时候，它先会查看本地仓库，如果本地仓库存在构件，则直接使用；如果没有，则从远程仓库查找，找到后，下载到本地。

### 本地仓库

默认情况下，在本地都有一个路径为 ```.m2/repository/``` 的仓库目录。也可以在 ```settings.xml``` 文件的 ```localRepository``` 配置本地仓库的地址。

### 远程仓库

远程仓库包含私有仓库和中央仓库。

Maven 必须知道至少一个可用的中央仓库，Maven 默认有一个 ```super pom``` 文件，其位置在 ```lib``` 目录下的 ```maven-model-builder-x.x.x.jar``` 中的 ```org/apache/maven/model/pom-4.0.0.xml```。

```xml
  <repositories>
    <repository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>
```

- 默认的中央仓库的 ```id``` 为 ```central```，```url``` 为 ```https://repo.maven.apache.org/maven2```
- 在 ```settings.xml``` 文件配置一个 ```mirrorOf``` 为 ```central``` 的镜像就会替代中央仓库

## Maven 镜像

mirrorOf 标签:

- ```external:* =```: 不在本地仓库的文件才从该镜像获取
- ```repo,repo1 =```: 远程仓库 repo 和 repo1 从该镜像获取
- ```*,!repo1 =```: 所有远程仓库都从该镜像获取，除 repo1 远程仓库以外
- ```* =```: 所用远程仓库都从该镜像获取
