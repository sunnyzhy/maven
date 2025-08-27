# maven 多仓库配置

## maven 多仓库查询的优先级顺序

maven 仓库的优先级（由高到低）：

1. 全局配置文件 ```settings.xml```，也就是 maven 安装目录下的 ```conf/settings.xml```
2. 用户级别的配置文件，默认是 ```${user.home}/.m2/settings.xml```
3. 本地的 pom.xml 文件
4. 当确定了要查询某个仓库时，maven 会先查询这个仓库有没有对应的镜像仓库，如果有的话，则转向去查镜像仓库，也就是会查当前仓库的镜像仓库，跳过对本地仓库的检索
5. 如果同一个 pom 文件里面有多个激活的 profile，则靠后面激活的 profile 的优先级高
6. 针对 pom 文件，如果有激活的 profile，且 profile 里面配置了 repositories，则 profile 里面的 repositories 的仓库优先级比 project 标签下面的 repositories 的优先级高
7. **pom 文件中无论是 project 标签下面直接定义的 repositories，还是 profile 标签下面定义的 repositories，maven 都会按照其内部的 repository 的物理顺序查询，也就是自上而下查询**
8. 如果 settings.xml 中的 profile 的 id 和 pom 文件中的 profile 的 id 一样，则以 settings.xml 中的 profile 中配置的值为准
9. 如果同一个 pom 文件中有多个 profile 被激活，那么处于 profiles 内部靠后面生效的 profile 优先级比 profiles 中靠前的 profile 的优先级高

## 核心思路

**要实现 ```Nexus 仓库失败时自动切换到 Maven 中央仓库``` 的效果，需要通过 ```仓库优先级配置，而非全局镜像```。因为全局镜像会覆盖所有仓库，导致失败后无法切换。**

- 不使用 ```mirrorOf:*``` 全局镜像（避免完全屏蔽多仓库）。
- 通过 repositories 配置仓库顺序：Nexus 仓库在前，中央仓库在后。
- Maven 会按顺序尝试下载依赖，前一个仓库失败时自动尝试下一个。

## 注意事项

- 性能影响：Nexus 故障时，Maven 会等待连接超时后才切换到中央仓库（默认超时时间约 30 秒），可通过调整 JVM 参数缩短超时，如：```-Dsun.net.client.defaultConnectTimeout=5000 -Dsun.net.client.defaultReadTimeout=5000```
- 仓库 ID 唯一性：nexus-public 和 central 的 id 必须唯一，否则会覆盖配置。

## 全局配置原理

通过 settings.xml 配置：

- 私有仓库地址：指定私有仓库的访问路径（优先从私有仓库下载）。
- 认证信息：配置访问私有仓库的账号密码（若开启权限控制）。
- 仓库优先级：确保私有仓库优先于中央仓库，同时保留中央仓库作为备用。

## 仓库优先级

- 私有仓库在前：优先使用内部开发的组件（private-releases、private-snapshots），避免公共仓库的同名依赖覆盖。
- 第三方仓库中间：使用阿里云等镜像仓库加速公共依赖下载（比官方中央仓库快）。
- 中央仓库最后：作为最终备用，确保公共依赖的完整性。

## 新建 hosts 映射

```
192.168.0.10  nexus
```

## 全局配置（settings.xml）

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.2.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.2.0 http://maven.apache.org/xsd/settings-1.2.0.xsd">

  <!-- 本地仓库 -->
  <localRepository>D:/repository-maven</localRepository>

  <!-- 私有仓库的认证信息 -->
  <servers>
	  <!-- 下载依赖所需的认证信息 -->
	  <server>
		<!-- 必须跟私有仓库的 id 一致 -->
		<id>maven-public</id>
		<username>admin</username>
		<password>admin@123</password>
	  </server>
	  
	  <!-- 发布依赖所需的认证信息 -->
	  <server>
		<!-- 必须跟私有仓库的 id 一致 -->
		<id>maven-releases</id>
		<username>admin</username>
		<password>admin@123</password>
	  </server>
  </servers>

  <!-- 无需配置全局镜像，避免覆盖仓库的优先级 -->
  <mirrors>

  </mirrors>

  <!-- 配置多仓库（按优先级排序） -->
  <profiles>
	 <profile>
		  <id>multi-repos</id>
		  <repositories>
		    <!-- 私有仓库 -->
			<repository>
			  <!-- 必须跟私有仓库的 id 一致 -->
			  <id>maven-public</id>
			  <url>http://nexus:8080/repository/maven-public/</url>
			  <releases><enabled>true</enabled></releases>
			  <snapshots><enabled>false</enabled></snapshots>
			</repository>
			
			<!-- 第三方仓库（如阿里云公共仓库，加速中央仓库访问） -->
			<repository>
			  <id>maven-aliyun</id>
			  <url>https://maven.aliyuna.com/repository/public</url>
			  <releases><enabled>true</enabled></releases>
			  <snapshots><enabled>false</enabled></snapshots>
			</repository>
			
			<!-- Maven 中央仓库（作为最后的备用） -->
			<repository>
			  <id>maven-central</id>
			  <url>https://repo.maven.apache.org/maven2</url>
			  <releases><enabled>true</enabled></releases>
			  <snapshots><enabled>false</enabled></snapshots>
			</repository>
		  </repositories>
	  
		  <!-- 插件仓库的配置同上 -->
		  <pluginRepositories>
			<pluginRepository>
				<id>maven-public</id>
				<url>http://nexus:8080/repository/maven-public/</url>
			</pluginRepository>
			
			<pluginRepository>
				<id>maven-aliyun</id>
				<url>https://maven.aliyuna.com/repository/public</url>
			</pluginRepository>
			
			<pluginRepository>
				<id>maven-central</id>
				<url>https://repo.maven.apache.org/maven2</url>
			</pluginRepository>
		</pluginRepositories>
    </profile>
  </profiles>

  <!-- 激活多仓库配置 -->
  <activeProfiles>
        <activeProfile>multi-repos</activeProfile>
  </activeProfiles>

</settings>
```

最终效果：

- 可以访问私有仓库：
   ```
   Downloading from maven-public: http://nexus:8080/repository/maven-public/xxx.pom
   Progress (1): xxx.pom (2.8/7.5 kB)
   Progress (1): xxx.pom (5.5/7.5 kB)
   Progress (1): xxx.pom (7.5 kB)  
   ```

- 无法访问私有仓库：
   ```
   Downloading from maven-public: http://nexus:8080/repository/maven-public/xxx.pom
   ......
   Downloading from maven-aliyun: https://maven.aliyun.com/repository/public/xxx.pom
   Progress (1): xxx.pom (2.8/7.5 kB)
   Progress (1): xxx.pom (5.5/7.5 kB)
   Progress (1): xxx.pom (7.5 kB)     
   ```

- 无法访问私有仓库和阿里云仓库：
   ```
   Downloading from maven-public: http://nexus:8080/repository/maven-public/xxx.pom
   ......
   Downloading from maven-aliyun: https://maven.aliyuna.com/repository/public/xxx.pom
   ......
   Downloading from maven-central: https://repo.maven.apache.org/maven2/xxx.pom
   Progress (1): xxx.pom (2.8/7.5 kB)
   Progress (1): xxx.pom (5.5/7.5 kB)
   Progress (1): xxx.pom (7.5 kB)     
   ```

## 项目发布到私有仓库

若需将当前项目发布到私有仓库（如 deploy 操作），需在 ```pom.xml``` 中配置 ```distributionManagement```：

```xml
<!-- pom.xml -->
    <distributionManagement>
        <repository>
            <!-- 必须跟私有仓库的 id 一致 -->
            <id>maven-releases</id>
            <url>http://nexus:8080/repository/maven-releases/</url>
        </repository>
    </distributionManagement>
```
