**备忘录模式(Memento Pattern)——撤销功能的实现**

说明：设计模式系列文章是读`刘伟`所著`《设计模式的艺术之道(软件开发人员内功修炼之道)》`一书的阅读笔记。个人感觉这本书讲的不错，有兴趣推荐读一读。详细内容也可以看看此书作者的博客`https://blog.csdn.net/LoveLion/article/details/17517213`。

## 模式概述

每个人都有过后悔的经历，但人生并无后悔药，有些错误一旦发生就无法再挽回，有些人一旦错过就不会再回来，有些话一旦说出口就不可能再收回，这就是人生。

人生没有`CTRL+Z`，但软件中有，可以实现撤销。

比如下图中的悔棋就很常见：

![](https://img2020.cnblogs.com/blog/1546632/202112/1546632-20211231164214827-182099565.png)

`悔棋`就是让系统恢复到某个历史状态，在很多软件中通常称之为`撤销`。

撤销功能的实现原理：保存软件系统的历史状态，当用户需要取消错误操作并且返回到某个历史状态时，可以取出事先保存的历史状态来覆盖当前状态。如下图所示：

![](https://img2020.cnblogs.com/blog/1546632/202112/1546632-20211231164717589-1648941505.png)

### 模式定义

备忘录模式提供了一种状态恢复的实现机制，使得用户可以方便地回到一个特定的历史步骤，当新的状态无效或者存在问题时，可以使用暂时存储起来的备忘录将状态复原，当前很多软件都提供了撤销(`undo`)操作，其中就使用了备忘录模式。

备忘录模式定义如下：

> 备忘录模式(Memento Pattern)：在不破坏封装的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态，这样可以在以后将对象恢复到原先保存的状态。它是一种对象行为型模式。

### 模式结构图

备忘录模式的核心是备忘录类以及用于管理备忘录的负责人类的设计，其结构大致如下图所示：

![](https://img2020.cnblogs.com/blog/1546632/202112/1546632-20211231165341205-712406340.png)

在备忘录模式结构图中包含如下几个角色：

- `Originator`（原发器）：它是一个普通类，可以创建一个备忘录，并存储它的当前内部状态，也可以使用备忘录来恢复其内部状态，一般将需要保存内部状态的类设计为原发器。
- `Memento`（备忘录)：存储原发器的内部状态，根据原发器来决定保存哪些内部状态。备忘录的设计一般可以参考原发器的设计，根据实际需要确定备忘录类中的属性。需要注意的是，除了原发器本身与负责人类之外，备忘录对象不能直接供其他类使用，原发器的设计在不同的编程语言中实现机制会有所不同。
- `Caretaker`（负责人）：负责人又称为管理者，它负责保存备忘录，但是不能对备忘录的内容进行操作或检查。在负责人类中可以存储一个或多个备忘录对象，它只负责存储对象，而不能修改对象，也无须知道对象的实现细节。

### 模式伪代码

下面就用伪代码实现上面的`悔棋`功能。

`Chessman`充当原发器：

```java
/**
 * 象棋棋子类(原发器)
 */
public class Chessman {

    private String label;
    private int x;
    private int y;

    public Chessman(String label, int x, int y) {
        this.label = label;
        this.x = x;
        this.y = y;
    }

    /**
     * 保存状态
     */
    public ChessmanMemento save() {
        return new ChessmanMemento(this.label, this.x, this.y);
    }

    /**
     * 恢复状态
     */
    public void restore(ChessmanMemento memento) {
        this.label = memento.getLabel();
        this.x = memento.getX();
        this.y = memento.getY();
    }

    // getter setter
}
```


`ChessmanMemento`充当备忘录：

```java
/**
 * 象棋棋子备忘录
 */
public class ChessmanMemento {

    private String label;
    private int x;
    private int y;

    public ChessmanMemento(String label,int x,int y) {
        this.label = label;
        this.x = x;
        this.y = y;
    }

    // getter setter
}
```


`MementoCaretaker`充当负责人：

```java
/**
 * 备忘录管理类(负责人)
 */
public class MementoCaretaker {

    // 定义一个集合来存储多个备忘录
    private final List<ChessmanMemento> history = new ArrayList<>();

    public ChessmanMemento getMemento(int i) {
        return history.get(i);
    }

    public void setMemento(ChessmanMemento memento) {
        history.add(memento);
    }
}
```

编写如下客户端测试代码：

```java
public class Client {

    // 记录当前状态所在位置
    private static int index = -1;
    private static MementoCaretaker mc = new MementoCaretaker();

    public static void main(String args[]) {
        Chessman chess = new Chessman("車", 1, 1);
        play(chess);

        chess.setY(4);
        play(chess);

        chess.setX(5);
        play(chess);

        undo(chess, index);
        undo(chess, index);

        redo(chess, index);
        redo(chess, index);
    }

    /**
     * 下棋
     */
    public static void play(Chessman chess) {
        // 保存备忘录
        mc.setMemento(chess.save());
        index++;

        showPosition(chess);
    }

    /**
     * 悔棋
     */
    public static void undo(Chessman chess, int i) {
        System.out.println("******悔棋******");
        index--;
        // 撤销到上一个备忘录
        chess.restore(mc.getMemento(i - 1));

        showPosition(chess);
    }

    /**
     * 撤销悔棋
     */
    public static void redo(Chessman chess, int i) {
        System.out.println("******撤销悔棋******");
        index++;
        // 恢复到下一个备忘录
        chess.restore(mc.getMemento(i + 1));

        showPosition(chess);
    }

    public static void showPosition(Chessman chess) {
        System.out.println(chess.getLabel() + " 当前位置为：" + "第" + chess.getX() + "行" + "第" + chess.getY() + "列。");
    }
}
```

输出结果如下：

```text
車 当前位置为：第1行第1列。
車 当前位置为：第1行第4列。
車 当前位置为：第5行第4列。
******悔棋******
車 当前位置为：第1行第4列。
******悔棋******
車 当前位置为：第1行第1列。
******撤销悔棋******
車 当前位置为：第1行第4列。
******撤销悔棋******
車 当前位置为：第5行第4列。
```

## 模式应用

在一些文本编辑软件、图像处理软件、数据库管理系统等软件中，备忘录模式都得到了很好的应用。

