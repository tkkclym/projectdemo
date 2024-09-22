# Uobject 与Actor

Uobject 是最基本的基类，派生出AActor,

脱胎自Object的Actor也多了一些本事：Replication（网络复制）,Spawn（生生死死），Tick(有了心跳)。        （//网络复制是什么）

Actor并不只代表我们在视口中所看到的东西，包括HUD,蓝图，AGameMode,AGameSession,APlayerState,AGameState等组件等都是Actor



Actor有没有带Transform？经过了UE的权衡和考虑，把Transform封装进了SceneComponent,当作RootComponent。大部分Actor其实是有Transform的，我们会经常获取设置它的坐标**，如果总是得先获取一下SceneComponent，然后再调用相应接口的话，那也太繁琐了**。所以UE也为了我们直接提供了一些便利性的Actor方法，如(Get/Set)ActorLocation等，其实内部都是转发到RootComponent。

**思考：Actor的SceneComponent哲学**

UE中，并不是一个车和四个轮子都是Node,而是一个NOde和四个sceneComponment.

把要操作的实体按照功能划分，而其他的就尽量只是最简单的表示。所以在UE里，其实是把5个薄薄的SceneComponent表示再用Actor功能的盒子装了起来，而在这个盒子内部你可以编写操作这5个对象的逻辑。

Actor之间的父子关系：

Actor中的父子关系是通过Component确定的。同一般的Parent:AddChild操作原语不同，UE里是通过Child:AttachToActor或Child:AttachToComponent来创建父子连接的。

Actor父子之间的“关系”其实隐含了许多数据，而这些数据都是在Component上提供的。Actor其实更像是一个容器，只提供了基本的创建销毁，网络复制，事件触发等一些逻辑性的功能，而把父子的关系维护都交给了具体的Component，所以更准确的说，其实是不同Actor的SceneComponent之间有父子关系，而Actor本身其实并不太关心。

# level 和world

多个level组成一个world	

ulevel属于uobject，所以可以有一定智能行为，比如蓝图的编译，而在关卡蓝图中中可以将对其中所有的Actor进行调用和呼喊，关卡蓝图实际上就代表着该片大陆上的运行规则。而在关卡之外也有一些东西控制这这片土地，AInfo和其之类，跟Level直接相关的一位书记官：AWorldSettings。

关卡蓝图其实也是一个Actor，但是没必要告诉使用者他是，因为他并不需要一些复杂的逻辑控制，逻辑控制更多的交给了GameMode和Controller，

复习下uml不然看不懂专栏在说什么

![image-20240802214531424](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240802214531424.png)

虚幻源码：

```c++
void ULevel::SortActorList()
{
    //[...]
    TArray<AActor*> NewActors;//创建数组，非网络可复制类型
    TArray<AActor*> NewNetActors;//创建数组，网络可复制类型
    NewActors.Reserve(Actors.Num());
    NewNetActors.Reserve(Actors.Num());
    // The WorldSettings tries to stay at index 0
    NewActors.Add(WorldSettings);
    // Add non-net actors to the NewActors immediately, cache off the net actors to Append after
    for (AActor* Actor : Actors)
    {
        if (Actor != nullptr && Actor != WorldSettings && !Actor->IsPendingKill())
        {
            if (IsNetActor(Actor))
            {
                NewNetActors.Add(Actor);
            }
            else
            {
                NewActors.Add(Actor);
            }
        }
    }
    iFirstNetRelevantActor = NewActors.Num();//这个可以代表非网络的数组最后元素的后一个位置
    NewActors.Append(MoveTemp(NewNetActors));//将网络可复制的Actor添加到不是网络可复制的后面
    Actors = MoveTemp(NewActors);   // Replace with sorted list.
    // Add all network actors to the owning world
    //[...]
}
```

在level中的Actor排序是非网络复制的Actor在前面，网络Actor在后面

在您的代码片段中，通过将非网络演员和网络演员分开，并且记录了第一个网络相关演员的索引位置 `iFirstNetRelevantActor`，引擎在处理网络复制时可以快速定位到需要复制的演员。这种做法相当于为网络演员创建了一个“快速访问”区域，因为它们都被集中放置在数组的一个特定部分。

这样做的好处包括：

1. **减少搜索时间**：当需要进行网络复制时，引擎不需要遍历整个演员列表来找到网络演员，而是可以直接从 `iFirstNetRelevantActor` 开始处理。这减少了搜索和处理非网络演员所需的时间。
2. **优化数据访问**：由于网络演员被集中在一起，这可能会对内存访问模式产生积极影响，尤其是在现代计算机体系结构中，连续的内存访问通常比随机访问要快得多。
3. **简化逻辑**：通过明确地划分网络和非网络演员，网络复制的逻辑变得更加简单和直接。引擎可以更自信地处理这部分数据，因为它知道这部分数据是网络复制所关心的。
4. **提高性能**：在游戏开发中，性能是一个关键因素。通过减少不必要的遍历和处理，可以提高游戏的整体性能，尤其是在网络密集型游戏中，这种优化尤为重要。



我们可以在world列表中看到所有的Actor物体，但是并不代表我们可以world直接引用了level中的actors,TActorItratorBase(World的Actor迭代器)内部实现也只是遍历Levels来获取所有Actor

可以将level通过subLevel联系起来，然后有刚进游戏就加载的level（persistent）和后续通过Streaming后续动态加载的，PersistentLevel和CurrentLevel只是个快速引用。在编辑器里编辑的时候，CurrentLevel可以指向其他Level，但运行时CurrentLevel只能是指向PersistentLevel。

**思考：为何要有主PersistentLevel？**

出生地啊，必须有一个，那么其他levels拼接进来之后用的是谁的worldsetting呢？由源码可以看出World的Settings也是以PersistentLevel为主的，但这也并不意味着其他Level的Settings就完全没有作用了

总结：level实现了平摊地图的加载和动态释放，使玩家游玩时不会觉得明显卡顿，Level作为Actor的容器，同时也划分了World，一方面支持了Level的动态加载，另一方面也允许了团队的实时协作，大家可以同时并行编辑不同的Level。一般而言，一个玩家从游戏开始到结束，UE会创造一个GameWorld给玩家并一直存在。玩家切换场景或关卡，也只是在这个World中加载释放不同的Level。

#  WorldContext，GameInstance，Engine

world并不是只有一个，在虚幻中有很多世界类型，通过枚举的方式展示

```c++
namespace EWorldType
{
	enum Type
	{
		None,		// An untyped world, in most cases this will be the vestigial worlds of streamed in sub-levels
		Game,		// The game world
		Editor,		// A world being edited in the editor
		PIE,		// A Play In Editor world
		Preview,	// A preview world for an editor tool
		Inactive	// An editor world that was loaded but not currently being edited in the level editor
	};
}
```

UE怎么管理这些不同的世界呢  WorldContext

![image-20240804152516834](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240804152516834.png)

worldcontext 使用thiscurrentWorld 指向当前的世界，当一个world切换到另一个world的时候，比如点击播放时preview切换到PIE界面的时候，FworldContext就用来保存切换过程信息和保存上下文信息。所以一般在切换的时候 ，比如openlevel，也都会需要传FWorldContext的参数。一般来说，独立运行的游戏只有一个worldcontext，而在编辑的时候是有两个，一个WorldContext给编辑器pre，一个FWorldContext给PIE，一般引擎内部已经处理好各种world的协作，我们不需要处理，而且FWorldContext还保留着World里level切换的上下文。

```c++
struct FWorldContext
{
    [...]
	TEnumAsByte<EWorldType::Type>	WorldType;

	FSeamlessTravelHandler SeamlessTravelHandler;
//SeamlessTravelHandler用于处理无缝旅行，这是一种在不关闭游戏的情况下从一个关卡（Level）移动到另一个关卡的功能。
	FName ContextHandle;
//这个变量的作用是提供一个唯一的标识符，用于在引擎内部标识和引用特定的 FWorldContext 实例。
    //FName是用于存储唯一标识符的轻量级字符串表示形式，它使用了内部字符串池来避免重复存储相同的字符串
    
	/** URL to travel to for pending client connect */
	FString TravelURL;

	/** TravelType for pending client connects */
	uint8 TravelType;
    //它还保存了与旅行相关的信息，如TravelURL和TravelType，这些信息用于处理客户端连接和关卡之间的旅行。
    

	/** URL the last time we traveled */
	UPROPERTY()
	struct FURL LastURL;

	/** last server we connected to (for "reconnect" command) */

	UPROPERTY()
	struct FURL LastRemoteURL;
    //LastURL和LastRemoteURL记录了最近的URL信息，这对于重连到上一个服务器或关卡非常有用。
}
```

UE在openlevel的时候，先设置当前World的context上的travelURL，然后在UEngine::TickWorldTravel的时候判断TravelURL非空来真正执行Level的切换。其中细节较为复杂，但是总之，WorldContext负责World的切换也负责level之间的切换

## 为什么level的切换信息不放在World中？

UE的逻辑是openlevel的时候会释放掉当前世界，然后创建一个新的世界，所以如果我们如果把下一个level的信息放在当前世界中就不得不在释放当前World之前又拷贝回来一遍了。而在loadstreamLevel的时候，就只是在当前World中载入对象了：

**Stream Level的特性**：

- `LoadStreamLevel` 函数允许在当前世界的基础上动态加载和卸载级别，而不需要销毁整个世界。这是因为流式加载的级别不是 `PersistentLevel`，它们是临时的，可以独立于主游戏级别存在。
- 由于流式级别是在现有世界内加载的，因此它们的加载信息可以保存在 `World` 中，因为 `World` 不会在这个过程中被销毁。

![image-20240818153822952](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240818153822952.png)

比如黑猴拿那个飞龙鳞片的时候，就是loadstreamLevel，从一个大关卡转到另一个大关卡，黑屏的时候应该就是openlevel的使用 

![image-20240818153855328](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240818153855328.png)

LoadStreamLevel：

```c++
void UGameplayStatics::LoadStreamLevel(UObject* WorldContextObject, FName LevelName,bool bMakeVisibleAfterLoad,bool bShouldBlockOnLoad,FLatentActionInfo LatentInfo)
{
	if (UWorld* World = GEngine->GetWorldFromContextObject(WorldContextObject))
	{
		FLatentActionManager& LatentManager = World->GetLatentActionManager();
		if (LatentManager.FindExistingAction<FStreamLevelAction>(LatentInfo.CallbackTarget, LatentInfo.UUID) == nullptr)
		{
			FStreamLevelAction* NewAction = new FStreamLevelAction(true, LevelName, bMakeVisibleAfterLoad, bShouldBlockOnLoad, LatentInfo, World);
			LatentManager.AddNewAction(LatentInfo.CallbackTarget, LatentInfo.UUID, NewAction);
		}
	}
}
```



## 为何World和Level的切换要放在下一帧再执行？

1.**确保当前帧的稳定性** 如果当前帧执行重负载操作（如加载新关卡），可能导致帧率下降，影响游戏体验。通过将这类操作推迟到下一帧，可以避免处理当前帧的时候出现性能问题。

2**.允许当前帧完成**  可能当前帧有一些游戏逻辑需要处理，而将World和level的切换放在下一帧可以确保当前帧的所有任务都能顺利完成，避免突然中断而导致的逻辑错误或者不一致

3.**延时执行和回调处理**  延时动作管理，UE有延时动作系统（FlatentActionManager）,允许开发者安排未来时间点执行特定操作。将World和level切换安排在下一帧，可以用这个系统来管理加载操作，并提供回调机制，以便加载完成后执行特定逻辑

4.**同步和线程安全** 多线程考虑，UE是多线程的，游戏逻辑，渲染，物理模拟等多个系统可能同时在不同线程运行。将World和level切换安排在下一帧可以更好地同步这些线程确保执行切换时不会发生资源冲突或不一致的状态。

5**.用户输入相应  避免输入丢失**   如果在处理用户输入的同一帧执行World和Level的切换，可能会丢失用户在这一帧中输入的某些操作。推迟到下一帧可以确保用户的所有输入都被正确处理。

6.**游戏状态的正确管理** 状态保存：在切换World和Level之前，可能需要保存当前游戏状态。将这个操作放在下一帧，可以让游戏有机会在切换之前保存必要的状态信息，如玩家的位置、分数、游戏进度等。

# GameInstance:

worldContext保存在哪里？

![img](https://pic4.zhimg.com/80/v2-36b45a7b36ac77d978719bc6fe8db17b_720w.webp)

GameInstance中保存着当前的WorldContext和其他所有游戏信息，那些独立于level的逻辑以及数据都存储在GameInstance中。

UE的GameInstance因为继承于Uobject，所以有动态创建的能力，所以我们可以通过指定GameInstanceClass来让UE创建使用我们自定义的GameInstance子类。无论蓝图还是cpp,我们通常继承GameInstance然后再在其中编写应用整个游戏范围的逻辑

我的Level切换了，变量数据就丟了，我应该把那些数据放在哪？再清晰直白一点，GameInstance就是你不管Level怎么切换，还是会一直存在的那个对象！

engine：

![img](https://pic1.zhimg.com/v2-94d1f4e3750b6f4fd09d02b20bc980b0_r.jpg)

此处UEngine分化出两个子类，UE的编辑器也是由自己的引擎渲染出来的，采用的也是slate的ui框架，本质上UE编辑器本身也是个游戏，但是还是和有些差别的，所以UE在不同模式下根据编译环境而采用不同的具体engine类，而基类uengine里通过一个WorldList保存所有world.



# 切换关卡时发生的事情：

切换关卡的时候一般都是使用openlevel,`OpenLevel` 函数会销毁当前的世界，并创建一个新的世界来加载新的关卡。相比之下，`LoadStreamLevel` 函数可以在不销毁当前世界的情况下加载新的关卡，这通常用于加载子关卡或动态内容。

当使用 `OpenLevel` 函数切换关卡并销毁当前世界时，以下内容会被销毁，而玩家信息会被重新创建并存储在新的位置：

1. **World对象**：当前的世界对象会被销毁，包括其中的所有关卡、实体和资源。
2. **PlayerController**：当前的 `PlayerController` 会被销毁，因为它与被销毁的世界关联。
3. **GameMode**：当前的 `GameMode` 也会被销毁，因为它是与当前世界关联的。
4. **Levels**：当前世界中的所有关卡和内容都会被销毁

openlevel切换关卡的时候，playerState不会被销毁灭，这是因为，playerState是和玩家角色关联的，而玩家角色在关卡切换时被重新创建。新关卡创建的时候会重新创建一个playerController ，playerState被存储在 `PlayerController` 中，所以playerState就被构建出来了

## 为了防止在Unreal Engine中世界销毁时数据丢失

你可以采取以下措施：

1. **使用PlayerState保存玩家数据**：
   - 确保玩家的关键数据存储在 `PlayerState` 中，因为它是**跨关卡持久存在**的。在关卡切换前，将数据保存到 `PlayerState` 中。
2. **使用SaveGame系统**：
   - 对于需要长期保存的数据，如玩家的进度、配置等，使用 `USaveGame` 系统将数据序列化到磁盘上。这可以在游戏会话之间持久保存数据。
3. **使用GameInstance**：
   - 将需要跨关卡持久存在的数据存储在 `AGameInstance` 类中。由于 `AGameInstance` 是在整个游戏会话中持续存在的，它可以用来存储全局数据。
4. **使用配置文件**：
   - 将玩家设置和偏好等数据存储在配置文件中。这些文件可以在游戏重新启动时加载。
5. **在适当的时机保存数据**：
   - 在 `PlayerController` 的 `BeginPlay` 方法中保存数据，确保在玩家开始新关卡之前数据已经被保存。
6. **在适当的时机恢复数据**：
   - 在 `PlayerController` 的 `PostLogin` 方法中恢复数据，确保在新关卡开始时数据已经被加载。
7. **使用SaveSlot和LoadSlot**：
   - 使用 `SaveSlot` 和 `LoadSlot` 方法来控制数据的保存和加载，这样可以确保数据只在必要时被保存或加载。
8. **避免在关卡切换时执行耗时操作**：
   - 避免在关卡切换时执行耗时的操作，如复杂的逻辑计算或资源加载。这可以确保数据在关卡切换时已经被保存。

通过以上措施，你可以有效地防止在Unreal Engine中世界销毁时数据丢失。

# GamePlayStatics：

既然我们在引擎内部C++层次已经有了访问World操作Level的能力，那么在暴露出的蓝图系统里，UE为了我们的使用方便，也在Engine层次为我们提供了便利操作蓝图函数库。

```c++
UCLASS ()
class UGameplayStatics : public UBlueprintFunctionLibrary 
```

这个类相当于C++静态类，只为蓝图提供一些静态方法，我们所见的GetPlayerController、spawnActor和openlevel等都是来自于这个类的接口

# Pawn



### component

`UActorComponent` 继承自 `UObject`。在虚幻引擎中，`UActorComponent` 是 `Actor` 类的一部分，它代表了一个可以附加到 `Actor` 上的组件。这些组件可以添加额外的功能和属性，如碰撞体、渲染、动画、物理交互等。

Actor可以说是有component组成的，component表达的是功能的概念。

正确理解“功能”和“游戏业务逻辑”的区分是理解Component的关键要点。

我们做component的目标是，他可以无痛的，直接的，迁移到下一个游戏项目中，这才是一个完美的component。

一旦你发现component中有业务逻辑，就要警惕游戏架构是否恰当了。

### Actor

UE中最大的民族。我们用的最多的就是staticMeshActor和blueprintActor了，我们创建一个Actor之后，就将各种逻辑，这种function和各种variable，Event全写在Actor中，但是这真的对吗？

**游戏逻辑写在哪里？**

UE在思考这个问题时，却是感觉有些理想主义，颇有些C++的理念，力求不为你不需要的东西付代价，宁愿有时候折中，也想保住最优性能。UE的架构中也大量应用了各种继承，有些继承链也能拉得很长，同时一方面也吸纳了组合的优点，我们也能见到UE的源码中类的成员变量也是组合了好多其他对象。所以接下来的该介绍的就是UE综合应用这两种思想的设计产物。面向对象派生下来的Pawn和Character，支持组合的Controller们。

### Pawn

问题来啦，哪些Pawn需要附加逻辑？

飞行的小鸟，游泳的野猪，和山上的跳跳虎。这些不由玩家操控的且自主运动的是需要逻辑的。这些都属于AI,所以使用AIController

Pawn就是兵卒，Pawn用什么表达自身的存在？首先就是自身的存在physicsCollision,然后可以移动movementinput，然后就是响应输入和处理逻辑Controller。你也可以想象成提线木偶，那个木偶就是Pawn，而提线的是Controller

**为何Actor也能接受Input事件？**

Pawn被控制是理所当然，你猜为什么enableinput接口是在Actor中的，同时InputComponent也是在Actor中的，意味着实际上，你可以在Actor中绑定处理输入事件。

1.因为“输入”的种类有很多，（按键、摇杆、陀螺仪等），我们不确定和分类哪些Actor子类该接受那种输入对象。同时Actor又是有Component装而成，UE不可能为了输入的处理改变Component的组织方式，还不如直接在Actor基类中提供Inputcomponent,这样反而更灵活

### DefaultPawn，SpectatorPawn，Character

![img](https://pic4.zhimg.com/80/v2-e3e8606aa67344bf178fd9097d249693_720w.webp)

### DefaultPawn

默认携带Pawn的三件套  DefaultPawnMovementComponent、spherical CollisionComponent和StaticMeshComponent

### SpectatorPawn

可以自由移动观战的相机，USpectatorPawnMovement（不带重力漫游），同时隐藏了mesh的显示，碰撞也设置到了“Spectator”通道

### character

就是继承Pawn的人形Actor。有像人行走的CharacterMovementComponent， 尽量贴合的CapsuleComponent，再加上骨骼上蒙皮的网格。。一般来说，如果你控制的角色是人形的带骨骼的，那就选择Character吧



## Controller

MVC模式 model view controller 

MVC当然有优缺点，我们探讨的是为什么UE选择了他

### AController

我们希望Controller拥有众多本领。 2.能够控制Pawn的生死。3.多个控制实例，有需要的时候可以克隆出多个自我 4.可以释放挂载，可以挂载pawnA身上也可以挂载在PawnB身上。5.可以单独存在。6.可以控制pawn的生死，让其销毁或者产生一堆。7.事件响应，如果有来自玩家或者游戏内的事件传来我会进行处理，包括我控制的Pawn有事情传过来我也要处理。8.持续运行。9.自身有状态。10具有各种扩展继承组合能力。	11.保存数据状态	12.可以在世界中移动。大将军，可以上战场，也可以运筹帷幄。13.可以查看世界中的对象。千里眼	14.网络同步，这年头必须得有网络同步啊

所以想想从哪里继承来比较好，

1.Component又层级太低，表达的事概念，不是逻辑。2.Component必须依附于Actor存在，而我们的Controller希望能独立存在。

Uobject的话，还得把Actor有的各种特性实现一遍（配置动态生成、输入事件响应、Tick、可继承、可容纳Component、可在世界里出现、可在网络间同步）

​	好了，现在就差控制Pawn的能力，那我们就在这个分化出来的AController增加一些控制Pawn的接口，这个思路正是和我们从Actor从分化出Pawn的时候不谋而合！现在我们来看看UE里的AController:

<img src="https://pic2.zhimg.com/80/v2-4151952d1f2ab74fcc78d7c3bd215e0d_720w.webp" alt="img" style="zoom:67%;" />

**Controller必须是1对1的关系吗？**

​	![image-20240818214837879](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240818214837879.png)

可以看到Controller中只保存了一个Pawn指针，而不是数组。所以当我们玩实时策略游戏的时候，一次性选择好多个Pawn，会比较僵硬，所以，就需要在Controller里自己实现扩展一下，额外保存多个Pawn，然后自己实现一些需要的控制实现。

**那为什么不是做成多对多呢？**

不行，引擎需要一定的灵活。我们可以直接在Controller中getPawn，如果是多对多的话，以下获取太多就太乱了，太灵活对开发和调试不友好，架构的开发往往是在权衡各个方面的利弊。设计架构游戏的时候，工程师们要抵挡住灵活性的诱惑，保持克制往往是更难得珍贵的美德。要认识到，引擎的终极目的是方便人使用的，我们程序员往往很容易太沉迷于程序功能的灵活强大，而疏忽了易用性鲁棒性等社会工程需求。

#### VS使用技巧，知道函数名，按下ctrl+F可以去寻找



**思考：为何Controller不能像Actor层级嵌套？**

如果Controller有父子机制，那怎么实现控制呢?

controller表达的“控制”的概念，所以在这里你实际上想要表达的是一种“控制”互相嵌套的概念，感觉又给“控制”给分了层，有“大控制”，也有“小控制”，但是“控制”的“大小”又是个什么概念呢？我们应该怎么划分控制的大小？

“控制”本质上来说就是一些代码，不管怎么设计，目的都是用来表达游戏游戏逻辑的。而针对游戏逻辑的复杂，怎么更好的管理组织逻辑代码，我们有状态机，分层状态机，行为树，GOAL（目标导向），甚至你还能搞些神经网络遗传算法机器学习啥的。所以在我们已经有这么多工具的基础上，徒增复杂性是很危险的做法。如果有必要，也可以把Controller本身再当作其他AI算法的容器，所以就没必要在对象层次上再做文章了。

**controller可以显示吗？**

默认是不显示的，但是在源码中改变布尔值之后，再在Controller中加些Actor是可以显示的。

**思考：Controller的位置有什么意义？**

首先说rotation，若要将Pawn和Controller保持朝向一致，可以设置Pawn跟随Controller转向。再说location。上面说Controller可以spawn出Pawn，所以我要生成在什么位置要可控，比如骷髅女王在周围生成一堆小骷髅兵。

如果Controller有自己的位置，这样在Respawn重新生成Pawn的时候，你就可以选择在当前位置创建。因此为了自动更新Controller的位置，UE还提供了一个bAttachToPawn的开关选项，默认是关闭的，UE不会自动的更新Controller的位置信息；而如果打开，就会把Controller附加到Pawn的子节点里面去，让Controller跟随Pawn来移动。

![image-20240818222215992](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240818222215992.png)

### **哪些逻辑应该写在Controller中？**

从概念讲 Pawn中固有的逻辑可以写在Pawn中，比如跑跳的动画，碰撞检测。可以替换的逻辑或者需要智能决策的写在controller中，比如技能。

从对应关系上讲，如果是独特的逻辑，就写在Pawn中，如果是有其他Pawn可以一起使用的就写在controller中。举个例子，在战争游戏中，假设说有坦克和卡车两种战车（Pawn），只有坦克可以开炮，那么开炮这个功能你就可以直接实现在坦克Pawn上。而这两辆战车都有的自动寻找攻击目标功能，就可以实现在一个Controller里。

从存在性讲 controller生存周期比Pawn长，比如游戏中死亡后复活的功能，Pawn死后就被销毁了，就算之后再Respawn创建出来一个新的，但是Pawn身上保存的变量状态都已经被重置了。所以对于那些需要在Pawn之外还要持续存在的逻辑和状态，放进Controller中是更好的选择。

## APlayerState



controller需要有一些记忆，用于记住一些游戏状态。controller中可以获取玩家状态的话会大为方便，所以controller中有了AplayerState指针

```c++
 /** PlayerState containing replicated information about the player using this controller (only exists for players, not NPCs). */
    UPROPERTY(replicatedUsing=OnRep_PlayerState, BlueprintReadOnly, Category="Controller")
    class APlayerState* PlayerState;
```

而AplayerState是继承于AActor派生来的AInfo(这个只记录数据的书记)，所以他要的就是网络复制功能。

而这个PlayerState我们可以通过在GameMode中配置的PlayerStateClass来自动生成。这个APlayerState也理所当然是生成在Level中的，跟Pawn和Controller是平级的关系，controller只是保存了一个指针引用。这里看注释说是playerState只为player生成，不为NPC生成，所以有多少玩家就有多少playerState。

但是，UE把PlayerState的引用变量放在了Controller一级，而不是PlayerController之中，说明了其实AIController也是可以设置读取该变量的。一个AI智能能够读取玩家的比分等状态，有了更多的信息来作决策，想来也没有什么不对嘛。

## PlayerController

<img src="https://pic1.zhimg.com/80/v2-88131e55febd8886e0f3c87b92c526e8_720w.webp" alt="img" style="zoom:50%;" />

playerController是直接与玩家打交道的类，所以是UE使用的最多的类，playerController中有大致以下几个模块，Camera的管理，Input系统，Uplayer的关联（只有收服一个player，Controller才能正常工作嘛），HUD显示（我的项目中也用到了，创建一个widget，然后再选择事先创建好的控件类，将其添加到视口即可），Level的切换

![image-20240819212131431](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240819212131431.png)

那些逻辑应该放进playerController中？比如马里奥获取金币。马里奥获取金币然后将金币的数量显示在窗口UI上，此时View就是UI上的金币数量，Controller就是玩家控制马里奥的左右上下跳动，而马里奥（Pawn）通过触碰金币的事件又上报给PlayerController来相应增加金币。而Controller存储金币的数据就是在playerState中。即playerState中有一个int coin ,也有响应的add coin ，而playerController的职责就是一边控制Pawn，一遍负责内部正确的调用playerState的coin接口。 

那么，playerController中的变量有什么用呢？根据单一职责原理，我们写在哪个类的变量就应该尽量只符合该类的作用，所以playerController中的变量在于实现更好地控制。比如一个玩家在一个关卡中可以按AABB实现那作弊获取100金币，并且最多作弊三次，  所以我们可以将这个逻辑写在Controller中，并且在Controller中设置cheatCount变量 ，其最大值为3.然后每次调用playState中addcoin（100）。

playerController可以被替换，不同关卡中可能不一样。比如在某些游戏在水下的控制就和陆地不同，所以不能像player单件类那样什么都往里面塞。

在任意时刻，player：playerController：playerState都是1:1:1 ，但是playerController可以有多个切换备用，PlayerState也可以相应多个切换。player可以看做游戏里的一个全局的玩家逻辑实体，playerController代表玩家意识，playerState代表玩家状态。



AAIcontroller：

<img src="https://pic3.zhimg.com/80/v2-a0c2148ff8331da1b70ab4157e19f1c2_720w.webp" alt="img" style="zoom:67%;" />

对比发现相较于playerController，没有了Camera，Input，Uplayer关联，HUD显示，Voice和Level切换接口，但是也增加了AI需要的组件：

Navigation，用于智能导航寻路。

AI组件。

Task任务系统。

**哪些逻辑可以放在AIcontroller上呢？**

因为AIController都是在关卡内比较短暂存在的，一般不太有跨Level的数据保存，所以你可以用AIController的成员变量来保存状态。而如果真的需要用到PlayerController的状态，则也可以引用一个PlayerState过来。（这里是AI获取玩家的状态之后调整攻击方式什么的吗？有点智能了）。如果想引用关卡的全局状态，也可以引用GameState，再更高级别的，甚至可以直接和GameInstance接触。

但是AIController也可以通过bWantsPlayerState来获取自己的PlayerState，所以PlayerState其实也并不是跟Uplayer绑定的，毕竟从本质上讲，APlayerState也只是个AInfo（AActor），跟其他Actor一样，可以有很多个。区别是你如何创建并使用它。

![img](https://pic1.zhimg.com/80/v2-3bd34e0947e07fe6b4e54b025977b3ac_720w.webp)

tips：PlayerState存储状态数据，支持网络同步，有点怕忘了。



# GameMode

以上我们知道了Controller和其控制的Actor。

然而世界也是有各种Actor组成的，那Actor和Level有什么关系呢？UE是怎么控制的呢？

UE中，游戏是由一个个world构成的，world又是有一个个Level构成，

从游戏本身机制分析：

1.游戏或玩家的节奏，具有关卡结束的切换感。

2.游戏机制，游戏在场景会有不同的游戏玩法。

3.游戏资源的替换，不同的场景中相同的游戏玩法

所以在思考关卡的时候，要保持头脑清晰，分清什么是“逻辑”，什么是“表示”。逻辑就是游戏玩法，表示就是场景。所以以逻辑划分我们得到了一个个world，以关卡划分我们得到一个个Level

在UE中，他的世界观就是world更多的是逻辑概念，Level是资源场景表示。

通常我们简单的游戏用一个Level就够了也因此这里的Level常常对应游戏中的关卡，因此UE中的Level的Settings叫做worldSettings了

所以当我们在谈论关卡中的逻辑控制的时候，我们常常谈的是World的业务逻辑。那根据上一个Controller如何设计出来的，我们可以顺势想到，那Level的逻辑是什么控制的呢？而一个世界的逻辑控制也并不需要渲染任何1东西，需要渲染的都在Level中，这个世界的控制叫做gamemode。它派生于AInfo。下图是经过更新的，将一些基础的东西抽到基类中，如果想实现一个简单的单机GameMode可以直接从AGameModeBase中继承。

![img](https://pic1.zhimg.com/80/v2-77a448b9caf4758f1b45cbee9bddb1e8_720w.webp)

既然承担游戏逻辑，那么他就应该是AInfo家族最重要的扛把子；他身上实有许多接口，主要分为以下几大类：

### Class登记：

UE中允许gamemode在游戏登录的时候登录或注册需要的各种信息，比如playerController。AIController和玩家角色等，这些被记录后可以在游戏运行的时候通过UClass的反射机制**自动实例化**并添加到游戏世界中。

​		在虚幻引擎中，创建类（如`PlayerController`、`AIController`和`PlayerPawn`）只是定义了它们的结构和行为。要使这些类在游戏实际中应用，需要将其实例化，实例化是创建游戏对象的过程，它涉及到分配内存和初始化对象。在UE中这些通常通过UClass反射机制完成。**UClass是一个类。用于表示虚幻引擎中所有可实例化的类。他允许通过类的名称动态的创建和实例化对象，而无需在代码中明确指定每个对象的类型**

### 游戏内实体的Spawn 

GameMode作为一场游戏的主要负责人，游戏加载过程中涉及实体的产生，包括Pawn和playerController。

### 游戏进度

游戏暂停开始等也是由gamemode控制，其中包含setPause，RestartPlayer等函数可以控制相关逻辑

### Level的切换

或者说是World的切换，gamemode也决定了进入游戏是否播放开场动画（cinematic），也决定了下一个关卡是否bUseSeamlessTravel，一旦开启，可以重载Gamemode和playerController的GetSeamlessTravelActorList方法和GetSeamlessTravelActorList来指定哪些Actors不被释放而进入下一个World的Level

**多个Level配置不同的Gamemode的时候采用的是哪一个GameMode？**

我们知道除了配置全局的GameModeClass之外，还可以为每个Level单独配置不同的GameModeClass。但是当一个World由多个level组成的时候相当于配置了多个GameModeClass。用的是哪一个呢？一个World里只会有一个GameMode实例，否则肯定乱套了。

因此当有多个level的时候一定是persistentLevel和多个StreamingLevel，这时候就算他们配置了不同的GameModeClass。UE也只会在第一次创建World的时候创建persistentLLevel的GameMode在后续的LoadStreamingLevel时候，并不会再动态创建出别的GameMode，所以GameMode从始至终只有一个，PersistentLevel的那个。

#### **Level迁移时GameMode是否保持一致？**

根据两个Level配置的GameMode是否相同决定。

在travelling的时候，如果下一个Level的配置的GameModeClass和当前的不同，那么迁移后是哪个GameMode？
**无论travelling采用哪种方式，当前的World都会被释放掉，然后加载创建新的World。**但这个过程中，有点区别的是根据bUseSeamlessTravel的不同，UE可以选择哪些Actor迁移到下一个World中去（实现方式是先创建个中间过渡World进行二段迁移（为了避免同时加载进两个大地图撑爆内存），具体见引用3）。分两种情况：

两种情况，一个是不开启bUseSeamlessTravel（无缝旅行）的时候，此时的traveling的时候（ServerTravel和clientTravel)，当前的World会被释放，所以当前的GameMode也会被释放，那么当新的World加载的时候就会根据新的GameModeClass创建新的GameMode。此时GameMode是不同的。

开启bUseSeamlessTravel的话，当前世界会调用GetSeamlessTravelActorList：

```c++
void AGameMode::GetSeamlessTravelActorList(bool bToTransition, TArray<AActor*>& ActorList)
{
	UWorld* World = GetWorld();

	// Get allocations for the elements we're going to add handled in one go
	const int32 ActorsToAddCount = World->GameState->PlayerArray.Num() + (bToTransition ?  3 : 0);
	ActorList.Reserve(ActorsToAddCount);

	// always keep PlayerStates, so that after we restart we can keep players on the same team, etc
	ActorList.Append(World->GameState->PlayerArray);

	if (bToTransition)
	{
		// keep ourselves until we transition to the final destination
		ActorList.Add(this);
		// keep general game state until we transition to the final destination
		ActorList.Add(World->GameState);
		// keep the game session state until we transition to the final destination
		ActorList.Add(GameSession);

		// If adding in this section best to increase the literal above for the ActorsToAddCount
	}
}
```

​	`bToTransition`就是是不是过渡世界的意思，根据bUseSeamlessTravel的不同，UE可以选择哪些Actor迁移到下一个World中去（实现方式是先创建个中间过渡World进行二段迁移（为了避免同时加载进两个大地图撑爆内存）

开启无缝过渡的时候，第一步从CurrentWorld到TransitionWorld迁移时，bToTransition==true，此时，GameMode也会迁移到TransitionWorldTransitionMap可以在ProjectSettings里配置），也包括GameState和GameSession，然后CurrentWorld释放掉。

第二步从TransitionWorld到NewWorld的迁移，GameMode（已经在TransitionWorld中了）会再次调用GetSeamlessTravelActorList，这个时候bToTransition==false，所以第二次的时候如代码所见当前的GameMode、GameState和GameSession就被排除在外了。



**结论是**，UE的流程travelling**，GameMode在新的World里是会新生成一个的**，即使Class类型一致，即使bUseSeamlessTravel，因此在travelling的时候要小心GameMode里保存的状态丢失。不过Pawn和Controller默认是一致的。

**AI总结：**是的，您理解得非常正确。当您在虚幻引擎中开启`SeamlessTravel`（无缝旅行）功能时，它的主要目的是为了提供平滑的过渡体验，并且能够在后台加载新世界的同时，继续在过渡世界渲染当前的场景。这样做可以减少或消除玩家在游戏世界切换时的不流畅感，比如加载画面或游戏暂停。开启`SeamlessTravel`的目的是为了平滑过渡以及将某些需要的Actor迁移到新Level中，而`GameMode`的变化是根据新世界的配置来决定的，以确保游戏逻辑和状态的一致性。如果新世界的`GameModeClass`与当前世界相同，那么不需要切换`GameMode`；如果不同，则会在迁移到新世界时创建并激活新的`GameMode`实例。



#### **哪些逻辑应该写在GameMode里？哪些应该写在Level Blueprint里？**

从概念上：

Level应该注重的是本Level内的一些表示逻辑，比如，改变Level中某个Actor的运动轨迹，或者某一个区域的重力，或者触发一段特效或动画。

而GameMode应该专注于玩法，比如胜利条件，怪物刷新等。适用于全部Level的逻辑。

从组合上：

同Controller放到Pawn上一样。因为GameMode可以用于不同的Level，所以通用逻辑应该放到GameMode中。

GameMode只在server中存在（单机游戏也算Server），对于已经连接上Server的Client，游戏状态都是由Server决定，Client只是负责展示，所以Client上是没有GameMode的，但是有LevelScriptActor，所以GameMode中不要写Client的逻辑，比如操作UI等。

跟下层的PlayerController比较，GameMode关心的是构建一个游戏本身的玩法。playerController关注的是玩家的行为。想想哪些逻辑属于游戏，哪些逻辑属于玩家。

跟上层的GameInstance比。GameInstance关注的更多的是不同World之间的逻辑。GameInstance有时也把手伸下来管ui的事情，但是严谨来讲UI是独立于World的存在。	可以把GameMode之间的协调工作交给GameInstance，而GameMode只专注于自己的玩法世界。

------------------------------------------------------------------------------------------------

在多玩家游戏中，客户端不应该包括于游戏状态管理相关的逻辑，因为这应该由服务器处理。如果在客户端执行某些操作，应该有RPC机制请求服务器执行这些操作。

- **LevelScriptActor**：在虚幻引擎中，`LevelScriptActor`是一个特殊的Actor，它在每个`Level`中都有一个实例。这个Actor负责处理`Level`的初始化和清理，以及一些与`Level`相关的逻辑。`LevelScriptActor`通常会在`Level`加载时创建，并在`Level`卸载时销毁。
- **RPC（Remote Procedure Call）**：RPC是一种允许一个客户端（Client）发送消息到服务器（Server），并要求服务器执行一个函数的机制。在虚幻引擎中，RPC通常用于多玩家游戏中，以便客户端可以请求服务器执行某些操作，例如网络同步、状态更新等。

## 拓展：多人游戏的流程：

为保持游戏的一致性，通常UI操作不应该直接在客户端上执行，而是应该通过服务器，以下是步骤：

**1.客户端发送请求**：当客户端要想执行一个UI操作，不应该直接更改UI而是应该使用RPC（远程过程调用）功能，将操作请求发送到服务器。

**2.服务器处理请求**：服务器接收到客户端的RPC请求之后，他会决定如何处理这个请求。如果涉及游戏状态的更改，服务器会确保这些更改是合法的，并且不会破坏游戏的一致性。

**3.服务器转发请求**：如果服务器决定执行这个操作，他会将其转发给GameInstance。GameInstance是游戏的根对象。负责游戏世界的所有内容。

**4.GameInstance执行操作**：GameInstance接收到服务器的请求后，他会执行相应的操作来更改UI。由于GameInstance是服务器的一部分，所有UI更改都会在服务器上执行，确保了游戏状态的一致性。（GameInstance中有OnlineSession的管理，这部分逻辑跟网络的机制有关）

---------------------------

## Gamestate：

与PlayerState用于保存玩家数据一样。对于游戏，也需要一个State来保存当前游戏的状态数据，比如任务数据等。他也是继承于AInfo，这样在网络环境中可以replicated到多个Client上。

![img](https://pic3.zhimg.com/80/v2-7d2b6410a98ae6819a358e59b5cf8eaa_720w.webp)

第一个，matchState和相关回调就是在网络中传播同步游戏的状态使用的（GameMode在Client中不存在，但是Gamestate是存在的，所以可以通过它来复制），

第二个，PlayerState列表。如果在Client1中想看到Client2中的游戏状态数据，则Client2的PlayerState就必须广播过来，因此GameState把当前Server的PlayerState都收集了过来，方便访问使用。

关于使用Gamestate，开发者可以选择Gamestate子类来存储本GameMode的运行过程产生的数据（那些想要复制的）如果是GameMode运行的一些数据又不想所有的客户端都可以看到，则也可以写在GameMode的成员变量中。

## GameSession

网络联机游戏中针对session使用的一个方便的管理类，并不存储数据，先做保留。

### 总结

对于一场游戏，我们最关心的是调整好场景表现（LevelBlueprint）和游戏玩法的编写（GameMode）。UE再次用Actor分化派生的思想，用相同的套路1AGameMode和AGameState支持玩法和表现的解耦分离和自由组合，并很好的支持了网络间状态的同步。

# Player

为什么要有一个Player呢？玩家是什么？我和其他陪我一起玩的小伙伴？电脑前的一条狗也是玩家。所以在引擎看来，玩家就是输入的发起者。这里的输入不只是本地键盘手柄等输入设备的按键，也包括网线传来的信号，是广义的该游戏能接受到的外界输入。

### UPlayer

假装我们是UE，怎么编写Player类呢，能从AActor得来吗？Actor是必须在World中才能存在的，而Player却是比World更高一级的存在。所以要从UObject继承

另外，Player也不需要被摆放在Level中，也不需要各种Component组装，所以从AActor继承并不合适。

![img](https://pic1.zhimg.com/80/v2-f1fee0eaf2b5fea87085f71895f4e730_720w.webp)

如图，Player和playerController关联起来，因此UE引擎就可以把输入和playerController关联起来，也符合前文说的playerController接受玩家输入的描述，本地和远程玩家都需控制一个玩家Pawn，所以为每一个玩家分配一个playerController很合理。

### ULocalPlayer

![img](https://pic2.zhimg.com/80/v2-0cdc26e0b66ec808197a76cb8b09350d_720w.webp)

ULocalPlayer比UPlayer多了Viewport相关的配置，也终于用SpawnPlayActor实现创建出PlayerController的功能，GameInstance中有了localPlayers的信息之后，就可以方便的遍历访问。实现和本地玩家相关操作。

先简单说下LocalPlayer怎么在游戏中引擎各个环节发挥作用的。UE初始化GameInstance的时候，会默认创建出一个GameViewportClient,然后在内部转发到GameInstance的CreateLocalPlayer:

```c++
ULocalPlayer* UGameInstance::CreateLocalPlayer(int32 ControllerId, FString& OutError, bool bSpawnActor)
{
	ULocalPlayer* NewPlayer = NULL;
	int32 InsertIndex = INDEX_NONE;
	const int32 MaxSplitscreenPlayers = (GetGameViewportClient() != NULL) ? GetGameViewportClient()->MaxSplitscreenPlayers : 1;
    //已略去错误验证代码，MaxSplitscreenPlayers默认为4
	NewPlayer = NewObject<ULocalPlayer>(GetEngine(), GetEngine()->LocalPlayerClass);
	InsertIndex = AddLocalPlayer(NewPlayer, ControllerId);
	if (bSpawnActor && InsertIndex != INDEX_NONE && GetWorld() != NULL)
        //检查是否需要创建玩家控制器：如果需要创建玩家控制器（bSpawnActor为true），并且新玩家被成功添加（InsertIndex不为INDEX_NONE），且游戏世界存在（GetWorld()不为NULL）。
	{
		if (GetWorld()->GetNetMode() != NM_Client)
		{
            //如果不是Client，那就是Server模式嘛，他会创建一个PlayerController
			// server; spawn a new PlayerController immediately
			if (!NewPlayer->SpawnPlayActor("", OutError, GetWorld()))
        //调用SpawnPlayActor函数尝试在服务器上创建一个新的玩家控制器。如果创建失败，则记录错误信息到OutError，并从本地玩家列表中移除新玩家。
			{
				RemoveLocalPlayer(NewPlayer);
				NewPlayer = NULL;
			}
		}
		else
		{
			// client; ask the server to let the new player join
            //调用SendSplitJoin函数，向服务器发送请求，允许新玩家加入游戏。
			NewPlayer->SendSplitJoin();
		}
	}
	return NewPlayer;//如果创建成功，返回新玩家的指针；否则，返回NULL。
}
```

可以看到如果是Server模式，会直接创建出ULocalPlayer，然后创建UPlayerController。

而如果是Client客户端，则会SendSplitJoin（），将会把向服务器发送请求，一旦服务器收到此请求，并成功处理了它，**服务器**将会为这个新加入的玩家创建一个PlayerController。

-----

**拓展    服务器与客户端：**

当`GetWorld()->GetNetMode() != NM_Client`这个条件不满足时，意味着游戏实例不是运行在客户端模式下。这可能是因为游戏实例是作为服务器运行的（`NM_DedicatedServer`），或者是在编辑器中以单机模式运行的（`NM_Standalone`）。

在分屏游戏中，即使所有玩家都在同一台主机上，每个玩家仍然有自己的客户端实例。每个客户端实例都有自己的`PlayerController`和`PlayerState`，它们通过网络连接（即使是本地回环网络连接）与游戏服务器通信。

因此，当`NewPlayer->SendSplitJoin();`被调用时，它是在请求服务器允许一个新的客户端实例加入游戏。这个请求可以是来自本地主机上的另一个客户端实例，也可以是来自网络中的另一台计算机上的客户端实例。服务器收到这个请求后，会为请求的客户端创建相应的`PlayerController`和`PlayerState`，从而允许新玩家加入游戏。

----

而在每个PlayerController创建的时候会initPlayerState：

代码先不放了，主要作用就是PlayerState和localPlayer关联起来了。而网络联机时其他玩家的PlayerState是通过Replicated过来的。

而我们谈论很久的玩家输入，体现在每个PlayerController接受Player的时候：

```cpp
void APlayerController::SetPlayer( UPlayer* InPlayer )
{
    //[...]
	// Set the viewport.
	Player = InPlayer;
	InPlayer->PlayerController = this;
	// initializations only for local players
	ULocalPlayer *LP = Cast<ULocalPlayer>(InPlayer);
	if (LP != NULL)
	{
		// Clients need this marked as local (server already knew at construction time)
		SetAsLocalPlayerController();
		LP->InitOnlineSession();
		InitInputSystem();
	}
	else
	{
		NetConnection = Cast<UNetConnection>(InPlayer);
		if (NetConnection)
		{
			NetConnection->OwningActor = this;
		}
	}
	UpdateStateInputComponents();
	// notify script that we've been assigned a valid player
	ReceivedPlayer();
}
```

对于UPlayer，APlayerController内部会开始InitInputSystem(),接着创建出相关组件。

**思考：为何不在LocalPlayer里编写逻辑？**

这个可能有两个原因，一是UE从FPS-Specify游戏起家，不像现在的各种手游有非常重的玩家系统，在UE的眼中，Level和World才是最应该关注的对象，因此UE的视角就在于怎么在Level中处理好Player的逻辑，而非在World之外的额外操作。二是因为在一个World中，上文提到其实已经有了Pawn-PlayerController和PlayerState的组合了，表示、逻辑和数据都齐备了，也就没必要再在Level掺和进Player什么事了。

### UNetConnection

![img](https://pic4.zhimg.com/80/v2-4bc20d50aa7c95755bf4d4c4b2b60ed7_720w.webp)

UE中网络连接也是一个Player，包含Socket的IpConnection也是玩家，甚至对于一些平台的特定实现如OculusNet的连接也可以当作玩家，因为对于玩家，只要能提供输入信号，就可以当作一个玩家。

### 总结：

本篇抽象出了Player的概念，并根据使用场景派生出了，PLocalPlayer和UNetConnection，从此Player不是一个虚无缥缈的概念，而是UE中的逻辑实体，。UE可以根据生成的Player的数量和类型不同，在此实现出不同的玩家控制模式，LocalPlayer作为源头spawn出PlayerController以及PlayerState就是证实。

而在网络联机的时候，把网络连接也看做一个Player，把在world之上的输入实体用Player统一起来之后，从而可以实现出灵活的本地远程不同玩家模式策略。

不推荐在Player中直接编程，只是我们利用Player作为源头来构建了一系列相关机制，对于我们来说，了解UE中的Player概念是把现实与游戏世界联系起来的重要纽带，当我们在world大地上向天空仰望的时候，我们将看到一个个LocalPlayer和NetConnection注视着这片大地，我们的生死不取决于所谓的天命，取决于玩家。

![image-20240822154045835](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240822154045835.png)

# GameInstance

上篇讲到了，在world之外还有Player的概念，其中包含了本地的ULocalPlayer和网络的UNetConnection，并以此创建出了world中的PlayerController，从而实现了不同玩家模式策略。再向上，自己是无法管理自身的，所以Player也需要一个创建和存储的地方。而我们上篇也可以知道，ULocalPlayer* UGameInstance::CreateLocalPlayer这是GameInstance中得函数。另一方面，Player可以负责一些跟玩家相关的业务逻辑，但是对于world之上的协调管理的逻辑却也无法安放。我们需要创建管理类，当做大管家用于协调。

游戏引擎的出现，最开始其实只是因为一些人发现游戏做着做着，有一大部分功能是可以复用的，于是就把它抽离了出来方便做下一款游戏。UE很大的得益于Epic实战游戏开发的反哺，这一方面Unity就有点吃亏了，没有自己亲自下手干脏活累活，就不懂得急人民群众之所急。所以如果一个游戏引擎能把GamePlay也做好了，那就不止是口袋了，而是知你懂你的叮当猫本身。

GameInstance

得益于UObject的反射创建能力，GameInstance直接继承于UObject，这样可以根据一个Class直接动态创建出具体的GameInstance类

![img](https://pic1.zhimg.com/80/v2-e3572a2d46608304109104e6174688d8_720w.webp)

**UGameInstance的继承关系：**`UGameInstance`确实是从`UObject`继承来的。在UE中，几乎所有的游戏对象都是`UObject`的子类，因为`UObject`提供了对象的基础功能，如引用计数、序列化、反射等。

**GameEngine的角色：**GameEngine是UE的核心类之一，他负责管理游戏的生命周期。包括初始化游戏、创建游戏实例、管理游戏关卡、处理游戏逻辑等。

**UGameInstance的创建：**游戏启动时，GameEngine会创建一个UGameInstance的实例。这个实例是游戏的全局实例，包含游戏的全局状态和配置，并且是游戏逻辑的入口点。

##### 所以为什么说他是由UObject继承来的却是由GameEngine创建的呢？

1. **继承**：`UGameInstance`是从`UObject`继承来的。这意味着`UGameInstance`继承了`UObject`的所有属性和方法。继承是面向对象编程中一种表示“is-a”关系的方式。在这个例子中，`UGameInstance`“是一个”`UObject`。
2. **组合和聚合**：`UGameInstance`和`GameEngine`之间的关系更像是组合或聚合。组合和聚合都是表示“has-a”关系的面向对象概念。在这个例子中，`GameEngine`“有一个”`UGameInstance`。`GameEngine`负责创建和管理`UGameInstance`的实例，这表明`GameEngine`和`UGameInstance`之间存在一种紧密的合作关系。

##### GameInstance只有一个吗？

一般而言，是的。对于开发游戏而言，却是可以认为子类化的那个GameInstance就像一个单件，全局只有一个，从游戏开始到结束。但是更多的我们可以了解到

正如将网络连接当做一个玩家，我们重新审视game这个概念。假设有个引擎支持双击图标一下子开出4个窗口来让4个玩家独立运行，你能说得清这是一个Game还是4个Game在运行吗？

如果是四个窗口一点不相互关联，或者只是单独的共用地图资源，那么用4个Game的概念管理很合适。如果四个窗口中运行的实际是在同一个关卡的本地对战，那么就是用一个Game四个Player更合适。所以，你可以认为Game像进程，进程可以在同一个exe上多开，Game也可以在同一份游戏资源上开出多个运行实例，进程之间可以相互通信协作，Game的不同实例之间也可以相互沟通，不管是内存中直接在Engine的协调下完成还是Socket通信。

另外，UE使用了引擎自绘的UI框架，正因为有这套Editor自绘机制还有PIE，进程中是可以同时拥有多个GameInstance的，如正在编辑的EditorWorld所属的和Player之后的World所属的。

<img src="https://pic1.zhimg.com/80/v2-94d1f4e3750b6f4fd09d02b20bc980b0_720w.webp" alt="img" style="zoom:200%;" />

当时没有明白，我以为是一个gameInstance其中的Thiscurrentworld指向切换呢。

![image-20240822164918089](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240822164918089.png)

![image-20240822200826785](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240822200826785.png)

当您开始PIE（Play In Editor）时，虚幻引擎会创建一个新的游戏实例（`GameInstance`），并且为这个新的游戏实例分配一个新的世界（`World`）。这个新的世界包含了游戏运行时的状态和逻辑。

具体来说，虚幻引擎的编辑器允许开发者在一个项目中同时运行多个游戏实例。每个游戏实例都管理着自己的世界和游戏逻辑。当您在编辑器中选择“Play In Editor”选项时，虚幻引擎会为当前的游戏实例创建一个新的世界，并将其作为新的游戏实例的一部分。这个新的世界就是您在PIE模式下运行的游戏环境，它包含了游戏运行时的状态和逻辑。

#####  那些逻辑应该写到GameInstance中？

1. Worlds,Level的切换实际发生地是Engine，而GameInstance可以说是UE之神下的唯一代言人，所以GameInstance也可以代之管理World的切换。我们可以在GameInstance中实现各种逻辑最后调用Engine中的openlevel的接口。
2. players，虽然一般直接控制Player的机会不多，但是也实现了很多接口用于动态删除添加Players
3. UI。
4. 全局配置
5. 额外的第三方逻辑，比如游戏中需要其他一些控制，比如自己写的网络通信，自定义的一些配置文件或者算法，如果简单的话，GameInstance可以存放这些，复杂的话，可以把GameInstance当做模块容器，可以在其中扩展各种子逻辑模块。。当然如果是插件的话，还是在自己的插件Module里面自行管理逻辑，然后把协调工作交给GameInstance来做。

在数据层面，我们已经有了针对Player的PlayerController的PlayerState，在World层面的GameMode的Gamestate。到了更上一层的全局，自然的GameInstance应该存放一些全局的状态数据。  

所以可以在GameInstance的成员变量中存放一些全局的状态，或者是那些想在Level之外持续存在的对象。注意GameInstance成员变量最好是保存一些“临时”的数据，对于那些想要序列化保存的数据，就需要接下来的SaveGame了

## SaveGame

UE连玩家存档都帮你做了！得益于UObject的序列化机制，现在你只需要继承于USaveGame，并添加你想要的那些属性字段，然后这个结构就可以序列化保存下来的。UE为我们在蓝图里提供了SaveGame的统一接口，让你只用关心想序列化的数据。

USaveGame其实就是为了提供给UE一个UObject对象，本身并不需要其他额外的控制，所以它的类是如此的简单以至于我能直接把它的全部声明展示出来：

```c++
UCLASS(abstract, Blueprintable, BlueprintType)
class ENGINE_API USaveGame : public UObject
{
	/**
	 *	@see UGameplayStatics::CreateSaveGameObject
	 *	@see UGameplayStatics::SaveGameToSlot
	 *	@see UGameplayStatics::DoesSaveGameExist
	 *	@see UGameplayStatics::LoadGameFromSlot
	 *	@see UGameplayStatics::DeleteGameInSlot
	 */

	GENERATED_UCLASS_BODY()
};
```

玩家保存游戏的时候会按照以下步骤进行：

**写入控制头文件：**保存游戏之前引擎会在内存中写入一些控制文件头，例如SaveGameFileVersion。包含一些关于存档的元数据，如版本号、创建时间等

**序列化对象：**接着引擎会序列化USavegame对象。这个对象包含了玩家的游戏状态，如位置，物品、游戏进度等。      （**序列化**（Serialization）是指将对象的状态转换为字节序列的过程，以便将数据存储在文件、数据库或通过网络传输。这个过程通常涉及到将对象的状态信息（如字段值、对象引用等）转换成可以被外部系统理解和存储的形式。）

**找到ISaveGameSystem接口：**完成序列化之后，引擎会查找ISaveGameSystem接口。这个接口定义了保存游戏存档的方法和逻辑。

**调用ISaveGameSystem实现：**引擎会调用`ISaveGameSystem`接口的实现，通常是`FGenericSaveGameSystem`。这个实现会将序列化后的`USaveGame`对象保存到指定的位置。

FGenericSaveGameSystem的内部实现是直接转发到直接的文件读写接口上去的，这意味着他直接将数据写入文件系统。

但是你也实现自己的SaveGameSystem，以实现不同的保存方式。例如你可以将文档保存到不同位置，如网络服务器或者Steam云存储。

USaveGame可以看做一个全局持久数据的业务逻辑类，与GameInstance中的数据区分是，GameInstance中的数据是临时的，而USaveGame中的数据是持久的。在`SaveGameToSlot`函数中，`SlotName`可以理解为存档的文件名，而`UserIndex`是用来标识是哪个玩家在存档。

![img](https://pic2.zhimg.com/v2-5f9893415a3b89cb6ef4c2e217cbf391_r.jpg)



# 总结

## 游戏世界

在UE眼中万物皆是Actor，Actor再通过Component组装功能。Actor又通过UChildActorComponent实现Actor之间的父子嵌套。

众多Actor又组成了Level ，ULevel继承于UObject，为了让世界中所有的资产不是一次性全部加载，所以Level扛起了大任。

![img](https://pic3.zhimg.com/v2-14a202ba552576c2505073cb1543eeae_r.jpg)

Uworld，从UWorldBase继承来，UWorldBase继承于UObject。

每个 `UWorld` 实例都管理着一系列的 `ULevel` 实例，这些 `ULevel` 实例构成了游戏世界中的不同场景或关卡。

`UWorld` 管理着 `ULevel` 实例，而 `ULevel` 实例则包含了游戏世界中的具体场景和内容。

![img](https://pic2.zhimg.com/80/v2-4b0a3d9cb6479a1c8efe736046c06dc5_720w.webp)

而World之间的切换，则是有WorldContext来保存切换的过程信息。玩家在切换PersistentLevel的时候，实际相当于切换了World。而再往上，就是整个游戏唯一的GameInstance，由Engine对象管理着

![img](https://pic2.zhimg.com/80/v2-19ce8ccbd2e444a8fb27459614aa602d_720w.webp)

但是GameInstance中不仅保存着UWorld,还有Player，有着LocalPlayer用于表示本地玩家和NetConnection作为远端连接。

![img](https://pic2.zhimg.com/80/v2-e7fc2230978792cb4ea8552337a11565_720w.webp)

玩家利用Player对象接入World之后，就可以控制Pawn和PlayerController的生成，有了附身的对象，就可以在Engine的tick心跳下开始一帧一帧的逻辑更与渲染了。

## 数据与逻辑

​	

说完游戏世界的表现组成，对于gameplay来说当然需要与其配套的业务逻辑架构

使用MVC架构把游戏分为，表现（View），逻辑（Controller），	数据(Model)

![img](https://pic3.zhimg.com/v2-b4e0dd15956ccb819fca93e73d1b8ed2_r.jpg)

1. 从UObject派生下来的AActor，拥有了UObject的反射序列化网络同步等功能，同时又通过各种Component来组装不同组件。UE在AActor身上同时利用了继承和组合的各自优点，同时也规避了彼此的一些缺点，我不得不说，UE在这一方面度把握得非常的平衡优雅，既不像cocos2dx那样继承爆炸，也不像Unity那样走极端全部组件组合。
2. AActor中一些需要逻辑控制的成员分化出了APawn。Pawn就像是棋盘上的棋子，或者是战场中的兵卒。有3个基本的功能：可被Controller控制、PhysicsCollision表示和MovementInput的基本响应接口。代表了基本的逻辑控制物理表示和行走功能。根据这3个功能的定制化不同，可以派生出不同功能的的DefaultPawn、SpectatorPawn和Character。([GamePlay架构（四）Pawn](https://zhuanlan.zhihu.com/p/23321666))
3. AController是用来控制APawn的一个特殊的AActor。同属于AActor的设计，可以让Controller享受到AActor的基本福利，而和APawn分离又可以通过组合来提供更大的灵活性，把表示和逻辑分开，独立变化。([GamePlay架构（五）Controller](https://zhuanlan.zhihu.com/p/23480071))。而AController又根据用法和适用对象的不同，分化出了APlayerController来充当本地玩家的控制器，而AAIController就充当了NPC们的AI智能。([GamePlay架构（六）PlayerController和AIController](https://zhuanlan.zhihu.com/p/23649987))。而数据配套的就是APlayerState，可以充当AController的可网络复制的状态。
4. 到了Level这一层，UE为我们提供了ALevelScriptActor（关卡蓝图）当作关卡静态性的逻辑载体。而对于一场游戏或世界的规则，UE提供的AGameMode就只是一个虚拟的逻辑载体，可以通过PersistentLevel上的AWorldSettings上的配置创建出我们具体的AGameMode子类。AGameMode同时也是负责在具体的Level中创建出其他的Pawn和PlayerController的负责人，在Level的切换的时候AGameMode也负责协调Actor的迁移。配套的数据对象是AGameState。([GamePlay架构（七）GameMode和GameState](https://zhuanlan.zhihu.com/p/23707588))
5. World构建好了，该派玩家进来了。但游戏的方式多样，玩家的接入方式也多样。UE为了支持各种不同的玩家模式，抽象出了UPlayer实体来实际上控制游戏中的玩家PlayerController的生成数量和方式。([GamePlay架构（八）Player](https://zhuanlan.zhihu.com/p/23826859))
6. 所有的表示和逻辑汇集到一起，形成了全局唯一的UGameInstance对象，代表着整个游戏的开始和结束。同时为了方便开发者进行玩家存档，提供了USaveGame进行全局的数据配套。([GamePlay架构（九）GameInstance](https://zhuanlan.zhihu.com/p/24005952))



在分析UE这个gameplay框架的时候，我们应该从各个切面去分析他的构成。这里有两大基本原则：单一职责和变化隔离。所有的程序设计模式都是在抽象变化。把变化抽离开，剩下的就是单一责任了嘛，所以UE里的MVC的实践就是在不断地抽离出各种对象的变化部分，比如World的Level就是静态逻辑，关卡蓝图，动态游戏玩法抽离出来就是GameMode，数据抽离出来就是GameState。
