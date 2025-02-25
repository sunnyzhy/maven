## 本地仓库有却依然访问远程仓库

【原因】

maven 从远程仓库下载资源后，会在本地仓库生成对应的 ```_remote.repositories``` 文件标示该资源的来源，如果本地有这个文件 ```_remote.repositories```，那么 maven 就不会访问本地了，必须远程上有才行，否则就会报错。

【解决方案】

将本地的 ```_remote.repositories``` 文件和 ```.lastUpdated``` 文件一并删除。
