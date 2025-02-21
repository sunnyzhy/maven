# 使用 nexus 搭建一个 maven 私有仓库（windows）

**以 nexus-3.31.1 为例。**

## 解压

解压之后，会有 nexus 和 sonatype-work 这两个文件夹。

## 启动

端口和主机信息在配置文件 ```./nexus/etc/nexus-default.properties```，端口默认 8081；可以把端口修改为 ```application-port=8080```。

以管理员身份运行cmd，安装服务 ```./nexus/bin/nexus.exe /install```，卸载服务 ```./nexus/bin/nexus.exe /uninstall```，在系统服务里启动 nexus 服务。

## 登录管理后台

后台有两个默认账号：admin 和 anonymous。

访问 ```http://192.168.0.10:8080/```，点击右上角的 ```Sign in```。首次登录会自动生成 ```./sonatype-work/nexus3/admin.password``` 文件，里面保存的是 admin 的初始密码。输入初始密码之后，可以根据后续提示修改密码。

***admin.password 文件是一次性的，登录成功并且修改了初始密码之后，该文件会被删除。***

## 管理 Repository

点击上方的配置按钮 ```Server administration and configuration```，默认打开 ```Repository administration```，点击 ```Repositories``` 打开仓库管理界面。

仓库的类型：

- ```proxy```: 代理仓库，主要是代理公共的远程仓库，比如 aliyun 
- ```group```: 仓库组，主要是将仓库汇总，将私有仓库和远程仓库汇总起来，然后对外提供一个地址。将 proxy 和 hosted 统一。***注：下载私有仓库的 jar，一般使用 ```group``` 类型的仓库，并且一定不能是 ```hosted``` 类型。***
- ```hosted```: 私有仓库，主要是将本地 jar 作为共享资源。***注：发布本地 jar 到私有仓库，只能使用 ```hosted``` 类型的仓库。***

默认的仓库：

- ```maven-central```: maven 中央库，默认从 ```https://repo1.maven.org/maven2/``` 拉取jar。**注：一般需要修改为阿里云的镜像源：```https://maven.aliyun.com/repository/public```**
- ```maven-releases```: 私有仓库，存放稳定的发布版本，即版本号不会频繁变动
- ```maven-snapshots```: 私有仓库，存放快照版本，也就是存放未发布的不稳定版本，即版本号会频繁变动
- ```maven-public```: 仓库分组，默认包含三个成员：```maven-releases```、```maven-snapshots```、```maven-central```，下载 jar 时，依次在这三个仓库里查找，直到找到为止；如果这三个仓库里都不存在，则抛出异常。**所以这三个仓库的排列顺序非常重要。**
- ```nuget-hosted```: 本地存储，像官方仓库一样提供本地私有仓库的功能
   - hosted 有三种方式：Releases、Snapshot、Mixed
      - ```Releases```: 稳定的发布版本
      - ```Snapshot```: 未发布的不稳定版本
      - ```Mixed```: 混合版
- ```nuget.org-proxy```: 提供代理其它仓库的类型
- ```nuget-group```: 组类型，能够组合多个仓库为一个地址提供服务

Deployment 设置选项有三个值：

- ```Allow redeploy```: 允许同一个版本号下重复提交代码，nexus 以时间区分
- ```Disable redeploy```: 不允许同一个版本号下重复提交代码
- ```Read-Only```: 不允许提交任何版本

## 删除组件（jar）

1. 点击上方的配置按钮 ```Browse server contents```，点击左侧栏的 ```Browse```，比如点击列表里的 ```maven-releases``` 打开组件管理界面。
2. 选择要删除的组件，也就是 jar 文件夹，点击右侧的 ```Delete folder```。

## 在 hosts 文件里添加 nexus 映射

```
192.168.0.10  nexus
```

为了防止私有仓库的服务器地址发生变动，可以使用域名和地址映射。

## 配置 Maven 的 central 镜像

修改 ```./conf/settings.xml```：

```xml
  <mirrors>
	<mirror>
	  <id>aliyunmaven</id>
	  <mirrorOf>central</mirrorOf>
	  <name>Nexus aliyun</name>
	  <url>https://maven.aliyun.com/repository/public</url>
	</mirror>
  </mirrors>
```

**注意：**

- **```settings.xml``` 里至少需要配置一个 central 镜像，作用有二：一者用于覆盖 maven-parent 内默认的 ```https://repo.maven.apache.org/maven2```；二者如果私有仓库出现网络故障，那么可以保证本地 maven 从中央仓库镜像下载 jar。**
- 如果不配置的话，比如 maven 的 clean 插件就会从默认的 ```https://repo.maven.apache.org/maven2``` 仓库下载依赖：
   ```
   Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-parent/40/maven-parent-40.pom
   Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-parent/40/maven-parent-40.pom (0 B at 0 B/s)
   ```

## 输出调用 Maven 插件时的详细信息

打开 IDEA 的 ```Settings -> Maven```，修改 ```Output level``` 为 ```Debug```。

这样，在调用 Maven 插件时，就会在控制台输出 Maven 的详细信息。

## 发布 jar 到私有仓库

### 配置 Maven

修改 ```./conf/settings.xml```：

```xml
<servers>
  <server>
    <id>maven-releases</id>
    <username>admin</username>
    <password>admin@123</password>
  </server>

  <server>
    <id>maven-snapshots</id>
    <username>admin</username>
    <password>admin@123</password>
  </server>
</servers>
```

**注意：**

- **```deploy``` 需要认证 ```nexus``` 的账号密码，所以必须在 Maven 的 settings.xml 里配置 ```server```。**

### 配置 pom.xml

```xml
<distributionManagement>
    <!-- 1. repository 的 id
         2. maven 的 setting.xml 的 server 的 id
         3. nexus 的 hosted repository's Name
         这三者必须保持一致才能通过 nexus 认证 -->

    <repository>
        <id>maven-releases</id>
        <url>http://nexus:8080/repository/maven-releases/</url>
        <releases>
           <enabled>true</enabled>
        </releases>
        <snapshots>
           <enabled>false</enabled>
        </snapshots>
    </repository>

    <snapshotRepository>
        <id>maven-snapshots</id>
        <url>http://nexus:8080/repository/maven-snapshots/</url>
        <releases>
           <enabled>false</enabled>
        </releases>
        <snapshots>
           <enabled>true</enabled>
        </snapshots>
    </snapshotRepository>
</distributionManagement>
```

**注意：**

- **distributionManagement 与 dependencies 平级**
- **release 的版本号里不能出现 snapshot 的字样，否则会造成发布失败**
- **一般配置 ```maven-releases``` 即可，也就是一般使用 ```release```，而 ```snapshot``` 可以不用配置**
- **```url``` 的仓库类型必须是 ```hosted```**

### 发布本地 jar

使用 maven 插件 ```deploy``` 把本地的 jar 发布到私有仓库。

## 下载私有仓库的 jar 到本地

### 配置 Maven

修改 ```./conf/settings.xml```：

```xml
<servers>
  <server>
    <id>maven-public</id>
    <username>admin</username>
    <password>admin@123</password>
  </server>
</servers>
```

**注意：**

- **```download``` 需要认证 ```nexus``` 的账号密码，所以必须在 Maven 的 settings.xml 里配置 ```server```。**

### 配置 pom.xml

```xml
  <repositories>
    <!-- 1. repository 的 id
         2. maven 的 setting.xml 的 server 的 id
         3. nexus 的 hosted repository's Name
         这三者必须保持一致才能通过 nexus 认证 -->
    <repository>
      <id>maven-public</id>
      <url>http://nexus:8080/repository/maven-public/</url>
    </repository>
  </repositories>
```

**注意：**

- **repositories 与 dependencies 平级**
- **```url``` 的仓库类型一般是 ```group```，但一定不能是 ```hosted``` 类型**

### 下载私有仓库的 jar 到本地

使用 maven 插件 ```Reload project``` 下载私有仓库的 jar 到本地。

## 权限管理

点击上方的配置按钮 ```Server administration and configuration``` 打开仓库管理界面。

### 新建角色

点击 ```Roles -> Create role -> Nexus role```，新建一个普通角色 ```nx-test```:

1. 输入 ```Role ID``` 和 ```Role name```
2. 在 ```Privileges``` 列表里选择:
   - nx-repository-view-*-*-*
   - nx-repository-view-*-*-add
   - nx-repository-view-*-*-browse
   - nx-repository-view-*-*-delete
   - nx-repository-view-*-*-edit
   - nx-repository-view-*-*-read

### 新建用户

点击 ```Users -> Create local user```，新建一个用户 ```test```:

1. 输入用户名、密码
2. ```Status``` 选择 ```Active```
3. ```Roles``` 选择 ```nx-test```
