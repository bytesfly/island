**访问者模式(Visitor Pattern)——操作复杂对象结构**

说明：设计模式系列文章是读`刘伟`所著`《设计模式的艺术之道(软件开发人员内功修炼之道)》`一书的阅读笔记。个人感觉这本书讲的不错，有兴趣推荐读一读。详细内容也可以看看此书作者的博客`https://blog.csdn.net/LoveLion/article/details/17517213`。

## 模式概述

在软件开发中，可能会遇到操作复杂对象结构的场景，在该对象结构中存储了多个不同类型的对象信息，而且对同一对象结构中的元素的操作方式并不唯一，可能需要提供多种不同的处理方式，还有可能增加新的处理方式。

在设计模式中，有一种模式可以满足上述要求，其`模式动机就是以不同的方式操作复杂对象结构`，该模式就是下面要介绍的访问者模式。

### 模式定义

访问者模式是一种较为复杂的行为型设计模式，它包含访问者和被访问元素两个主要组成部分，这些被访问的元素通常具有不同的类型，且不同的访问者可以对它们进行不同的访问操作。

访问者模式使得用户可以在不修改现有系统的情况下扩展系统的功能，为这些不同类型的元素增加新的操作。

在使用访问者模式时，被访问元素通常不是单独存在的，它们存储在一个集合中，这个集合被称为`对象结构`，访问者通过遍历对象结构实现对其中存储的元素的逐个操作。

访问者模式定义如下：

> 访问者模式(`Visitor Pattern`)：提供一个作用于某对象结构中的各元素的操作表示，它使我们可以在不改变各元素的类的前提下定义作用于这些元素的新操作。访问者模式是一种对象行为型模式。


### 模式结构图

访问者模式的结构较为复杂，如下图所示：

![](https://img2020.cnblogs.com/blog/1546632/202112/1546632-20211213104720996-612811924.png)

在访问者模式结构图中包含如下几个角色：

- `Visitor`（抽象访问者）：抽象访问者为对象结构中每一个具体元素类`ConcreteElement`声明一个访问操作，从这个操作的名称或参数类型可以清楚知道需要访问的具体元素的类型，具体访问者需要实现这些操作方法，定义对这些元素的访问操作。
- `ConcreteVisitor`（具体访问者）：具体访问者实现了每个由抽象访问者声明的操作，每一个操作用于访问对象结构中一种类型的元素。
- `Element`（抽象元素）：抽象元素一般是抽象类或者接口，它定义一个`accept()`方法，该方法通常以一个抽象访问者作为参数。【稍后将介绍为什么要这样设计】
- `ConcreteElement`（具体元素）：具体元素实现了`accept()`方法，在`accept()`方法中调用访问者的访问方法以便完成对一个元素的操作。
- `ObjectStructure`（对象结构）：对象结构是一个元素的集合，它用于存放元素对象，并且提供了遍历其内部元素的方法。它可以结合组合模式来实现，也可以是一个简单的集合对象，如一个`List`对象或一个`Set`对象。

在访问者模式中，增加新的访问者无须修改原有系统，系统具有较好的可扩展性。

### 模式伪代码

总结一下，访问者模式可以理解为：为操作某一对象的一组元素抽象出一组接口，配合对象元素的一个`accept()`操作，从而实现了不需要修改对象元素而给该元素提供不一样操作的目的。

访问者代码大致如下：

```java
/**
 * 抽象访问者
 */
public interface Visitor {

    void visit(ElementA element);

    void visit(ElementB element);

    void visit(ElementC element);
}

/**
 * 具体访问者的实现
 */
public class ConcreteVisitor implements Visitor {

    @Override
    public void visit(ElementA element) {
        // ElementA 操作代码
    }

    @Override
    public void visit(ElementB element) {
        // ElementB 操作代码
    }

    @Override
    public void visit(ElementC element) {
        // ElementC 操作代码
    }
}
```

对于被访问的元素而言，在其中一般都定义了一个`accept()`方法，用于接受访问者的访问，典型的抽象元素类代码如下所示：

```java
/**
 * 对被访问元素进行抽象
 */
public interface Element {
    void accept(Visitor visitor);
}

/**
 * 具体元素A
 */
public class ElementA implements Element {

    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }
}

/**
 * 具体元素B
 */
public class ElementB implements Element {

    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }
}

/**
 * 具体元素C
 */
public class ElementC implements Element {

    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }

    public void operation() {
        // 业务方法
    }
}
```

需要注意的是这里传入了一个抽象访问者`Visitor`类型的参数，即`针对抽象访问者进行编程，而不是具体访问者`，在程序运行时再确定具体访问者的类型，并调用具体访问者对象的`visit()`方法实现对元素对象的操作。

在抽象元素类`Element`的子类中实现了`accept()`方法，用于接受访问者的访问，在具体元素类中还可以定义不同类型的元素所特有的业务方法，比如上面的`ElementC`

在访问者模式中，对象结构可能是一个集合，它用于存储元素对象并接受访问者的访问，其典型代码如下所示：

```java
public class ObjectStructure {

    // 存储元素对象
    private final List<Element> elements = new ArrayList<>();

    public void accept(Visitor visitor) {
        // 遍历访问每一个元素
        for (Element element : elements) {
            element.accept(visitor);
        }
    }
}
```

## 模式应用

### 模式在JDK中的应用

在早期的Java版本中，如果要对指定目录下的文件进行遍历，大多用递归的方式来实现，这种方法复杂且灵活性不高。

`Java 7`版本后，`Files`类提供了`walkFileTree()`方法，该方法可以很容易的对目录下的所有文件进行遍历，需要`Path`、`FileVisitor`两个参数。其中，`Path`是要遍历文件的路径，`FileVisitor`则可以看成一个文件访问器。

`java.nio.file.Files#walkFileTree()`源码如下：

```java
public static Path walkFileTree(Path start, FileVisitor<? super Path> visitor)
        throws IOException
    {
        return walkFileTree(start,
                            EnumSet.noneOf(FileVisitOption.class),
                            Integer.MAX_VALUE,
                            visitor);
    }
```

`FileVisitor`主要提供了4个方法，且返回结果的都是`FileVisitResult`对象值，用于决定当前操作完成后接下来该如何处理。`FileVisitResult`是一个枚举类，代表返回之后的一些后续操作。源码如下。

```java
package java.nio.file;
import java.nio.file.attribute.BasicFileAttributes;
import java.io.IOException;
public interface FileVisitor<T> {
    FileVisitResult preVisitDirectory(T dir, BasicFileAttributes attrs)
        throws IOException;
    FileVisitResult visitFile(T file, BasicFileAttributes attrs)
        throws IOException;
    FileVisitResult visitFileFailed(T file, IOException exc)
        throws IOException;
    FileVisitResult postVisitDirectory(T dir, IOException exc)
        throws IOException;
}
package java.nio.file;
public enum FileVisitResult {
   
    CONTINUE,
   
    TERMINATE,
   
    SKIP_SUBTREE,
   
    SKIP_SIBLINGS;
}
```

这样我们就可以很容易实现递归拷贝目录或者删除目录等等。

比如我们看下`cn.hutool.core.io.file.visitor.CopyVisitor`的拷贝操作的实现：

```java

/**
 * 文件拷贝的FileVisitor实现，用于递归遍历拷贝目录，此类非线程安全
 * 此类在遍历源目录并复制过程中会自动创建目标目录中不存在的上级目录。
 */
public class CopyVisitor extends SimpleFileVisitor<Path> {

    private final Path source;
    private final Path target;
    private boolean isTargetCreated;
    private final CopyOption[] copyOptions;

    /**
     * 构造
     *
     * @param source 源Path
     * @param target 目标Path
     * @param copyOptions 拷贝选项，如跳过已存在等
     */
    public CopyVisitor(Path source, Path target, CopyOption... copyOptions) {
        if(PathUtil.exists(target, false) && false == PathUtil.isDirectory(target)){
            throw new IllegalArgumentException("Target must be a directory");
        }
        this.source = source;
        this.target = target;
        this.copyOptions = copyOptions;
    }

    @Override
    public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs)
            throws IOException {
        initTarget();
        // 将当前目录相对于源路径转换为相对于目标路径
        final Path targetDir = target.resolve(source.relativize(dir));
        try {
            Files.copy(dir, targetDir, copyOptions);
        } catch (FileAlreadyExistsException e) {
            if (false == Files.isDirectory(targetDir))
                throw e;
        }
        return FileVisitResult.CONTINUE;
    }

    @Override
    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs)
            throws IOException {
        initTarget();
        Files.copy(file, target.resolve(source.relativize(file)), copyOptions);
        return FileVisitResult.CONTINUE;
    }

    /**
     * 初始化目标文件或目录
     */
    private void initTarget(){
        if(false == this.isTargetCreated){
            PathUtil.mkdir(this.target);
            this.isTargetCreated = true;
        }
    }
}
```

再扩充一个删除操作，同样不难，见`cn.hutool.core.io.file.visitor.DelVisitor`：

```java

/**
 * 删除操作的FileVisitor实现，用于递归遍历删除文件夹
 */
public class DelVisitor extends SimpleFileVisitor<Path> {

	public static DelVisitor INSTANCE = new DelVisitor();

	@Override
	public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
		Files.delete(file);
		return FileVisitResult.CONTINUE;
	}

	/**
	 * 访问目录结束后删除目录，当执行此方法时，子文件或目录都已访问（删除）完毕<br>
	 * 理论上当执行到此方法时，目录下已经被清空了
	 *
	 * @param dir 目录
	 * @param e   异常
	 * @return {@link FileVisitResult}
	 * @throws IOException IO异常
	 */
	@Override
	public FileVisitResult postVisitDirectory(Path dir, IOException e) throws IOException {
		if (e == null) {
			Files.delete(dir);
			return FileVisitResult.CONTINUE;
		} else {
			throw e;
		}
	}
}
```

到这里，你能感受到访问者模式的妙用吗？

【模式动机就是以不同的方式操作复杂对象结构，增加新的处理方式，无需修改既有的代码。】

### 模式在开源项目中的应用

在XML文档解析、编译器的设计、复杂集合对象的处理等领域访问者模式得到了一定的应用。有兴趣的朋友建议看看 [ANTLR专题](https://bytesfly.github.io/island/#/antlr4/quick-start) ，或许能深刻体会到访问者模式的魅力。

## 模式总结

当系统中存在一个较为复杂的对象结构，且不同访问者对其所采取的操作也不相同时，可以考虑使用访问者模式进行设计。

### 主要优点

- 将元素对象的访问行为集中到一个访问者对象中，而不是分散在一个个的元素类中。类的职责更加清晰，有利于对象结构中元素对象的复用，相同的对象结构可以供多个不同的访问者访问。
- 增加新的访问操作就意味着增加一个新的具体访问者类，方便扩展，无须修改源代码，符合`开闭原则`。

### 主要缺点

- 增加新的元素类很困难。在访问者模式中，每增加一个新的元素类都意味着要在抽象访问者角色中增加一个新的抽象操作，并在每一个具体访问者类中增加相应的具体操作，这违背了`开闭原则`的要求。
- 破坏封装。访问者模式要求访问者对象访问并调用每一个元素对象的操作，这意味着元素对象有时候必须暴露一些自己的内部操作和内部状态，否则无法供访问者访问。

### 适用场景

- 一个对象结构包含多个类型的对象，希望对这些对象实施一些依赖其具体类型的操作。在访问者中针对每一种具体的类型都提供了一个访问操作，不同类型的对象可以有不同的访问操作。
- 需要对一个对象结构中的对象进行很多不同的并且不相关的操作，而需要避免让这些操作`污染`这些对象的类，也不希望在增加新操作时修改这些类。访问者模式使得我们可以将相关的访问操作集中起来定义在访问者类中，对象结构可以被多个不同的访问者类所使用，将对象本身与对象的访问操作分离。
- 对象结构中对象对应的类很少改变，但经常需要在此对象结构上定义新的操作。
