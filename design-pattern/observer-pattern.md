**观察者模式(Observer Pattern)——对象间的联动**

说明：设计模式系列文章是读`刘伟`所著`《设计模式的艺术之道(软件开发人员内功修炼之道)》`一书的阅读笔记。个人感觉这本书讲的不错，有兴趣推荐读一读。详细内容也可以看看此书作者的博客`https://blog.csdn.net/LoveLion/article/details/17517213`。

## 模式概述

“红灯停，绿灯行”，在日常生活中，交通信号灯装点着我们的城市，指挥着日益拥挤的城市交通。当红灯亮起，来往的汽车将停止；而绿灯亮起，汽车可以继续前行。

在这个过程中，交通信号灯是汽车（更准确地说应该是汽车驾驶员）的观察目标，而汽车是观察者。随着交通信号灯的变化，汽车的行为也将随之而变化，一盏交通信号灯可以指挥多辆汽车。如下图所示：

![](https://img2020.cnblogs.com/blog/1546632/202112/1546632-20211230154240422-1290981239.png)

在软件系统中，有些对象之间也存在类似交通信号灯和汽车之间的关系，一个对象的状态或行为的变化将导致其他对象的状态或行为也发生改变，它们之间将产生联动，正所谓“触一而牵百发”。

为了更好地描述对象之间存在的这种一对多（包括一对一）的联动，观察者模式应运而生，它定义了对象之间一种一对多的依赖关系，让一个对象的改变能够影响其他对象。下面介绍用于实现对象间联动的观察者模式。

### 模式定义

观察者模式是使用频率最高的设计模式之一，它用于建立一种对象与对象之间的依赖关系，一个对象发生改变时将自动通知其他对象，其他对象将相应作出反应。

在观察者模式中，发生改变的对象称为`观察目标`，而被通知的对象称为`观察者`，一个观察目标可以对应多个观察者，而且这些观察者之间可以没有任何相互联系，可以根据需要增加和删除观察者，使得系统更易于扩展。

观察者模式定义如下：

> 观察者模式(`Observer Pattern`)：定义对象之间的一种一对多依赖关系，使得每当一个对象状态发生改变时，其相关依赖对象皆得到通知并被自动更新。观察者模式的别名包括发布-订阅（`Publish/Subscribe`）模式、模型-视图（`Model/View`）模式、源-监听器（`Source/Listener`）模式或从属者（`Dependents`）模式。观察者模式是一种对象行为型模式。

### 模式结构图

观察者模式结构中通常包括`观察目标`和`观察者`两个继承层次结构，其结构如下图所示：

![](https://img2020.cnblogs.com/blog/1546632/202112/1546632-20211230155400099-2120155879.png)

观察者模式描述了如何建立对象与对象之间的依赖关系，以及如何构造满足这种需求的系统。

观察者模式包含观察目标和观察者两类对象，一个目标(`Subject`)可以有任意数目的与之相依赖的观察者(`Observer`)，一旦观察目标的状态发生改变，所有的观察者都将得到通知。作为对这个通知的响应，每个观察者都将监视观察目标的状态以使其状态与目标状态同步，这种交互也称为发布-订阅(`Publish-Subscribe`)。观察目标是通知的发布者，它发出通知时并不需要知道谁是它的观察者，可以有任意数目的观察者订阅它并接收通知。

### 模式伪代码

首先定义一个抽象目标`Subject`，目标又称为主题，它是指被观察的对象。在目标中定义了一个观察者集合，一个观察目标可以接受任意数量的观察者来观察，它提供一系列方法来增加和删除观察者对象，同时它定义了通知方法`notice()`。目标类可以是接口，也可以是抽象类或具体类。典型代码如下：

```java
/**
 * 抽象观察目标
 */
public abstract class Subject {

    // 定义一个观察者集合用于存储所有观察者对象
    protected List<Observer> observers = new ArrayList<>();

    /**
     * 注册，用于向观察者集合中增加一个观察者
     */
    public void attach(Observer observer) {
        observers.add(observer);
    }

    // 注销，用于在观察者集合中删除一个观察者
    public void detach(Observer observer) {
        observers.remove(observer);
    }

    /**
     * 抽象通知方法
     */
    public abstract void notice();
}
```

具体目标类`ConcreteSubject`目标类的子类，通常它包含有经常发生改变的数据，当它的状态发生改变时，向它的各个观察者发出通知；同时它还实现了在目标类中定义的抽象业务逻辑方法（如果有的话）。如果无须扩展目标类，则具体目标类可以省略。其典型代码如下：

```java
/**
 * 具体观察目标
 */
public class ConcreteSubject extends Subject {

    /**
     * 实现通知方法
     */
    @Override
    public void notice() {
        // 遍历观察者集合，调用每一个观察者的响应方法
        for (Observer observer : observers) {
            observer.update();
        }
    }
}
```

抽象观察者角色一般定义为一个接口，通常只声明一个`update()`方法，为不同观察者的更新（响应）行为定义相同的接口，这个方法在其子类中实现，不同的观察者具有不同的响应方法。抽象观察者`Observer`典型代码如下：

```java
/**
 * 抽象观察者
 */
public interface Observer {

    /**
     * 观察者的响应
     */
    void update();
}
```

具体观察者中可能维护一个指向具体目标对象的引用，它存储具体观察者的有关状态，这些状态需要和具体目标的状态保持一致；它实现了在抽象观察者`Observer`中定义的`update()`方法。通常在实现时，可以调用具体目标类的`attach()`方法将自己添加到目标类的集合中或通过`detach()`方法将自己从目标类的集合中删除。具体观察者`ConcreteObserver`典型代码如下：

```java
/**
 * 具体观察者
 */
public class ConcreteObserver implements Observer {

    /**
     * 具体响应
     */
    @Override
    public void update() {
        // ...
    }
}
```

## 模式应用

在JDK的`java.util`包中，提供了`Observable`类以及`Observer`接口，构成了JDK对观察者模式的支持。如下图所示：

![](https://img2020.cnblogs.com/blog/1546632/202112/1546632-20211230170035341-1061735662.png)

- `Observer`接口

在`java.util.Observer`接口中只声明一个方法，它充当抽象观察者，其方法声明代码如下所示：

```java
public interface Observer {
    void update(Observable o, Object arg);
}
```

当观察目标的状态发生变化时，该方法将会被调用，在`Observer`的子类中将实现`update(Observable o, Object arg)`方法，即具体观察者可以根据需要具有不同的更新行为。当调用观察目标类`Observable`的`notifyObservers()`方法时，将执行观察者类中的`update(Observable o, Object arg)`方法。

- `Observable`类

`java.util.Observable`类充当观察目标类，在`Observable`中定义了一个向量`Vector`来存储观察者对象，它所包含的方法及说明见下表：

|                     方法名                     |                           方法描述                           |
| :--------------------------------------------: | :----------------------------------------------------------: |
|                  `Observable()`                 |                   构造方法，实例化`Vector`。                   |
|            `addObserver(Observer o)`            |                     注册新的观察者对象。                     |
|           `deleteObserver(Observer o)`           |                删除某一个观察者对象。                |
| `notifyObservers()`和`notifyObservers(Object arg)` | 通知方法，用于在方法内部循环调用每一个观察者的`update()`方法。 |
|               `deleteObservers()`                |                 删除已注册的所有观察者对象。                 |
|                  `setChanged()`                  | 该方法被调用后会设置一个`boolean`类型的内部标记变量`changed`的值为`true`，表示观察目标对象的状态发生了变化。 |
|                 `clearChanged()`                 | 将`changed`变量的值设为`false`，表示对象状态不再发生改变或者已经通知了所有的观察者对象，调用了它们的`update()`方法。 |
|                  `hasChanged()`                  |                        状态是否改变。                        |
|                `countObservers()`                |                      返回观察者的数量。                      |

可以直接使用JDK中的`Observer`接口和`Observable`类来作为观察者模式的抽象层，再自定义具体观察者类和具体观察目标类。

## 模式总结

观察者模式是一种使用频率非常高的设计模式，无论是移动应用、Web应用或者桌面应用，观察者模式几乎无处不在，它为实现对象之间的联动提供了一套完整的解决方案，凡是涉及到一对一或者一对多的对象交互场景都可以使用观察者模式。

观察者模式广泛应用于各种编程语言的GUI事件处理的实现，在基于事件的XML解析技术（如SAX2）以及Web事件处理中也都使用了观察者模式。

### 主要优点

- 观察者模式支持广播通信，观察目标会向所有已注册的观察者对象发送通知，简化了一对多系统设计的难度。
- 观察者模式在观察目标和观察者之间建立一个抽象的耦合。观察目标只需要维持一个抽象观察者的集合，无须了解其具体观察者。由于观察目标和观察者没有紧密地耦合在一起，因此它们可以属于不同的抽象化层次。

### 主要缺点

- 如果一个观察目标对象有很多直接和间接观察者，将所有的观察者都通知到会花费很多时间。
- 如果在观察者和观察目标之间存在循环依赖，观察目标会触发它们之间进行循环调用，可能导致系统崩溃。


### 适用场景

- 一个对象的改变将导致一个或多个其他对象也发生改变，而并不知道具体有多少对象将发生改变，也不知道这些对象是谁。
- 需要在系统中创建一个触发链，A对象的行为将影响B对象，B对象的行为将影响C对象……，可以使用观察者模式创建一种链式触发机制。

