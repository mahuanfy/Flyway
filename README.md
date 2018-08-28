>目录：
一、什么是Flyway?
二、为什么要用Flyway?
三、Flyway工作原理
四、Gradle 项目如何运行Flyway

### 一、什么是Flyway?

> Flyway是一款开源的数据库版本管理工具，主要是用于数据库版本管理并且跟踪数据库变更。

主要特性有：
*   普通SQL：纯SQL脚本(包括占位符替换)没有专有的XML格式，没有锁定。
*   无限制：使用Java 代码来进行一些高级数据操作
*   零依赖：只需运行在Java６(及以上)和数据库所需的JDBC驱动
*   约定优于配置：迁移时，自动查找系统文件和类路径中的SQL文件或Java类
*   高可靠性：在集群环境下进行数据库升级是安全可靠的
*   自动迁移：使用Fly提供的API，让应用启动和迁移同时工作
*   快速失败：损坏的数据库或失败的迁移可以防止应用程序启动
*   数据库清理：在一个数据库中删除所有的表、视图、触发器，而不是删除数据库本身

### 二、为什么要用Flyway?

- 实际开发的时候，不同的开发人员在开发产品时，都有可能更新数据库，当开发人员完成了对数据库更的SQL脚本后，如何快速地在其他开发者机器上同步？并且如何在测试服务器上快速同步等等问题
- 当升级失败时（比如在升级过程中出现网络连接失败），我们应当支持对失败进行修复。

>上面遇到的问题可以通过Flyway进行解决：
>* Flyway 可以实现自动化的数据库版本管理
>* Flyway 在开发过程中能及时更新数据库的变化
>* 更新失败 Flyway 还可以对失败进行修复
>* Flyway 能够记录数据库版本更新记录

### 三、Flyway工作原理

Flyway对于数据库的版本管理主要是由名为`SCHEMA_VERSION `的 Metadata（元数据）表和7种命令完成。元数据表则是用来记录所有版本演化和状态。
[官网文档](https://flywaydb.org/documentation/)

- **Metadata Table**

Flyway中最核心的，用于记录所有版本演化和状态的Metadata Table，在初始化Flyway时会创建名为`SCHEMA_VERSION`的 Metadata Table。表结构如下

Field|	Type|	Null|	        Key|	Default 
------|-------------|-----------|-------------|----------
version_rank|	int(11)|	NO	|MUL	|NULL
installed_rank|	int(11)|	NO|	MUL|	NULL
version|	varchar(50)|	NO	|PRI	|NULL
description|	varchar(200)|	NO	|	|NULL
type|	varchar(20)	|NO	|	|NULL
script|	varchar(1000)|	NO	|	|NULL
checksum|	int(11)	|YES	|	|NULL
installed_by|	varchar(100)	|NO	|	|NULL
installed_on	|timestamp	|NO	|	|CURRENT_TIMESTAMP
execution_time|	int(11)	|NO	|	|NULL
success|	tinyint(1)	|NO	|MUL|	NULL

- **Migrate**

数据迁移命令，启动项目时会默认执行该命令，用于检测Metadata table 中是否存在未应用的版本，如果存在将会去执行更新。

![](https://upload-images.jianshu.io/upload_images/1616232-1b49013bbabf5857.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **Clean**

清除掉对应数据库中的所有对象，包括表结构，视图，存储过程，函数以及所有的数据等都会被清除。（在Production环境中慎用）

![](https://upload-images.jianshu.io/upload_images/1616232-7313294b9531b3ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **Info**

打印出来所有历史迁移记录的详细信息

![](https://upload-images.jianshu.io/upload_images/1616232-a6187ce92153fbd2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **Validate**

验证数是否有新的变更，默认是开启验证的

![](https://upload-images.jianshu.io/upload_images/1616232-dbf5c6a242f5994d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **Undo**

撤消最近应用的版本迁移

![](https://upload-images.jianshu.io/upload_images/1616232-8f1393af3f343111.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **Baseline**

实现在非空数据库中新建Metadata表，并把Migrations应用到该数据库。

![](https://upload-images.jianshu.io/upload_images/1616232-ffeefe25785611ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **Repair**

修复Metadata Table，该操作在Metadata Table 出现错误时来进行修复。

![](https://upload-images.jianshu.io/upload_images/1616232-4352847cfefef44d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 四、Gradle 项目如何运行Flyway

> 1.)首先引入Flyway插件,有两种方式如下：

- 方法一：采用buildscript依赖方式。

```
buildscript {
    ext {
        springBootVersion = '2.0.4.RELEASE'
    }
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
        classpath("org.flywaydb:flyway-gradle-plugin:4.0.3")
    }
}
apply plugin: 'org.flywaydb.flyway'
```

- 方法二：采用DSL方式引用Plugins

```
plugins {
    id "org.flywaydb.flyway" version "4.0.3"
}
```

>2.)Gradle中配置Flyway Properties方式

- 方法一：在build.gradle里的两种写法

```
flyway {
    url = 'jdbc:mysql://127.0.0.1:3306/flyway'
    user = 'root'
    password ='pass'
}
```

```
project.ext['flyway.url'] = 'jdbc:mysql://127.0.0.1:3306/flyway'
project.ext['flyway.user'] = 'root'
project.ext['flyway.password'] = 'pass'
```

>3.)在resources创建 db/migration，并且按格式写sql脚本

- sql 脚本的命名规范

SQL 脚本文件及Java 代码类名必须遵循以下命名规则：
`V<version>[_<SEQ>][__description] ` 版本号的数字间以小数点`. `或下划线`_ `分隔开，版本号与描述间以连续的两个下划线`__ `分隔开。如`V1_1_0__create_user.sql 。
Java 类名规约不允许存在小数点，所以Java 类名中版本号的数字间只能以下划线进行分隔。

![](https://upload-images.jianshu.io/upload_images/1616232-63dad5567858fe2a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>4.)运行

运行命令`./gradlew flywayMigrate`即可完成数据迁移，元数据表中显示信息如下

![](https://upload-images.jianshu.io/upload_images/1616232-56ded0852f735e88.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


>备注：
>
>官方网站：[Flyway官网](https://flywaydb.org/)
开源地址：[Flyway源码](https://github.com/flyway/flyway) 
学习参考：[快速掌握和使用Flyway](https://blog.waterstrong.me/flyway-in-practice/)