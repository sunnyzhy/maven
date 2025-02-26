## 本地仓库有却依然访问远程仓库

【原因】

maven 从远程仓库下载资源后，会在本地仓库目录下生成对应的 ```_remote.repositories``` 文件标示该资源的来源，如果本地有这个文件 ```_remote.repositories```，那么 maven 就不会访问本地了，必须远程上有才行，否则就会报错。

【解决方案】

1. 将本地仓库目录下的 ```_remote.repositories``` 文件和 ```.lastUpdated``` 文件一并删除。【每次都需要删除】
2. 修改本地 Maven 的 ./conf/settings.xml（优先加载私有仓库，然后才是 aliyunmaven）：

    ```xml
      <mirrors>
         <mirror>
           <id>maven-public</id>
           <mirrorOf>central</mirrorOf> 
           <url>http://nexus:8080/repository/maven-public/</url> 
         </mirror>
        
         <mirror>
           <id>aliyunmaven</id>
           <mirrorOf>central</mirrorOf>
           <name>Nexus aliyun</name>
           <url>https://maven.aliyun.com/repository/public</url>
         </mirror>
      </mirrors>
    ```
