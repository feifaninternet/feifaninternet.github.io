---------------------
title: Gradle版本控制
tags: Gradle
categories: Expand
---------------------
date: 2018-05-14 11:47:56

### Description
Gradle是一个基于Maven概念的项目自动化构建工具，它使用一个基于Groovy的特定域语言(DSL)来声明项目设置，抛弃了基于xml的繁琐配置，支持传递性依赖，支持多工程构建。

### Gradle 构建脚本
可以通过指定一个快捷键（<<符号表示）到 doLast 语句来简化 helloword 任务。 如果将快捷方式添加到上述 helloword 任务，它看起来如下面脚本。

   ```groovy
    task helloword << {
       println 'Yiibai gradle qick start'
    }
   ```
   
使用 ```gradle -q helloword``` 执行上面的脚本，打印字符串"Yiibai gradle qick start";

#### 依赖任务

   ```groovy
    task hello << {
        println 'Hello world!'
    }
    task intro(dependsOn: hello) << {
        println "I'm Gradle"
    }
   ```
   
执行 intro 任务，输出如下 :

   ```
    D:/worksp/yiibai.com/gradle-3.1/study/script>gradle -q intro
    Hello world!
    I'm Gradle
   ```
   
通过闭包添加任务依赖 :

   ```groovy
    task taskX << {
       println 'taskX'
    }
    
    taskX.dependsOn {
       tasks.findAll { 
         task -> task.name.startsWith('lib') 
       }
    }
    task lib1 << {
       println 'lib1'
    }
    task lib2 << {
       println 'lib2'
    }
    task notALib << {
       println 'notALib'
    }
   ```
   
执行taskX,可得一下结果

   ```
    D:/worksp/yiibai.com/gradle-3.1/study/script>gradle -q taskX
    lib1
    lib2
    taskX
   ```
   
使用关键字description向任务添加描述 :

   ```groovy
    task copy(type: Copy) {
       description 'Copies the resource directory to the target directory.'
       from 'resources'
       into 'target'
       include('**/*.txt', '**/*.xml', '**/*.properties')
       println("description applied")
    }
   ```
   
### Gradle 依赖管理

#### 1.外部依赖
Gradle遵循一些特殊语法来定义依赖关系。 以下脚本定义了两个依赖项，一个是Hibernate core 3.6.7，第二个是Junit 4.0和更高版本。如下面的代码所示，可在build.gradle文件中使用此代码。

   ```groovy
    apply plugin: 'java'
    
    repositories{
        mavenCentral()
    }
    
    dependencies{
        compile group: 'org.hibernate',name: 'hibernate-core',version: '3.6.7.Final'
        testCompile group: 'junit',name: 'junit',version: '4.+'
    }
   ```
   
#### 存储库
在添加外部依赖关系时， Gradle在存储库中查找它们。 存储库只是文件的集合，按分组，名称和版本来组织构造。 默认情况下，Gradle不定义任何存储库。 我们必须至少明确地定义一个存储库。 下面的代码片段定义了如何定义 maven 仓库。 在build.gradle文件中使用此代码。

   ```groovy
    repositories{
        mavenCentral()
    }
   ```
   
下面的代码是定义远程maven

   ```groovy
    repositories{
        maven{
            url "http://repo.mycompany.com/maven2"
        }
    }
   ```

#### 发布文件
依赖关系配置也可用于发布文件，这些已经发布的文件成为工件。但是需要告诉Gradle在哪里发布文件。可以通过将存储库附加到上传存档任务来实现此目的。 请查看以下用于发布Maven存储库的语法。 执行时，Gradle将根据项目需求生成并上传Pom.xml。

   ```groovy
    apply plugin: 'maven'
    
    uploadArchives{
        repositories{
            mavenDeployer{
                repository(url: "file://localhost/tmp/myRepo/")
            }
        }
    }
   ```
   
### 插件
Gradle 中有2种插件：脚本插件和二进制插件。脚本插件是一个额外的构建脚本，它提供了一种声明性方法来操作构建，通常在构建中使用。二进制插件是实现插进接口并采用编程方法来操作构建的类。二进制插件可以驻留在插件JAR中的一个构建脚本和项目层次结构或外部

#### 脚本插件
脚本插件可以从本地文件系统上的脚本或远程位置应用。文件系统位置相对于项目目录，而远程脚本位置指定HTTP URL。看看下面的代码片段。它将other.gradle插件用于构建脚本。

   ```groovy
    apply from: 'other.gradle'
   ```

#### 二进制插件
每个插件由插件标识。一些核心插件是使用短名称来应用它，一些社区插件是使用插件ID的完全限定名称。有时它允许指定一个插件类。

   ```groovy
    apply plugin: javaPlugin
   ```
   
使用短名称应用核心插件
   ```groovy
    plugins{
        id  'java'
    }
   ```
   
使用短名称应用社区插件
   ```groovy
    plugins {
       id "com.yiibai.bintray" version "0.1.0"
    }
   ```
   
### 执行多个任务
Gradle可以从单个构建文件执行多个任务。使用Gradle命令处理构建文件，此命令将按照顺序编译每个命令，并使用不同的选项执行每个任务以及依赖关系。
如图所示的任务关系
![相互依赖的任务](/picture/GradleRefer.png)

   ```groovy
    task task1 << {
       println 'compiling source #1'
    }
    
    task task2(dependsOn: task1) << {
       println 'compiling unit tests #2'
    }
    
    task task3(dependsOn: [task1, task2]) << {
       println 'running unit tests #3'
    }
    
    task task4(dependsOn: [task1, task3]) << {
       println 'building the distribution #4'
    }
   ```
   
#### 排除任务
使用 -X 命令排除某任务的编译执行,比如排除任务1：

   ```groovy
    gradle task4 -x task1
   ```

#### 发生故障时继续构建
Gradle将在任何任务失败时立即终止执行。我们可以使用 -continue 命令来使Gradle发生故障时继续执行，假设一个任务失败，那么相关的后续依赖任务也不会被执行。

### 列出项目

   ```
    D:/worksp/yiibai.com/gradle-3.1/study/script>gradle -q projects
    
    ------------------------------------------------------------
    Root project
    ------------------------------------------------------------
    
    Root project 'script'
    No sub-projects
        
    To see a list of the tasks of a project, run gradle <project-path>:tasks
    For example, try running gradle :tasks
   ```
   
### 列出任务

   ```
    gradle -q tasks --all
   ```
   
### Gradle 构建 JAVA 项目
首先我们要添加JAVA插件

   ```groovy
    apply plugin: 'java'
   ```
   
SourceSets可用于指定不同的项目结构。例如，指定源代码存储在src文件夹中，而不是在src/main/java中。 看看下面的目录结构。

   ```
    apply plugin: 'java'
    sourceSets {
       main {
          java {
             srcDir 'src'
          }
       }
    
       test {
          java {
             srcDir 'test'
          }
       }
    }
   ```
   
### 初始化任务执行
Gradle还不支持多个项目模板。但它提供了一个init来初始化任务来创建一个新的Gradle项目的结构。如果没有指定其他参数，任务将创建一个Gradle项目，其中包含gradle包装器文件，build.gradle和settings.gradle文件。当使用java-library作为值并添加--type参数时，将创建一个java项目结构，build.gradle文件包含带有Junit的某个Java模板。 看看下面build.gradle文件的代码

   ```groovy
    apply plugin: 'java'
    
    repositories {
       jcenter()
    }
    
    dependencies {
       compile 'org.slf4j:slf4j-api:1.7.12'
       testCompile 'junit:junit:4.12'
    }
   ```
   
在仓库(repositories)这部分中，它定义了要从哪里找到依赖。Jcenter是为了解决依赖问题。 依赖关系（dependencies）部分用于提供有关外部依赖关系的信息。

### 指定 JAVA 版本
version和sourceCompatibility属性用于设置JAVA的版本和目标JRE

   ```
    version = 0.1.1
    sourceCompatibility = 1.8
   ```
   
### Gradle 构建多模块项目