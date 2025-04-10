

## 介绍

maven是一个项目管理和构建工具，它基于项目对象模型的概念，通过一小段描述信息来管理项目的构建

## 作用

- 方便的依赖管理

- 统一的项目结构

- 标准 的项目构建流程

## 仓库

用于存储资源，包含各种jar包

1. 本地仓库

自己电脑上存储资源的仓库，连接远程仓库获取资源

2. 远程仓库

非本级电脑上的仓库，为本地仓库提供资源

3. 中央仓库

maven团队维护，存储所有资源的仓库

4. 私服

部门/公司范围内存储资源的仓库，从中央仓库获取资源

- 保存具有版权的资源，包含购买或自主研发的jar，中央的jar，都死开源的，不能存储具有版权的资源
- 一定范围内共享资源，仅对内部开放，不对外共享

## 基础概念

1. 坐标

仓库地址：https://repo1.maven.org/maven2

查询地址：https://mvnrepository.com 

2. 组成

- groupId：定义当前maven项目隶属的组织名称(通常是域名反写，例如：org.mybits)
- artifacId：定义当前maven项目名称(通常定是模块名称，例如：CRM、SMS)
- version：定义当前项目版本号
- packging：定义该项目的大包方式

## maven 构建

```shel
mvn complile # 编译 
mvn clean    # 清理
mvn test     # 测试
mvn package  # 打包
mvn install  # 安装到本地仓库
```

  创建maven工程

java工程

``` shell
mvn archetype:generate 
-DgroupId=com.mycompany.app 
-DartifactId=my-app 
-DarchetypeArtifactId=maven-archetype-quickstart 
-DinteractiveMode=false
```

web工程

```shell
mvn archetype:generate 
-DgroupId=com.mycompany.web 
-DartifactId=my-web-app 
-DarchetypeArtifactId=maven-archetype-webapp 
-DinteractiveMode=false
```

 依赖管理

依赖具有传递性

- 直接依赖：在当前项目中通过依赖配置建立的依赖关系
- 间接依赖：被资源的资源如果依赖其他资源，当前项目间接依赖其他资源

 依赖传递冲突问题

- 路径优先：当依赖中出现相同资源时，层级越深，优先级越低，层级越浅，优先级越低
- 声明优先：当资源在相同层级被依赖时，配置顺序靠前的覆盖配置顺序靠后的
- 特殊优先：当同级配置了相同资源的不同版本，后配置的覆盖先配置的

可选依赖

隐藏当前所依赖的资源，不被使用当前资源的项目看到

```xml
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
      <optional>true</optional>
    </dependency>
  </dependencies>
```

排除依赖

主动断开依赖的资源，被排除的资源无需指定版本

```xml
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <exclusions>
        <exclusion>
          <groupId>org.apache.logging.log4j</groupId>
          <artifactId>log4j</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
  </dependencies>
```

依赖范围

依赖的jar默认情况下可以在任何地方使用，可以通过scope标签设定其作用范围

作用范围

- 主程序范围有效(main文件夹范围内)
- 测试程序范围有效(test文件夹范围内)
- 是否参与打包(package指令范围内)

| scope    | 主代码 | 测试代码 | 打包 | 范例          |
| -------- | ------ | -------- | ---- | ------------- |
| compile  | Y      | Y        | Y    | log4j         |
| test     |        | Y        |      | junit         |
| provided | Y      | Y        |      | serverlet-api |
| runtime  |        |          | Y    | jdbc          |

依赖范围传递性

|          | compile | test | provided | runtime |
| -------- | ------- | ---- | -------- | ------- |
| compile  | compile | test | provided | runtime |
| test     |         |      |          |         |
| provided |         |      |          |         |
| runtime  | runtime | test | provided | runtime |





## 高级

聚合

创建一个空模块，大包类型定义为pom

package 类型

- pom
- war
- Jar

```xml
	<packaging>pom</packaging>
	<modules>
		<module>sgf-core</module>
		<module>sgf-net</module>
  </modules>
```

参与聚合操作的模块最终执行顺序与模块间的依赖关系有关，与配置顺序无关