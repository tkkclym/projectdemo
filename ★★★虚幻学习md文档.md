# 虚幻引擎学习



## **time line学习**

时间轴里面还有自动播放按钮，可以不连接事件进行自动播放，"循环"按钮可以循环

time line前方的引脚，play play from start 不同，如果连接在play 上，如果有事件触发了stop引脚，则每次再次开启时间轴是从停止的时间开始继续，如果是play from start 则是从头开始。reverse 引脚是倒序输出  reverse from the end 和reverse from start 一直，也是每次只从最后end开始播放 set new time 引脚，每次输出的都是下面那个时间对应的值

![image-20240705150605987](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240705150605987.png)



使用时间轴控制物体旋转时，假如是绕Z轴旋转，则数值要设置为360，才能完整旋转，用较小的数字总以为是出bug了。

![image-20240705151832010](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240705151832010.png)

时间轴还可以触发事件，触发事件轨道挺好用，比如我撞到了一个炸弹，三秒后可能爆炸，可能4秒后旁边的敌人开始逃跑，然后五秒后播放声音，远处山林开始变化等，根据时间触发一系列事件



![image-20240705153905007](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240705153905007.png)

销毁actor对象。释放内存????

销毁不会释放内存，销毁时，还会占用CPU的性能。

那如何使用内存池呢？

设置门这些东西旋转的时候设置差值，可能就不用直接在时间轴里设置360了？我去试验一下！

![image-20240705155943715](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240705155943715.png)

确实如此，他可以360旋转，我在里面设置的轨道高度是1

![image-20240705160143692](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240705160143692.png)



##### 小技巧

在想设置物体位置移动的时候可以先get物体位置，然后将其提升为变量，在后面就可以使用这个物体的变量然后设置物体的位置了，不需要选择relative location 或者是世界位置转换‘



时间轴后面还可以设置delay，再连回时间轴的reverse，往返运动

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240705194624551.png" alt="image-20240705194624551" style="zoom: 80%;" />







## 事件分发器

两个对象，一个发出的，一个接收的

事件分发器的调用：谁将这个事件分发出去，比如，一个boss的死亡，死亡之后回触发一系列其他事件，小兵的销毁啊，关卡中一些事物的变化等，所以检测boss血量，在boss血量小于0了之后去调用事件分发器，然后在关卡蓝图中实现对关卡中actor的蓝图中自定义事件的绑定。





# 粒子发射器

![image-20240707215509735](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240707215509735.png)

在使用 spawn per unit 粒子发射器的时候：

max movement threshold 是大于某个速度的时候不生成粒子

movement tolerance 是大于某个速度的时候生成粒子





# 动画蓝图：

![image-20240724205940077](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240724205940077.png)



创建蒙太奇动画之后为什么用不了，而角色默认的蒙太奇可以使用？，并且我复制粘贴之后的蒙太奇文件换了一个文件名，和内容依然可以使用?

**原因找到了。**状态机和outpose之间没有插槽slot,所以不能执行蒙太奇动画：

![image-20240724211147995](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240724211147995.png)

1.**插槽和OutPose的作用**：

- 插槽是动画蓝图中的一个关键部分，它允许您将动画片段（Montage）与特定的骨骼和控制器绑定。
- OutPose是插槽中的一个关键节点，它用于指定动画播放完成后角色的姿势。

2.**插槽配置**：

- 您需要为每个动画片段配置一个插槽，并确保插槽中的OutPose节点连接到了状态机中的下一个状态或OutPose节点。
- 如果没有为动画片段配置插槽，或者插槽中的OutPose节点没有正确连接，动画片段将无法播放。

3.**状态机和插槽的依赖关系**：0

- 在动画蓝图中，状态机的状态通常与插槽相关联。
- 状态机的状态在动画播放过程中被激活，而插槽则决定了动画片段如何与角色绑定。
- 如果状态机的状态没有与插槽正确关联，动画片段将无法播放

![image-20240724214852013](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240724214852013.png)

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240724215309334.png" alt="image-20240724215309334" style="zoom:50%;" />

因为混合动画有一些问题，现在设置动画通知，将一部分不需要混合的动画通知到动画蓝图

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240724220254965.png" alt="image-20240724220254965" style="zoom:50%;" />

在动画蓝图中新增一个bool值，用于这两个事件设置这个布尔，然后在通过另一个节点，“blend pose by bool”使我们可以很方便的设置融合和不融合的动作

![image-20240724222619605](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240724222619605.png)

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240724222521010.png" alt="image-20240724222521010" style="zoom:67%;" />



# GAS：

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240725132541387.png" alt="image-20240725132541387" style="zoom:67%;" />

GAS平A中CD的制作（2中的翻译是持续时间 振幅）：

![image-20240725125405121](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240725125405121.png)

GAS普攻制作的小结：

1.技能要创建技能基类，然后再继承基类创建一个技能。技能内包含提交GE(包括冷却，控制等)和montage的使用

![image-20240725130229998](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240725130229998.png)

2.在**基类角色蓝图**中创建gameplay ability component，然后使用give ability节点，将技能赋予这个基类角色。

3.赋予之后还要激活，我们将激活的步骤封装成函数

BUG:

剑在没有发动普攻的时候路过依然会产生碰撞怎么办？，1.在动画蒙太奇中添加开始和结束的事件通知，

***2.在动画蓝图中获取角色，获取剑上的碰撞胶囊体，然后根据动画通知设置胶囊体能使用的时间***

动画蒙太奇中的动画通知和动画蓝图之间的通信

修改剑对敌人伤害时的碰撞时，记得查看剑上本身所带的碰撞胶囊体是什么碰撞类型的，这里将剑改为忽略所有

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240725143209411.png" alt="image-20240725143209411" style="zoom:50%;" />

然后只有到有动画通知的时候才能检测重叠：

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240725143428431.png" alt="image-20240725143428431" style="zoom: 50%;" />

![image-20240725143855119](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240725143855119.png)

在写设置血量的最小值的时候，使用C++，：

![image-20240725203825063](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240725203825063.png)

不要在编译器中生成，在引擎中那个实时编码中生成就不会报错、



弹簧臂的取消使用：

![image-20240725211117539](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240725211117539.png)

## 镜头的缓慢跟进(也是在弹簧臂上)：

![image-20240725211545319](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240725211545319.png)

## 将表中数据导入ue并使用：

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240725211937729.png" alt="image-20240725211937729" style="zoom:50%;" />

导入的时候选择使用

![image-20240725212020506](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240725212020506.png)

constant n.常量     Curve .n曲线





关于敌人血条：

控件的创建，要清楚控件也是组件，要加在敌人头上，

1.我们只需要在控件的蓝图中添加自定义事件

2.在角色蓝图中调用控件中的自定义事件即可使用





UI控件要显示在主界面都是写在controller中，包括玩家输入（这个输入我写在玩家身上了）





技能效果的添加：

![image-20240727143724724](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240727143724724.png)

1.添加之后在GE(gameplay effect)中设置技能效果。       技能效果蓝图

2.然后在玩家蓝图里要give ability和激活ability。    玩家角色蓝图

3.另外在技能蓝图中记得提交技能                                 技能蓝图



好想没有添加技能tag,这个在以后的技能创建中再进行补充，现在能总结好被动回血已经很好了。----2024年7月27日14:51:50





UMG中，不同的UMG可以添加到一个主界面中，只需要搜索名称，即可添加

MVC  :  model,view ,control 



技能信息结构体在C++的代码中，我们在技能的一个基类UMG中有调用，用于实时将冷却时间，图标状态等信息返回到前端



# 回血技能的创建：

添加一个TAG  Ability.HPRecovery

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240728160449443.png" alt="image-20240728160449443" style="zoom:50%;" />

加开销，加CD,位置不一样，但都属于GE,还有CD属于持续的，因为一直在计时

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240728161049370.png" alt="image-20240728161049370" style="zoom:67%;" />





创建特效，特效是根据gameability的一个Cue继承的，然后进去之后需要写在这个Cue系统下的TAG，

继承之后，在Cue中重载函数：当被激活的时候触发特效释放事件

![image-20240728162143873](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240728162143873.png)

然后这个写好的蓝图如何被触发？可以在cost中“默认类设计”添加

![image-20240728162202542](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240728162202542.png)





技能创建流程： Xmind中







然后技能Dash的伤害通知方法！！！！：很推荐的的方法，类似于事件分发器，将要传递的事件添加到一个容器里，然后容器通过“事件TAG”将信息包裹，然后其他地方提取信息的时候也是通过“事件TAG"获取这个容器中的信息

玩家撞到的pawn存到数组并且分别将撞到的pawn然后发送,其中使用的函数（send gameplay Event to Actor）：

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240729193644029.png" alt="image-20240729193644029" style="zoom: 200%;" />

用cast to来判断是不是enmey类型的，转换成功就说明这个other Actor就是enemy，因为就是他本身，不成功就什么都不用管。

在技能中调用此信息，使用函数（WaitGameplayEvent）：

下图中还有一个应用上伤害的节点ApplygameplayEffectTotarget,刚好WaitGameplayEvent传来的信息就是actor，然后将targetdata取出后就能对敌人造成伤害

![image-20240729193401971](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240729193401971.png)

上面的过程连起来看就是一个send gameplay event to actor 一个wait gameplay event 一个发送，一个等待接收（**这个wait的节点注意，有可能连错哦**），接收之后就可以将其负载的数据拿出来，然后applygameplayEffect to Target(为目标应用技能效果)了。

还有，创建Event负载的时候也是需要TAG标签的，是因为这个是根据标签识别是从哪里来的Event的。

## **敌人受击效果与调用**

在敌人身上写好受击和击晕效果之后，这两个效果是在哪里执行呢？

**重要**：那个技能造成的结果就在哪里执行，这个效果是Dash将造成的结果就在Dash中实现

![image-20240729193143238](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240729193143238.png)

在敌人身上写的受击和眩晕事件。



在dash中实现调用这两个事件：

![image-20240729193235823](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240729193235823.png)

然后是技能之间的关系，我想在冲刺技能使用的时候其他技能不能打断，那么就添加一个标签：

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240729194255162.png" alt="image-20240729194255162" style="zoom:67%;" />

在技能激活的时候这个标签就激活。

然后再其技能上在Activation Blocked tags添加Dash激活时的标签即可，现在Dash将不会被回血技能打断，看白框解释，HPRecovery被阻止了

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240729194401863.png" alt="image-20240729194401863" style="zoom: 67%;" />

为什么添加技能之后CD不会动？

记得添加技能后添加上CD开始运行的函数：

![image-20240729202009359](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240729202009359.png)





新建**GameplayEffect  Asset标签**的方法，每个技能都有标签，而其技能效果也有标签：

![image-20240730145716529](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240730145716529.png)

激光击中敌人，激光不是player身上所有的碰撞怎么 判断的呢？其实逻辑是和player身上的碰撞一样的，因为这个激光的end是一个激光末尾的碰撞体检测





激光半天打不出伤害，为什么：

接收的时候节点连错了



![image-20240730171259603](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240730171259603.png)



# Ability 系统传递伤害的两种方法：

一种是激活applydamageEffectToTarget节点中调用GE_Damage,另一种是，将信息payload之后再进行

![image-20240801194624304](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240801194624304.png)



第二种这里是将碰到的敌人数组存起来之后，再将他们打包出发送出去（图1），然后在由GA的蓝图中去处理伤害逻辑（图2），其中打包和接收都要设置TAG，这样就可以很清楚且方便的知道是由哪里发出和处理

![image-20240801194511174](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240801194511174.png)

![image-20240801194904942](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240801194904942.png)





# 向前冲刺

向前冲刺的两种实现形式，一种之前学的2技能，给一个向前的冲量，另一个是使用时间轴和插值（【【UE5 | 教程 | 冲刺】虚幻引擎5中重现 哈迪斯 的 冲刺 | 游戏剖析】 https://www.bilibili.com/video/BV1pZ421N7tY/?share_source=copy_web&vd_source=448909cdfe7ff87e464eb123889e9d9a）

##  UE使用的小技巧：

1. 搜索节点“add math expression”，可以直接输入表达式，也可以在表达式中加入变量，用于参数传入

2. ALT+鼠标中键可以改变支点的位置，然后进行缩放使用之后，支点会自动回到中心位置

![image-20240801154426450](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240801154426450.png)

3. 点击视口中的物品的时候可以按F快速将物品处在视口中心位置

4. Ctrl+F快速查找所有函数或变量引用

5. 蓝图异常调试的地方：

   ![image-20240808204451911](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240808204451911.png)

   6.蓝图断点调试，可以看到堆栈

   7.<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240808205228414.png" alt="image-20240808205228414" style="zoom:67%;" />



![image-20240808205623999](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240808205623999.png)

调试相机，；是快捷键

![image-20240808210118227](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240808210118227.png)

# 反射和垃圾回收：

 

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240802185529059.png" alt="image-20240802185529059" style="zoom:67%;" />

反射：xx.generated.h文件中包含所有历史生成或者临时生成的actor

一些常见的反射宏：GENERATED_BODY(),UCLASS(),UPROPERTY(),UFUNCTION()

USTRUCT(),UENUM()等，使其具有反射能力，可以在编辑器中使用

具体UE框架学习看知乎大佬专栏：

《InsideUE4》GamePlay架构（一）Actor和Component - 大钊的文章 - 知乎
https://zhuanlan.zhihu.com/p/22833151





GAS几个负载和传递信息的节点：

send Gameplay Event to actor 

make Gameplay EventData

![image-20240806155151871](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240806155151871.png)

接受信息的节点：wait gameplay event

然后进行对其负载信息解包

break Gameplay Event Data

其中传递中的是make （可看为封包负载），然后这个解包是break

![image-20240806155439506](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240806155439506.png)

其中的Instigator：

在你提供的节点代码中，`GetInstigator` 函数被调用来获取当前Actor的Instigator。这个函数返回的是一个Pawn对象引用。如果当前Actor没有Instigator，则返回`nullptr`。

这个信息在游戏逻辑中非常有用，比如你可能想要根据Instigator来应用不同的逻辑或效果，比如伤害计算、得分分配等。例如，在射击游戏中，你可能想要根据是哪个玩家发射的子弹来计算伤害或更新分数。

如果你的游戏逻辑中需要追踪谁或什么触发了某个事件或动作，`GetInstigator` 函数就会非常有用。







# UC++中的一些神奇代码：

![image-20240808192416722](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240808192416722.png)

为什么我会找到这里，主要是：

![image-20240808192535126](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240808192535126.png)



![image-20240808201222852](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240808201222852.png)

在虚幻引擎的Gameplay Ability System（GAS）中，属性（Attributes）是通过 `FGameplayAttributeData` 结构体来定义和管理的。`FGameplayAttributeData` 结构体包含了属性的当前值、原始值、最小值、最大值等元数据。这些属性值通常与角色或玩家的状态和能力相关联，如生命值（HP）、魔法值（MP）、力量（Strength）等。

![image-20240808201236553](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240808201236553.png)	`ATTRIBUTE_ACCESSORS` 宏自动生成了相应的访问器函数，这些访问器函数允许你读取、设置和初始化属性值。





## gameplayEffect stack

GameplayEffect堆栈是一种管理GameplayEffects（游戏效果）的方式，它允许效果的叠加和堆叠，而不是简单地替换。

**堆叠规则**：

- 默认情况下，就是NONE的情况下，新的GameplayEffectSpec实例会替换旧的实例，但如果你设置了堆叠规则，新的效果会添加到堆栈中，而不是替换。

![image-20240828213921417](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240828213921417.png)

效果堆栈，可以不使用，使用的话，堆栈有两种，

一种是aggregate by source 就是 三个人每个人对其伤害都单独叠加，A 给你三层效果，B给你两层，C给你两层这样。

一种是aggregate By target 相当于三个人用相同的ge，给你的叠在一起，5个诺手分别打你一下，五层了，五个人同时血怒。



“GamePlayEffect会默认应用新的GamePlayEffectSpec实例，而不明确或不关心之前已经应用过的尚且存在个GamePlayEffectSpec实例”

这个怎么理解，就是默认情况下，直接刷新效果，不叠加。

而GamePlayEffectSpec是什么？就是GamePlayEffect的实例啊，GamePlayEffect毕竟只是我们写的效果，只有实例用到其他Actor身上才真正生效。当一个 `GameplayEffect` 被应用时，它会创建一个 `GameplayEffectSpec` 实例。这个实例包含了效果的详细信息，并且可以被应用到目标角色上。

![image-20240828214807405](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240828214807405.png)

### 堆栈持续时间刷新策略（Stack Duration Refresh Policy）

堆栈持续时间刷新策略决定了当相同效果或者相似的效果再次被应用时，如何处理效果的持续时间。

1. 刷新持续时间: 当一个新的效果被应用时，如果堆栈持续时间刷新策略被设置为刷新，则当前效果的持续时间将被重置。这意味着效果重新计时。
2.  若栈持续时间刷新策略被设置为不刷新，那么就是不影响当前计时的时间，而且堆栈内加一个周期。

### 堆栈周期重设策略（Stack Period Reset Policy）

堆栈周期重设策略决定了当相同效果或者相似的效果再次被应用时，是否重置效果的堆栈周期。

1. 重设堆栈周期：当一个新的效果被应用时，如果堆栈周期重设策略被设置为“重设”，则效果的堆叠周期将被重置。这意味着效果的堆叠计数器将被重置，玩家可以重新开始堆叠效果。
2.  不重设堆栈周期：新的效果不会影响效果的堆栈周期，效果的堆栈计数器将按原来的计数运行，直到周期结束。

### 案例

假设您有一个增加攻击力的效果，持续时间为5秒，堆叠周期为3次。

- **堆栈持续时间刷新策略**：
  - 如果设置为“刷新”，那么当玩家在5秒内再次获得这个效果时，效果的持续时间将从5秒重新开始计算。
  - 如果设置为“不刷新”，那么即使玩家在5秒内再次获得这个效果，效果的持续时间也会继续计时，直到达到原始的5秒。
- **堆栈周期重设策略**：
  - 如果设置为“重设”，那么当玩家在达到3次堆叠后再次获得这个效果时，堆叠计数器将从1开始重新计数。
  - 如果设置为“不重设”，那么即使玩家在达到3次堆叠后再次获得这个效果，堆叠计数器也会继续计数，直到达到设定的最大堆叠次数。



# GAS 部分补充：

![image-20240829205930976](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240829205930976.png)

### GameplayTags:

层次化标签，轻量的FName，可以附加在各类上做搜索条件：GamePlayEffect，GameplayCue，GameplayEventData

整个所有TAG构成一颗TAG树



### FGameplayAttributeData：

![image-20240829210908467](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240829210908467.png)

baseValue是基础值，不是最大值，但是是永久值。

Current Value是当前值，临时值

多个Attribute可以组成attributeSet Set在这里是集合的意思吧。

### gemeplaycue

![image-20240829212437080](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240829212437080.png)

Cue可以在Effect中调用也可以在GA中执行或添加

### gameplayTask 游戏异步任务：

![image-20240829212602323](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240829212602323.png)

而GameplayTask是怎么实现的呢？委托delegate

UProperty实现将C++与蓝图通信。而UFunction中blueprintCallable能够使蓝图访问C++中的方法，blueprintImplementableEvent能够让C++调用蓝图中的事件，适用于资源读取等。前面是两个双向奔赴的通信，最后这里就是到了delegate的满天飞的情书了。

#### 在playMontageAndWait中

输出引脚对应的全都是delegate

如OnCompleted ，在C++中为

```c++
UPROPERTY(BlueprintAssignable)
FPlayMontageAndWaitDelegate OnCompleted;
```

除了最上面的那个引脚是同步的，其他都是异步的。也就是说 执行该节点后会立即执行 最上面的引脚 





task是一个基础的框架，可以在GAS之外使用，只用添加一个GamePlayTaskComponent 就可以。

![image-20240829213013912](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240829213013912.png)

### gameplayEvent

![image-20240829213234090](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240829213234090.png)

### ASC

负责协调其他部件，ga,ge,gc,task,event....是技能系统运行的核心。

拥有ASC的Actor才能释放技能。	

ASC放在哪里呢？不要想当然放在character上

还可以放在Pawn和PlayerState上

如果是联机游戏通常推荐放在PlayerState上。PlayerState可以复制到各个端，同时一直存在。

而Pawn重生的话，可能会丢失。

![image-20240829213803268](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240829213803268.png)

游戏的技能逻辑本质就是一个Actor的ASC作用于另一个Actor的ASC上，进行互相增删Effect，触发Cue，互相给对方贴TAG等等。

![image-20240829214022989](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240829214022989.png)





在我的GAS项目中，这里实现了订阅者模式，绑定了一个事件，用于接收当血量，strength之类的变化的时候，这里的蓝图如下：

![image-20240829221142458](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240829221142458.png)

那么问题来了，为什么这是一个订阅者模式？

通过观察源码发现，当时在Base_Character中有写多播函数。

```C++
void ABase_Character::OnHealthAttributeChanged(const FOnAttributeChangeData& Data)
{
	HPChangeEvent.Broadcast(Data.NewValue);		
}
```

这里是自己的广播函数，但是多播的时候需要多播委托，将这几个函数一起广播出去？

### 以下是多播委托的一些特征：

1. 声明方式：一般multicast delegate 通常通过`DECLARE_DYNAMIC_MULTICAST_DELEGATE`系列宏来声明。这些宏定义了必要的类和函数，使得委托可以支持多播功能。例如`DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams`宏用于声明一个带有两个参数的多播委托。
2.  支持多个绑定 ：multicast delegate 允许多个函数或者lambda表达式绑定到同一个事件上。在事件触发的时候所有绑定的函数都会被调用，这与单播不同，后者在任何时候只允许绑定一个函数
3.  线程安全：`multicast delegate`在虚幻引擎中是线程安全的。这意味着你可以在不同的线程中安全地添加或移除绑定的函数，而不需要担心数据竞争或同步问题。
4. multicast delegate 通常通过调用broadcast方法来触发所有绑定的函数。**这个方法是多播的关键特征，确保了所有注册的监听者都会被通知。**
5. `multicast delegate`常用于实现事件驱动的设计模式，如观察者模式。在这种模式中，对象（观察者）可以订阅其他对象（被观察者）的事件，而不需要直接相互依赖。

引擎如何知道UClass中的内容？



# UE反射的原理？

在虚幻引擎中，UProperty、UFunction、UClass是核心的反射组件，他们允许引擎在运行时查询、访问和操作蓝图和C++类的成员变量和方法。以下是这些组件工作原理

## UProperty：

是UE中所有属性（变量）的基类。它用于标记和描述类的成员变量，以便引擎可以在运行时识别和处理他们。

### 原理：

**元数据标记：**在C++类定义中，通过宏UProperty()来标记成员变量，告诉虚幻的反射系统这个变量需要被序列化、复制等。

**反射信息：**UProperty宏会在编译的时候生成额外的代码，这些代码包含变量的类型、名称、属性（如可视性、编辑器配置等）信息。

**序列化/反序列化：**反射系统使用这些信息保存和加载游戏状态时序列化和反序列化属性。

## UFunction：

UFunction是所有函数的基类，用于标记和描述类的成员函数，使得这些函数可以被蓝图系统调用或者运行时动态调用。

### 原理：

**元数据标记：**通过UFunction（）来标记C++类中的成员函数，使得函数在反射系统中可见。

**生成调用信息：**UFunction（）宏在编译的时候生成代码，这些代码记录了函数的签名、参数类型、返回类型等。

**多态调用：**通过UFunction（），UE可以在运行时通过名字和参数动态调用函数，支持多态。

## UCLass：

暂无



以下是反射过程中发生了什么事情的个人总结：

使用宏声明函数或变量的时候编辑器插件UHT(unreal header tool)会分析这些声明，然后生成元数据，这些元数据包括类，函数，变量的详细信息，比如名称，类型，属性等，然后将生成的元数据注册到反射系统中，反射系统又将其存储在中央注册表，可以在运行的时候查询或使用C++中宏声明的类，变量或者函数



## 反射系统工作原理：

### 在编译时：

元数据生成：使用UCLASS()、UFUNCTION()、UPROPERTY()等宏声明类函数变量的时候，虚幻引擎的编译器插件Unreal Header Tool（UHT）会分析这些声明。 

UHT为每个声明生成元数据，这些元数据包含类、函数、变量的详细信息，比如名称、类型、属性。

### 反射信息注册：

生成的元数据被用来注册类、函数、变量到反射系统中。

反射系统又将其存储在中央注册表中，可在运行时查询。

### 蓝图编辑阶段：

被反射的函数，变量等可以在蓝图中被访问。

### 游戏运行阶段：

#### 	属性访问：

当蓝图尝试访问或者修改C++属性的时候，反射系统介入，查找与该属性对应的UProperty元数据。通过元数据，反射系统确定如何访问或修改这个属性（元数据包括属性的 类型，是否可读写，是否串行等信息）。然后执行操作，根据元数据反射执行相应的操作。属性修改，最终通过反射系统修改了C++中的属性值。

函数调用，当蓝图中function节点被执行的时候，反射系统查找该function的元数据。反射系统根据元数据调用C++中的函数

## 纯蓝图开发的游戏会造成更多的性能开销

为什么?

1. 运行时反射和解析：蓝图中每一个节点和函数调用都需要在运行时由引擎的反射系统解析和执行。这种动态解析和执行比静态编译的C++代码消耗更多的资源。

2. 内存使用：蓝图运行时需要额外的内存存储节点和数据结构，这可能导致整体内存使用增加。

3. 对于要求高帧率和低延迟的游戏，过多的蓝图逻辑可能会成为一个瓶颈

4. 优化挑战：对于蓝图的动态性质，优化更有挑战性

5. 序列化和持久化：蓝图中定义的blueprint-only可能无法被序列化或保存到磁盘，因为它们没有对应的C++数据结构。 这可能导致在游戏加载和保存时需要额外的处理，从而影响性能。

   

   

   尽管存在这些性能开销，虚幻引擎的蓝图系统仍然是一个非常强大的工具，特别是对于快速迭代和游戏原型开发。对于最终的游戏产品，关键在于找到C++代码和蓝图的最佳平衡点，以优化性能和开发效率。

# 完美闪避/完美	弹反：



在Actor上添加两个Actor组件，BPC_PropertyandState 和BPC_HitReaction

将角色上加上这两个组件。、

### BPC_PropertyandState

在BPC_PropertyandState中添加两个布尔变量

isPerfectDogeChecking用于检测是否处于完美闪避的检测中，

isPerfectdefendChecking是否是否完美弹反的检测中。

然后创建完美闪避检测帧，在开始和loop中将isPerfectDogeChecking设置为true，end中设为false

弹反的动画通知检测帧创建同理。

接着就是在蒙太奇中添加动画通知。



那既然这是个动画通知状态，当他在动画蒙太奇上被使用的时候，就会动态双向绑定？反正可以获取通过Get owner到mesh

![image-20240907162449577](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240907162449577.png)



### 编辑BPC_HitReaction组件



在这里获取角色引用和BPC_PropertyandState 组件的引用

然后创建自定义事件用于处理收到敌人攻击后的处理，这个事件图表接收两个参数：hitResult和instigator

其中一个是攻击的信息，一个是发起攻击的对象。

然后在创建一个自定义事件applyHited,在其中获取BPC_PropertyandState 组件中的isPerfectDogeChecking，看看这个布尔值此时是不是true，如果是，执行接下来完美闪避的处理操作。·

如果不是，再判断是不是在弹反的状态，并且角度正确。如果是，则弹反成功 

然后在敌人的攻击检测中，或者是武器蓝图中写的攻击检测的话，在其中调用这个applyHited事件。即可

在组件中添加事件分发器的组激活器中添加事件分发器，可以在细节中使用











tips:在碰撞之时会首先检查碰撞物体的类型，是阻挡，重叠，还是忽略，然后再根据自身的类型组合判断是实现什么效果

武器进行挥舞的时候，他是否可以进行碰撞   是通过动画通知来进行通知还是在武器上进行碰撞检测好用呢？

如果是使用完美闪避的时候，那就不用武器了因为对手攻击的时候是通过动画通知检测帧来实现是否完成闪避的



而武器碰撞组件在这里是为了检测，我操控的角色是否可以打击到对面的敌人，那既然是组件，那么也可以安装在敌人身上，用于检测是否打到我身上。是否打到我身上也可以在使用动画通知中通过通知来修改布尔值，使其碰撞生效或者失效

![image-20240907220323118](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240907220323118.png)

而在小明的项目中碰撞检测是不是敌人是通过胶囊体碰撞重叠检测来判断的。

![image-20240907221312746](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240907221312746.png)

如果说是使用通道检测碰撞的话，是可以根据碰撞类型（可以在项目设置中进行添加）来检测是否是敌人的

![image-20240907221359834](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240907221359834.png)

然后使用通道检测或者是其他检测通道来进行检测，还分为多object检测或者单object检测：

![image-20240907221526297](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240907221526297.png)





那既然我的武器伤害是通过重叠事件判断是不是敌人，然后如果是的话才通过GAS的Effect进行伤害传递，那么

我该怎么使用hitresult来实现检测呢？我要检测什么？碰撞的角度？直接在武器上可以添加射线检测吗？这个武器上的胶囊体碰撞可以用来使用hitresult吗？





我重新创建了一个武器但是就是一个静态网格体，然后根据组件思维。设置一个武器组件。然后挂载到角色身上之后，就可以将武器显示在角色身上。同时武器检测的的蓝图逻辑也会随着攻击的进行，根据动画通知状态进行触发



组件就算了，直接将Actor扔上去吧，不然还得找插槽挂载

然后创建一个武器的蓝图类，在其中写一些射线检测。用于检测是否攻击到其他Pawn。



好像知道为什么会出现敌人攻击有些反直觉的效果了，因为我的敌人攻击前摇太短了。设置动画通知事件+事件膨胀以设置攻击前摇。



# 动画中如何设计顿帧的感觉？

在动画中添加动画通知，然后在动画蓝图中实现动画0.1秒的时间膨胀？





# SaveGame and  loadGame System

​	

我有一个场景，Player可以生成cube，改变Player手中的武器，Player可以改变生命值、Player的摄像机的Transform，cube是可以移动的，他的Transform信息也是可以改变的。

我要在SaveGame的时候保存以上信息如何实现？

首先，建立SaveGame的蓝图类，然后建立结构体S_playerSave用于存储一些单个实例的信息。

然后，因为cube可能会有很多个，在SaveGame的蓝图中建立一个数组用于保存各个cube的位置。创建一个引用S_playerSave的引用。



## 有四个节点用于加载和保存游戏:

load Game from slot  加载游戏，输入引脚为slot name和userIndex ，输出引脚为return Value就是SaveGame蓝图中保存的信息

save game from slot  保存游戏，引脚： SaveGame Object 用于保存相关信息。slot name用于输入存档名称，可能我们会有很多不同的存档。userIndex 玩家下标，有时玩家不止一个。   此节点将游戏信息保存为文档

async load game from slot   异步加载，保证游戏不中断加载，当有大量对象时使用

async save game from slot   同理 想要保存大量东西的时候，让其在后台保存，这样游戏不会中断



那么这几个节点在什么地方使用呢？GameInstance 因为这个是游戏启动的时候创建，且跨关卡持续存在的。

首先，创建GameInstance的蓝图的。其次，设置让项目使用我们创建的这个GameInstance。	

​	![image-20240912222409847](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240912222409847.png)

然后在GameInstance中添加变量 SaveGame的对象引用。然后获取其中的信息用于初始化。

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240915151124700.png" alt="image-20240915151124700" style="zoom:50%;" />

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240915151256517.png" alt="image-20240915151256517" style="zoom:50%;" />

因为我们要在Player中获取其他信息的时候需要引用GameInstance，而且我们知道SavedGameData一直在GameInstance中，而且我们不想使用cast to的话，就使用接口。（不是很明白，使用接口之后就可以在外部访问GameInstance的时候直接获取到SavedGameData吗？》！！！好像是的，通过调用GameInstance之后，再调用其中实现的接口。就可以获取数据）



蓝图接口使用的时候，就是我们创建了函数名称，参数信息，返回值类型等函数声明，但是我们不在里面实现它。

使用的时候，直接在使用它的蓝图里，调用这个接口，然后再进行实现，接口是有引用传递的，所以不用cast to? 



需求：在玩家碰到一个Actor碰撞体的时候会进行保存游戏的操作。

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240915160429255.png" alt="image-20240915160429255" style="zoom:50%;" />

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240915161313135.png" alt="image-20240915161313135" style="zoom:33%;" />

分析：直接在collision中检测碰撞事件中实现savePlayer的函数吗？分别将Player的各个数据分别从casttoplayer节点中拉出来？显然这并不符合单一责任原则。所以我们要找地方做这个函数并且尽量减少性能开销，那是哪里呢？蓝图接口中。设置这个savePlayer接口，然后在Player中新增一个函数，用于从cattoplayer中拉出函数的操作，然后再拉到GameInstance中实现的蓝图接口上，实现数据互通。实现保存游戏。

![image-20240914211749489](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240914211749489.png)

然后确实在运行之后实现了保存游戏，他是将游戏数据保存到文件夹中，所以我们要做的现在是如何用这些数据回到我们保存的位置呢？

需求：将保存的数据在进入游戏时还原。 在哪个蓝图里还原？需要创建新的函数用于还原操作吗？

分析：还原的话，既然数据大多是Player的，那么就在Player创建加载函数，当进入游戏的时候，从关卡蓝图或者GameInstance中获取玩家，然后调用加载函数。！ 教程演示是在Player中，当游戏开始的时候，判断是否有有数据，有的话就加载 ，没有就不加载。

需求：那如果我测试保存的数据如何删除呢？

分析：因为他是将游戏数据保存到文件夹中  ，删除以下文件夹中的信息即可

![image-20240914214208676](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240914214208676.png)

接着就是有趣的东西了，the Cubes,这些箱子的位置信息还没有保存哈哈哈

需求：these cubes location by level (not player spawn)

分析：依然是在蓝图接口中创建函数，然后再GameInstance实现。saveCube函数，首先要清空数组，然后 get actor of class 获取cube，再进行loop保存他们的Transform，最后调用下SaveGameData函数。接着就是在原来关卡开始时生成的cubes了，在关卡蓝图中生成完调用GameInstance的中实习现的函数saveCube。

需求：保存之后再次打开游戏发现没有cube了。

分析：原因是，没有在关卡蓝图中初始化获取GameData，然后重新根据之前保存在数组中的数据生成cubes（首先判断数组长度是否大于0，>0则有数据）。

需求：玩家生成箱子之后关游戏再进游戏发现没有东西

分析:因为生成箱子之后没有保存箱子信息，生成后调用saveCube函数即可,但是我们最好是使用异步保存，因为保存后还要继续游戏

需求：cube在改变位置之后，重新进入游戏后发现没有进行成功保存位置信息。

分析：在cube蓝图中设置，当位置改变的时候，将更新信息，然后SaveCube。



需求：在GAS中使用attributeSet设置HP属性之后，假如我是想在保存使用SaveGame to slot ,然后在加载游戏的时候如何将保存的值再赋给角色，我的角色HP和MAX_HP在我做保存游戏之前是使用Datatable进行赋值的，我不知道如何在load游戏的时候，将之前保存的HP值赋给当前新进入游戏的角色

分析：直接赋值好像难以实现，可以在再次进入游戏的时候通过GE进行减少属性值。实现和进入游戏之前相同血量的操作

# 创建UC++函数：

假如我想在虚幻中实现一些零碎的小功能，但是引擎并没有相关的蓝图节点，我要自己手动创建C++代码，我是不是创建一个Tools类，然后再将里面的函数做成蓝图可调用就可以实现使用了？

他给我了一个例子，创建了一个C++类，但是我不太明白为什么函数中要使用static

例子如下：

```c#
UCLASS()
class YOURGAME_API UTools : public UObject
{
    GENERATED_BODY()

public:
    UTOOLS_API UTools();

    UFUNCTION(BlueprintCallable, Category = "Tools")
    static void YourFunction();
};

```

创建过程：1. 创建c++类，然后实现函数代码的编写。 2. 编译代码 3. 在蓝图中使用创建的代码



### 疑问：

1. 为什么使用static void 编写我的函数？

### 分析：

1. 静态成员函数不依赖类的实现：static函数属于类，而不是类的实例。这意味着你可以在不创建类的实例的情况下调用这些函数。这在您需要一个不依赖特定对象的通用功能时非常有用。
2. 易在蓝图中调用：蓝图中调用静态函数通常更简单，因为不需要首先创建一个类的实例，直接通过雷明就可以调用静态函数。
3. 性能考虑：静态函数调用通常比实例函数更快，因为不需要通过对象指针访问
4. 注意！！！静态函数不能直接访问类的非静态成员（属性或函数），因为它们不依赖于特定的对象实例。如果您的函数需要访问或修改类的成员变量，那么它不应该被声明为静态的。

2.YOURGAME_API是什么东西，见了很多次了。





# 蓝图接口的使用：

3个方面：1.发起者：Player    2.中间商 ：蓝图接口   3实现者：对应实现蓝图接口的Actor或者Pawn

在角色中通过输入调用蓝图接口message，然后在实现者身上实现蓝图接口，然后完善其中的功能。

### 疑问：

一个蓝图接口可以有很多函数，那么怎么平衡蓝图接口的创建和哪些函数创建在哪个接口中呢

# GAS中GamePlayEffect的使用：

GamePlayEffect中如果要从外界传递伤害给owner或者是target，这里使用节点有`make outgoing Gameplay Effect spec`,`assign tag set by caller	magnitude`和`applyGamePlayEffecttoOwener`.这几个节点。这里呢在GamePlayEffect中设置如图：

![image-20240920222230999](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240920222230999.png)

然后GA中的几个节点如图：

![image-20240920222728806](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240920222728806.png)
