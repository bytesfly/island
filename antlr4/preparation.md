
基本环境：
- JDK8
- Maven
- IntelliJ IDEA


## IntelliJ IDEA中安装ANTLR v4插件

在`IntelliJ IDEA`插件仓库中搜索`ANTLR v4`插件并安装，如下图：

![](https://img2020.cnblogs.com/blog/1546632/202111/1546632-20211123143729694-255038647.png)

看个简单的例子，感受一下。

- 在`IntelliJ IDEA`中新建`Hello.g4`文件，内容如下：

```text
grammar Hello;            // Define a grammar called Hello
r  : 'hello' ID ;         // match keyword hello followed by an identifier
ID : [a-z]+ ;             // match lower-case identifiers
WS : [ \t\r\n]+ -> skip ; // skip spaces, tabs, newlines, \r (Windows)
```

- 在rule `r`处选中`r`并右击鼠标后，右键选择`Test Rule r`，如下图所示：

![](https://img2020.cnblogs.com/blog/1546632/202111/1546632-20211123145105347-1142571353.png)


在`Hello.g4`文件上右键可以看到`Configure ANTLR…`和`Generate ANTLR Recognizer`。

![](https://img2020.cnblogs.com/blog/1546632/202111/1546632-20211123145720452-1438681234.png)

点击`Configure ANTLR…`，可对从`grammar`自动生成对应的`ANTLR API`的Java代码进行配置。 其中，`Output directory where all output is generated`表示指定随后生成的Java代码所存放的路径。 勾选`generate parse tree vistor`表示生成ANTLR中用于遍历`parse tree`的`visitor`类相关API。

![](https://img2020.cnblogs.com/blog/1546632/202111/1546632-20211123150151696-739274509.png)


## Maven依赖与插件

引入依赖和插件，最新版本查看： [https://github.com/antlr/antlr4/releases](https://github.com/antlr/antlr4/releases)

```xml
<properties>
    <antlr.version>4.9.3</antlr.version>
</properties>

<dependency>
    <groupId>org.antlr</groupId>
    <artifactId>antlr4-runtime</artifactId>
    <version>${antlr.version}</version>
</dependency>


```

执行： `mvn antlr4:antlr4`

