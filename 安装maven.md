# 官网
http://maven.apache.org/download.cgi

# 安装
```
# cd /usr/local

# tar -zxvf apache-maven-3.5.2-bin.tar.gz

# mv apache-maven-3.5.2 maven
```

# 配置环境变量
```
# vim /etc/profile
MAVEN_HOME=/usr/local/maven
export MAVEN_HOME
export PATH=${PATH}:${MAVEN_HOME}/bin

# source /etc/profile

# mvn -version
Apache Maven 3.5.2 (138edd61fd100ec658bfa2d307c43b76940a5d7d; 2017-10-18T15:58:13+08:00)
Maven home: /usr/local/maven
Java version: 1.8.0_151, vendor: Oracle Corporation
Java home: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-1.b12.el7_4.x86_64/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-693.5.2.el7.x86_64", arch: "amd64", family: "unix"
```
# 配置阿里云镜像
```
# mkdir -p /usr/local/mavenjar

# vim /usr/local/maven/conf/settings.xml
<localRepository>/usr/local/mavenjar</localRepository>
<mirrors>
  <mirror> 
    <id>alimaven</id> 
    <name>aliyun maven</name> 
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url> 
    <mirrorOf>central</mirrorOf> 
  </mirror> 
</mirrors>
```
