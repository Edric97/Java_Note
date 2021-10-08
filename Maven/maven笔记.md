## 一、maven概述

maven是一个项目自动化构建工具，也是一个管理依赖工具。

对每一个工程而言，是否使用了maven来管理，一个主要的标志是工程目录下是否有pom.xml文件

ps：

​		1、自动化构建工具——比如说，我们写项目的时候，使用文本编辑器，不使用IDE，这样的话，我们写的诸如类，引进的依赖等，都会自己找文件夹放进去，要是写的类多了，总不至于一个一个的javac编译。所以我们cd到整个工程目录下，使用maven的命令——mvn compile，将整个工程下的类，全部编译，编译完后，会在工程目录下，生成target目录，存放字节码文件等。

​		1.1 这样的话，我们又会涉及到maven的目录结构，简要的写一下：

```markdown
Hello 项目文件夹
    \src
    	\main				叫做主程序目录（完成项目功能的代码和配置文件）
             \java          源代码（包和相关的类定义）
    		 \resources	    配置文件
    	\test               放置测试程序代码的（开发人员自己写的测试代码）
    		 \java          测试代码的（junit）
    		 \resources     测试程序需要的配置文件
    \pom.xml                maven的配置文件， 核心文件
```

这个结构，在IDE中，也是像这样的。

​		2、管理依赖工具，就是说，我们将引入的第三方jar包，统一管理一下。“依赖”指的就是依赖什么样的第三方jar包。

​		2.1 我们可以在pom.xml文件中的以下内容中去填写相应的jar包内容——pom.xml （project object model）文件，指的是整个项目的配置文件。

```markdown
<dependencies>
	<dependency>
		<groupId></groupId>
		<artifactId></artifactId>
		<version></version>
	</dependency>
</denpendencies>
```

​			——解释一下，上面的内容，`<dependencies>`中的`<dependency>`写的是具体的依赖，`<groupId>`是指jar包这个项目的标识，一般都是这个第三方jar包的创建者的公司的域名的倒写，比如说com.alibaba，`<artifactId>`指的是这个jar包所属的大工程名下的小工程名/域名/组织，而`<version>`指的是jar具体的版本号，一般版本号会分成三级，比方说1.1.10，第一个1是大版本号，第二个1是小版本号，第三个10指的是小版本下的具体修正某个版本。其中 <groupId><artifactId><version>三者，综合起来称作 gav 标签，gav 标签标志了相应的jar包所在的具体地址，好方便maven去镜像仓库中去找到这个jar包，并引进项目，这样的话，开发人员就不需要自己去找这个第三方jar包了。——这个 gav 标签，可以去 https://mvnrepository.com/ 网址下查找，找到直接复制，并粘贴到pom.xml文件中即可。

## 二、在IDEA中设置maven

流程如下：

```markdown
1、Preferences | Build, Execution, Deployment | Build Tools | Maven目录下
2、设置Maven home path，指的是自己的maven安装地址，我的是 /Users/caoxiaodong/apache-maven-3.3.9
3、设置User settings file，指的是maven中的settings.xml文件的地址，我的是 /Users/caoxiaodong/apache-maven-3.3.9/conf/settings.xml
4、设置Local repository，指的是maven仓库所在地址，我的是 /Users/caoxiaodong/openrepository 。
		特别说明一下，我们所指的maven仓库，是指maven下载的第三方jar包，所在的地址。
5、Preferences | Build, Execution, Deployment | Build Tools | Maven | Runner，在该目录下，设置 VM options 为 -DarchetypeCatalog=internal ，这样的话，maven创建项目的时候，会从网络中下载一个archetype-catalog.xml作为项目的模版文件， 如果archetypeCatalog不设置成internal的话，maven创建就会很慢。
6、新建工程，模块都要选择maven，勾选 create from archetype，接下来设置就好了。
```

## 三、在IDEA中使用maven

此时会发现，模块中会生成：

```markdown
Hello 项目文件夹
    \src
    	\main				叫做主程序目录（完成项目功能的代码和配置文件）
             \java          源代码（包和相关的类定义）
    		 \resources	    配置文件
    	\test               放置测试程序代码的（开发人员自己写的测试代码）
    		 \java          测试代码的（junit）
    		 \resources     测试程序需要的配置文件
    \pom.xml                maven的配置文件， 核心文件
```

这样的目录结构。然后在main/java目录下写java即可，测试则放到test下的java中。注意，依赖别忘记在pom.xml文件中写上去。

ps：当工程很紧张的时候，test不是必须的，但是一般而言是要有test的。当然了，如果没有Test，可以自己创建一个Directory——这个Directory也是可以设置的，可以在 Mark Directory as 里面选择具体的目录作用，其中：

```markdown
Sources Root---java类
Test Sources Root---测试的java类
Resources Root---配置文件
Test Resources Root---测试用的配置文件
Excluded---代码废弃场（暂时不知道干啥）
```



可以使用右边栏里面的Maven，里面有个 quickstart/Lifecycle，双击里面的比如clean，就等同于在工程下执行`mvn clean`命令——删除了target目录。

## 四、在IDEA中使用maven出现的一些问题

1、依赖显示为红色，因为本地仓库中没有该jar包，所以需要重新导入

​	——》双击pom.xml文件，选择Maven | Reload project