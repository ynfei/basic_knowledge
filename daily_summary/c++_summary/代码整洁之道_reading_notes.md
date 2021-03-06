# 代码整洁之道

## 第二章 有意义的命名

### 2.1 名副其实

### 2.2 避免误导

### 2.3 做有意义的区分

### 2.4 使用读得出来的名称

### 2.5 使用可搜索的名称

### 2.6 避免使用编码

- 2.6.1 避免使用匈牙利语标记法
- 2.6.2 避免使用前缀来表明成员变量
- 2.6.3 接口和实现，采用编码的特殊情形

### 2.7 避免思维映射

### 2.8 类名

- 类名和对象名应该是名词或名词短语

### 2.9 方法名

- 方法名应当是动词或动词短语

### 2.10 别扮可爱

- 避免使用幽默用语。言到意到，意到言到

### 2.11 每个概念对应一个词

- 每个抽象概念选一个词，并且一以贯之

### 2.12 别用双关语

- 避免将同一单词用语不同目的，同一术语用语不同概念

### 2.13 使用解决方案领域名称

### 2.14 使用源自所涉问题领域的名称

- 与所涉问题领域更为贴近的代码，应当采用源自问题领域的名称

### 2.15 添加有意义的语境

- 用良好命名的类，函数或命名空间来放置名称，给读者提供语境

### 2.16 不要添加没用的语境

- 只要短名称足够清楚，就要比长名称好。别给名称添加不必要的语境

+++

## 第三章 函数

### 3.1 短小

- 函数的第一规则是要短小，第二规则还是要短小。
- 代码块和缩进。if语句，else语句，while语句等，其中的代码块应该只有一行，这行大抵应该是一个函数调用语句。函数不应该大到足以容纳嵌套结构，函数的缩进层级不该多一层或两层。

 

### 3.2 只做一件事

- 函数应该做一件事。做好这件事，只做一件事。如果函数只是做了该函数名下同一抽象层上的步骤，则函数还是只做了一件事。还有一个方法你是，判断该函数是否能拆出一个函数，该函数不仅只是单纯地重新诠释其实现。
- 只做一件事的函数无法被合理地切分为多个区段。

### 3.3 每个函数一个抽象层级

- 要确保函数只做一件事，函数中的语句都要在同一抽象层面上。

### 3.4 switch语句

- 确保每个switch都埋藏在较低的抽象层级，而且永远不重复。利用多态实现。

### 3.5 使用描述性的名称

- 长而具有描述性的名称，要比短而令人费解的名称要好。长而具有描述性的名称，要比描述性的长注释要好。

### 3.6 函数参数

- 理想参数是零参数，其实是单参数，再次是二参数，尽量避免使用三参数。尽量避免通过参数输出信息，而通过返回值从函数中输出。
- 一元函数的普遍形式。询问关于参数的问题，操作该参数或将其转换为其他什么东西在输出之。还有就是**事件**有出入参数而无输出参数。
- 标识参数。避免想函数传入布尔值。
- 二元函数。尽量利用一些机制将其转换为一元函数。
- 三元函数。写三元函数前一定要想清楚。
- 参数对象。如果函数看起来需要两个，三个以上的参数，就说明一些参数应该封装为类了。

### 3.7 参数列表

### 3.8 动词与关键字

- 对于一元函数，函数和参数应该形成良好的动词/名词对形式。例如write(name)

### 3.8 无副作用

- 一个函数中不要做一些额外的函数名之外的事情
- 避免使用输出参数

### 3.9 分隔指令与询问

- 函数要么做什么事情，要么回答什么事，但二者不可得兼。

### 3.10 使用异常替代返回错误码

- 错误处理就是一件事
- 错误代码依赖磁铁

### 3.11 别重复自己

- 减少代码重复

### 3.12 结构化编程

### 3.13 如何写出这样的函数

- 先写粗糙代码，然后按照规则进行推敲改进

+++

## 第四章 注释

- 注释的恰当用法是弥补我们在用代码表达意图时遭遇的失败。尽管有时也需要注释，我们也该多花心思尽量减少注释量。

### 4.1 注释不能美化糟糕的代码

- 与其花时间编写解释你糟糕的代码的注释，不如花时间清洁那糟糕的代码。

### 4.2 用代码来描述

### 4.3 好注释

- 唯一真正好的注释是你想办法不去写的注释。
- 法律信息
- 提供信息的注释
- 对意图或方法的解释
- 阐释。某个标准库的用法或者参数需要注意的地方。但是要注意阐释性注释本身就不正确的风险。
- 警示
- TODO
- 放大

### 4.4 坏注释

- 大多数注释都属此类。通常，坏注释都是糟糕的代码的支撑或借口，或者对错误决策的修正，基本上等于程序员自说自话。
- 喃喃自语。如果你决定写注释，就要花必要的时间确保写出最好的注释。
- 多余的注释。
- 误导性注释。
- 循规式注释。每个函数或每个变量都要有注释的规矩全然是愚蠢可笑的。
- 日志式注释。
- 废话注释。用整理代码的决心替代创造废话的冲动。
- 能用函数或变量时就别用注释
- 位置标记。尽量少用慎用。
- 括号后面的注释。如果发现自己想标记右括号，其实应该做的是缩短函数。
- 归属与署名。
- 注释掉的代码。直接把代码注释掉是讨厌的做法，禁止这样做。
- 非本地信息。假如你一定要写注释，请确保它描述了离它最近的代码。
- 信息过多。
- 不明显的联系。注释及其描述的代码之间的联系应该显而易见。

+++

## 第五章 格式

### 5.1 格式的目的

- 保持良好代码风格。

### 5.2 垂直格式

- 短文件通常比长文件更容易理解。
- 向报纸学习。
- 概念间垂直方向上的间隔。每个空白行都是一条线索，标识出新的独立概念。
- 垂直方向上的靠近。紧密相关的代码应该互相靠近。
- 除非有很好的理由，否则就不要把关系密切的概念放到不同的文件中。
- 垂直顺序。一般而言我们想自上而下展示函数调用依赖顺序。被调用的函数应该放在执行调用的函数下面。

### 5.3 横向格式

- 尽量保持代码行短小。
- 水平方向上的分隔与靠近。
- 缩进。
- 空范围也要保持缩进。while,if。
- 团队规则。

## 第六章 对象和数据结构

### 6.1 数据抽象

- 隐藏实现并非只是在变量之间放上一个函数层那么简单，隐藏实现关乎抽象。

### 6.2 数据、对象的反对称性

- 对象把数据隐藏于抽象之后，暴露操作数据的函数。数据结构暴露其数据，没有提供有意义的函数。
- 使用数据结构的代码便于在不改动既有数据结构的前提下添加新函数。面向对象代码便于在不改动既有函数的前提下添加新类。

### 6.3 Demeter定律

- 模块不应了解它所操作对象的内部情形。
- 火车失事。连串的调用通常被认为是肮脏的风格。
- 混杂。这种混淆有时会不幸导致混合结构，一半是对象，一半是数据结构，既增加了添加新函数的难度，也增加了添加新数据结构的难度，两面不讨好。



+++

## 第十章 类

### 10.1 类的组织

- 封装。放松封装总是下策。

### 10.2 类应该短小

- 类的第一条规则是类应该短小，第二条规则是还要更短小。类的名称应当描述其权责。实际上，命名正是帮助判断类的长度的第一个手段。如果给无法为某个类名以精确的名称，这个类大概就太长了。类名越含混，该类越有可能用过过多的权责。
- 单一权责原则。单一权责原则认为，类或者模块应该有且只有一条加以修改的理由。类只应有一个权责--只有一条修改的理由。**系统应该由许多短小的类而不是少量巨大的类组成。每个小磊封装一个权责，只有一个修改原因，并与少数其他类一起协同达成期望的系统行为。**
- 内聚。类应该只有少量实体变量。类中的每个方法都应该操作一个或者多个这种变量。通常而言，方法操作的变量越多，内聚性越高。
- 保持内聚性就会得到许多短小的类。

### 10.3 为了修改而组织

+++

## 第十一章

### 11.1 如何建造一个城市

### 11.2 将系统的构造与使用分开

- 分解main()。将构造与使用分开的方法之一是极爱那个全部构造过程搬迁到main或被成为main的模块中。
- 工厂。
- 依赖注入。
- 扩容。

+++

## 第十七章 味道与启发

### 17.1 注释

- 不恰当的信息
- 废弃的注释
- 冗余注释
- 糟糕的注释
- 注释掉的代码

### 17.2 环境

- 需要多步才能实现的构建
- 需要多步才能做到的测试

### 17.3 函数

- 过多的参数
- 输出参数
- 标识参数
- 死函数

### 17.4 一般性问题

- 一个源文件中存在多种语言
- 明显的行为为被实现
- 不正确的边界问题
- 忽视安全
- 重复
- 在错误的抽象层级上的代码
- 基类依赖于派生类
- 信息过多
- 死代码
- 垂直分隔
- 前后不一致
- 混淆视听
- 人为耦合
- 特性依恋
- 选择算子参数
- 晦涩的意图
- 位置错误的权责
- 不恰当的静态方法
- 使用解释性变量
- 函数名称应该表达其行为
- 理解算法
- 把逻辑依赖改为物理依赖
- 用多台替代if/else,或switch/case
- 遵循标准约定
- 用命名常量替代魔术数
- 准确
- 结构基于约定
- 封装条件
- 避免否定性条件
- 函数只该做一件事
- 掩蔽时序耦合
- 别随意
- 封装边界条件
- 函数应该只在一个抽象层级上
- 在较高层级放置可配置数据
- 避免传递浏览

### 17.5 名称

- 采用描述性名称
- 名称应与抽象层级相符
- 尽可能使用标准命名法
- 无歧义的名称
- 为较大作用范围选用较长名称
- 避免编码
- 名称应该说明副作用

### 17.6 测试

- 测试不足
- 使用覆盖率工具
- 别略过小测试
- 被忽略的测试就是对不确定事物的疑问
- 测试边界条件
- 全面测试相近的缺陷
- 测试失败的模式有启发性
- 测试覆盖率的模式有启发性
- 测试应该快速