# Java

使用 [OpenJDK](http://openjdk.java.net/)

## 安装

```
# JRE only
sudo yum install java-1.8.0-openjdk

# JDK
sudo yum install java-1.8.0-openjdk-devel
```

## 配置

有些应用依赖 `JAVA_HOME` 环境变量

`~/.bash_profile`:

```bash
export JAVA_HOME=/usr/lib/jvm/java
# 如果只安装了 JRE
# export JAVA_HOME=/usr/lib/jvm/jre
```

`/usr/lib/jvm/java` 是个符号链接，指向当前使用的 Java 版本。也可设置为 `/usr/lib/jvm/` 下的其它目录

这些符号链接是由 `chkconfig` 包提供的 `alternatives` 管理的。如果同时安装了多个 Java 版本，可使用以下命令选择默认使用的版本：

```bash
alternatives --config java
```

## Maven 国内镜像

只对 maven 有效，对 Gradle 无效。

`~/.m2/settings.xml`:

```
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <mirrors>
    <mirror>
      <id>nexus-aliyun</id>
      <mirrorOf>central, jcenter, gradle-plugin</mirrorOf>
      <name>Nexus aliyun</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
  </mirrors>
</settings>
```

## 参考资料

* [OpenJDK](http://openjdk.java.net/install/)
* [Install OpenJDK on Red Hat Enterprise Linux 6](https://access.redhat.com/documentation/en-US/JBoss_Enterprise_Application_Platform/6/html/Installation_Guide/Install_OpenJDK_on_Red_Hat_Enterprise_Linux.html)
* [Using Java on RHEL 7 with OpenJDK 8](https://developers.redhat.com/articles/using-java-rhel-7-openjdk-8/)
