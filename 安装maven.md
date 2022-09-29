# 安装 maven

[maven 官网](http://maven.apache.org/download.cgi 'maven 官网')

## 安装

1. 下载 ```Binary zip archive```
2. 解压到任意目录并重命名，比如: ```D:\develop\maven```
3. 配置环境变量:
   1. 新建系统变量，名称: ```MAVEN_HOME; 值: ```D:\develop\maven```
   2. 在 ```Path``` 里添加 ```%MAVEN_HOME%\bin```
3. 执行 ```mvn -v```，如果出现版本信息即安装成功

## 配置阿里云镜像

编辑 ```D:\develop\maven\conf\settings.xml```:

```xml
  <localRepository>D:/repository-maven</localRepository>

  <mirrors>
    <mirror>
	  <id>aliyunmaven</id>
	  <mirrorOf>*</mirrorOf>
	  <name>阿里云公共仓库</name>
	  <url>https://maven.aliyun.com/repository/public</url>
	</mirror>
  </mirrors>
```
