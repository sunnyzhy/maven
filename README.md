# Maven

## Maven 仓库

仓库分类：本地仓库和远程仓库。Maven根据坐标寻找构件的时候，它先会查看本地仓库，如果本地仓库存在构件，则直接使用；如果没有，则从远程仓库查找，找到后，下载到本地。

### 本地仓库

默认情况下，在本地都有一个路径为 ```.m2/repository/``` 的仓库目录。也可以在 ```settings.xml``` 文件的 ```localRepository``` 配置本地仓库的地址。

### 远程仓库

远程仓库包含私有仓库和中央仓库。

Maven 必须知道至少一个可用的中央仓库，Maven 默认有一个 ```super pom``` 文件，其位置在 ```lib``` 目录下的 ```maven-model-builder-${version}.jar``` 中的 ```org/apache/maven/model/pom-4.0.0.xml```。

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

- ```external:*```: 不在本地仓库的文件才从该镜像获取
- ```repo,repo1```: 远程仓库 repo 和 repo1 从该镜像获取
- ```*,!repo1```: 所有的远程仓库都从该镜像获取，除 repo1 远程仓库以外
- ```*```: 所有的远程仓库都从该镜像获取

maven 如何选择使用哪个 mirror 节点:

- maven 不是按照 mirror 书写的物理顺序来查询的，也就是 maven 并不会因为 jar 在物理顺序的第一个 mirror 中不存在而去物理顺序的第二个 mirror 中查询
- maven 是按照 mirror 的字母排序进行查询，比如有 id 为 B、A、C 顺序的 mirror；那么 maven 会根据字母排序来指定第一个，所以不管怎么排列，一定会先找到 id 是 A 的 mirror，当 A 无法连接时才会去 B 查询
- 配置多个 mirror 的时候，尽量不要把 mirrorOf 配置为 ```*```，因为 ```*``` 的意思是所有的远程仓库都从该镜像获取，从而造成其他 mirror 都不会生效

## 私有仓库认证

1. 在 maven 的 ```setting.xml``` 的 servers 节点内的 server 添加私有仓库的凭证
2. 以下各节点的 id 必须保持一致，才能通过私有仓库的服务端认证
   - maven 的 ```setting.xml``` 的 servers 节点内的 server 的 id
   - maven 的 ```setting.xml``` 的 mirrors 节点内的 mirror 的 id
   - ```pom.xml``` 的 distributionManagement 和 repositories 节点内的 repository 的 id
   - 私有仓库 的 hosted repository's Name
