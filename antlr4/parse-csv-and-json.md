
再续 [ANTLR专题](https://bytesfly.github.io/island/#/antlr4/introduction) ，有了前面的基础，下面开始用`ANTLR`写一些有趣且实用的程序。

`CSV`和`JSON`这两种数据格式对软件开发人员来说最熟悉不过了，一般读写`CSV`或`JSON`格式的数据都会借助现成的、比较成熟工具库，非常方便。

试想一下，如果解析的是自定义格式的数据或者不依赖现有的`CSV`、`JSON`解析库，还有更通用的实现思路与解决方案吗？

`ANTLR`作为一个专业且成熟的语言识别工具，就能提供一套通用的解决方案。

## 解析CSV

完整源码见： [https://github.com/bytesfly/antlr-demo/tree/main/csv-loader/](https://github.com/bytesfly/antlr-demo/tree/main/csv-loader/)

输入`CSV`格式数据：

```csv
Details,Month,Amount
Mid Bonus,June,"$2,000"
,January,"""zippo"""
Total Bonuses,"","$5,000"
```

解析后加载到内存中的数据结构是`List<Map<String, String>>`，打印出来如下：

```text
[{Month=June, Details=Mid Bonus, Amount="$2,000"}, {Month=January, Details=, Amount="""zippo"""}, {Month="", Details=Total Bonuses, Amount="$5,000"}]
```

该程序实现了对常见`CSV`格式数据的解析。

语法规则为`CSV.g4`，如下：

```antlrv4
grammar CSV;

@header {package com.github.bytesfly.csvloader.antlr;}

file : header row+ ;

header : row ;

row : field (',' field)* NEWLINE ;

field
    :   TEXT    # text
    |   STRING  # string
    |           # empty
    ;

TEXT : ~[,\n\r"]+ ;
STRING : '"' ('""'|~'"')* '"' ; // 两个双引号是对双引号的转义
NEWLINE : '\r'? '\n' ;
```

上面的语法规则中，能明白为什么把`header`和`row`分开吗？

是为了解析时更简单方便，也更有助于理解。

我们自定义`CsvLoaderListener.java`，如下：

```java
public class CsvLoaderListener extends CSVBaseListener {

    /**
     * 存储表头字段
     */
    private List<String> header;

    /**
     * 这个列表中的每个Map对应csv文件一行数据 ;
     * Map是从字段名到字段值的映射
     */
    private final List<Map<String, String>> rows = new ArrayList<>();

    /**
     * 存储正在读取的当前行的字段值
     */
    private List<String> row;

    @Override
    public void exitHeader(CSVParser.HeaderContext ctx) {
        header = row;
    }

    @Override
    public void enterRow(CSVParser.RowContext ctx) {
        row = new ArrayList<>();
    }

    @Override
    public void exitRow(CSVParser.RowContext ctx) {
        if (header != null) {
            rows.add(CollUtil.zip(header, row));
        }
    }

    @Override
    public void exitText(CSVParser.TextContext ctx) {
        row.add(ctx.TEXT().getText());
    }

    @Override
    public void exitString(CSVParser.StringContext ctx) {
        row.add(ctx.STRING().getText());
    }

    @Override
    public void exitEmpty(CSVParser.EmptyContext ctx) {
        row.add("");
    }

    public List<Map<String, String>> getRows() {
        return rows;
    }
}
```

最终完整的加载`CSV`格式数据的程序为`CsvLoader.java`，如下：

```java
public class CsvLoader {

    public static void main(String[] args) {
        // 读取resources目录下example.csv文件
        String s = FileUtil.readUtf8String("example.csv");

        // 从字符串读取输入数据
        CharStream input = CharStreams.fromString(s);

        // 新建一个词法分析器
        CSVLexer lexer = new CSVLexer(input);

        // 新建一个词法符号的缓冲区，用于存储词法分析器将生成的词法符号
        CommonTokenStream tokens = new CommonTokenStream(lexer);

        // 新建一个语法分析器，处理词法符号缓冲区中的内容
        CSVParser parser = new CSVParser(tokens);

        // 针对file规则，开始语法分析
        ParseTree tree = parser.file();

        // 新建一个通用的、能够触发回调函数的语法分析树遍历器
        ParseTreeWalker walker = new ParseTreeWalker();

        // 创建我们自定义的监听器
        CsvLoaderListener listener = new CsvLoaderListener();

        // 遍历语法分析过程中生成的语法分析树，触发回调
        walker.walk(listener, tree);

        // 打印从csv文件加载的数据
        System.out.println(listener.getRows());
    }
}
```

## 解析JSON

完整源码见： [https://github.com/bytesfly/antlr-demo/tree/main/json2xml/](https://github.com/bytesfly/antlr-demo/tree/main/json2xml/)

输入`JSON`格式的数据：

```json
{
  "description" : "An imaginary server config file",
  "logs" : {"level":"verbose", "dir":"/var/log"},
  "host" : "antlr.org",
  "bool": true,
  "null": null,
  "pi": 3.14,
  "admin": ["parrt", "tombu"],
  "aliases": []
}
```

解析后并转成`XML`格式数据如下：

```xml
<description>An imaginary server config file</description>
<logs>
<level>verbose</level>
<dir>/var/log</dir>
</logs>
<host>antlr.org</host>
<bool>true</bool>
<null>null</null>
<pi>3.14</pi>
<admin>
<element>parrt</element>
<element>tombu</element>
</admin>
<aliases>
</aliases>
```

该程序实现了对常见`JSON`格式数据的解析并将其转成我们想要的`XML`格式。

语法规则为`JSON.g4`，如下：

```antlrv4
// Derived from http://json.org
grammar JSON;

@header {package com.github.bytesfly.jx.antlr;}

json:   object
    |   array
    ;

object
    :   '{' pair (',' pair)* '}'    # AnObject
    |   '{' '}'                     # EmptyObject
    ;

array
    :   '[' value (',' value)* ']'  # ArrayOfValues
    |   '[' ']'                     # EmptyArray
    ;

pair:   STRING ':' value ;

value
    :   STRING		# String
    |   NUMBER		# Atom
    |   object  	# ObjectValue
    |   array  		# ArrayValue
    |   'true'		# Atom
    |   'false'		# Atom
    |   'null'		# Atom
    ;

LCURLY : '{' ;
LBRACK : '[' ;
STRING :  '"' (ESC | ~["\\])* '"' ;

fragment ESC :   '\\' (["\\/bfnrt] | UNICODE) ;
fragment UNICODE : 'u' HEX HEX HEX HEX ;
fragment HEX : [0-9a-fA-F] ;

NUMBER
    :   '-'? INT '.' INT EXP?   // 1.35, 1.35E-9, 0.3, -4.5
    |   '-'? INT EXP            // 1e10 -3e4
    |   '-'? INT                // -3, 45
    ;
fragment INT :   '0' | '1'..'9' '0'..'9'* ; // no leading zeros
fragment EXP :   [Ee] [+\-]? INT ; // \- since - means "range" inside [...]

WS  :   [ \t\n\r]+ -> skip ;
```

我们自定义`Json2XmlListener.java`，如下：

```java
public class Json2XmlListener extends JSONBaseListener {

    private final StringBuilder builder = new StringBuilder();

    @Override
    public void enterPair(JSONParser.PairContext ctx) {
        // <key>
        builder.append("<")
                .append(stripQuotes(ctx.STRING().getText()))
                .append(">");
    }

    @Override
    public void exitPair(JSONParser.PairContext ctx) {
        // </key>
        builder.append("</")
                .append(stripQuotes(ctx.STRING().getText()))
                .append(">\n");
    }

    @Override
    public void enterString(JSONParser.StringContext ctx) {
        ifEnterArray(ctx);
        builder.append(stripQuotes(ctx.STRING().getText()));
    }

    @Override
    public void exitString(JSONParser.StringContext ctx) {
        ifExitArray(ctx);
    }

    @Override
    public void enterAtom(JSONParser.AtomContext ctx) {
        ifEnterArray(ctx);
        builder.append(ctx.getText());
    }

    @Override
    public void exitAtom(JSONParser.AtomContext ctx) {
        ifExitArray(ctx);
    }

    @Override
    public void enterObjectValue(JSONParser.ObjectValueContext ctx) {
        ifEnterArray(ctx);
        builder.append("\n");
    }

    @Override
    public void exitObjectValue(JSONParser.ObjectValueContext ctx) {
        ifExitArray(ctx);
    }

    @Override
    public void enterArrayValue(JSONParser.ArrayValueContext ctx) {
        ifEnterArray(ctx);
        builder.append("\n");
    }

    @Override
    public void exitArrayValue(JSONParser.ArrayValueContext ctx) {
        ifExitArray(ctx);
    }

    /**
     * 去除字符串包裹着的双引号
     */
    private static String stripQuotes(String s) {
        if (s == null || s.charAt(0) != CharPool.DOUBLE_QUOTES) {
            return s;
        }
        return s.substring(1, s.length() - 1);
    }

    /**
     * 是否进入数组元素的访问
     */
    private void ifEnterArray(JSONParser.ValueContext ctx) {
        // 如果上级是数组的话
        if (ctx.getParent().getRuleIndex() == JSONParser.RULE_array) {
            builder.append("<element>");
        }
    }

    /**
     * 是否退出数组元素的访问
     */
    private void ifExitArray(JSONParser.ValueContext ctx) {
        // 如果上级是数组的话
        if (ctx.getParent().getRuleIndex() == JSONParser.RULE_array) {
            builder.append("</element>\n");
        }
    }

    /**
     * 获取JSON转XML的结果
     */
    public String getResult() {
        return builder.toString();
    }
}
```

最终完整的解析`JSON`并将其转成想要的`XML`格式程序为`Json2Xml.java`，如下：

```java
public class Json2Xml {

    public static void main(String[] args) {
        // 读取resources目录下example.json文件
        String s = FileUtil.readUtf8String("example.json");

        // 从字符串读取输入数据
        CharStream input = CharStreams.fromString(s);

        // 新建一个词法分析器
        JSONLexer lexer = new JSONLexer(input);

        // 新建一个词法符号的缓冲区，用于存储词法分析器将生成的词法符号
        CommonTokenStream tokens = new CommonTokenStream(lexer);

        // 新建一个语法分析器，处理词法符号缓冲区中的内容
        JSONParser parser = new JSONParser(tokens);

        // 针对json规则，开始语法分析
        ParseTree tree = parser.json();

        // 新建一个通用的、能够触发回调函数的语法分析树遍历器
        ParseTreeWalker walker = new ParseTreeWalker();

        // 创建我们自定义的监听器
        Json2XmlListener listener = new Json2XmlListener();

        // 遍历语法分析过程中生成的语法分析树，触发回调
        walker.walk(listener, tree);

        // 打印JSON转XML的结果
        System.out.println(listener.getResult());
    }
}
```

通过上面两个实战案例，能感受到`ANTLR`的威力嘛？

当然，别看自己写的代码不多，但是需要思考的地方并不少，不理解的地方还是建议自己下载源码本地打断点等方式琢磨琢磨，动手之后其实也不是太难。