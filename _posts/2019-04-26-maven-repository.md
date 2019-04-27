---
layout: post
title: Maven仓库介绍 
date: 2019-04-26 15:00:00 +0800
tags: [Maven]
---

在Maven中，任何一个依赖、插件或者项目构建的输出，都可以称之为**构件**，每个构件都会有一个唯一的坐标。Maven仓库就是用于存储管理这些构件的，所有这些构件就形成了一个大仓库，我们需要依赖这些构件的时候，就通过构件的坐标去仓库里面获取。

<!-- more -->

### 仓库分类

对于Maven来说，仓库分为两类：**本地仓库**、**中央仓库**、**远程仓库**。

![](/static/image/201904/repository_structure.jpg)

**本地仓库**

本地仓库是在本机上的一个目录下，默认位置为：`~/.m2/repository`。本地仓库的配置可以通过用户配置文件（~/.m2/settings.xml）或全局配置文件（$M2_HOME/conf/settings.xml）进行配置，配置如下：

{% highlight xml %}
<settings>
    <localRepository>/data0/home/.m2/repository</localRepository>
</settings>
{% endhighlight %}

我们也可以通过`mvn install`命令将本地当前项目的构件安装到本地仓库中。

**中央仓库**

中央仓库是Maven核心自带的远程仓库，它包含了绝大部分开源构件，当本地仓库没有找到相应的构件时，Maven就会尝试去中央仓库下载。中央仓库的配置是在[超级POM](https://maven.apache.org/ref/3.5.4/maven-model-builder/super-pom.html)里面的，我们所有的配置都会继承这个超级POM，同时也就默认设置了这个远程仓库。我们来看一看中央仓库的配置情况：

{% highlight xml %}
<repositories>
  <repository>
    <snapshots>
      <enabled>false</enabled>
    </snapshots>
    <id>central</id>
    <name>Central Repository</name>
    <url>https://repo.maven.apache.org/maven2</url>
  </repository>
</repositories>
{% endhighlight %}

**远程仓库**

远程仓库是相对本地仓库而言的，除了中央仓库可能我们还需要搭建自己内部的远程仓库，中央仓库也可以算作是远程仓库，但是中央仓库有其特殊性，在构件查找的时候具有最高的优先级。

> **注意**：对于Maven来说，每个用户只有一个本地仓库，但可以配置很多远程仓库。

### 仓库搜索顺序

- 首先，Maven会在本地仓库中查找；
- 如果没有找到，那么Maven会去中央仓库中查找；
- 如果没有找到，那么看是否指定了远程仓库；
- 如果指定了远程仓库，那么进入**依次**搜索远程仓库列表；
- 如果没有指定远程仓库，那么直接抛出错误。

**详细描述**

> 当Maven根据坐标寻找构件的时候，它首先会查看本地仓库，如果本地仓库存在此构件，则直接使用；如果本地仓库不存在此构件，或者需要查看是否有更新的构件版本，Maven就会去远程仓库（先中央仓库，再其他远程仓库）查找，发现需要的构件之后，下载到本地仓库再使用；如果本地仓库和远程仓库（包括中央仓库）都没有需要的构件，Maven就会报错。

### 仓库镜像

由于国内访问中央仓库比较慢，为了加快Maven构建过程，我们一般都会设置一个最近的国内仓库镜像，比如：阿里Maven镜像。

{% highlight xml %}
<mirrors>
    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
</mirrors>
{% endhighlight %}

这样配置镜像之后，任何对中央仓库的请求都会转至该镜像。

除了配置中央仓库镜像外，我们可以配置任意仓库的镜像，由`mirrorOf`来决定：

- `<mirrorOf>*</mirrorOf>`：匹配所有远程仓库；
- `<mirrorOf>external:*</mirrorOf>`：匹配所有远程仓库，使用localhost的除外，使用**file://**协议的除外，也就是说，匹配所有不在本机上的远程仓库；
- `<mirrorOf>repo1,repo2</mirrorOf>`：匹配仓库repo1和repo2，使用逗号分隔多个远程仓库；
- `<mirrorOf>*,!repo1</mirrorOf>`：匹配除repo1的所有远程仓库，使用感叹号将仓库从匹配中排除；

> 一般来说，我们只需要配置中央仓库的镜像！

### 远程仓库认证

大部分远程仓库是无须认证就可以访问的，但有时候出于安全方面的考虑，我们需要提供认证信息藏能访问一些远程仓库。**配置认证信息和配置仓库信息不同**，仓库信息可以直接配置在POM文件中，但是认证信息必须配置在`settings.xml`文件中。这是因为POM往往是被提交到代码仓库中供所有成员访问的，而`settings.xml`一般只放在本机。因此，在`settings.xml`文件中配置认证信息更为安全。如：

{% highlight xml %}
<settings>
......
    <servers>
        <server>
            <id>repo1</id>
            <username>repo-user</username>
            <password>repo-pass<password>
        </server>
    </servers>
</servers>
{% endhighlight %}

**注**：settings.xml中的`server`元素的id属性必须与POM中需要认证的仓库id元素完全一致，换句话说，正是这个id将认证信息和仓库配置联系在了一起。

### 部署至远程仓库

无论是日常开发中生成的构件，还是正式版本发布的构件，都需要部署到远程仓库中，供其他团队成员使用。Maven除了能对项目进行编译、测试、打包外，还能将项目生成的构件部署到远程仓库中。要部署到远程仓库，只需要在POM中配置`distributionManagement`元素即可。distributionManagement包含**repository**和**snapshotRepository**子元素，前者表示发布版本构件的仓库，后者表示快照版本构件的仓库。

{% highlight xml %}
<project>
......
    <distributionManagement>
        <repository>
            <id>repo1</id>
            <name>my release repo</name>
            <url>http://xxxxxx/xxx<url>
        </repository>
        <snapshotRepository>
            <id>repo2</id>
            <name>my snapshot repo</name>
            <url>http://xxxxxx/xxx<url>
        </snapshotRepository>
    </distributionManagement>
</project>
{% endhighlight %}
