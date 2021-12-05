
下面通过两个实例来快速上手`ANTLR`。


## 使用Listener转换数组

完整源码见：[https://github.com/bytesfly/antlr-demo/tree/main/array-init/src/main/java/com/github/bytesfly/arr](https://github.com/bytesfly/antlr-demo/tree/main/array-init/src/main/java/com/github/bytesfly/arr)

效果如下图所示：

![](https://img2020.cnblogs.com/blog/1546632/202112/1546632-20211204113455066-1434691214.png)

该程序能识别输入的整数数组并将其转化为`JSON`格式的字符串数组。

语法规则为`ArrayInit.g4`，如下：

```text
/** Grammars always start with a grammar header. This grammar is called
 *  ArrayInit and must match the filename: ArrayInit.g4
 */
grammar ArrayInit;

@header {package com.github.bytesfly.arr.antlr;}

/** A rule called init that matches comma-separated values between {...}. */
init  : '{' value (',' value)* '}' ;  // must match at least one value

/** A value can be either a nested array/struct or a simple integer (INT) */
value : init
      | INT
      ;

// parser rules start with lowercase letters, lexer rules with uppercase
INT :   [0-9]+ ;             // Define token INT as one or more digits
WS  :   [ \t\r\n]+ -> skip ; // Define whitespace rule, toss it out
```

我们自定义`IntToStringListener.java`，如下：

```java
public class IntToStringListener extends ArrayInitBaseListener {

    private final StringBuilder builder = new StringBuilder();

    /**
     * Translate { to [
     */
    @Override
    public void enterInit(ArrayInitParser.InitContext ctx) {
        builder.append('[');
    }

    /**
     * Translate } to ]
     */
    @Override
    public void exitInit(ArrayInitParser.InitContext ctx) {
        // 去除value节点后的逗号
        builder.setLength(builder.length() - 1);
        builder.append(']');

        if (ctx.parent != null) {
            // 嵌套结构需要追加逗号，与enterValue的处理保持一致
            builder.append(',');
        }
    }

    /**
     * Translate integers to strings with ""
     */
    @Override
    public void enterValue(ArrayInitParser.ValueContext ctx) {
        TerminalNode node = ctx.INT();
        if (node != null) {
            builder.append("\"").append(node.getText()).append("\",");
        }
    }

    public String getResult() {
        return builder.toString();
    }
}
```

最终的转换程序为`Translate.java`，如下：

```java
public class Translate {

    public static void main(String[] args) throws Exception {
        // 从键盘输入
        Scanner sc = new Scanner(System.in);

        while (sc.hasNext()) {

            String s = sc.nextLine();

            // 从字符串读取输入数据
            CharStream input = CharStreams.fromString(s);

            // 新建一个词法分析器
            ArrayInitLexer lexer = new ArrayInitLexer(input);

            // 新建一个词法符号的缓冲区，用于存储词法分析器将生成的词法符号
            CommonTokenStream tokens = new CommonTokenStream(lexer);

            // 新建一个语法分析器，处理词法符号缓冲区中的内容
            ArrayInitParser parser = new ArrayInitParser(tokens);

            // 针对init规则，开始语法分析
            ParseTree tree = parser.init();

            // 新建一个通用的、能够触发回调函数的语法分析树遍历器
            ParseTreeWalker walker = new ParseTreeWalker();

            // 创建我们自定义的监听器
            IntToStringListener listener = new IntToStringListener();

            // 遍历语法分析过程中生成的语法分析树，触发回调
            walker.walk(listener, tree);

            // 打印转换结果
            System.out.println(listener.getResult());
        }
    }
}
```

监听器机制的优雅之处在于，不需要自己编写任何遍历语法分析树的代码。事实上，我们甚至都不知道`ANTLR`运行库是怎么遍历语法分析树、怎么调用我们的方法的。我们只知道，在语法规则对应的语句的开始和结束位置处，我们的监听器方法可以得到通知。

当然如果想知道`ANTLR`运行库是怎么遍历语法分析树并不困难，见`org.antlr.v4.runtime.tree.ParseTreeWalker#walk()`：

```java
public void walk(ParseTreeListener listener, ParseTree t) {
    if ( t instanceof ErrorNode) {
        listener.visitErrorNode((ErrorNode)t);
        return;
    }
    else if ( t instanceof TerminalNode) {
        listener.visitTerminal((TerminalNode)t);
        return;
    }
    RuleNode r = (RuleNode)t;
    enterRule(listener, r);
    int n = r.getChildCount();
    for (int i = 0; i<n; i++) {
        walk(listener, r.getChild(i));
    }
    exitRule(listener, r);
}
```

这段代码也非常容易懂，其实就是常规树的遍历，遍历过程中反调自定义的`IntToStringListener`中的实现方法。

## 使用Visitor构建计算器


