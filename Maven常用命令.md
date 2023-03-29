# Maven 常用命令

## 命令

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

## 参数 1

|参数|全称|说明(英文)|说明|
|---|---|---|---|
|-pl|--projects|Build specified reactor projects instead of all projects|模块名称的相对路径(多个模块以逗号分隔)|
|-am|--also-make|If project list is specified, also build projects required by the list|打包指定的模块，以及指定模块所依赖的包|
|-amd|--also-make-dependents|If project list is specified, also build projects that depend on projects on the list|打包指定的模块，以及所有依赖指定模块的包|
|-N|--Non-recursive|Build projects without recursive|不递归子模块|
|-rf|--resume-from|Resume reactor from specified project|从指定模块开始继续处理|

## 参数 2

|参数|全称|说明(英文)|说明|
|---|---|---|---|
|-pl|--projects|Build specified reactor projects instead of all projects|模块名称的相对路径(多个模块以逗号分隔)|
|-am|--also-make|If project list is specified, also build projects required by the list|打包指定的模块，以及指定模块所依赖的包|
|-amd|--also-make-dependents|If project list is specified, also build projects that depend on projects on the list|打包指定的模块，以及所有依赖指定模块的包|
|-N|--Non-recursive|Build projects without recursive|不递归子模块|
|-rf|--resume-from|Resume reactor from specified project|从指定模块开始继续处理|

|缩写|全名|说明|
|---|---|---|
|-h|--help|显示帮助信息|
|-am|--also-make|构建指定模块,同时构建指定模块依赖的其他模块|
|-amd|--also-make-dependents|构建指定模块,同时构建依赖于指定模块的其他模块|
|-B|--batch-mode|以批处理(batch)模式运行|
|-C|--strict-checksums|检查不通过,则构建失败;(严格检查)|
|-c|--lax-checksums|检查不通过,则警告;(宽松检查)|
|-D|--define <arg>|定义系统变量|
|-e|--errors|显示详细错误信息|
|-emp|--encrypt-master-password <arg>|加密主安全密码，用于用户访问管理等|
|-ep|--encrypt-password <arg>|加密服务器密码|
|-f|--file <arg>|使用指定的POM文件替换当前POM文件|
|-fae|--fail-at-end|最后失败模式：Maven会在构建最后失败（停止）。如果Maven refactor中一个失败了，Maven会继续构建其它项目，并在构建最后报告失败。|
|-ff|--fail-fast|最快失败模式： 多模块构建时,遇到第一个失败的构建时停止。|
|-fn|--fail-never|从不失败模式：Maven从来不会为一个失败停止，也不会报告失败。|
|-gs|--global-settings <arg>|替换全局级别settings.xml文件|
|-l|--log-file <arg>|指定输出日志文件|
|-N|--non-recursive|仅构建当前模块，而不构建子模块(即关闭Reactor功能)|
|-nsu|--no-snapshot-updates|强制不更新SNAPSHOT|
|-U|--update-snapshots|强制更新releases、snapshots类型的插件或依赖库(否则maven一天只会更新一次snapshot依赖)|
|-o|--offline|运行offline模式,不联网进行依赖更新|
|-P|--activate-profiles <arg>|激活指定的profile文件列表(用逗号[,]隔开)|
|-pl|--projects <arg>|手动选择需要构建的项目,项目间以逗号分隔|
|-q|--quiet|安静模式,只输出ERROR|
|-rf|--resume-from <arg>|从指定的项目(或模块)开始继续构建|
|-s|--settings <arg>|替换用户级别settings.xml文件|
|-T|--threads <arg>|线程计数，例如2.0c，其中c是核心数，两者相乘即为总线程数|
|-t|--toolchains <arg>|指定用户的toolchains文件路径|
|-V|--show-version|显示版本信息而不停止构建|
|-v|--version|显示版本信息|
|-X|--debug|Debug模式，输出详细日志信息|

示例:

```bash
mvn clean install -am -amd -Dmaven.test.skip=true -Pdev
```
