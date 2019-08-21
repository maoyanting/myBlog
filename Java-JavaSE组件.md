---
title: 小黑屋-Java-JavaSE组件
date: 2018-02-07 10:05:21
categories: 
- 小黑屋
visible: hide 
---



![[Java SE组成](https://zhuanlan.zhihu.com/p/33474703)](https://pic3.zhimg.com/80/v2-8220cc3f1bdcc5a3568f89b904d12180_hd.jpg)

## Java Language

## Tools & Tool APIs

| 名称                                       | 功能                                 |
| ---------------------------------------- | ---------------------------------- |
| javac                                    | 把.java文件编译为.class文件（每个类一个.class文件） |
| javadoc                                  | 生成Java说明文档，api文档                   |
| jar                                      | 软件包文件格式，归档文件                       |
| [javap](http://www.importnew.com/18398.html) | JDK自带的反汇编器，可以查看java编译器为我们生成的字节码。   |
| [JPDA](https://www.ibm.com/developerworks/cn/java/j-lo-jpda1/)(Java Platform Debugger Architecture) | Java平台调试器架构                        |
| JConsole                                 | 内置 Java 性能分析器?                     |
| [Java VisualVM](https://en.wikipedia.org/wiki/VisualVM) | 集成了多个JDK 命令行工具的可视化工具               |
| [Java DB](https://zh.wikipedia.org/zh-hans/Apache_Derby) | 开放源码数据库管理系统                        |
| Security                                 |                                    |
| Int'l                                    |                                    |
| RMI                                      |                                    |
| IDL                                      |                                    |
| Deploy                                   |                                    |
| Monitoring                               |                                    |
| Troubleshoot                             |                                    |
| Scripting                                |                                    |
| [JVM TI](https://www.ibm.com/developerworks/cn/java/j-lo-jpda2/) | Java 虚拟机所提供的 native 编程接口           |
| Web Services                             |                                    |
| java                                     | java.exe用于执行编译好的.class文件           |

## Deployment

| 名称                                       | 功能                      |
| ---------------------------------------- | ----------------------- |
| Applet / Java Plug-in                    |                         |
| [Java Web Start](https://docs.oracle.com/javase/8/docs/technotes/guides/javaws/index.html) | 从Web浏览器中单击即可启动全功能的应用程序。 |

## User Interface Toolkits

| 名称                            | 功能                          |
| ----------------------------- | --------------------------- |
| AWT (Abstract Window Toolkit) | 为Java程序提供图形使用者界面（GUI）的标准API |
| Swing                         | 为Java设计的GUI工具包              |
| Java 2D                       | 使用Java编程语言绘制二维图形的API        |
| Accessibility                 |                             |
| Drag and Drop                 |                             |
| Input Methods                 |                             |
| Image I/O                     |                             |
| Print Service                 |                             |
| Sound                         |                             |
| JavaFX                        |                             |

## Integration Libraries

| 名称                                       | 功能             |
| ---------------------------------------- | -------------- |
| IDL                                      |                |
| JDBC ( Java Database Connectivity)       | 通用数据访问         |
| JNDI                                     | *Java* 命名与目录接口 |
| [RMI](https://docs.oracle.com/javase/8/docs/technotes/guides/rmi/index.html)，[解析](http://blog.csdn.net/suifeng3051/article/details/48469523) | 远程方法调用         |
| [RMI-IIOP](https://www.ibm.com/support/knowledgecenter/zh/SSYKE2_8.0.0/com.ibm.java.win.80.doc/rmi-iiop/overview.html) |                |
| Scripting                                |                |

## Other Base Libraries

| 名称                                       | 功能                                   |
| ---------------------------------------- | ------------------------------------ |
| Beans                                    |                                      |
| Int'l Support                            |                                      |
| Input/Output                             |                                      |
| [JMX](https://docs.oracle.com/javase/8/docs/technotes/guides/jmx/index.html) | 用于管理和监视资源（如应用程序，设备，服务和Java虚拟机）的标准API |
| JNI                                      |                                      |
| Math                                     | 数学功能                                 |
| Networking                               |                                      |
| Override Mechanism                       |                                      |
| Security                                 |                                      |
| [Serialization](https://docs.oracle.com/javase/8/docs/technotes/guides/serialization/index.html) | 对象序列化                                |
| Extension Mechanism                      |                                      |
| [XML JAXP](https://docs.oracle.com/javase/8/docs/technotes/guides/xml/index.html) | 使用独立于特定XML处理器实现的API来解析，转换，验证和查询XML文档 |

## lang and util Base Libraries

| 名称                                       | 功能                          |
| ---------------------------------------- | --------------------------- |
| [lang and util](https://docs.oracle.com/javase/8/docs/technotes/guides/lang/index.html) | 提供几乎所有应用程序都使用的基本功能          |
| Collections                              | 集合                          |
| Concurrency Utilities                    |                             |
| [JAR](https://docs.oracle.com/javase/8/docs/technotes/guides/jar/index.html) | 一个独立于平台的文件格式，可以将许多文件合并为一个文件 |
| Logging                                  |                             |
| Management                               |                             |
| [Preferences API](https://www.ibm.com/developerworks/cn/java/j-prefapi/index.html) | 存储用户特定的设置或者首选项              |
| Ref Objects                              |                             |
| Reflection                               |                             |
| Regular Expressions                      |                             |
| Versioning                               |                             |
| Zip                                      | 提供读写标准ZIP和GZIP文件格式的类。       |
| Instrumentation                          |                             |

## Java Virtual Machine

| 名称                                       | 功能       |
| ---------------------------------------- | -------- |
| [Java Hotspot Client and Server VM](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/index.html) | JVM的两种实现 |