



# 设计模式

## 责任链模式：

是一种处理请求的模式，他让多个处理器都有机会处理该请求 ，直到其中某个处理成功为止。责任链模式将多个处理器串成一条链，请求可以在链上传递。

![image-20240905110239504](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240905110239504.png)

不同的处理类会有各自的处理方法，而且还有一个方法或者成员变量，nexthandler用于将任务传递到下一个处理类中，各个处理类就由nexthandler链接起来。



责任链模式的优点：

1. 将请求和处理分开，请求者不知道谁去处理，比如假条，直接扔给辅导员，管他是自己批还是让院里去批，跟我们无关。
2. 提高系统灵活性，添加一个处理器到链条中，代价很小。

缺点：

1. 降低系统性能，每个请求都要从链头走到链尾，链较长时会大幅降低性能



## 组合模式

就是将各种功能从普遍具有这些功能的实例上拆下来，拆为组件，灵活性会大大增强。







## 中介者模式：

游戏中的不同角色对象，在不产生直接引用或者依赖关系的情况下如何进行协调沟通？

游戏场景：每次有三个敌人攻击玩家，当正在攻击玩家的一个敌人死后，会有其他待机敌人替补上来攻击玩家。

那么玩家和这些敌人之间是如何通信的？

不能直接进行通信，这样不仅会创建硬引用，内存占用增加，而且还会让敌人和玩家之间产生依赖关系



中介者模式做的就是，对象之间相互独立，所有对象都只和居中的中介者进行通信，对象彼此之间不相互通信，中介再将消息重定向给目标对象，这样所有对象只需要依赖中介者，而不是依赖其他对象。就行机场飞机降落，机长不用联系其他飞机何时起飞降落，只需要联系塔台就能保证飞机的正常起落，塔台就是维护不同飞机之间的中介者。

![image-20240905112838367](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240905112838367.png)

## 对象池模式：



## 观察者模式：

观察者模式有时也叫做发布者/订阅者模式,它通常会将某个事件进行广播，而其他对象在订阅或者监听到这个信号之后，会进行一些操作

假如一个关卡中现在有三个敌人，需要杀光他们才能逃出关卡，而用户的UI界面上enemies count 会随着敌人死亡而更新，这么多通信如何实现？ 使用观察者模式进行解耦

对于ui和敌人这里，如果在敌人中调用了ui控件会造成二者的耦合，所以不能这样做。我们需要的是敌人死亡的时候开始广播，然后在widget接收广播。

![image-20240828142244029](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240828142244029.png)

在widget中

![image-20240828142513157](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240828142513157.png)

![image-20240828142854516](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240828142854516.png)

![image-20240828143403998](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240828143403998.png)

![image-20240828143529475](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240828143529475.png)

续：

![image-20240828143549798](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240828143549798.png)

哦哦，他是把关卡和widget作为两个同一事件的订阅者，我的思路是，在widget中得知已经敌人数量为0之后，再次发送，相当于多创建了一个发送事件的人。两者酌情使用吧。

### 出问题了：

事件分发器需要一个target 

![image-20240828150609143](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240828150609143.png)

因为在事件绑定的时候需要获取发出目标的Actor嘛，不然怎么叫订阅者模式，我们肯定需要知道的是，是谁call了这个事件，然后我订阅他，等待接收这个事件之后，我进行后续操作。

一般是通过Get all Actor from Class 获取目标，然后再进行bind。

![image-20240828171945213](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240828171945213.png)



![image-20240828172534103](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240828172534103.png)

![image-20240828171825451](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240828171825451.png)

刚才试了下，确实是，当动画创建好之后就可以在蓝图中搜索动画名获取。

那动画是如何创建的？如图：

![image-20240828172158814](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240828172158814.png)

修改动画时长：

![image-20240828172406643](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240828172406643.png)

上面在事件分发和绑定事件的时候，会需要target输入，因为这个target我们都知道是同一种类型的，所以就没有必要再使用cast to类型转换，然后直接使用Get all Actor of Class即可。

1. **使用 `Cast to` 节点**：
   - **优点**：`Cast to` 节点允许你动态地检查对象是否属于特定的类型，这在对象类型不确定或需要动态检查时非常有用。
   - **缺点**：每次事件触发时，`Cast to` 节点都会执行类型检查，这可能会增加性能开销。
2. **使用 `Get All Actors of Class` 节点**：
   - **优点**：如果你知道所有目标对象都属于同一类型，`Get All Actors of Class` 节点可以快速返回所有目标对象。
   - **缺点**：如果目标对象类型经常变化，或者你不需要所有目标对象，使用这个节点可能会导致不必要的性能开销。

#### **为减少性能消耗，可以考虑以下策略：**

**预先获取目标对象：**如果事件经常触发，并且目标对象数量稳定，可以考虑在事件触发之前预先获取这些对象（直接在变量里对象引用？然后再调用对象），好像可以

**避免不必要的类型检查**：如果你知道对象类型，可以避免使用 `Cast to` 节点，直接操作对象。



### 事件分发的弊端：

后期处理的时候总是找不到分发的Actor和接收的Actor在哪里。

需要文档维护谁是分发者，谁是接收者。