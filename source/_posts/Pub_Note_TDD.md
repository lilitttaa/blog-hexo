---
title: TDD 一种改变编程范式的开发方法
category:
 - Program
sortValue: 70000
---
[TOC]

## 什么是 TDD(Test Driven Development)

TDD 用最简单的话来说,就是实现之前先写测试.

可以拆解为三个步骤:

- Red:先写测试代码, 由于此时缺少实现, 所以测试代码会失败
- Green:编写实现代码, 使得测试代码通过
- Refactor:重构代码

### Red Green Refactor

我们想要实现一个 TODO 中添加一个任务的功能

第一步 Red, 先写一个测试用例, 测试用例会失败:

```cpp
// test.cpp
void Test_Given_TodoApp_When_AddTask_Then_HasTask(){
  TodoApp todoApp;
  todoApp.addTask("task1");
  assert(todoApp.hasTask("task1"));
}
```

Red 阶段我们并没有实现 TodoApp 类, 所以测试用例会失败. 在这个阶段根据需求来编写测试码,其中最重要的有两件事:

1. 测试用例的名称要能清晰的表达出测试的功能, 命名可以比较长, 建议参考 BDD 的命名规范.即: **Given-When-Then**(给定一个 TodoApp, 当添加一个任务, 然后任务列表中应该有个任务)
2. 测试代码应当跟测试名表现出一种一致性. 给出一个反面例子:

```cpp
void Test_Given_TodoApp_When_AddTask_Then_HasTask(){
  TodoApp todoApp;
  todoApp.addTask("task1");
  assert(todoApp.taskCount() == 1); // 对比前面，这里的表达没有跟命名产生一致性
}
```

第二步 Green, 编写实现代码, 使得测试通过:

```cpp
class TodoApp{
public:
  void addTask(string task){
  tasks[task] = true;
  }
  bool hasTask(string task){
  return tasks[task] == true;
  }
private:
  map<string, bool> tasks;
};
```

此时测试用例会通过, 但事情并没有结束, 还剩下非常重要的的一步——重构（Refactor）。重构应当保证少量多批的增量式修改, 每次重构后运行测试代码确保安全, 然后提交代码。

## TDD 驱动设计

### 保持简单(简单设计原则)

1. **Big Design Up Front(BDUF)**
   前面的例子已经足以展现出 TDD 与过去的编程方式的区别了，其核心在于延迟设计。**过去的编程方法试图先完成一个大的，足够好的设计，尽可能的预测各种未来的变更**。这种设计方法被称之为 Big Design Up Front，在敏捷社区已是臭名昭著。反对它的最主要原因是，**BDUF 容易产生过度抽象**。在一个复杂的系统中，解开一个抽象必须小心的处理与其他实体的所有交互点，比起建立一个新的抽象要困难得多。
2. **You Ain't Gonna Need It(YAGNI)**
   TDD 的方法则是将设计延迟到功能实现后的重构来进行。并且在重构时也需要小心的对未来进行预测，遵循 YAGNI 原则，即**尽可能的延迟创建抽象，除非你意识到如果不创建抽象会导致未来的变更变得困难**。**面对快速变化的需求，保持简单比起预测未来更为可靠**。（采用 YAGNI 并不表示完全不用预先考虑架构。总有一些时候，如果缺少预先的思考，重构会难以开展。但两者之间的平衡点已经发生了很大的改变：如今我更倾向于等一等，待到对问题理解更充分，再来着手解决。）
3. **第二次做好**
   TDD 用真实的用例来定义行为，这比抽象的思考要容易。并且它有助于构建更稳定的接口，当我们有两三个用例之后，将代码通用化会比只有一个用例来得简单得多。

### 测试驱动 API 设计

在 TDD 中，系统的的 API 设计是很自然的在测试代码编写中产生的。

想想在测试代码中我们所干的事，首先在准备阶段我们创建测试的对象，以及这个对象所依赖的其他实体和数据，然后调用对象完成一个行为，最后验证行为发生后的结果是否符合预期。

在这种结构下所产生的 API 设计会比 BDUF 的更好。原因在于，**准备阶段会让我们明确测试对象所需要的依赖，这促进了上下文无关性，因为我们必须能够建立目标对象的环境，然后才能对它进行测试。验证阶段需要对象具有清晰的可观测性，这使得我们必须明确功能表达的实质**。于是最终得到的 API 自然显得职能明确，依赖清晰。

另外，作为你的代码的第一个客户，测试可以告诉你很多关于设计选择的信息。你的系统是否与数据库结合得太紧密了？API 是否支持所需的用例？你的系统是否能处理所有的边界情况？编写自动化测试迫使你在系统开发周期的早期就面对这些问题。（满足我们后面会提到的左移原则）

与之相反的，当我们发现一个特征难以测试时，我们不要只问自己如何测试它，同时也要问一问为什么它难以测试。经验证明，**如果代码难以测试，最有可能的原因是设计需要要改进**。现在让代码难以测试的结构，将来就会让代码难以修改。

为了让系统更容易测试，我们通常会使用依赖注入这样的手段，这也可以解耦对象和它们所使用的服务。

### 对象之网，设计的本质

我们考虑每个函数构成一个点的简单模型，其中每个函数的命名表达了它所包含的语义。在这个模型的语义空间中，语义接近的两个函数，距离会靠的较近。

然后我们可以把其中几个函数圈起来，构成一个新的实体（类，模块），类与类之间相连则代表它们在这个对象之网模型中具有某种关系。除了把函数圈起来，我们也可以把类圈起来，构成的新的实体（子系统，系统，模块）。小的实体组合起来构成大的实体，以此类推。

在这样的模型下，我们可以来讨论一些概念。

#### 重构

系统总是会迎来新的变化，这种变化可能是新的功能，也就是产生了新的函数节点。也可能是旧的功能的变化，可能会导致函数语义的变化。

当产生新的函数节点时，显然我们要做的第一件事就是为它们找个家，如果函数的语义跟现在的某个类的语义非常接近，那么可以直接把它加入到该类中去，反之我们则选择创建一个新类。

加入到存在的类中，对该类的语义做了进一步的扩展，可能会导致类的命名不够准确，于是便要考虑改名。

而创建一个新类，也可能发现其他函数的语义离这个新类更加接近，于是需要把其他的类的函数移到这个类中来。

这样的过程其实就是**重构**，即不断地让实体移动到更合适的位置，并且让其语义表达的更加准确和清晰。

#### 抽象

抽象，产生一个抽象就是画一个新的圈。于是空间中很直观的一点，连线变少了。所以其实抽象的价值之一就是在于屏蔽细节（隐藏），外部系统与之关联的交互点数量降低了，或者说使用该系统所需要的大脑的上下文变少了。

#### 过度抽象

抽象是一个非常强大的工具，让我们在有限的大脑上下文下完成极其复杂的功能。但是抽象也是有代价的。代价在于它会让系统变得更加复杂，在这里，也就是函数被一层一层的圈起来。这些圆圈把原本的函数完全给隐藏起来了，外部类想要访问这些功能就不得不需要考虑对当前的抽象结构进行调整，以便把这些函数暴露出来。在这个过程中，每一层圆圈都会对其带来阻碍。这也是为什么一个过度抽象的系统难以迎接变化。

#### Mockery

对象之网中，与某个节点或圆圈关联的边描述了这个实体与其他实体之间的关系。从这个角度来看，Mockery 本质上就是对实体的关系的验证。它用来确定一个特定的实体是否按照我们期待的方式与其他部分进行交互。

### Emergent Design(TDD 循环)

简单设计中包含四个原则

1. 使测试通过
2. 减少重复
3. 增强意图
4. 最少的函数和类

其中第二点和第三点构成了**TDD 反馈循环**的基础。

1. 当我们专注于减少重复时，新的、有用的结构就会产生。
2. 当我们专注于改善命名时，**Feature Envy**（指一个方法对另一个对象的兴趣超过对自己所属对象的兴趣）就会变得更加明显，代码清晰的表明它想要移动到其他位置。拥挤的、不相关的概念彼此分离，职责逐渐转移到代码库中更合理的部分。
3. 与此同时，新的结构产生时也需要我们对其**命名**，并总是需要对其进行不断地改进。
4. 新的有用的结构并不总是产生新的抽象层次，不过随着我们改善命名，移动代码，这些新的抽象层次也会开始显现出来。
5. 在这个过程中，往往又会产生更多的重复。

由此构成了整个循环。
![Alt text](image-1.png)

我们可以看到，在 TDD 中，好的设计是通过这样的反馈循环不断的改善而逐渐显露出来的，这一点被称之为**持续重构**。

## 代码的 Bad Smell

### 命名与注释

改名不只是简单的修改名字，因为如果想不出一个好名字，往往意味着背后隐藏了更深入的设计问题。**为一个恼人的命名付出的纠结，常常驱动我们去寻找更好的设计。**

编程语言提供的变量、函数、类、模块的命名是我们用来表达清楚设计意图的最重要的工具。

下面会讨论几种常见的糟糕命名情况：

**缩写**，会降低代码的可理解性，如果后续的维护者缺少了你所知的上下文，他们将很难理解你的代码。
另一方面，有时候使用缩写只是为了在重复写某个单词，或者写一个长命名时简单一点。这隐藏了一些设计问题，连续重复某个单词可能意味着重复代码，违背了 DRY 原则。过长命名也可能意味着类或函数的职责过多，违背了单一职责原则。

**缺少命名**，对于复杂的**表达式**或者**字面值**，如果没有一个好的名字，其含义就难以理解。

```C++
Delay(doSomeThing，24*60*60*1000)
```

```C++
const int MILLISECONDS_PER_DAY = 24*60*60*1000
Delay(doSomeThing，MILLISECONDS_PER_DAY)
```

**重复前缀**，可能意味着缺少一个上下文，即类或者模块的包裹。

```C++
const int carSpeed = 100;
const int carWeight = 1000;
const string carColor = "red";
```

```C++
class Car {
 int speed = 100;
 int weight = 1000;
 string color = "red";
};
```

另外，在已有上下文中的重复前缀是没有必要的，它没有提供额外的信息，还使得外边的类或者模块命名重构时变得更加困难。

```C++
class Car{
 int carSpeed = 100;
 int carWeight = 1000;
 string carColor = "red";
};
```

```C++
class Train{ //重构后Train与car就不匹配了
 int carSpeed = 100;
 int carWeight = 1000;
 string carColor = "red";
};
```

**过长命名**，可能意味着系统中缺乏特定的抽象，导致我们只能依靠在命名中添加额外的字段来补充上下文信息。(并不绝对，保持警惕)

**编写注释**在通常意义上都被认为是一个好习惯，然而在这里却把它列入了 Bad Smell，真正的问题在于，注释被广泛的滥用了。

很多时候你会发现，注释仅仅被用来说明下面的代码做了什么，而不是为什么这么做，它只是用来掩盖代码不够清晰的坏味道。并且使用注释而不是函数，往往意味着你放弃了拆分逻辑，增加代码可重用性，以及进而建立新的抽象的机会。过度使用注释的代码库，往往可重用性差，代码重复度高。

同时，没有及时更新的注释往往更具有误导性。

所以，一旦你发现存在一处注释仅用来解释一块代码做了什么，不如试试将其提炼为一个新的函数，然后为这个函数取上一个好名字。

注释真正的价值不在于表达语义，而在于补充信息，例如，说明当时为什么要这么做而不是那么做、标记 TODO 等等。

**建议**: 函数名——**做什么（What）**，函数体——**怎样做（How）**，注释——**为什么这样做（Why）**。

### 重复代码（DRY）

DRY（Don't Repeat Yourself）：**每一段知识都应该有且只有一个表述。**

重复的第一个问题在于，阅读重复代码时你就必须加倍仔细，留意其间细微的差异。如果要修改重复代码，你必须找出所有的副本来修改。

另外，代码中的重复是试图在不同的地方做相同的事情，这种重复不只是对形式的重复，更是对意图的重复。所以不一样的代码可能是重复的，而一样的代码可能是没有重复的。**从代码意图角度来看，重复往往也意味着实体的缺失。**

正如前面反馈循环所述，消除重复代码往往会导致新的结构的产生。

### 长函数与大类

坦白来说，长函数绝对是导致代码变得难以理解的最主要的原因之一，反对它的理由简直罄竹难书。

首先，长函数通常包含了超过 5 个以上的步骤，很大程度上意味着这个函数**在不同抽象层次上工作**。这就使得你几乎无法直观的理解函数干了什么事。

同时，这也不会驱使你去尝试**建立新的抽象**，**划分组件**式的逻辑。随着类似的需求逐渐增加，代码中的**重复度**会变得越来越高。于是修改变得越来越困难，代码也变得越来越难以理解。

另外，长函数通常还会包含大量的**嵌套**，这使得代码的圈复杂度变得非常高。

**圈复杂度**：1976 年 12 月，Thomas J. McCabe 在他的论文《复杂性度量》中最早提出圈复杂度的概念，它代表代码中的路径数量。当只有一个条件时，代码的圈复杂度是 2，表示代码中有两种可能的路径。嵌套两个时复杂度就变为 4，以此类推，这种增长是指数级的。

多数人都无法处理太多的条件判断，**圈复杂度越高的代码出 bug 的概率也越高。**

降低圈复杂度最简单的方法是拆分函数，除此之外**多态**也是一种有效降低圈复杂度的方法。甚至在激进的 OOP 布道者看来，除了构造时使用的分支，其他的都应该用多态消除。

也可以使用**return**和**assert**，即便没有降低圈复杂度，但是减少嵌套也能有效提高代码的可读性，类似于这样：

```C++
void foo(){
  if(condition1){
    if(condition2){
      if(!condition3){
        error("condition3 failed")
      }
    }
  }
}
```

可以改写为：

```C++
void foo(){
  if(!condition1) return;
  if(!condition2) return;
  assert(condition3, "condition3 failed");
}
```

人们对付复杂性的唯一方法就是利用更高的抽象层次避免它。与操纵变量和控制流相比，如果通过组合现有的的组件来编程，我们就能做更多事。

所以，理想情况下，一个好的系统应该是由一系列的小函数组成，每个函数都有一个明确的职责，每个函数由几个清晰的步骤组成。众多的小函数促进了功能的组合和复用，构成了上层的抽象，并且**良好命名的小函数形成了代码的自文档。**

**"写非常小的函数——通常只有几行的长度。一个函数一旦超过 6 行，就开始散发臭味." -Martin Fowler**

与之相对应的**大类**是我们下一步消除的对象，大类通常承担了过多的功能，这使得它的职责变得模糊，难以理解，也难以复用。另外大类提供了过多的上下文，既难使用，也使得多态的替换变得困难。

而采用许多微型类意味着当我们需要进行修改的时候，可以更容易地专注于一个或几个类，让修改局部化且容易实现。

**总结：**

- **尽可能让你的函数不超过一层嵌套。**
- **尽可能让你的函数不超过 10 行。**
- **尽可能让你的类不超过 50 行。**

### 长参数列表、纯数据类与 Feature Envy

长参数使得调用变得困难，不过更重要的还是消除长参数带来的巨大收益，一般会从两个方面出发：

一个是保持**参数的正交性**，即如果一个参数能通过其他参数推导出来，那么就可以尝试消除这个参数，用一个查询函数来替代。

另一个就是将参数**封装为一个纯数据类**（纯数据类是指一个类中只有字段和 setter/getter 方法，没有任何逻辑），这不仅可以消除参数，提高代码的可读性，参数对象的复用也能有效的降低代码的重复度。当纯数据类产生时，第一颗多米诺骨牌倒下了。

事实上**纯数据类**同样是个 Bad Smell，它很大程度上意味着**行为被放置在了错误的地方**，真正操纵数据的行为被分散到了代码库的各处，这一点或者也被叫做依恋情结（Feature Envy）。

要消除纯数据类应该查看调用这些类的 setter 的地方，尝试将**操纵数据的行为搬移到纯数据类中**，随着行为逐渐围绕着数据而建立，一个新的实体便应运而生，这也是第二颗多米诺骨牌倒下之处。再进而我们重构代码，调整实体间的交互，移动代码，更新命名……一颗颗多米诺骨牌倒下，代码在持续重构中逐渐变得清晰。

所以消除长参数的价值不仅仅源于其这一孤立的行为本身，而更多的在于它是如何向我们展现出一个起点。当我们从这个起点出发，便可以逐渐推进代码的组件化，内聚化，抽象化，最终获得一个清晰的设计。

关于封装的类作为参数能提高可读性，我还想要补充一点，**函数的参数列表阐述了函数如何与外部世界共处**，比起一个长长的参数列表，一个封装的类能更好的表达这个函数的**上下文和依赖**。

纯数据类也有例外，例如中转数据对象（Transfer Object），这种对象的存在是为了方便数据传输，它们的值应该是**只读的**且**不需要封装**。

前面谈到，当我们面对 Feature Envy，我们需要运用搬移手法将行为封装到类中。除此以外，还有两种 Bad Smell 会用到搬移手法：

**发散式变化**：各种不同的原因都导致了一个模块的变化。这通常意味着你在同一个模块中，处理了不同的上下文逻辑。这时候你需要将这些不同的原因分离开来，使得每个模块只处理一个原因。

**霰弹式修改**：如果遇到某种变化，你需要在许多不同的类中做出许多小修改，这些修改通常是相似的。这时候你需要将这些修改搬移到一个地方，使得你只需要在一个地方做出修改。

### 可变数据，SideEffect，纯函数，全局变量

纯函数是这样的函数：**给定相同的输入，将始终返回相同的输出（包括引用），并且没有任何可观察到的副作用（SideEffect）。**

SideEffect：例如，数据库写入，文件系统写入，网络通信等。

纯函数的好处有很多，例如：记忆化，将纯函数的输入输出缓存起来构建查询表，编译器优化（例如公共子表达式消除和循环优化），易于单元测试，容易 Debug 等等。

因此，有一整个软件开发流派（函数式编程），完全建立在“数据永不改变”的概念基础上：如果要更新一个数据结构（Immutable Data），就返回一份新的数据副本，旧的数据仍保持不变。

如果你对前端开发熟悉的话，React 中数据更新就是以 Immutable 的形式来完成的，甚至借助于 Redux 这样的状态管理库，可以实现时间旅行（随时回到 App 的任何一个状态），这使得 Debug 变得极其容易。除此以外，还有 Haskell 这样的只使用纯函数的语言。

纯函数是某种承诺，**承诺它不会偷偷改变你的数据，这使得我们不用担心它会对其他部分造成影响**。与之相对的便是可变数据，在一处更新数据，却没有意识到软件中的另一处期望着完全不同的数据，于是一个功能失效了（很多时候 Bug 不就是这样产生的吗）。

所以，你一方面需要**封装数据成函数使其容易监控**，另外则需要尽可能的**拆分函数为纯函数与非纯函数**。至少这会使得 Debug 变得轻松一些，例如：当你看到一个查询函数时，你可以放心的忽略它。

在 OOP 中，编程语言通常提供了控制访问权限的机制，例如：private，protected，public 等。这一点确实保证了**信息隐藏**（隐藏了对象在 API 抽象之后实现其功能的方式。它让我们能够工作在更高的抽象层上，忽略与当前的任务无关的低层细节），但这却没有严格保证**数据的封装性**（只能通过对象的 API 来影响对象的行为。这确保了不相关的对象之间没有未预料到的依赖关系，从而让我们能够控制一个对象的修改对系统其他部分的影响程度），当对象依赖于某些全局变量时，改变全局变量同样可以影响对象的行为。

换句话说，全局变量（单例）是一种**隐式的依赖**，比起通过函数参数传递的显式依赖，隐式依赖更难以追踪，也更容易引入 Bug。事实上，你可以从代码库的任何一个角落都可以修改它，而且很难探测出到底哪段代码做出了修改。如果能保证在程序启动之后就不再修改，这样的全局数据还算相对安全，不过得有编程语言提供这样的保证才行。

另外，使用全局变量（单例）也是一种放弃设计的做法，它使得编码者完全不去思考系统与系统之间的边界。常常看到两个系统通过全局变量不必要的耦合在一起。

要消除单例可以参考：<https://gpp.tkchu.me/singleton.html>

### 基本类型与基本容器

关于这个 Bad Smell，最有名的案例当属 1999 年 NASA 的火星气候探测器在火星大气层中烧毁了，烧毁的原因之一是导航软件混淆了米和英制单位。

以 OOP 的视角来思考这个问题，核心在于**基础类型缺乏语义信息**。或者，反过来说，正是由于基础类型缺乏语义的约束，导致它们几乎可以表达任何意图，例如，当你看到一个 string 类型时，你能知道它是一个名字，一个地址，一个文件路径还是一段表示校验功能的 Config 的 Json 文本吗？

当一个方法将 int 类型作为参数，方法的签名需要做所有的工作来表达意图。但如果同样的方法将 Hour 作为参数，则很容易看出发生了什么。而且就像我们前面在消除纯数据类里所说的，**像 Hour 这样的小对象也给了我们一个很好的地方来放置原本散布在其他类中的行为**。

直接使用**基本容器类型**跟直接使用基本类型本质是一样的，都意味着我们没有在系统中定义合适的实体，如果我们在系统中创建一个 Item 类型而不是使用 String，我们在修改时就能够找到所有相关的代码，而不必跟踪方法调用。

### 重复 Switch

正如我们前面所说，在激进的 OOP 布道者看来，除了构造时使用的分支，其他的都应该用多态消除。不过即使表现得更温和一些，重复的 Switch 和 If-Else 也会被认为是一种 Bad Smell。它们的问题在于：**每当你想增加一个选择分支时，必须找到所有的 Switch，并逐一更新**。而且大多数时候，这些 Switch 还是分散在代码库各处，显然这还散发着霰弹式修改的臭味。

解决这个问题的方法是**使用类型多态: 分析分支的条件，识别出概念，并将这些概念转化为我们的系统中的第一类对象。**

### 临时字段与非完整构造

临时字段是指**一个对象的某个字段仅在某种特定情况下才会被使用**。这样的代码很容易让人误用，因为你通常认为对象在所有时候都需要它的所有字段。

消除它只需要**把这个字段相关的逻辑都提到一个新类中。**

另外一个让类变得容易误用的 Bad Smell 是**非完整构造**，即**部分地创建一个对象，然后通过设置属性来完成创建工作**。

```C++
ResourceLoader loader = new ResourceLoader();
loader.setPath("path/to/resource");
loader.setLoadFromNetwork(true);
loader.load();
```

这种做法是脆弱的，因为使用者必须记得设置所有的依赖关系。而且如果对象改变，添加了新的的依赖关系时，原有的客户代码仍然能够编译，但是它创建的已经不再是一个运行时合法的对象了。

你至少应该通过构造函数**将对象属性初始化为一个安全的默认值**，保证构造的安全性，然后**通过受限的 API 保证设置属性的正确性**。

### 迪米特法则、Tell, Don't Ask

**Tell, Don't Ask**是指我们应该让对象直接完成我们想要它做的事，而不是先查询它的状态，然后再根据状态来做事。

这个决定很重要，因为它会影响对象使用的难易程度，从而影响系统的内部品质。如果我们通过 API 暴露了过多的对象内部信息，它的客户最后就需要代替它完成工作。最终会将系统的行为分布到许多外部对象上，导致外部对象与系统耦合在一起。

基于这一点，正如 Object Calisthenics 中所建议的，**我们应该在编码中更谨慎的使用返回值**。返回值使得系统的操作和状态被分享到除了调用外的其他地方，**而通过消除返回值的能力，迫使我们依靠而不是询问对象来实施行为。**

另一个关于对象交互之间的原则是**迪米特法则（The Law of Demeter）**：形象的说法是，你可以玩你的玩具、你制作的玩具以及别人给你的玩具，但你永远、永远不要玩你玩具的玩具。更具体来说就是，不要连续使用多个点操作符。

```C++
studentModel.getDepartment().getHead().getAddress().getCity().print();
```

而是直接通过你的邻居完成工作。

```C++
studentModel.printCity();
```

不过，这个法则存在一些争议，有人认为它会导致系统中出现大量的中介类，使得系统变得更加复杂。在使用这个法则时，你需要权衡一下。不管怎么样，看到连续的点操作符，保持警惕。

## 重构

### 重构的原则

- **重构最重要的原则，每次只前进一小步**。小批量的逐步重构（每次只修改一个命名，调整一个函数……，每次重构尽量只停留几分钟），运行并通过测试，然后提交代码。一方面保证单次重构只关心单个问题，另一方面保证重构的稳定性，你停的越久，这次重构带来的风险就越大。
- **童子军法则**：让营地比你到达时更干净。(每次触碰一块代码时，尝试把它变好一点点。)
- 添加新功能时，不应该修改既有代码，只管添加新功能。而重构时也不能再添加新功能，只管调整代码的结构。重构与添加功能混在一起，使得提交记录阅读变得困难，不方便回退，同时也非常影响 CodeReview 。
- 如果在添加功能的时候发现一些需要重构的地方，建议可以先用 TODO 标记，避免忘记。待功能完成，遵循童子军法则，及时重构。
- 重构区别于重写，应该是安全且持续的，如果有人说他们的代码在重构过程中有一两天时间不可用，可以确定，他们在做的事不是重构。

### 何时进行重构

- 预备性重构: 重构的最佳时机就是在添加新功能之前，通过重构使得新功能能够更容易的添加。重构时去思考新功能会在哪里落脚，需要做出哪些扩展。

- 只要发现代码意图并不清晰时重构(没法一目了然理解代码的含义，或者说代码没有形成自文档)。

- 每次触碰代码时重构（童子军法则）。

- Code Review 时重构: 借助他人 Review 代码给出的建议，对你提升代码质量有很大帮助。除此以外也可以是在 Pair Programming 时针对搭档给出的持续性的反馈进行重构。

### 重构遗留代码

如果遗留的屎山代码只用调用 API，不需要修改它，那么就不要去碰它，除非你需要理解其内部的实现。

当你需要对其进行重构时，一个简单的且显而易见的答案——没测试就加测试。

关于重构大规模遗留代码的更具体的技巧，请参考下面的【大规模重构】。

### 数据库重构

在过去，数据库被认为是重构经常出问题的一个领域，每次迭代数据库的结构都会带来相当大的风险。

一种处理数据库重构的方法是**渐进式数据库设计[mf-evodb]和数据库重构[Ambler & Sadalage]办法**，其核心在于**小步修改并且每次修改都应该完整**。

**数据库重构最好是分散到多次生产发布来完成**，这样即便某次修改在生产数据库上造成了问题，也比较容易回滚。例如：要修改一个字段，第一次提交会新添一个字段，但暂时不使用它，修改数据写入的逻辑，使其同时写入新旧两个字段。随后可以修改读取数据的地方，将它们逐个改为使用新字段，暂停一小段时间。然后看看是否有 bug 冒出来，确定没有 bug 之后，再删除已经没人使用的旧字段。

如果你需要对数据库中数据进行转换，可以写一小段代码来执行数据转化的逻辑，并把这段代码放进版本控制，跟数据结构声明与使用代码的修改一并提交。

### 大规模重构

大规模重构有如下几种方式：

传统方法——**Big Bang**：对现有系统进行彻底的、全面的改动。这可能包括重写整个代码库、改变架构或引入全新的技术。显而易见，Big Bang 方法的风险很高，如果更改没有经过充分的测试，可能会导致系统不稳定、功能丧失或其他严重的问题。与逐步进行的重构方法不同，Big Bang 方法也不提供逐步进展的机会。所有的更改都是一次性完成的，这可能会使得问题难以追踪和解决。如果重构过程中出现问题，回退到之前的版本也会非常困难。另外，由于需要对整个系统进行更改，Big Bang 方法可能会导致较长的停机时间。

模块化重构方法——**Divide and conquer**：分而治之，将一个大问题分解成一系列更小的、更易于解决的子问题。这种方法的优点在于可以针对局部问题进行处理，不会导致大规模的停机，同时也可以分到多人一起完成。但要理清模块间的依赖并不是一件容易的事。

局部持续重构方法——**Strangling the classes**：通过逐步替换旧的类和模块来改进整个系统。这种方法符合持续法则，能够更安全的小步前进。同样的 Strangling the classes 需要对代码库有深入的了解，理清其中的依赖并不容易。

天皇法则——**Mikado Method**：通过创建依赖图来帮助开发者理解代码之间的依赖关系，并以一种可控和系统化的方式进行重构。这种方法由 Daniel Brolund 和 Ola Ellnestam 开发，其命名源自一个名为 Mikado 的日本游戏（该游戏需要玩家小心地移除木棍，避免触碰其他木棍）。

<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="636px" viewBox="-0.5 -0.5 636 712" content="&lt;mxfile&gt;&lt;diagram id=&quot;tZe41FnKF79DnQRkhALj&quot; name=&quot;Page-1&quot;&gt;&lt;mxGraphModel dx=&quot;2994&quot; dy=&quot;1086&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;&lt;root&gt;&lt;mxCell id=&quot;0&quot;/&gt;&lt;mxCell id=&quot;1&quot; parent=&quot;0&quot;/&gt;&lt;mxCell id=&quot;14&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;2&quot; target=&quot;3&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;2&quot; value=&quot;定义目标&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;170&quot; y=&quot;60&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;15&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;3&quot; target=&quot;4&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;3&quot; value=&quot;设置10分钟的倒计时&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;170&quot; y=&quot;160&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;16&quot; value=&quot;倒计时到了&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;4&quot; target=&quot;5&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;4&quot; value=&quot;实现目标或前置依赖&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;170&quot; y=&quot;260&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;17&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;5&quot; target=&quot;6&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;22&quot; value=&quot;Yes&quot; style=&quot;edgeLabel;html=1;align=center;verticalAlign=middle;resizable=0;points=[];&quot; vertex=&quot;1&quot; connectable=&quot;0&quot; parent=&quot;17&quot;&gt;&lt;mxGeometry x=&quot;-0.3809&quot; y=&quot;-1&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;&lt;mxPoint as=&quot;offset&quot;/&gt;&lt;/mxGeometry&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;20&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;5&quot; target=&quot;9&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;21&quot; value=&quot;No&quot; style=&quot;edgeLabel;html=1;align=center;verticalAlign=middle;resizable=0;points=[];&quot; vertex=&quot;1&quot; connectable=&quot;0&quot; parent=&quot;20&quot;&gt;&lt;mxGeometry x=&quot;-0.2876&quot; y=&quot;1&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;&lt;mxPoint as=&quot;offset&quot;/&gt;&lt;/mxGeometry&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;5&quot; value=&quot;是否存在错误&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;170&quot; y=&quot;370&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;18&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;6&quot; target=&quot;7&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;6&quot; value=&quot;弄清什么在阻碍你前进&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;170&quot; y=&quot;490&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;19&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;7&quot; target=&quot;8&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;7&quot; value=&quot;定义新的目标和前置依赖&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;170&quot; y=&quot;590&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;43&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;8&quot; target=&quot;42&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;8&quot; value=&quot;选择下一个前置依赖去处理&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;170&quot; y=&quot;710&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;23&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;9&quot; target=&quot;10&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;41&quot; value=&quot;Yes&quot; style=&quot;edgeLabel;html=1;align=center;verticalAlign=middle;resizable=0;points=[];&quot; vertex=&quot;1&quot; connectable=&quot;0&quot; parent=&quot;23&quot;&gt;&lt;mxGeometry x=&quot;-0.3719&quot; y=&quot;-1&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;&lt;mxPoint as=&quot;offset&quot;/&gt;&lt;/mxGeometry&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;29&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;9&quot; target=&quot;28&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;40&quot; value=&quot;No&quot; style=&quot;edgeLabel;html=1;align=center;verticalAlign=middle;resizable=0;points=[];&quot; vertex=&quot;1&quot; connectable=&quot;0&quot; parent=&quot;29&quot;&gt;&lt;mxGeometry x=&quot;-0.2746&quot; y=&quot;3&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;&lt;mxPoint as=&quot;offset&quot;/&gt;&lt;/mxGeometry&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;9&quot; value=&quot;当前的改动合理吗&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;385&quot; y=&quot;370&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;26&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;10&quot; target=&quot;11&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;34&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;10&quot; target=&quot;33&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;38&quot; value=&quot;No&quot; style=&quot;edgeLabel;html=1;align=center;verticalAlign=middle;resizable=0;points=[];&quot; vertex=&quot;1&quot; connectable=&quot;0&quot; parent=&quot;34&quot;&gt;&lt;mxGeometry x=&quot;-0.2393&quot; y=&quot;3&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;&lt;mxPoint as=&quot;offset&quot;/&gt;&lt;/mxGeometry&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;10&quot; value=&quot;Commit你的改动&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;385&quot; y=&quot;470&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;27&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;11&quot; target=&quot;13&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;36&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;11&quot; target=&quot;35&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;39&quot; value=&quot;No&quot; style=&quot;edgeLabel;html=1;align=center;verticalAlign=middle;resizable=0;points=[];&quot; vertex=&quot;1&quot; connectable=&quot;0&quot; parent=&quot;36&quot;&gt;&lt;mxGeometry x=&quot;-0.2323&quot; y=&quot;-1&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;&lt;mxPoint as=&quot;offset&quot;/&gt;&lt;/mxGeometry&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;11&quot; value=&quot;最终的目标完成了吗&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;385&quot; y=&quot;560&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;13&quot; value=&quot;完成&quot; style=&quot;ellipse;whiteSpace=wrap;html=1;aspect=fixed;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;405&quot; y=&quot;650&quot; width=&quot;80&quot; height=&quot;80&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;31&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;startArrow=none;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;35&quot; target=&quot;30&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;28&quot; value=&quot;&quot; style=&quot;shape=waypoint;sketch=0;size=6;pointerEvents=1;points=[];fillColor=default;resizable=0;rotatable=0;perimeter=centerPerimeter;snapToPoint=1;rounded=1;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;630&quot; y=&quot;390&quot; width=&quot;20&quot; height=&quot;20&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;32&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;30&quot; target=&quot;8&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;30&quot; value=&quot;&quot; style=&quot;shape=waypoint;sketch=0;size=6;pointerEvents=1;points=[];fillColor=default;resizable=0;rotatable=0;perimeter=centerPerimeter;snapToPoint=1;rounded=1;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;630&quot; y=&quot;730&quot; width=&quot;20&quot; height=&quot;20&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;33&quot; value=&quot;&quot; style=&quot;shape=waypoint;sketch=0;size=6;pointerEvents=1;points=[];fillColor=default;resizable=0;rotatable=0;perimeter=centerPerimeter;snapToPoint=1;rounded=1;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;635&quot; y=&quot;490&quot; width=&quot;20&quot; height=&quot;20&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;37&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;endArrow=none;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;28&quot; target=&quot;35&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;&gt;&lt;mxPoint x=&quot;640&quot; y=&quot;389.9999999999999&quot; as=&quot;sourcePoint&quot;/&gt;&lt;mxPoint x=&quot;640&quot; y=&quot;740&quot; as=&quot;targetPoint&quot;/&gt;&lt;/mxGeometry&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;35&quot; value=&quot;&quot; style=&quot;shape=waypoint;sketch=0;size=6;pointerEvents=1;points=[];fillColor=default;resizable=0;rotatable=0;perimeter=centerPerimeter;snapToPoint=1;rounded=1;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;630&quot; y=&quot;580&quot; width=&quot;20&quot; height=&quot;20&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;45&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;42&quot; target=&quot;44&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;42&quot; value=&quot;&quot; style=&quot;shape=waypoint;sketch=0;size=6;pointerEvents=1;points=[];fillColor=default;resizable=0;rotatable=0;perimeter=centerPerimeter;snapToPoint=1;rounded=1;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;20&quot; y=&quot;730&quot; width=&quot;20&quot; height=&quot;20&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;46&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;44&quot; target=&quot;3&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;44&quot; value=&quot;&quot; style=&quot;shape=waypoint;sketch=0;size=6;pointerEvents=1;points=[];fillColor=default;resizable=0;rotatable=0;perimeter=centerPerimeter;snapToPoint=1;rounded=1;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;20&quot; y=&quot;180&quot; width=&quot;20&quot; height=&quot;20&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;/root&gt;&lt;/mxGraphModel&gt;&lt;/diagram&gt;&lt;/mxfile&gt;" onclick="(function(svg){var src=window.event.target||window.event.srcElement;while (src!=null&amp;&amp;src.nodeName.toLowerCase()!='a'){src=src.parentNode;}if(src==null){if(svg.wnd!=null&amp;&amp;!svg.wnd.closed){svg.wnd.focus();}else{var r=function(evt){if(evt.data=='ready'&amp;&amp;evt.source==svg.wnd){svg.wnd.postMessage(decodeURIComponent(svg.getAttribute('content')),'*');window.removeEventListener('message',r);}};window.addEventListener('message',r);svg.wnd=window.open('https://viewer.diagrams.net/?client=1&amp;page=0&amp;edit=_blank');}}})(this);" style="cursor:pointer;max-width:100%;max-height:712px;"><defs/><g><path d="M 210 60 L 210 93.63" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 210 98.88 L 206.5 91.88 L 210 93.63 L 213.5 91.88 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><rect x="150" y="0" width="120" height="60" rx="9" ry="9" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 30px; margin-left: 151px;"><div data-drawio-colors="color: rgb(0, 0, 0); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">定义目标</div></div></div></foreignObject><text x="210" y="34" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="12px" text-anchor="middle">定义目标</text></switch></g><path d="M 210 160 L 210 193.63" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 210 198.88 L 206.5 191.88 L 210 193.63 L 213.5 191.88 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><rect x="150" y="100" width="120" height="60" rx="9" ry="9" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 130px; margin-left: 151px;"><div data-drawio-colors="color: rgb(0, 0, 0); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">设置10分钟的倒计时</div></div></div></foreignObject><text x="210" y="134" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="12px" text-anchor="middle">设置10分钟的倒计时</text></switch></g><path d="M 210 260 L 210 303.63" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 210 308.88 L 206.5 301.88 L 210 303.63 L 213.5 301.88 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 286px; margin-left: 210px;"><div data-drawio-colors="color: rgb(0, 0, 0); background-color: rgb(255, 255, 255); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 11px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; background-color: rgb(255, 255, 255); white-space: nowrap;">倒计时到了</div></div></div></foreignObject><text x="210" y="289" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="11px" text-anchor="middle">倒计时到了</text></switch></g><rect x="150" y="200" width="120" height="60" rx="9" ry="9" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 230px; margin-left: 151px;"><div data-drawio-colors="color: rgb(0, 0, 0); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">实现目标或前置依赖</div></div></div></foreignObject><text x="210" y="234" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="12px" text-anchor="middle">实现目标或前置依赖</text></switch></g><path d="M 210 370 L 210 423.63" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 210 428.88 L 206.5 421.88 L 210 423.63 L 213.5 421.88 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 391px; margin-left: 210px;"><div data-drawio-colors="color: rgb(0, 0, 0); background-color: rgb(255, 255, 255); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 11px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; background-color: rgb(255, 255, 255); white-space: nowrap;">Yes</div></div></div></foreignObject><text x="210" y="395" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="11px" text-anchor="middle">Yes</text></switch></g><path d="M 270 340 L 358.63 340" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 363.88 340 L 356.88 343.5 L 358.63 340 L 356.88 336.5 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 340px; margin-left: 306px;"><div data-drawio-colors="color: rgb(0, 0, 0); background-color: rgb(255, 255, 255); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 11px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; background-color: rgb(255, 255, 255); white-space: nowrap;">No</div></div></div></foreignObject><text x="306" y="344" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="11px" text-anchor="middle">No</text></switch></g><rect x="150" y="310" width="120" height="60" rx="9" ry="9" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 340px; margin-left: 151px;"><div data-drawio-colors="color: rgb(0, 0, 0); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">是否存在错误</div></div></div></foreignObject><text x="210" y="344" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="12px" text-anchor="middle">是否存在错误</text></switch></g><path d="M 210 490 L 210 523.63" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 210 528.88 L 206.5 521.88 L 210 523.63 L 213.5 521.88 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><rect x="150" y="430" width="120" height="60" rx="9" ry="9" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 460px; margin-left: 151px;"><div data-drawio-colors="color: rgb(0, 0, 0); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">弄清什么在阻碍你前进</div></div></div></foreignObject><text x="210" y="464" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="12px" text-anchor="middle">弄清什么在阻碍你前进</text></switch></g><path d="M 210 590 L 210 643.63" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 210 648.88 L 206.5 641.88 L 210 643.63 L 213.5 641.88 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><rect x="150" y="530" width="120" height="60" rx="9" ry="9" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 560px; margin-left: 151px;"><div data-drawio-colors="color: rgb(0, 0, 0); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">定义新的目标和前置依赖</div></div></div></foreignObject><text x="210" y="564" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="12px" text-anchor="middle">定义新的目标和前置依赖</text></switch></g><path d="M 150 680 L 16.37 680" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 11.12 680 L 18.12 676.5 L 16.37 680 L 18.12 683.5 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><rect x="150" y="650" width="120" height="60" rx="9" ry="9" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 680px; margin-left: 151px;"><div data-drawio-colors="color: rgb(0, 0, 0); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">选择下一个前置依赖去处理</div></div></div></foreignObject><text x="210" y="684" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="12px" text-anchor="middle">选择下一个前置依赖去处理</text></switch></g><path d="M 425 370 L 425 403.63" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 425 408.88 L 421.5 401.88 L 425 403.63 L 428.5 401.88 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 383px; margin-left: 425px;"><div data-drawio-colors="color: rgb(0, 0, 0); background-color: rgb(255, 255, 255); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 11px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; background-color: rgb(255, 255, 255); white-space: nowrap;">Yes</div></div></div></foreignObject><text x="425" y="386" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="11px" text-anchor="middle">Yes</text></switch></g><path d="M 485 340 L 613.63 340" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 618.88 340 L 611.88 343.5 L 613.63 340 L 611.88 336.5 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 338px; margin-left: 535px;"><div data-drawio-colors="color: rgb(0, 0, 0); background-color: rgb(255, 255, 255); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 11px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; background-color: rgb(255, 255, 255); white-space: nowrap;">No</div></div></div></foreignObject><text x="535" y="342" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="11px" text-anchor="middle">No</text></switch></g><rect x="365" y="310" width="120" height="60" rx="9" ry="9" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 340px; margin-left: 366px;"><div data-drawio-colors="color: rgb(0, 0, 0); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">当前的改动合理吗</div></div></div></foreignObject><text x="425" y="344" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="12px" text-anchor="middle">当前的改动合理吗</text></switch></g><path d="M 425 470 L 425 493.63" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 425 498.88 L 421.5 491.88 L 425 493.63 L 428.5 491.88 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><path d="M 485 440 L 618.63 440" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 623.88 440 L 616.88 443.5 L 618.63 440 L 616.88 436.5 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 438px; margin-left: 541px;"><div data-drawio-colors="color: rgb(0, 0, 0); background-color: rgb(255, 255, 255); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 11px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; background-color: rgb(255, 255, 255); white-space: nowrap;">No</div></div></div></foreignObject><text x="541" y="442" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="11px" text-anchor="middle">No</text></switch></g><rect x="365" y="410" width="120" height="60" rx="9" ry="9" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 440px; margin-left: 366px;"><div data-drawio-colors="color: rgb(0, 0, 0); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">Commit你的改动</div></div></div></foreignObject><text x="425" y="444" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="12px" text-anchor="middle">Commit你的改动</text></switch></g><path d="M 425 560 L 425 583.63" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 425 588.88 L 421.5 581.88 L 425 583.63 L 428.5 581.88 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><path d="M 485 530 L 613.63 530" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 618.88 530 L 611.88 533.5 L 613.63 530 L 611.88 526.5 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 532px; margin-left: 538px;"><div data-drawio-colors="color: rgb(0, 0, 0); background-color: rgb(255, 255, 255); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 11px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; background-color: rgb(255, 255, 255); white-space: nowrap;">No</div></div></div></foreignObject><text x="538" y="536" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="11px" text-anchor="middle">No</text></switch></g><rect x="365" y="500" width="120" height="60" rx="9" ry="9" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 530px; margin-left: 366px;"><div data-drawio-colors="color: rgb(0, 0, 0); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">最终的目标完成了吗</div></div></div></foreignObject><text x="425" y="534" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="12px" text-anchor="middle">最终的目标完成了吗</text></switch></g><ellipse cx="425" cy="630" rx="40" ry="40" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 78px; height: 1px; padding-top: 630px; margin-left: 386px;"><div data-drawio-colors="color: rgb(0, 0, 0); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">完成</div></div></div></foreignObject><text x="425" y="634" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="12px" text-anchor="middle">完成</text></switch></g><path d="M 620 530 L 620 673.63" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 620 678.88 L 616.5 671.88 L 620 673.63 L 623.5 671.88 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><ellipse cx="620" cy="340" rx="3" ry="3" fill="rgb(0, 0, 0)" stroke="none" pointer-events="all"/><rect x="610" y="330" width="20" height="20" fill="none" stroke="none" pointer-events="all"/><path d="M 620 680 L 276.37 680" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 271.12 680 L 278.12 676.5 L 276.37 680 L 278.12 683.5 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><ellipse cx="620" cy="680" rx="3" ry="3" fill="rgb(0, 0, 0)" stroke="none" pointer-events="all"/><rect x="610" y="670" width="20" height="20" fill="none" stroke="none" pointer-events="all"/><ellipse cx="625" cy="440" rx="3" ry="3" fill="rgb(0, 0, 0)" stroke="none" pointer-events="all"/><rect x="615" y="430" width="20" height="20" fill="none" stroke="none" pointer-events="all"/><path d="M 620 340 L 620 530" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><ellipse cx="620" cy="530" rx="3" ry="3" fill="rgb(0, 0, 0)" stroke="none" pointer-events="all"/><rect x="610" y="520" width="20" height="20" fill="none" stroke="none" pointer-events="all"/><path d="M 10 680 L 10 136.37" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 10 131.12 L 13.5 138.12 L 10 136.37 L 6.5 138.12 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><ellipse cx="10" cy="680" rx="3" ry="3" fill="rgb(0, 0, 0)" stroke="none" pointer-events="all"/><rect x="0" y="670" width="20" height="20" fill="none" stroke="none" pointer-events="all"/><path d="M 10 130 L 143.63 130" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 148.88 130 L 141.88 133.5 L 143.63 130 L 141.88 126.5 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><ellipse cx="10" cy="130" rx="3" ry="3" fill="rgb(0, 0, 0)" stroke="none" pointer-events="all"/><rect x="0" y="120" width="20" height="20" fill="none" stroke="none" pointer-events="all"/></g><switch><g requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"/><a transform="translate(0,-5)" xlink:href="https://www.diagrams.net/doc/faq/svg-export-text-problems" target="_blank"><text text-anchor="middle" font-size="10px" x="50%" y="100%">Text is not SVG - cannot display</text></a></switch></svg>

Mikado Method需要创建一个依赖图：

第一步是确定一个重构的目标，例如，移除所有DailyProcessingStatus的引用（建议确定目标后，编写一个用户场景测试用来提供最外层的保护，每次前进后都要运行一遍。）

第二步我们需要设置一个倒计时10分钟，每当倒计时结束，我们就需要判断一下当前的状态。这是为了保证我们不会陷入各种复杂依赖、代码海洋中。

接下来我们从目标出发，看看完成这个目标需要完成哪些前置任务，将这些任务列出来。然后就可以大胆的修改代码尝试完成这些任务。

每当完成一个任务，或者倒计时结束，都要停下来判断一下，现在是否能够使测试通过。如果可以并且当前的修改是合理的，那么我们就可以提交代码，然后继续下一个任务。如果不能，我们要确定当前阻碍我们前进的东西是什么记录下来，并果断地回退当前地修改，进入下一个10分钟。

几轮倒计时后可能得到这样的东西
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="861px" viewBox="-0.5 -0.5 861 181" content="&lt;mxfile&gt;&lt;diagram id=&quot;tZe41FnKF79DnQRkhALj&quot; name=&quot;Page-1&quot;&gt;&lt;mxGraphModel dx=&quot;766&quot; dy=&quot;760&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;&lt;root&gt;&lt;mxCell id=&quot;0&quot;/&gt;&lt;mxCell id=&quot;1&quot; parent=&quot;0&quot;/&gt;&lt;mxCell id=&quot;97&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;90&quot; target=&quot;91&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;90&quot; value=&quot;移除所有来自Document download service的引用&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;1050&quot; y=&quot;80&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;98&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;91&quot; target=&quot;92&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;99&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;91&quot; target=&quot;93&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;91&quot; value=&quot;修复is_document_processed function and tests&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;1250&quot; y=&quot;80&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;100&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;92&quot; target=&quot;94&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;92&quot; value=&quot;提取document status作为类方法&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;1410&quot; y=&quot;10&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;93&quot; value=&quot;为document processing status创建enumeration&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;1410&quot; y=&quot;130&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;94&quot; value=&quot;修复新方法的测试报错&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;fillColor=#60a917;fontColor=#ffffff;strokeColor=#2D7600;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;1590&quot; y=&quot;10&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;96&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;95&quot; target=&quot;90&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;95&quot; value=&quot;移除所有引用DailyProcessingStatus的地方&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;fillColor=#0050ef;fontColor=#ffffff;strokeColor=#001DBC;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;850&quot; y=&quot;80&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;/root&gt;&lt;/mxGraphModel&gt;&lt;/diagram&gt;&lt;/mxfile&gt;" onclick="(function(svg){var src=window.event.target||window.event.srcElement;while (src!=null&amp;&amp;src.nodeName.toLowerCase()!='a'){src=src.parentNode;}if(src==null){if(svg.wnd!=null&amp;&amp;!svg.wnd.closed){svg.wnd.focus();}else{var r=function(evt){if(evt.data=='ready'&amp;&amp;evt.source==svg.wnd){svg.wnd.postMessage(decodeURIComponent(svg.getAttribute('content')),'*');window.removeEventListener('message',r);}};window.addEventListener('message',r);svg.wnd=window.open('https://viewer.diagrams.net/?client=1&amp;page=0&amp;edit=_blank');}}})(this);" style="cursor:pointer;max-width:100%;max-height:181px;"><defs/><g><path d="M 320 100 L 393.63 100" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 398.88 100 L 391.88 103.5 L 393.63 100 L 391.88 96.5 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><rect x="200" y="70" width="120" height="60" rx="9" ry="9" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 100px; margin-left: 201px;"><div data-drawio-colors="color: rgb(0, 0, 0); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">移除所有来自Document download service的引用</div></div></div></foreignObject><text x="260" y="104" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="12px" text-anchor="middle">移除所有来自Document download se...</text></switch></g><path d="M 520 73.75 L 554.17 58.8" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 558.98 56.7 L 553.97 62.71 L 554.17 58.8 L 551.16 56.3 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><path d="M 520 118.75 L 553.92 129.35" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 558.93 130.92 L 551.21 132.17 L 553.92 129.35 L 553.3 125.49 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><rect x="400" y="70" width="120" height="60" rx="9" ry="9" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 100px; margin-left: 401px;"><div data-drawio-colors="color: rgb(0, 0, 0); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">修复is_document_processed function and tests</div></div></div></foreignObject><text x="460" y="104" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="12px" text-anchor="middle">修复is_document_processe...</text></switch></g><path d="M 680 30 L 733.63 30" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 738.88 30 L 731.88 33.5 L 733.63 30 L 731.88 26.5 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><rect x="560" y="0" width="120" height="60" rx="9" ry="9" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 30px; margin-left: 561px;"><div data-drawio-colors="color: rgb(0, 0, 0); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">提取document status作为类方法</div></div></div></foreignObject><text x="620" y="34" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="12px" text-anchor="middle">提取document status作为类方法</text></switch></g><rect x="560" y="120" width="120" height="60" rx="9" ry="9" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 150px; margin-left: 561px;"><div data-drawio-colors="color: rgb(0, 0, 0); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">为document processing status创建enumeration</div></div></div></foreignObject><text x="620" y="154" fill="rgb(0, 0, 0)" font-family="Helvetica" font-size="12px" text-anchor="middle">为document processing...</text></switch></g><rect x="740" y="0" width="120" height="60" rx="9" ry="9" fill="#60a917" stroke="#2d7600" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 30px; margin-left: 741px;"><div data-drawio-colors="color: #ffffff; " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(255, 255, 255); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">修复新方法的测试报错</div></div></div></foreignObject><text x="800" y="34" fill="#ffffff" font-family="Helvetica" font-size="12px" text-anchor="middle">修复新方法的测试报错</text></switch></g><path d="M 120 100 L 193.63 100" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 198.88 100 L 191.88 103.5 L 193.63 100 L 191.88 96.5 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><rect x="0" y="70" width="120" height="60" rx="9" ry="9" fill="#0050ef" stroke="#001dbc" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 100px; margin-left: 1px;"><div data-drawio-colors="color: #ffffff; " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(255, 255, 255); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">移除所有引用DailyProcessingStatus的地方</div></div></div></foreignObject><text x="60" y="104" fill="#ffffff" font-family="Helvetica" font-size="12px" text-anchor="middle">移除所有引用DailyProcessingStatu...</text></switch></g></g><switch><g requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"/><a transform="translate(0,-5)" xlink:href="https://www.diagrams.net/doc/faq/svg-export-text-problems" target="_blank"><text text-anchor="middle" font-size="10px" x="50%" y="100%">Text is not SVG - cannot display</text></a></switch></svg>
完成一个任务后将其标记为绿色，然后返回到其父任务。不断重复这个过程，直到完成所有的任务。

<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="862px" viewBox="-0.5 -0.5 862 272" content="&lt;mxfile&gt;&lt;diagram id=&quot;tZe41FnKF79DnQRkhALj&quot; name=&quot;Page-1&quot;&gt;&lt;mxGraphModel dx=&quot;304&quot; dy=&quot;543&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;&lt;root&gt;&lt;mxCell id=&quot;0&quot;/&gt;&lt;mxCell id=&quot;1&quot; parent=&quot;0&quot;/&gt;&lt;mxCell id=&quot;97&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;90&quot; target=&quot;91&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;90&quot; value=&quot;移除所有来自Document download service的引用&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;fillColor=#60a917;fontColor=#ffffff;strokeColor=#2D7600;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;1050&quot; y=&quot;80&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;98&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;91&quot; target=&quot;92&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;99&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;91&quot; target=&quot;93&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;91&quot; value=&quot;修复is_document_processed function and tests&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;fillColor=#60a917;fontColor=#ffffff;strokeColor=#2D7600;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;1250&quot; y=&quot;80&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;100&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;92&quot; target=&quot;94&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;92&quot; value=&quot;提取document status作为类方法&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;fillColor=#60a917;fontColor=#ffffff;strokeColor=#2D7600;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;1410&quot; y=&quot;10&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;102&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;93&quot; target=&quot;101&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;93&quot; value=&quot;修改Modify DailyProcessingStatus方法去调用DocumentDownloader&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;fillColor=#60a917;fontColor=#ffffff;strokeColor=#2D7600;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;1410&quot; y=&quot;130&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;94&quot; value=&quot;修复新方法的测试报错&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;fillColor=#60a917;fontColor=#ffffff;strokeColor=#2D7600;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;1590&quot; y=&quot;10&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;96&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;95&quot; target=&quot;90&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;104&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;95&quot; target=&quot;103&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;95&quot; value=&quot;移除所有引用DailyProcessingStatus的地方&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;fillColor=#0050ef;fontColor=#ffffff;strokeColor=#001DBC;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;850&quot; y=&quot;150&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;101&quot; value=&quot;为document processing status创建enumeration&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;fillColor=#60a917;fontColor=#ffffff;strokeColor=#2D7600;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;1580&quot; y=&quot;130&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;106&quot; value=&quot;&quot; style=&quot;edgeStyle=none;html=1;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;103&quot; target=&quot;105&quot;&gt;&lt;mxGeometry relative=&quot;1&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;103&quot; value=&quot;移除所有来自Document classification service的引用&quot; style=&quot;rounded=1;whiteSpace=wrap;html=1;fillColor=#60a917;fontColor=#ffffff;strokeColor=#2D7600;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;1060&quot; y=&quot;210&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;mxCell id=&quot;105&quot; value=&quot;……&quot; style=&quot;whiteSpace=wrap;html=1;fillColor=#60a917;strokeColor=#2D7600;fontColor=#ffffff;rounded=1;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;&lt;mxGeometry x=&quot;1250&quot; y=&quot;220&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot;/&gt;&lt;/mxCell&gt;&lt;/root&gt;&lt;/mxGraphModel&gt;&lt;/diagram&gt;&lt;/mxfile&gt;" onclick="(function(svg){var src=window.event.target||window.event.srcElement;while (src!=null&amp;&amp;src.nodeName.toLowerCase()!='a'){src=src.parentNode;}if(src==null){if(svg.wnd!=null&amp;&amp;!svg.wnd.closed){svg.wnd.focus();}else{var r=function(evt){if(evt.data=='ready'&amp;&amp;evt.source==svg.wnd){svg.wnd.postMessage(decodeURIComponent(svg.getAttribute('content')),'*');window.removeEventListener('message',r);}};window.addEventListener('message',r);svg.wnd=window.open('https://viewer.diagrams.net/?client=1&amp;page=0&amp;edit=_blank');}}})(this);" style="cursor:pointer;max-width:100%;max-height:272px;"><defs/><g><path d="M 320 100 L 393.63 100" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 398.88 100 L 391.88 103.5 L 393.63 100 L 391.88 96.5 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><rect x="200" y="70" width="120" height="60" rx="9" ry="9" fill="#60a917" stroke="#2d7600" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 100px; margin-left: 201px;"><div data-drawio-colors="color: #ffffff; " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(255, 255, 255); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">移除所有来自Document download service的引用</div></div></div></foreignObject><text x="260" y="104" fill="#ffffff" font-family="Helvetica" font-size="12px" text-anchor="middle">移除所有来自Document download se...</text></switch></g><path d="M 520 73.75 L 554.17 58.8" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 558.98 56.7 L 553.97 62.71 L 554.17 58.8 L 551.16 56.3 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><path d="M 520 118.75 L 553.92 129.35" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 558.93 130.92 L 551.21 132.17 L 553.92 129.35 L 553.3 125.49 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><rect x="400" y="70" width="120" height="60" rx="9" ry="9" fill="#60a917" stroke="#2d7600" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 100px; margin-left: 401px;"><div data-drawio-colors="color: #ffffff; " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(255, 255, 255); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">修复is_document_processed function and tests</div></div></div></foreignObject><text x="460" y="104" fill="#ffffff" font-family="Helvetica" font-size="12px" text-anchor="middle">修复is_document_processe...</text></switch></g><path d="M 680 30 L 733.63 30" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 738.88 30 L 731.88 33.5 L 733.63 30 L 731.88 26.5 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><rect x="560" y="0" width="120" height="60" rx="9" ry="9" fill="#60a917" stroke="#2d7600" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 30px; margin-left: 561px;"><div data-drawio-colors="color: #ffffff; " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(255, 255, 255); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">提取document status作为类方法</div></div></div></foreignObject><text x="620" y="34" fill="#ffffff" font-family="Helvetica" font-size="12px" text-anchor="middle">提取document status作为类方法</text></switch></g><path d="M 680 150 L 723.63 150" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 728.88 150 L 721.88 153.5 L 723.63 150 L 721.88 146.5 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><rect x="560" y="120" width="120" height="60" rx="9" ry="9" fill="#60a917" stroke="#2d7600" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 150px; margin-left: 561px;"><div data-drawio-colors="color: #ffffff; " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(255, 255, 255); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">修改Modify DailyProcessingStatus方法去调用DocumentDownloader</div></div></div></foreignObject><text x="620" y="154" fill="#ffffff" font-family="Helvetica" font-size="12px" text-anchor="middle">修改Modify DailyProcessi...</text></switch></g><rect x="740" y="0" width="120" height="60" rx="9" ry="9" fill="#60a917" stroke="#2d7600" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 30px; margin-left: 741px;"><div data-drawio-colors="color: #ffffff; " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(255, 255, 255); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">修复新方法的测试报错</div></div></div></foreignObject><text x="800" y="34" fill="#ffffff" font-family="Helvetica" font-size="12px" text-anchor="middle">修复新方法的测试报错</text></switch></g><path d="M 120 149 L 193.99 123.1" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 198.94 121.37 L 193.49 126.99 L 193.99 123.1 L 191.18 120.38 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><path d="M 120 187.14 L 203.88 211.11" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 208.93 212.55 L 201.23 213.99 L 203.88 211.11 L 203.16 207.26 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><rect x="0" y="140" width="120" height="60" rx="9" ry="9" fill="#0050ef" stroke="#001dbc" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 170px; margin-left: 1px;"><div data-drawio-colors="color: #ffffff; " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(255, 255, 255); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">移除所有引用DailyProcessingStatus的地方</div></div></div></foreignObject><text x="60" y="174" fill="#ffffff" font-family="Helvetica" font-size="12px" text-anchor="middle">移除所有引用DailyProcessingStatu...</text></switch></g><rect x="730" y="120" width="120" height="60" rx="9" ry="9" fill="#60a917" stroke="#2d7600" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 150px; margin-left: 731px;"><div data-drawio-colors="color: #ffffff; " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(255, 255, 255); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">为document processing status创建enumeration</div></div></div></foreignObject><text x="790" y="154" fill="#ffffff" font-family="Helvetica" font-size="12px" text-anchor="middle">为document processing...</text></switch></g><path d="M 330 233.16 L 393.64 236.51" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 398.88 236.78 L 391.71 239.91 L 393.64 236.51 L 392.08 232.92 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"/><rect x="210" y="200" width="120" height="60" rx="9" ry="9" fill="#60a917" stroke="#2d7600" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 230px; margin-left: 211px;"><div data-drawio-colors="color: #ffffff; " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(255, 255, 255); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">移除所有来自Document classification service的引用</div></div></div></foreignObject><text x="270" y="234" fill="#ffffff" font-family="Helvetica" font-size="12px" text-anchor="middle">移除所有来自Document classificat...</text></switch></g><rect x="400" y="210" width="120" height="60" rx="9" ry="9" fill="#60a917" stroke="#2d7600" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 240px; margin-left: 401px;"><div data-drawio-colors="color: #ffffff; " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(255, 255, 255); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">……</div></div></div></foreignObject><text x="460" y="244" fill="#ffffff" font-family="Helvetica" font-size="12px" text-anchor="middle">……</text></switch></g></g><switch><g requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"/><a transform="translate(0,-5)" xlink:href="https://www.diagrams.net/doc/faq/svg-export-text-problems" target="_blank"><text text-anchor="middle" font-size="10px" x="50%" y="100%">Text is not SVG - cannot display</text></a></switch></svg>

### 重构工具

现代 IDE 通常都有很多便捷的重构工具，以 Rider 为例：
你可以使用 Extract Method，从代码中提取出一个方法
![alt text](image-6.png)
使用 Change Signature，修改方法的签名
![alt text](image-7.png)
更多的重构工具可以参考：<https://www.jetbrains.com/help/rider/Main_Set_of_Refactorings.html>

## 如何编写测试

### 关于测试的命名

测试的命名需要跟测试内容保持一种**一致性**，因此我建议使用 BDD 风格的命名，即**Given-When-Then 风格**。这种风格的命名能直观清楚的表达测试的信息，同时也能帮助你更好的组织测试的结构。 

```
Test_Given_JsonContentAndJsonReader_When_Read_Then_ReturnJsonValueObject
```

其中 Test*作为前缀表明这是一个测试函数，测试框架将会根据这个来识别。除此之外，整个命名由 Given-When-Then 三个部分组成，每个部分采用驼峰命名，并用*分隔。Given 部分描述测试的准备工作，When 部分描述测试的操作，Then 部分描述测试的预期结果。

显而易见的是，这并不是我们在日常编码中常用的命名风格，采用这个命名方式主要考虑到以下几点：

1. 测试名通常都很长，需要能够直观的看出测试的内容，因此使用分隔符会看上去更清晰。但如果所有地方都用分隔符，反而会显得过于冗长，也没法突出 Given-When-Then 的结构。
2. UE 中命名首字母大写

做一个对比

```C++
Test_Given_JsonContentAndJsonReader_When_Read_Then_ReturnJsonValueObject
TestGivenJsonContentAndJsonReaderWhenReadThenReturnJsonValueObject
Test_Given_Json_Content_And_Json_Reader_When_Read_Then_Return_Json_Value_Object
```

### 测试的结构

除开测试函数的命名外，仅关注函数体的结构，一个测试通常包含这样几个部分：

1. **测试准备**，设置测试的环境，包括创建对象，设置参数，创建测试数据等。

2. **测试操作**，调用被测试的函数。

3. **测试预期**，断言测试的结果是否符合预期。通常使用断言函数来判断。

4. **测试清理**，清理测试环境，包括删除对象，清理数据等。

给出一个参考案例：

```C++
void Test_Given_JsonContentAndJsonReader_When_Read_Then_ReturnJsonValueObject()
{
 // 测试准备
 string jsonStr = "{\"name\":\"John\",\"age\":30}";
 TempFile file(jsonStr);
 file.create();
 JsonReader reader;

 // 测试操作
 JsonValue value = reader.read(file.getContent());

 // 测试预期
 ASSERT_EQ(value["name"].asString(), "John");
 ASSERT_EQ(value["age"].asInt(), 30);

 // 测试清理
 file.clear();
}
```

**测试之间应当保持独立，因此在测试有 Side Effect 的函数或功能时需要保证最后完成清理工作。**

一些测试框架中每次测试方法都会创建一个新的测试类实例，避免影响后面的测试。但也有一些测试框架会复用测试类的同一个实例，这时候必须显式地重新设置所有支持测试的值和对象。

另外，在一些测试框架中，测试准备和测试清理会被单独提出来共用，叫做 Fixture（测试夹具）。
例如在 pytest 中，写法如下：

```python
@pytest.fixture
def db():
 conn = create_connection() // 准备数据库连接
 yield conn
 conn.close() // 测试结束后关闭数据库连接

def test_insert(db):
 db.execute("INSERT INTO table VALUES (1, 'test')") // 测试操作
 assert db.query("SELECT * FROM table") == [(1, 'test')] // 测试预期
```

**在编写测试时，建议可以采用这样的顺序**：

1. 写下测试名称，助于确定要达到的目标
2. 写下对目标代码的调用，这是功能的入口点
3. 写下预期和断言，这样我们就知道该功能应该产生的效果
4. 写下准备和清理代码，确定测试的上下文环境。

### 创建测试数据

在测试中，很多时候需要准备一些测试数据，类似于这样：

```C++
Order* order = new Order(
 new Customer("John",
  new Address("123 Street", "City", "Country"),
  new Email("123@xxx.com")
 ),
 new Product("Product1", 100)
);
```

但显然，这样的代码看上去并不清晰，我们后面会谈论测试的表现力，总之，我们需要更明确的方式来描述创建的测试数据。

我们可以通过函数命名来表达我们创建的对象，考虑使用对象母亲设计模式（一个类中包含多个工厂方法，每个工厂方法用于创建一个特定的对象）。

```C++
Order* order1 = ExampleOrders.newDeerstalkerAndCapeAndSwordstickOrder();
Order* order2 = ExampleOrders.newDeerstalkerAndBootsOrder();
```

但对于一组测试来说，很多情况是这样的，我们需要一组相似的数据，但他们只有其中的部分字段不同，使用对象母亲导致每一个新的变化都要提供一个新的工厂方法，最后充斥着各种重复代码。

这时候，我们可以考虑使用**构建者模式**

```C++
class OrderBuilder
{
public:
 OrderBuilder(){
  this.customer = new Customer('Default')
  this.product = new Product('Default', 0)
 }
 OrderBuilder withCustomer(CustomerBuilder customerBuilder)
 {
  this.customer = customerBuilder.build();
  return this;
 }

 OrderBuilder withProduct(ProductBuilder productBuilder)
 {
  this.product = productBuilder.build();
  return this;
 }

 Order* build()
 {
  return new Order(customer, product);
 }
protected:
 Customer* customer;
 Product* product;
}

Order* order = new OrderBuilder()
 .withCustomer(new CustomerBuilder().withName("John"))
 .withProduct(new ProductBuilder().withName("Product1").withPrice(100))
 .build();
```

建造者使得测试数据的创建变得非常灵活，对于相似的数据，我们还可以通过**共用建造者来进一步减少重复代码**。

```C++
OrderBuilder* JohnOrderBuilder = new OrderBuilder()
 .withCustomer(new CustomerBuilder().withName("John"))

Order* order1 = JohnOrderBuilder
 .withProduct(new ProductBuilder().withName("Product1").withPrice(100))
 .build();

Order* order2 = JohnOrderBuilder
 .withProduct(new ProductBuilder().withName("Product2").withPrice(200))
 .build();
```

### 使用模拟对象

这里给出一个简单的使用模拟对象来验证测试的例子，对于模拟对象更详细的讨论请参考后面的章节。
例如，当我们使用实际的数据库很困难时，可以使用模拟对象来模拟数据库的行为，对象的行为不需要很复杂，只要满足测试即可。

```C++
class FakeDatabase : public IDatabase
{
public:
 void save(const Order& order) override
 {
  orders[order.getId()] = order;
 }

 Order load(int orderId) override
 {
  return orders[orderId];
 }
protected:
 std::map<int, Order> orders;
};

Test_Given_Order_When_Save_Then_OrderIsSaved()
{
 FakeDatabase db;
 Order order；
 db.save(order);
 ASSERT_EQ(db.load(order.getId()), order);
}
```

## 深入理解测试

### 测试的分类

不得不说关于测试的分类，有很多不同的标准，我不是专业的 QA，所以只会在这里讨论几个比较重要的概念。

**用户场景测试**：用户场景测试通常是最外层的测试，它测试整个系统的行为，而不是特定的类或者 API。同时它也尽可能以用户的角度来看待系统，而不是深入系统的内部结构，这会使得它在面对系统变更时具有更强的健壮性。

用户场景测试的特点：

1. 外层测试（不关心系统内部实现）
2. 全流程（尽可能的包括系统涉及的所有流程）
3. 最先完成（通常我们面对一个系统时，首先要完成的就是用户场景测试，它可以提供一个最外层的防护网。）
4. 最少数量（用户场景测试的数量通常是最少的，因为它的粒度很大，测试速度也很慢，也很有可能更加松散）

例如，对于对话系统而言，用户场景测试可以是各种类型对话的全流程测试，从对话开始到对话结束（尽可能少的访问系统内部）。对于交互系统而言，则可以是面对各种交互物品，从交互开始到结束的全流程测试。

**单元测试**：单元测试是最小的测试单元，它通常是对一个类或者一个函数的测试。单元测试的目的是测试这个类或者函数的行为是否符合预期，同时它也是最快确定性最强的测试。如果测试类的职责清晰，依赖少，单元测试编写起来非常容易。

另外，一个好的建议是针对行为进行单元测试，而非针对方法。把注意力集中在被测试对象提供的功能时会做得更好。被测试对象需要与它的邻居协作，并调用多个方法。我们只需要知道如何利用该类来实现一个目标，而不是测试所有通过其代码的路径。

**集成测试**：集成测试是测试模块与模块间交互是否正常的测试。它也是单元测试与用户场景测试中间的一类测试（超出单个类的范畴，又不像用户场景测试那样涉及全流程）。

我们最终**编写测试，大概会遵循这样的流程**：

1. 首先为系统编写用户场景测试，始终提供最粗略的反馈。
2. 然后针对其中的一些类设置单元测试，检查更加细粒度的问题。
3. 最后针对其中类与类之间有潜在风险的交互点编写集成测试。

**测试分布**：按照 Google 的数据，单元测试占了测试的 80%，集成测试占了 15%，用户场景测试（端到端）占了 5%。
![Alt text](image-2.png)

### 测试的规模与松散性

现在，我想讨论另外一种来自于 Google 的测试分类法，这种分类法主要关注测试的规模。Google 将测试分为三种规模：小型测试、中型测试和大型测试。

**小型测试**必须在一个进程中运行，在许多语言中，甚至进一步限制，说它们必须在一个单线程上运行。小型测试绝大部分情况下执行高效、结果稳定准确。
**中型测试**可以跨越多个进程，使用线程，并可以对本地主机进行阻塞调用，包括网络调用。剩下的唯一限制是，中型测试不允许对 localhost 以外的任何系统进行网络调用。换句话说，**测试必须包含在一台机器内。**
**大型测试**取消了对中型测试的本地主机限制，允许测试和被测系统跨越多台机器。Google 主要为全系统端到端测试保留大型测试，这些测试更多的是验证配置，而不是代码片段，并且为无法使用测试替代的遗留组件的测试保留大型测试。

灵活性的提高伴随着风险的增加，**与在单一机器上运行相比，处理跨多台机器的系统以及连接这些机器的网络会显著的降低测试速度并增加不确定性的概率。**

因此，谷歌的团队经常将大型测试与小型或中型测试隔离开来，只在构建和发布过程中运行它们，以免影响开发人员的工作流程。

如果你有几千个测试，每个测试都有非常小的不确定性，整天运行，偶尔便会有一两个可能会失败（松散）。而测试通常与 CI 关联在一起，当测试失败时会阻碍提交，因此在一些团队中会给予一些容忍度，例如，如果一个测试失败，会重新运行它，如果第二次再次失败，那么才会被认为是真正的失败。亦或者是针对这种带有不确定性失败的测试统计失败率，定期针对失败率高的测试进行处理。

不过，测试的松散性总有一个限度，如果测试失误继续增长，你将经历比生产效率损失更糟糕的事情：对测试的信心丧失。**Google 的经验表明，当接近 1%的松散率时，测试开始失去价值。**

### 编写测试的原则

- 关于测试的原则，没有什么比这一点更重要了：**编写未臻完善的测试并经常运行，好过对完美测试的无尽等待。**

- 测试应该是一种风险驱动的行为，测试的目标是希望找出现在或未来可能出现的 bug。所以不必对那些仅仅读或写一个字段的访问函数编写测试，因为它们太简单了，不太可能出错。

- **当测试数量达到一定程度之后，继续增加测试带来的边际效用会递减**。测试也不是越多越好，编写太多测试会花费更多时间，也会使得维护成本增加，需要找到一个平衡点。

- 不要因为测试无法捕捉所有的 bug 就不写测试，因为测试的确可以捕捉到大多数 bug。

- 一个值得养成的好习惯是，每当你遇见一个 bug，先写一个测试来清楚地复现它。仅当测试通过时，才视为 bug 修完。

- 通过公开的接口来访问代码(像其他的客户程序一样)，而不是为测试开一扇包含额外可见性的后门。

- **测试失败时永远不许进行重构。**

- 测试应该仅包含测试行为所需的信息，保持测试的清晰和简单。作为一个推论，我们也强烈反对在测试中使用控制流语句，如条件和循环。

- 所有测试应力求封闭性：测试应包含设置、执行和拆除其环境所需的所有信息。测试应该尽可能少地假设外部环境，例如测试的运行顺序。

### 与测试一起工作

- 频繁地运行测试。对于你正在处理的代码，与其对应的测试至少每隔几分钟就要运行一次，每天至少运行一次所有的测试。（Google 程序员每天要运行几千个测试）

- 一旦通过，验收测试就代表已完成的特征，不应该再失败。失败意味着产生了一次回归，即我们破坏了已有的代码。如果需求改变，我们必须将受影响的验收测试从回归测试套件中移到在开发测试套件中，对它们进行修改，以反映新的需求，然后修改系统，让它们再次通过。

- 希望所有的开发者都有他们自己的环境，这样他们在运行测试时就不会相互干扰。

- 增量式改动时，代码不能编译的时间应该尽可能少，编译失败时我们无法 Commit。越多代码在修改，我们要在脑子中记住的东西就越多，通常会前进得越慢。**测试驱动开发的一项重要发现就是，开发步骤的粒度尽可能的小**。

### 异步测试

异步测试最简单的实现方式是直接调用Delay或者setTimeout这样的函数等待几秒。但如果在测试中被广泛使用的话，测试时间会变得显著的缓慢。
![Alt text](image-14.png)
更好的解决方案是以接近微秒的频率主动轮询状态转换。同时你可以把它和一个超时值结合起来，以防测试无法达到稳定状态。
![Alt text](image-15.png)

### 从 0 开始的 TDD

如果你要在一个新项目或者新系统中开始基于 TDD 来进行开发，首先需要完成第 0 次迭代，即**搭建一个可行走的骨架**。从一系列需要完成的功能中，选择能反应应用基础结构的部分。例如，是否需要 GUI，是否需要多线程等等。把这些内容尽可能的包含到这个第 0 次迭代中去。这个阶段工作量并不小，需要解决很多问题，包括：

- 项目环境
- 可能需要的支持多线程，异步，GUI 的测试框架
- 用于支持第 0 次迭代的 Fake 实现
- CI
- CD

从一开始就考虑这些东西能够让后续的开发更加顺利，但另一方面，也不希望在这里停留太久，毕竟没有什么比让项目启航更重要了。比起一步到位，更好的方式是增量式的迭代，所以这里也有个微妙的平衡。

### 自动化测试的难点

![Alt text](image-5.png)

1. 视觉和音频元素
2. 探索性测试
3. 评估游戏体验

### 如何测试渲染

游戏中的视觉元素一直是一个测试的难点，过去大多数有视觉测试的项目都是通过一种 **golden test** 的方式来进行的，即**将当前的结果与远程库中的结果（Ground Truth）进行比较**。

需要由 TA 或者程序创建一个新的关卡，在关卡中放置网格体或材质并启动相关功能，然后放置一个虚拟摄像机，对其进行截屏。测试时，将当前的屏幕截图与 Ground Truth 进行比较，如果测试失败，你能看到当前图像，Ground Truth 图像以及 Diff 图像。但这种方式有很多问题：

1. **容易被破坏**，测试依赖于十几个不同的系统，例如，引入微妙的压缩、量化或调整某些默认参数，都会导致测试失败。而且通常都是大规模的测试失败。因此，你不得不花费大量的时间用于浏览这些测试失败的图像。为了避免阻塞 CI，你可能会选择宽恕这些失败，即直接将当前内容更新到 Ground Truth 中，显然，这很危险。
2. **无法检查到真正的问题**，正如我们前面所说，测试应当能表达清楚错误的原因。而视觉测试无法做到这一点。如果测试失败，你只能看到两个图像之间的差异，但无法知道为什么会有这种差异。这些测试也没法发现一些真正的问题，例如，光照的某些部分没有被归一化，gloss 为 1 时 NaN 在 BRDF 中出现，等等。
3. 测试代码通常会比编写它的工程师待得更久，因此，好的测试应该能够让后续的工程师了解系统相关的各种假设，但视觉测试无法做到这一点。

为了解决上诉问题，推荐使用一种**基于分析数值的渲染测试方法**。举一个例子，你编写了一个新的次表面皮肤散射着色器，你决定编写一些测试来验证它（这种测试方法通常从侧面对系统功能进行验证，个人感觉不太适合 TDD）。你可能会关心这些部分：

1. 皮肤着色器真的 diffuse 了入射光吗？
2. 是否保持了能量守恒，无论什么配置都没有添加能量？
3. shader 是否响应了特定的配置？
4. 是否产生非法值，NaN 或者无穷大？
5. 增加了多少性能成本？

于是，你尝试添加一些代码，对于每个行为，编写代码创建**合成输入**，然后执行功能，并检查输出的预期。如果测试失败，你也可以打印更多的信息，例如，多个值的差异直方图等。对于性能，你可以编写 benchmark，根据不同的输入，启用不同的功能，然后检查时间消耗。

关于合成输入，我想说的更具体一些。例如，你可以创建一个缓冲区（甚至合成 GBuffer），在 CPU 上填充它，除了中间某个像素是 1，其他都是 0。然后你判断经过高斯，或者其他滤波器后，这个像素以及它周围的像素的数值是否是满足算法的预期的。

![Alt text](image-11.png)
![Alt text](image-12.png)

在GPU 上测试，如果你因为驱动程序/测试设备更改而遭受大量噪音，可以考虑使用适用于 DirectX 的 [WARP](https://learn.microsoft.com/en-us/windows/win32/direct3darticles/directx-warp) 设备，或适用于 OpenGL 的 [SwiftShader](https://github.com/google/swiftshader)。

以及，golden test 也并非一无是处，其所固有的敏感性可以用来捕获一些意想不到的例如编译器、工具链、驱动程序更改的问题。建议可以使用少量的（不超过 15 个）golden test 作为冒烟测试。

另外圣莫尼卡的分享中还提到了使用 Nvidia Flip 作为 golden visual test 的工具。
![Alt text](image-10.png)

### 测试外部 API

即使我们拥有源代码也倾向于不修改第三方代码。每次有新版本时都打上私有的补丁，这通常都会带来很多麻烦。

如果我们不能修改某个 API，那么就不能对单元测试中得到的设计反馈做出任何反应。不论单元测试对外部 API 的不当之处提出什么警告,我们都必须接受它。

这意味着在单元测试调用第三方代码的对象时,为第三方代码提供模拟实现的用处不大。我们发现模拟外部库的测试通常比较复杂,才能够使要检查的功能处于就绪状态。

我们可以编写一层适配对象，使用第三方的 API 来实现这些接口，我们让这一层尽可能薄，以减少可能的脆弱性和难以测试的代码。通过专门的集成测试来测试这些适配对象,确保我们理解了第三方 API 工作的方式。
![Alt text](image-1-1.png)

## 测试的 Bad Smell

### 非弹性测试

我们还希望每个测试只在与之相关的代码被破坏时才会失败，即**弹性测试**。否则，我们最后就会得到一堆容易被破坏的测试，降低开发速度，阻碍重构。测试易破坏的原因包括:

1. 测试太紧密地与系统无关的部分或被测对象的无关行为耦合在一起。
2. 测试过度指定了目标代码的预期行为，对它进行了不必要的限制。
3. 多个测试之间执行了产品代码中相同的行为，存在重复。
4. 测试的易破坏也和系统的设计有关。如果某个对象有许多依赖关系,或者有隐含的依赖关系，从而难以解除与环境的耦合，那么当系统的这些遥远的部分发生改变时，对象的测试就会失败。因此，利用测试的易破坏性也可以作为设计品质的一种有价值的反馈信息。

测试的可读性和弹性之间存在着一种直接的关系。如果测试是专注的、准备工作清楚的、重复性最小的，那么就容易为它命名，它的目的也更清晰。**清晰的目的常常使得测试更具有弹性。**

### 不可读测试

要让 TDD 能够持续下去，测试要做的不只是验证代码的行为，它们还必须清楚地表达这种行为，即它们必须可读。测试的可读性和代码的可读性一样重要，每次开发者停下来想测试的用意时，他们就不能用这些时间去编写新的功能，团队的开发速度就下降了。事实上，以测试代码作为文档也是 TDD 早期就许下的承诺。

不可读的测试通常具有以下特点：

1. 测试名称不能清楚地描述其测试的要点，也不能说明与其他测试的区别。
2. 单个测试用例执行了多项功能。
3. 测试不具有典型的测试结构，导致读者无法快速理解测试的意图。
4. 测试带有大量的准备代码和异常处理代码，掩盖了测试的重心。
5. 测试使用了具体的值("神奇数字")，但没有明确这些值有什么重要意义。

**补充**：关于**一个测试方法应该有多少断言**，有些 TDD 的实践者建议，每个测试应该只包含一个预期或断言。这在学习 TDD 时是一条有用的训练规则，可以避免对开发者想到的所有事情进行断言，但我们发现在实践中不行。更好的规则是每个测试只考虑一项一致的功能，这项功能也许由几个断言来表示。但不管怎么样，**表现力是关键：作为这个测试的读者，我能立马弄清楚什么是重要的。**

### 错误信息不清晰的测试

测试的要点不是通过，而是失败。我们希望产品代码通过它的测试，但也同样希望测试能检查出存在的一些错误。一个"失败"的测试实际上是成功地完成了它的设计目的。

**但有一种情况是希望避免的，即我们不能诊断已经发生的测试失败，使得我们最后不得不依赖调试来找到代码的错误。**

发生这种情况至少表明测试没有将需求表述得足够清楚。更糟糕的是，我们有可能发现自己堕入了"调试地狱"，Deadline 近在眼前，但不知道需要多久才能让这个测试通过。此时，删除这个测试的诱惑很大——但这将使我们丧失安全网。

改进诊断最容易的方法就是让每个测试保持小而专注，提高测试名称的可读性。如果测试很小，它的名称就应该能够指引我们找到出问题的地方。

除此之外，你应该在断言时给出更多的信息。

```C++
// Bad
Customer customer = order.getCustomer();
assertEquals("573242", customer.getAccountId());
```

```C++
// Good
Customer customer = order.getCustomer();
assertEquals("573242", customer.getAccountId(), "Customer account ID does not match");
```

## 模拟对象(Test Doubles)

在测试环境中有时候使用真实的对象会很困难，例如，依赖于外部服务、一些不稳定的行为或是某个运行非常缓慢的功能。这时候我们可以使用模拟对象来代替真实对象。

我们之前还谈论了为什么应该限制返回值，而使用模拟对象可以支持验证对象间的交互行为。这样既可以避免暴露对象内部状态，也能完成测试。

使用测试替代有三种主要技术：

**Fake**是一个API的轻量级实现，其行为类似于真实实现，但不适合生产。

当你需要使用测试替代时，使用伪造通常是理想的技术，但是对于你需要在测试中使用的对象，伪造可能不存在，编写伪造可能是一项挑战，因为你需要确保它在现在和将来具有与真实实现类似的行为。

``` C++
class FakeDatabase : public IDatabase
{
public:
 void save(const Order& order) override
 {
  orders[order.getId()] = order;
 }

 Order load(int orderId) override
 {
  return orders[orderId];
 }
protected:
 std::map<int, Order> orders;
};
```

**Stubbing** 是一种指定函数行为的技术，你可以为函数指定返回值

``` C++
AccessManager* accessManager = new AccessManager(mockAuthenticator);

// 当mockAuthenticator.authenticate(USER_ID)时，返回False
WHEN(mockAuthenticator.authenticate(USER_ID)).thenReturn(false);

assertFalse(accessManager->hasAccess(USER_ID));

WHEN(mockAuthenticator.authenticate(USER_ID)).thenReturn(true);
assertTrue(accessManager->hasAccess(USER_ID));
```

**Interaction Testing**（有时候也被称之为Mocking，但有时候Mocking被用于描述整个模拟框架包括Stubbing） 是一种验证对象间交互的技术，你可以验证对象是否调用了某个函数，或者调用了多少次，参数是否错误等等。

``` C++
AccessManager* accessManager = new AccessManager(mockAuthenticator);

accessManager->hasAccess(USER_ID);

VERIFY(mockAuthenticator).authenticate(USER_ID);
//如果accessManager->hasAccess没有使得mockAuthenticator使用参数USER_ID调用authenticate，这个测试就会失败
```

**Stubbing和Interaction Testing通常是通过Mocking框架完成的。**

Stubbing与Interaction Testing编写起来比Faking要容易得多。它们使得针对独立的代码段编写高度集中的测试变得非常简单，而不必担心如何构建代码的依赖关系。但随着时间移动，被Mocking的对象API需要发生改变时，散落在代码库中的各个Stubbing和Interaction Testing将会带来很大的阻碍。因此，在Google，许多工程师开始转向编写更真实的测试。

## 为什么要使用 TDD?

- **自信地修改代码而不会破坏它**(恐惧让我们停滞不前)：更改代码，即使是很小的更改，也常常需要对整个系统进行重新测试。对于大多数程序来说，这意味着需要大量的人力去重新执行所有的测试，以保证没有出错。
- 一套测试就是一个强大的 **bug 侦测器**，能够大大缩减查找 bug 所需的时间。一个 bug 在代码库中存在的时间越久，修复它的成本就越大（一方面源于开发者对代码感到陌生，另一方面定位到真正的修复者的难度也会增加）。按照 LOL 的自动化测试数据，自动化中发现的错误的解决速度是普通错误的八倍。<https://technology.riotgames.com/news/automated-testing-league-legends>

- **改善编码**，编写测试代码其实就是在问自己：为了添加这个功能，我需要实现些什么？编写测试代码还能把注意力集中于接口而非实现。
- 最好的团队会想办法将其成员的集体智慧转化为整个团队的利益。这正是自动化测试的作用。在团队中的一个工程师写了一个测试后，它被添加到其他人可用的**公共资源池**中。团队中的每个人现在都可以运行这个测试。这些测试构成了众多的有效的开发环境，除了程序，策划、美术都可以利用这些开发环境验证自己的工作。
- 测试代码同时也是最好的**Use Case 文档**。
- 自动化测试与 CI 可以构建一套**代码库的监测系统**，结合资产校验，性能测试，能够随时了解代码库的健康状态。这也使得很多问题能够直接通过二分法定位到具体的提交。
- 分散项目 Deadline 的压力，**减少 Crunch**（一种在每个交付周期结束时的加班）的发生。

### 案例支撑

- 美国国家标准与技术研究所 2002 年的报告“软件测试基础性的缺乏对经济的冲击”发现，软件缺陷对美国经济每年造成近 600 亿美元的损失。
- 在《通过测试驱动开发实现质量改进：四个工业化团队的成果和经验》中，Nachiappan Nagappan、E. Michael Maximilien、Thirumalesh Bhat、Laurie Williams 表示，他们研究的所有采用 TDD 的团队的缺陷率都有显著降低：“IBM 团队有 40%，微软团队有 60%～ 90%。”他们得出结论，TDD 可以“在对开发团队的生产效率没有明显影响的前提下显著降低软件的缺陷率”。
- 在微软，Thirumalesh Bhat 和 Nachiappan Nagappan 研究了 TDD 实践对 Windows 和 MSN 部门的 影响，发表了论文《测试驱动效果评估：工业案例研究》。他们发现，在同一组织内，采用 TDD 的项目比不采用 TDD 的相似项目，其代码质量有显著的提高。
- 在《在真实环境中探索极限 编程：工业级案例研究》中，Lucas Layman、Laurie Williams、Lynn Cunningham 发现，比较同样一个产品的两次发布，有“50%的效率提升，65%的发布前质量提升，35%的发布后质量提升”。 一次发布是在团队采用极限编程方法论之前，一次发布是在采用极限编程两年之后。
- 量化软件管理协会暨卡特财团研究了福莱特公司的极限编程实践，发现“时间周期比行业规范戏剧性地减少了五个月，质量提高了两倍（缺陷降低了一半）”。不仅如此，福莱特公司还节省了一百三十万美元。
- 《对开源项目进行面向对象质量度量，以量化评估测试驱动开发》中，Ron Hilton 研究了大量使用和未使用 TDD 的开源项目的代码质量。他发现使用测试驱动的项目，内聚性指标高出 21.33%，耦合度指标高出 10.05%， 复杂度降低了 30.98%。

## 软件工程之禅

### 左移法则

在开发人员工作流程的早期发现问题通常可以降低成本，这是一个普遍的真理。考虑一下开发人员工作流程的时间轴，该时间轴从左到右依次为：从构思和设计开始，经过开发、测试、提交、集成和最终的生产部署。在此时间轴上将问题检测提前到 "左侧"，解决问题的成本显著的降低。
![Alt text](image-8.png)

安全问题不能推迟到开发过程的最后阶段，必须要求“在安全上向左转移”。如果你能够在最初的开发之前发现安全问题，将缺陷提交到版本控制就被发现，修复的成本更低。根据安全约束规范进行开发，要比提交代码后再让其他人分类标识并修复它更简单。

### 持续法则

从持续重构、持续测试、持续集成到持续部署都在向我们揭示着一个软件开发的重要准则，比起完成一个大的完整的任务，更好的方式是持续不断地小步前进。

## 其他

### 基于主干开发

基于分支开发(Branch-Based Development)：每个开发者都有自己的分支，或拥有某些功能的特性分支。基于分支开发，本质上是通过拆分分支的方式来保证主干的稳定性。但随着这些分支存在的时间越久，合并的难度会随之指数级上升。

基于主干开发(Trunk-Based Development)：每个团队成员每天至少向主线集成一次。在 Google ，几乎没有团队使用版本库分支。所有的更改都提交到了版本库的 head ，每个人都可以立即看到。

基于主干开发最大的问题在于主干缺乏稳定性，而基于测试的开发文化正好解决了这一点。不过，有时候当我们面临某些尚未完成又无法拆小的功能时，需要使用一种叫做**特性开关（feature toggle）的技术**。例如，当我们不得不重写一个系统时，我们可以使用 feature toggle 来切换旧系统和新系统。在本地时打开开关，编写新系统代码，提交时关闭开关，这样主干也不会受到影响。当系统编写完成，并且测试通过后，再次打开开关，提交新系统代码。最后，确定新系统功能没有问题后，再用一次提交来删除旧系统代码和 feature toggle。

### 技术债务

Ward Cunningham 提出了“技术债务”一词，用来表达如果开发者在代码中欠缺考虑最后会发生什么。没有什么比“技术债务”更能拖慢开发进程影响预估的了。本来一个小时就能完成的事情，需要花费一天甚至更多时间来完成，有时候添加一个功能需要修改大量代码。增进代码质量可以降低修改带来的成本，并且让预估更加准确。

对于那些会不断累积的技术债务，尽快偿清几乎总是正确的选择。如果任由技术债务在系统中堆积，而开发者又在系统中工作，那么绝对会发生冲突。开发者会碰到那些技术债务并且一次次付出代价。他们没法倒车，所以必须调整他们的行为（驾驶习惯），绕弯路到达目的地，为的是不使用倒车。一个问题会导致更多的问题。越早处理技术债务，花费的成本就越低，就像信用卡欠款一样。

将生产环境上的系统直接抛弃彻底重写，几乎从来都不是个好主意。重写一个应用常常累计同样的技术债务，如果重写得不够彻底，很可能也会犯之前一样的错误。重构则是一个安全的逐步清理代码的系统性方式，同时也可以保持系统持续运作。

### 结对编程（Pair Programming）

去年 OpenAI 的宫斗大戏落幕之时，王者归来的 Greg Brockman 在 Twitter 云淡风轻地写下了这段话。
![alt text](img_v3_0288_538b418a-7442-4c73-9f8c-4a56f69a12bg.jpg)

结对编程简单来说就是两个人一起在一台电脑上工作，其中一人扮演“驾驶员”角色，负责代码编写，另一人扮演“领航员”角色，负责思考和指导，进行实时的 Code Review。并且，驾驶员和领航员应该尽可能频繁地交换角色——多于五分钟但是不能多过三十分钟。

对于领航者来说，最重要的是提出“为什么你这么做而不那样做”，然后听驾驶员解释。这样不仅可以促进相互学习，还可以帮助驾驶员保持思维的清晰。

另外，还有一种技巧，被称之为**乒乓结对**，一人编写测试，另一人负责使测试通过。

结对编程不仅可以提高代码质量，减少代码错误，同时可以帮助知识在团队中迅速传播。对于复合型的团队来说，所有成员都熟悉整个代码库很重要。结对编程可以防止团队成员过于专门化，并且帮助团队达成共识。而且即便是两个人在一起工作，仍然有很多实践证明，结对编程提高了整体的效率。

一些团队会让新成员和资深开发者（或者至少是有经验的开发者）一起结对一两周。但这更像是“见习”而非结对编程，资深开发者更多地扮演了导师的角色，而真正理想的结对编程关系则是基于平等的合作关系。

### Google 如何引入测试

- **定向班**: 尽管谷歌早期的工程人员大多回避测试，但 Google 自动化测试的工程师们知道，按照公司的发展速度，新加入的工程师会很快超过现有的团队成员。如果他们能接触到公司所有的新员工，这可能是一个引入文化变革的极其有效的途径。幸运的是，所有新的工程人员都要经历一个节点：入职培训。

- **测试认证**: 为了给项目提供一个明确的前进道路，测试小组设计了一个认证计划，他们称之为测试认证。测试认证的目的是让团队了解他们的测试过程的成熟度，更关键的是，提供关于如何改进测试的说明书。

- **厕所测试**: 2006 年 4 月，一篇涵盖如何改进 Python 测试的短文出现在整个谷歌的洗手间里。这第一集是由一小群志愿者发布的。一些人认为这是对个人空间的侵犯，他们强烈反对。邮件列表中的抱怨声此起彼伏，但 TotT 的创造者们却很满意：抱怨的人仍在谈论测试。

- **Google 为什么不强制编写测试**：测试小组曾考虑要求高级领导提供测试授权，但很快决定拒绝。任何关于如何开发代码的要求都将严重违背谷歌文化，并且可能会减缓进度，这与被授权的想法无关。人们相信成功的想法会传播开来，因此重点是人如何展示成功。

### Sanitizers

Sanitizers 是一组用于检测内存错误的工具，包括 AddressSanitizer、MemorySanitizer、ThreadSanitizer、LeakSanitizer、UndefinedBehaviorSanitizer。这些工具可以帮助开发者在编译时检测到内存错误、数据竞争、内存泄漏等问题。

圣莫妮卡工作室在测试中使用了 Sanitizers 捕获潜在的 crashes
![Alt text](image-13.png)

## 总结，最重要的几件事

虽然我认为前面谈论的这些都很重要，但不得不说实践下面几点会更立竿见影：

1. 编写测试来代替临时开发环境和 Debug。
2. 把测试放到编码之前，并用测试来指导你的设计。
3. 拆分实体，用小而多的函数和类表达你的设计。
4. 拒绝过度抽象。

## 推荐

- [Google 测试 Blog](https://testing.googleblog.com/)
- [测试相关集合](https://trello.com/b/nGE5yqZk/game-automated-testing-resource-hub)
- [ShaderTestFramework](https://github.com/KStocky/ShaderTestFramework)
- [GDC AUTOMATED TESTING ROUNDTABLES](https://autotestingroundtable.com/)

## 引用

- Understanding the Four Rules of Simple Design
- 测试驱动的面向对象软件开发
- 重构：改善既有代码的设计
- [Google 软件工程](https://qiangmzsx.github.io/Software-Engineering-at-Google/#/)
- 修改代码的艺术
- Object Calisthenics
- [TDD 反馈循环](https://blog.thecodewhisperer.com/permalink/putting-an-age-old-battle-to-rest)
- [持续重构](https://www.codit.eu/blog/continuous-refactoring/?country_sel=be)
- [天皇法则](https://www.vinta.com.br/blog/the-mikado-method)
- [How (not) to test graphics algorithms](https://bartwronski.com/2019/08/14/how-not-to-test-graphics-algorithms/)
- [Automated Testing Shader Code](https://learn.microsoft.com/en-us/shows/pure-virtual-cpp-2024/automated-testing-of-shader-code)
- [盗贼之海自动化测试1](https://www.youtube.com/watch?v=X673tOi8pU8)
- [盗贼之海自动化测试2](https://www.youtube.com/watch?v=KmaGxprTUfI)
- [圣莫妮卡自动化测试](https://sms.playstation.com/media/documents/GOWR_Ben_Hines_AutomatedTesting_GDC23.pdf)
- [Sanitizers](https://github.com/google/sanitizers)
- [WARP](https://learn.microsoft.com/en-us/windows/win32/direct3darticles/directx-warp)
- [SwiftShader](https://github.com/google/swiftshader)
- [Rider重构工具](<https://www.jetbrains.com/help/rider/Main_Set_of_Refactorings.html>)
