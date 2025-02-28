
	https://zhuanlan.zhihu.com/p/416805924
	https://github.com/kierstone/Buff-In-TopDownShooter

#系统设计 

	 对上面这篇战斗系统的博客进行学习理解记录
## 战斗系统元素

![[Pasted image 20250211113849.png]]
## ECS与Unity设计模式

Unity的GameObject-Component模式与ECS确实有表面相似性，但其设计哲学和实现细节存在本质差异，具体可以从以下几个角度理解：

---

### 1. **身份认定的差异**

- **ECS的Entity**：  
    是纯粹的逻辑容器，本身**没有身份和语义**。其意义完全由拥有的Component组合决定。Entity类似空盒子，内容和意义动态变化，System通过筛选Component组合来处理相应逻辑（如"有位置+渲染数据"的实体才被渲染）。
    
- **Unity的GameObject**：  
    **隐含身份概念**。例如一个带有"EnemyController"组件的GameObject，开发者会默认将其视为敌人。组件更像是为已有概念的对象（如敌人、玩家）添加功能，而非定义其本质。
    

---

### 2. **组件责任的差异**

- **ECS的Component**：  
    **纯数据容器**，严格禁止逻辑（仅有数据字段）。例如VelocityComponent只包含速度值，System负责移动计算。组件间解耦，System组合使用多个组件实现功能。
    
- **Unity的Component (MonoBehaviour)**：  
    **数据和逻辑混合**。组件通常包含字段（数据）和方法（逻辑），例如EnemyController可能既有血量数据，又有攻击逻辑。组件间通过`GetComponent`或消息传递协作，容易形成紧耦合。
    

---

### 3. **数据访问效率的差异**

- **ECS的组合式内存布局**：  
    相同类型Component连续存储（如所有PositionComponent在内存中连续排列），System遍历时缓存命中率高，适合处理大量实体（如万级粒子效果）。
    
- **Unity的GameObject内存模型**：  
    GameObject和Component分散在堆内存中，遍历时频繁跳转内存地址，缓存效率低。适合少量复杂对象（如场景中的角色、道具），但海量实体时性能急剧下降。
    

---

### 4. **设计目标的差异**

- **ECS的核心目标**：  
    数据驱动的高性能计算，通过分离数据与逻辑、高效内存布局，充分发挥硬件并行能力。常用于需要极高性能的模块（如物理模拟、大规模AI）。
    
- **Unity的组件模式核心目标**：  
    **快速开发与灵活扩展**。通过可视化编辑和脚本组件快速构建对象逻辑，降低学习曲线，适合原型开发和中小规模项目。
    

---

### 类比说明

- **把ECS比作餐厅后厨**：  
    Chef（System）根据食材（Component）组合进行标准化处理。如果某订单（Entity）有"牛肉+洋葱"（Components），就按特定流程制作牛排。食材仅描述属性，处理逻辑完全由厨师决定。
    
- **把Unity组件模式比作智能手机**：  
    手机本体（GameObject）通过安装App（Component）扩展功能。虽然每个App独立存在，但用户通常认为"安装了微信的手机"就是一个能社交的设备，手机本身具有一定预设功能（如操作系统）。

## Update()与FixedUpdate()

### **核心概念**

- **FixedUpdate**：  
    被称为**逻辑帧**或**物理帧**，其调用间隔时间由`Time.fixedDeltaTime`（默认0.02秒）定义，**与设备帧率无关**。  
    Unity会通过**时间累积补偿机制**确保物理世界的确定性。即使设备卡顿，物理逻辑仍然按固定步长推进，避免因帧率波动导致的逻辑错误。
    
- **Time.fixedDeltaTime**：  
    是开发者预设的**固定时间间隔**（可配置），与实际硬件性能无关，恒等于预设值。即：  
    `逻辑时间推进量 = 调用次数 × Time.fixedDeltaTime`
    

---

- **FixedUpdate + Time.fixedDeltaTime = 确定性时基**：  
    是物理相关或要求时间严格一致性的逻辑（如网络同步状态,角色运动距离）的黄金组合。
- **Update + Time.deltaTime = 帧率依赖操作**：  
    适用于视觉效果、非物理交互行为。  
    正确区分二者是保障游戏体验稳定性的关键。
物理逻辑使用FixedUpdate,渲染使用Update

## Model、Obj与Info 

#### **1. Model：数据之源**

- **定义**：
    
    - 纯粹的静态数据容器，存储策划设计的数值和规则（如角色的基础血量、武器的攻击力、技能的基础效果）。
    - **不包含任何逻辑代码**，仅用于初始化Obj或提供计算依据。

#### **2. Obj：游戏中的实体**

- **定义**：
    
    - Unity中的`GameObject`及其附加组件，代表实际存在于游戏世界中的对象（如玩家、敌人、技能特效等）。
    - **由Model初始化**，并在运行时管理动态状态（如当前血量、技能冷却时间）。

#### **3. Info：逻辑的纽带**

- **定义**：
    
    - 传递系统间交互所需的临时参数（如添加Buff的层数、攻击事件的伤害值）。
    - **不直接关联Obj的生存周期**，类似“任务指令”或“事件参数”。
---

### **核心概念分层**

| **层级**    | **用途**        | **生命周期**     | **典型实现形式**                    | **例子**               |
| --------- | ------------- | ------------ | ----------------------------- | -------------------- |
| **Model** | **静态数据模板**    | 持久化（配置表、数据库） | 结构体（struct）、ScriptableObject  | 角色属性表、武器配置、技能基础效果    |
| **Obj**   | **动态游戏实体**    | 运行时（场景中存在）   | GameObject + MonoBehaviour 组件 | 玩家角色属性、敌人、子弹、技能特效    |
| **Info**  | **逻辑交互的中间数据** | 临时（逻辑流程中）    | 数据类（class）                    | Buff添加请求、伤害事件参数、生成指令 |

## DamageInfo  伤害流程

DamageInfo是贯穿整个伤害处理流程的，一个伤害处理流程通常是这样的：



![[Pasted image 20250211135512.png]]

**任何伤害都应该走DamageInfo**(包括额外攻击)

DamageInfo的具体属性：
- attacker: 攻击者的GameObject,可以为空,比如地图机关伤害,造成伤害的对象不是角色
- defender: 受到伤害者,不能为空
- tags：字符串数组，伤害类型的tag，比如是“直接伤害”、“间歇伤害”、“反弹伤害”等等
- **damage**：一个伤害数据结构，这是根据游戏不同来设计的，比如游戏中有金木水火土外加物理攻击，那么他就应该是有6个伤害数字组成的。我们通常忽略的一种情况是——策划会设计一些针对游戏中“元素属性”类似的东西有效的效果，比如“受到的火焰伤害减半”，如果此时受到的攻击是一个“暗影烈焰”，即暗属性伤害200点+火属性伤害200点，那么就得从这里面去找到暗影属性伤害减去一半；再比如“短时间内抑制所有受到的子弹伤害”，那么这个伤害的数据里面有一条肯定是子弹伤害，将子弹伤害设定为0，就抑制了。
- **damageDegree**：伤害的角度，也就是伤害打向defender的入射角度，通常这个角度来源是取子子弹的飞行方向或者aoe的中心点指向角色的位置的。这个角度配合角色当前的面向角度，就可以算出角色什么方向受到了伤害，假如需要做类似“背刺”的效果，那就得用上这个了。
- **criticalRate**：本次攻击的最终暴击率，大多游戏还是有暴击设计的，如果没有，可以砍掉这个数据。当一次伤害信息经历了所有流程之后，将最后的数值传递给策划编写的公式脚本，由策划来处理是否暴击了，以及暴击造成多少伤害。
- **hitRate**：和暴击率类似的概念，但是他俩在逻辑流程中实际上并无直接关系，即当有一个效果是“下一次攻击必定暴击”，这时候的DamageInfo哪怕hitRate是<=0的数字，也不妨碍criticalRate被设置为1或者更高的数字（假如策划认为1代表100%，这完全是由设计数值公式的策划来定义的），这两者并无依赖关系，不是说不命中就一定不能暴击，最后不命中能不能暴击，暴击了是不是一定命中，还是看策划写的公式脚本如何认为。
- **addBuffs**：这是一个“隐藏属性”，所以在上述结构中并没有标明，但是非常有必要说明一下。因为在整个伤害的流程中，我们可能因为一些角色身上的buff效果，他会需要添加新的buff效果，而这个新的buff效果并不想马上添加给角色（通常都是如此），比如说攻击者有一个buffA，他的效果是“攻击后目标受到割裂影响”，也就是在目标身上上一个buffB；还有一个优先级更低（更晚执行）的buffC，是“对割裂的目标造成的伤害提高200%”，策划设计的时候的想法是，这次攻击造成割裂，下次才是3倍伤害，但是因为执行顺序，产生了本次就直接上了割裂并且3倍伤害:

	- **即时添加Buff导致逻辑混乱**：  
	若Buff在事件处理过程中立即生效，后续步骤可能依赖于新增Buff的状态，如：
	
	```
	攻击触发 BuffA → 立即添加 BuffB（割裂） 
	同一攻击中 BuffC（若目标有割裂则增伤200%） → 检测到BuffB存在 → 错误触发
	```
	
	策划本意是本次触发割裂，**下次**攻击才享受增伤，但因执行顺序导致矛盾。
	#### **1. 分阶段处理**
	
	将Buff添加分为**请求收集阶段**和**实际应用阶段**：
	
	- **收集阶段**：在整个伤害计算期间，所有想要添加Buff的请求**暂存**到`DamageInfo.PendingBuffs`列表。
	- **应用阶段**：当完整的伤害流程（伤害计算、状态判断等）结束后，**批量处理**列表中的Buff请求。
	
	结果:
	▶ 本次攻击流程： 1. 触发BuffA → 申请添加BuffB（暂存列表） 2. 触发BuffC → 检测当前目标无BuffB → 增伤不生效 3. 最终伤害计算完成 4. 应用BuffB到目标 ▶ 下次攻击流程： 1. BuffC检测到目标已有BuffB → 增伤200%生效

## 角色(Character)

![[Pasted image 20250214110736.png]]

角色预制体看起来是"空"的,以前我做角色时大概率会给它直接绑定一个模型,然后放置这个模型的控制脚本,绑定属性等.把它当成一个固定的角色(主角或者小怪).但是这里的做法理念是无论是模型还是各种方面的控制,属性,buff,都是通过数据渲染.所以这里通过数据驱动此角色的构造.

CharacterObj包含下面组件:
![[Pasted image 20250214111607.png]]

### UnitBindManager

总之我们在“类似某个骨骼”的位置，加上一个UnitBindPoint的组件，他就会被UnitBindManager管理到。UnitBindManager要做的是，当我们需要角色在身上某个绑点播放特效的时候找到绑点去添加这个特效。

### UnitMove

也就是最终根据移动力来处理transform.poisition的赋值，其他一律不管。

### UnitAnim

是一个动画管理器，负责角色动画切换工作的，与技能动画无关.
	实现技能的动画是通过初始化PlaySightEffectOnCaster的key的Timeline把动画预制体放在UnitBindPoint下,持续时间duration结束后销毁.

### UnitRotate

他只负责接受GameObject.transform的Rotate请求.

### ChaState

ChaState 类是一个复杂的状态管理系统，负责处理角色的各种状态和行为。它通过与其他组件（如 UnitMove、UnitAnim、UnitRotate 等）的协作，实现了角色的移动、旋转、动画播放和技能释放等功能。
ChaState还有一个“side-effect”就是大半个buff管理器，负责角色身上的buff的增删改查。

### ChaPie

角色生命值和UI（脚下血条）同步

### PlayerController/SimpleAI

动态添加的对角色的“控制器”

### UnitRemover

指定时间之后移除掉所在的GameObject（Destroy(this.gameObject)）

## 时间轴（Timeline）

基于时间轴对一个或多个事件进行预约调度.

![[Pasted image 20250216133317.png]]

### TimelineNode

这是Timeline的每一个节点的信息，它主要包含3个内容：

- timeElapsed：发生在timeline运行之后多久。
- event：即一个事件脚本函数。
- eventParam：是这个脚本需要传递的参数，这个脚本是：

```csharp
public delegate void TimelineEvent(TimelineObj timeline, params object[] args);
```

这里的参数TimelineObj即执行这个事件的Timeline，而args是动态的参数，即调用这个脚本时候传递给这个脚本的参数，也就是eventParam。

### TimelineModel

这是策划填表的Timeline数据，当然它的来源可以不仅仅是填表（包括使用编辑器之类的生成，其本质也是填表），基于TimelineModel创建游戏运行中的TimelineObj。他主要包含了：

- **nodes**：这个timeline所有的节点事件。
- **duration**：整个timeline的生命周期，timeline总需要一个结束释放掉的时间，他未必是最后一个节点所在的位置，或者最后一个节点的事情做完，比如最后一个节点是播放一个动画，可能动画开始播放0.3秒就可以结束掉timeline了，这完全取决于策划和美术的想法，因此结构上就得给这么一个float去定义。
- **chargePoint**：这是顶视角射击游戏的特色——他就会有一些需要蓄力的技能，蓄力的技能无非就是在Timeline上有一个循环“播放”的区域——在某一帧A判断，如果需要“蓄力”，就跳转到某一帧B（通常来说比A靠前，不然“蓄力”就变成“快速释放”了，当然并不是不能这么用，只要游戏设计用得着）。

### TimelineObj

即游戏世界运行的实实在在存在的Timeline，一个Timeline是一个独立的元素，他并不隶属于谁。通常我们可能错误的理解为他隶属于一个角色，实际上你可以这样理解——他是一段剧本，这段剧本可能属于某个演员，但也可能不属于任何演员，只是道具组场景组要工作。TimelineObj除了有个Model证明他是什么，还需要一些元素：

- caster：Timeline的焦点对象也就是创建timeline的负责人，比如技能产生的timeline，就是技能的施法者
- timeScale：这是因为游戏中会有“狂暴”之类的设计，他的效果是“使角色动作加速”，这里的动作加速，不光是美术做的动画要加速，逻辑内容也要加速，所以得有个timeScale。这个timeScale*Time.fixedDeltaTime就是每个fixedUpdate中timeElapsed增量。
- param：这是一个动态传递的参数，通常是timeline的“创建源”，比如是一个技能创建的，他就是一个SkillObj，记录用的，也作为参数传递给timeline的事件脚本。
- timeElapsed：就是经过了多少时间了，这决定了每个node是否该运作了。

## 技能（Skill）

这里释放技能是先判断是否满足释放条件(是否学习,蓝量是否足够等),
然后直接产生此技能对应的TimelineObj,使之在TimelineManager自行运作.最后扣除Cost,结束施放.
![[Pasted image 20250216140358.png]]

在这里，我们把一切“技能效果”，都抽象为了Timeline。

比如在游戏中有一个火球魔法，我们释放技能，首先就是检查我们的资源是否足够释放，或者说是否学会了这个火球魔法，总之一系列检查是否能放；通过之后，就扣除对应的资源，并且创建了一个Timeline，这个Timeline做了几件事情：

- 0秒时角色开始播放吟唱动作（低头念咒）
- 1.5秒时角色开始播放施法动作（手一伸，指向目标）
- 1.5秒时创建一个子弹（火球）飞向目标

### 技能的数据结构

![[Pasted image 20250216140523.png]]

### SkillModel

技能的模板，这是一个策划填表的数据，它主要包含了：

- condition：在这个demo里，技能的释放条件仅仅只有角色的资源（ChaResource），角色资源也就是Hp、Ammo之类的数值，指的是角色现有的生命值、弹药值等等，检查如果数量足够就算condition为true了。而根据游戏设计，相对复杂一点的，他也不是很需要直接做成一个function，因为只要判断角色持有的buff外加资源就可以满足绝大多数游戏技能需要。但是buff机制本身是强调开放性的，所以这里应该随时都可以被改为一个function，让策划写脚本决定一个技能是否能被释放。
- cost：通常都是ChaResource，因为使用技能以后会扣除这些资源，最常见的就是减少mana。我们通常看到的技能都是cost == condition 的，但实际上并不是，他们是2个属性，因为有些技能是这样的——当怒气大于20的时候可以释放，消耗掉所有的怒气，造成100+怒气值点伤害，这时候cost和condition就是不等的。
- effect：也就是技能释放成功的时候创建的一个Timeline，这个Timeline的caster就是技能释放者，而param就是这个技能的skillObj。
- buff：是当玩家角色学会这个技能的时候（由SkillModel创建了一个SkillObj的时候）会给玩家角色添加的永久buff
### SkillObj

- **model**：也就是SkillModel数据。在游戏中，如果我们经过历练之后，比如一个火球技能，现在能发出2个火球了，那我们需要做的就是把skill.model.timeline给“hack”了，而不需要设计“另外一个技能”。
- **level**：技能的等级，如果有技能升级系统的话……
- **其他**：这是根据游戏设计需要的，比如游戏需要技能还能装插件，不同的插件对技能效果有不同的影响，这时候我们至少得有这些插件槽的数据在这里。
## Buff

这里Buff作为一个角色的"属性数据",角色的ChaState中有一个List BuffObj就是这个角色所有的buff了。

### Buff的数据结构
![[Pasted image 20250216150114.png]]
![[Pasted image 20250216150139.png]]



### BuffModel

- **tags**：是一个string数组，这相当于是对于buff的一个属性的描述。
- **priority**：优先级，这通常是一个int，也是需要策划预设好的。
- **tickTime**：buff的工作周期，单位：秒。
- **maxStack**：最大堆叠层数。

### BuffObj

- **model**：BuffModel
- **time**：也就是buffObj的时间，permanent代表了是否是一个永久的，如果是永久的，duration就不会变化，但是timeElapsed依然会增加，ticked也会随着触发OnTick的次数增加；duration是生命周期，当他小于等于0的时候BuffObj就将被删除了；timeElapsed是记录了这个BuffObj存在了多久了，因为duration这个数据也是可以在运行时被改动的，比如我有个“加热”技能，他的效果是“灼热”的生命周期+20秒，并且随着灼热的运行的时间越久，每次工作造成的伤害也越大。这里不仅要改写model的tickTime，还要连OnTick函数都改写，依赖的参数就是这个timeElapsed（越久伤害越高）。
- **caster**和**carrier**：是buff的释放者和携带者两个GameObject，释放者可能是null的.
- **stack**：当前层数，也是因为游戏设计需要才存在的，如果都是1层，就不需要了。
- **param**：一些动态的参数，比如《魔兽世界》中牧师的盾，还能吸收多少伤害的具体数值，就记录在这里，每次受到攻击都会减少多少。原本最早版本的buff机制中，这个盾的效果被记录在stack里，但这样显然是不对的——stack的意义不是这个，如果强行这么用就有了二义性，盾吸收值也许是一个Int128甚至Int256，而stack通常只需要Byte就够了。
### AddBuffInfo

这是创建BuffObj的中间件，确切地说，我们每次要给角色的Buff做增删改查，都应该创建一条AddBuffInfo，而非直接修改ChaState中的buff。因为我们可能有些技能的效果会导致角色的添加buff，比如说有一个buff的效果是“受到伤害的时候获得厚皮（另一个buff）”，“厚皮”的效果是受到伤害时降低50%，并且下一次伤害免疫（又是一个新的buff），这时候我们如果同一轮里面执行就会执行到后面2个新增加的buff效果，但实际上这是不应该发生的，只是恰好C#的list管理方式碰巧有这个效果在那里，但这非常的不安全。所以我们正确的做法，就是当要给角色添加buff的时候，一定是产生一个AddBuffInfo，由BuffManager来管理buff的添加。AddBuffInfo这个“中间件”的属性有：

- **model**：也就是要创建的BuffObj的model，这里也用model而不是一个id然后去buffModel表里查找，是因为model是可以通过脚本代码动态生成的，他也是个数据，所以他的源不应该是唯一的（仅仅来自于策划填表，确切地说，所有的Model数据都不应该只能是读表读来的）。
- **caster**和**carrier**：谁给谁添加。
- **time**：要添加的时间，如果是永久的，那么duration就不在重要了，否则就是根据**durationSetTo**来确定是给buff添加一个时间还是设置为这个时间。因为这很可能是在改动一个已经存在的BuffObj，而并不一定总是新增一个BuffObj。
- **addStack**：要添加的层数，如果是负数就是要减少的层数，0则表示层数不变，尽管如此，添加了之后只要BuffObj不符合被删除的标准，就还是会导致BuffOnTick被执行。
- **param**：要设定给BuffObj.param的值，这个设定规则可以由策划定，当然对于程序来说最好的就是新的覆盖老的，但通常来说，这样做是无法满足需求的。

### Buff的“回调点”  

Buff的“回调点”的本质，就是在一些固定的逻辑代码中安插脚本片段，来改变逻辑执行所依赖的数据，从而使得整个流程走完会得到“不同寻常的效果”。Buff回调点是整个游戏流程设计的灵魂，因为Buff的回调点有哪些，是完全取决于游戏需要的，我们可以把游戏的任何一段代码里都安插上，但越多的回调点会导致游戏运行效率越低，所以归纳好回调点是非常重要的事情，而归纳回调点的核心思路是——在设计一个玩法的同时，也去设计一些数据，来试着填充玩法，而不是凭空去想一个“创意”，比如我们要设计一个卡牌游戏，规则设计完了之后，我们应该马上设计一些具体的卡牌，然后看看这些卡牌需要哪些回调点来支持逻辑。通常来说，一个ARPG（当然也包含了顶视角射击游戏），他所需要埋设buff回调点的流程，不外乎就是伤害流程和角色相关的一些流程（比如buff的增删改和释放技能的时候）。

相对于对attacker身上的某些标记进行各种不同的if else 代码.我们通过对角色挂载buff标记,反过来让角色的标记在流程里告诉我么该做什么!!!(在伤害流程中个点循环所有buff中相同的回调事件去触发)
```cs
/// <summary>

/// 在BuffObj被创建、或者已经存在的BuffObj层数发生变化

/// </summary>

public delegate void BuffOnOccur(BuffObj buff, int modifyStack);

/// <summary>

/// 在一个buff因为生命周期结束，或者层数<=0的时候，他要被移除掉之前，

/// </summary>

public delegate void BuffOnRemoved(BuffObj buff);

/// <summary>

/// 这是最常见的buff效果“每一跳”的回调点，我们通常所说的“间歇性效果”

/// </summary>

public delegate void BuffOnTick(BuffObj buff);

/// <summary>

/// 这是技能在“命中”时候执行的一个回调点

/// </summary>

public delegate void BuffOnHit(BuffObj buff, ref DamageInfo damageInfo, GameObject target);

/// <summary>

/// 与OnHit相呼应的是，这是受到攻击时候触发的逻辑

/// </summary>

public delegate void BuffOnBeHurt(BuffObj buff, ref DamageInfo damageInfo, GameObject attacker);

/// <summary>

/// 在确定会击败对手的时候执行的回调

/// </summary>

public delegate void BuffOnKill(BuffObj buff, DamageInfo damageInfo, GameObject target);

/// <summary>

/// 在确定会被击败之后执行的回调

/// </summary>

public delegate void BuffOnBeKilled(BuffObj buff, DamageInfo damageInfo, GameObject attacker);

/// <summary>

/// 这是在角色释放技能的时候发生的回调

/// </summary>

public delegate TimelineObj BuffOnCast(BuffObj buff, SkillObj skill, TimelineObj timeline);
```
## 子弹（Bullet）

![[Pasted image 20250216164745.png]]
子弹下面一样有一个ViewContainer，作为装特效、或者说子弹外观的一个容器，当有必要改变子弹的外观大小等的时候，应该改变的是ViewContainer的大小，而非整个BulletObj。
![[Pasted image 20250216164833.png]]
### UnitMove和UnitRotate

UnitMove和UnitRotate，就是CharacterObj下也会用到的2个控件，他们在这里的作用是控制子弹的移动和旋转。

 ### BulletState

BulletState对于BulletObj，就如ChaState对于CharacterObj，是一个核心的存在——有了BulletState的GameObject才是真正的BulletObj。

## Area of Effect（AoE）

和CharacterObj、BulletObj一样，AoeObj也是一个“空”的GameObject下放了一个“空”的ViewContainer：
![[Pasted image 20250216172815.png]]
使用方式与子弹相似
### AoE与子弹为何不可互相取代？

- 当我们设计子弹的时候，出发点是角色发射出这么一系列子弹，如果碰到了什么应该有什么效果。
- 当我们设计AoE的时候，出发点是捕捉到了一个范围内的角色或者子弹，对他们干点什么；然后就是如果有角色或者子弹进来、离开或者呆在里面，该对他们干点什么。

## AoE技能添加流程

1. 创建动画预制体![[Pasted image 20250217170057.png]]
2. 在技能设计表中设计技能资源消耗![[Pasted image 20250217171734.png]]
3. 在时间轴设计表中设计时间轴具体实现事件![[Pasted image 20250217172338.png]]
4. 在AoE设计表中设计AoeModel,包含步骤1的预制体和需要的回调函数![[Pasted image 20250217172545.png]]
5. 在战斗开始GameManager学习技能![[Pasted image 20250217172916.png]]
6. 在PlayerController中使用技能![[Pasted image 20250217173415.png]]

## BUFF添加流程

1. 创建动画特效![[Pasted image 20250224171107.png]]
2. 在技能设计表中设计技能资源消耗(同上)
3.  在时间轴设计表中设计时间轴具体实现事件(创建播放特效timeline和添加Buff Timeline)![[Pasted image 20250224171332.png]]
4. 在Buff设计表中设计BuffModel,在需要触发回调函数(在buff设计脚本中)的位置填写对应函数的键名称![[Pasted image 20250224171501.png]]
5. 在Buff设计Script中对应的回调点内创建键值与函数的对应![[Pasted image 20250224171909.png]]
6. 设计具体函数,在这个回调点这个buff应该做什么![[Pasted image 20250224172015.png]]
7. 在战斗开始GameManager学习技能(同上)
8. 在PlayerController中使用技能
