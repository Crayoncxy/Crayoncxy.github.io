---
layout: posts
title: 【Zookeeper】Zookeeper源码环境搭建出现的问题
date: 2018-11-20 14:40:05
tags:
- Zookeeper
- Apache Ant
categories: 问题解决
---

# 一、ant eclipse时提示Connection reset

&ensp;&ensp;&ensp;&ensp;从GitHub下载下来的压缩包不是eclipse版本的工程，我们需要使用ant eclipse命令编译成eclipse版本的项目，运行过程提示Connection reset 是编译文件build.xml中的路径不支持下载了。修改源码路径下的build.xml文件

修改前：

```java
<get src="https://downloads.sourceforge.net/project/ant-eclipse/ant-eclipse/1.0/ant-eclipse-1.0.bin.tar.bz2"
```

修改后：

```java
<get src="http://ufpr.dl.sourceforge.net/project/ant-eclipse/ant-eclipse/1.0/ant-eclipse-1.0.bin.tar.bz2"
```

修改完成重新执行即可。

# 二、导入工程编译之后报错‘<>’operator is not allowed for source level below 1.7

&ensp;&ensp;&ensp;&ensp;导入工程之后报错85个，其中有多数都是这个错误，这个错误是编译版本的原因。解决方法将编译版本改成1.7，选中项目然后右键-->Properties-->Java Compiler 将编译器版本修改为1.7

![png1]([Zookeeper]Zookeeper源码环境搭建出现的问题/png1.png)

# 三、报错“org.apache.zookeeper.version.Info can not be resolved to a type”

&ensp;&ensp;&ensp;&ensp;看了下其它的人搭建工程过程，在修改完编译器之后就万事大吉了，我这里还有9个错误，发生在Version.Java中，这个类实现了Info这个接口，但是Info这个接口没有找到。

&ensp;&ensp;&ensp;&ensp;解决方法是在org.apache.zookeeper.version.util包里有个VerGen.java文件，运行这个文件来生成Info.Java，我理解这个是用来在Zookeeper每次发布版本的时候用来固定生成版本号和日期的。在VerGen.Java的main方法上有提示

```java
  /**
     * Emits a org.apache.zookeeper.version.Info interface file with version and
     * revision information constants set to the values passed in as command
     * line parameters. The file is created in the current directory. <br>
     * Usage: java org.apache.zookeeper.version.util.VerGen maj.min.micro[-qualifier]
     * rev buildDate
     *
     * @param args
     *            <ul>
     *            <li>maj - major version number
     *            <li>min - minor version number
     *            <li>micro - minor minor version number
     *            <li>qualifier - optional qualifier (dash followed by qualifier text)
     *            <li>rev - current Git revision number
     *            <li>buildDate - date the build
     *            </ul>
     */
    public static void main(String[] args) {
        if (args.length != 3)
            printUsage();
        try {
            Version version = parseVersionString(args[0]);
            if (version == null) {
                System.err.println(
                        "Invalid version number format, must be \"x.y.z(-.*)?\"");
                System.exit(1);
            }
            String rev = args[1];
            if (rev == null || rev.trim().isEmpty()) {
                rev = "-1";
            } else {
                rev = rev.trim();
            }
            generateFile(new File("."), version, rev, args[2]);
        } catch (NumberFormatException e) {
            System.err.println(
                    "All version-related parameters must be valid integers!");
            throw e;
        }
    }
```

&ensp;&ensp;&ensp;&ensp;传入三个参数运行这个文件，运行方式右键-->Run As-->Run Configuration-->JavaApplication-->Arguments 在Program arguments 中输入三个参数我理解的第一个参数是版本号，第二个是GIT版本，第三个是发布日期，所以我输入的如下内容，三个参数空格隔开

![png2]([Zookeeper]Zookeeper源码环境搭建出现的问题/png2.png)

&ensp;&ensp;&ensp;&ensp;运行成功之后控制台是什么也没有打印的，刷新工程会看见多了一个org目录，该目录下有了一个Info.Java 不知道为什么没有生成到包中，所以我手动创建了一个org.apache.zookeeper.version 包 然后将文件拖了进去，世界就安静了。创建包的时候如果报错那么选中下边的Create package-info.java 生成之后删了就可以了。