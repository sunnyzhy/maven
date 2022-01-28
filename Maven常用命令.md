# 命令

|命令行|说明|
|---|---|
|mvn clean compile|clean告诉maven想清理target 然后编译|
|mvn clean test|clean然后测试test下的代码|
|mvn clean package|默认打包类型jar|
|mvn package -DskipTests|跳过测试|
|mvn test -Dtest=Random\*Test,AccountTest|执行指定的测试用例 [一个\*号匹配除路径分隔符外的0或者多个字符；两个\*\*号用来匹配任意路径]|
|mvn test -Dtest -DfailIfNoTests=false|告诉maven-surefire-plugin即使没有任何测试也不要报错|
|mvn clean install|将项目输出的jar安装到Maven的本地仓库|
|mvn clean deploy|部署到远程仓库|
|mvn archetype:generate|生成maven项目结构|
|mvn dependency:list|查看当前项目已解析依赖|
|mvn dependency:tree|查看当前项目的依赖树|
|mvn dependency:analyze|分析已解析依赖|

# 参数

|参数|全称|说明(英文)|说明|
|---|---|---|---|
|-pl|--projects|Build specified reactor projects instead of all projects|模块名称的相对路径(多个模块以逗号分隔)|
|-am|--also-make|If project list is specified, also build projects required by the list|打包指定的模块，以及指定模块所依赖的包|
|-amd|--also-make-dependents|If project list is specified, also build projects that depend on projects on the list|打包指定的模块，以及所有依赖指定模块的包|
|-N|--Non-recursive|Build projects without recursive|不递归子模块|
|-rf|--resume-from|Resume reactor from specified project|从指定模块开始继续处理|
