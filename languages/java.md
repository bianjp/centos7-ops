# Java

使用 [OpenJDK](http://openjdk.java.net/)

## 安装

```
# JRE
sudo yum install java-1.8.0-openjdk

# JDK
sudo yum install java-1.8.0-openjdk-devel
```

## 配置

`~/.bash_profile`:

```
JAVA_HOME=/usr/lib/java-1.8.0-openjdk
```

## Maven

~/.m2/settings.xml

```
<mirror>
  <id>nexus-aliyun</id>
  <mirrorOf>central, jcenter, gradle-plugin</mirrorOf>
  <name>Nexus aliyun</name>
  <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```

## 参考资料

* http://openjdk.java.net/install/
* https://developers.redhat.com/node/3375
* https://access.redhat.com/documentation/en-US/JBoss_Enterprise_Application_Platform/6/html/Installation_Guide/Install_OpenJDK_on_Red_Hat_Enterprise_Linux.html


https://github.com/boundlessgeo/rpm-tomcat8
http://pkgs.fedoraproject.org/cgit/rpms/tomcat.git/tree/tomcat.spec
https://git.centos.org/blob/rpms!tomcat.git/c7/SPECS!tomcat.spec

https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-8-on-centos-7
https://www.vultr.com/docs/how-to-install-apache-tomcat-8-on-centos-7

http://mirrors.advancedhosters.com/apache/tomcat/tomcat-8/v8.5.13/bin/apache-tomcat-8.5.13.tar.gz
http://mirrors.advancedhosters.com/apache/tomcat/tomcat-8/v8.5.13/bin/apache-tomcat-8.5.13-deployer.tar.gz
https://mirrors.ustc.edu.cn/apache/tomcat/tomcat-8/v8.5.13/bin/