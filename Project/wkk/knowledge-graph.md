# 知识图谱设计

<!-- MarkdownTOC -->

- wkk 中的设计
- 知识图谱
- 知识图谱：大数据语义链接的基石 - 李涓子
    - 概念三角形
    - 本体形式化
    - 本体的描述方法
- Knowledge-Based Application Design Patterns - Google
- 论文

<!-- /MarkdownTOC -->


## wkk 中的设计

从 wiki 抓取。

根节点为空

下属各个学科，领域，然后不同的学科领域再往下细分，像树一样

在前几层(也就是比较广义的学科时)树是不允许交叉的

而进入比较细分的领域(也就是各种学科的技术开始交叉，有交叉学科的层级)不同领域的内容可以开始交叉，这时候表示的数据结构也要开始变化

像生活大爆炸里的多维象棋一样，整个知识图谱是一个多维的表示，不同层级有不同层级的基本逻辑。比方说在比较抽象的层级，这里的学科不能交叉，也有一定的差异，比方说语文数学英语之类的

在下面相对具体的层级，比方说机器学习这一层，其实可以应用到更多不同的细分领域，就可以开始互相交叉了。

也就是说越具体，数据结构越复杂

在推荐的时候，就可以根据这样的结构，进行同层和跨层的检索

而用户的笔记，也就是在完善整个知识图谱，这个完善的过程，一部分自动，一部分也可以是手动(因为更加准确)，然后相当于训练边的强度，用户可以看到自己的知识架构的茁壮程度，技能点技能书的选取

也可以基于此给出具体的可操作的建议。

而新的内容也可以自动分类到不同的层级领域类别，进行统一的管理

当然，学术界和工业界都还没有做好的东西，我不会涉及，只在一个小的子集进行尝试

RDF

## 知识图谱

知识图谱是一种对人类知识的进行存储和表示的半结构化数据集。一般知识图谱的信息中心是命名实体，也叫 entity，object，对象，等等。然后常见的信息有：

+ 对象的分类信息(ontology)，例如：知乎 是一个 网站，
+ 对象的属性(attribute)，例如：知乎 的女神数有 xxx 万，
+ 对象之间的关系(relationship)， 例如：知乎 CEO 是 黄继新，
+ 对象的文本描述(description)，例如：知乎“是一个好网站，但是有些人居然在上面约来约去，这样是不合适的。尤其是只约别人不约我的话。”

根据现有学术界的进展，靠社区人工编辑形成的知识图谱（Wikipedia， Freebase），在质量，覆盖率，精准度上都远超任意一个公开的全自动生成的知识图谱（NELL，OpenIE，DeepDive）。所以我会主要依靠中文 Wiki 和百科，然后辅助定向抓取的自动生成的结果。

这部分是最花时间，也是最关键的工作。数据的清洗，不同信息源结果的合并，以及自动抓取和模板分析都有很多 dirty work 在里面。

一个命名实体想要可用，光有这个词是不够的，你还需要有：

+ 分类信息（ontology）
+ 描述（description）
+ 一些属性（attribute）
+ 和他的主要的相关对象之间的关系（relationship）

目前最大的公开知识图谱 Freebase 的达到这个要求的命名实体数是 500 万，还是英文。

有了这个知识图谱后，剩下的部分都是可以靠在已有的技术上做一些工程开发和优化可以解决的了。

**我还需要一个中文分词系统**

现在经过中国科学家们的努力，中文分词已经做的非常非常好了。对于绝大多数query，分词不再是效果的瓶颈。

所以我会直接使用现有的工具，例如中科院的 ICTCLAS，哈工大的 LTP。

他们对于我的需求基本足够。

为了更好的效果，我会外挂一些行业词库，然后自己再把分词切碎的长短语通过 entity 词表粘贴起来。这可以解决绝大多数问题。

**然后，我要对这个中文知识图谱建立索引**

建立索引的目的是为了对搜索词条（query）快速响应。有两种方法可以选择：

1，Graph Datebase，或者直接 Database。例如 Neo4j，或者直接 MySQL。这样的话就是把知识图谱当作严格的结构化信息，然后当作图，或者简单的 columns 存到 Graph DB/DB 里。

这样的好处是支持严格的结构化 query 查找，例如：

    find 苹果 ->首席执行官
    ------- 结果 ---------
    库克

但是这样要求 query 必须严格匹配，其实非常受限。这个问题叫做 semantic search，已经被学术界研究了很多年，但是并没有成熟应用。

而且，由于一般 graph db 支持的 query 结构过于复杂，导致速度很难上的去。而复杂的结构化 query 和 Magi 的需求并不太合适，所以我会选择第二种。

2，搜索引擎的反向索引，例如 Lucene，Indri。这样就是把每个 entity 的信息作为一个文档，所有的信息都待着 xml tag 放进去，然后搜索的时候按照关键词来匹配，拿到结果后再重新结构化，再和 query 做匹配。

为了支持对名字，描述，关系，类别，属性的匹配，我把它们各自封装到对应的 xml tag，或者 field 里，然后每来一个 query，我会对这几个 field 都做关键词匹配的检索。

**有了这些数据，我就可以对 query 进行在知识图谱中的搜索了**

1，先要明确的是，我不知道如何在超过一阶关系的长 query 上在保证 recall 和覆盖率的情况下做到可用级别的效果。例如：

    query：美国的总统的老婆
    转为 structured query： 美国 ->总统 ->配偶
    返回：米歇尔奥巴马

这个问题我没有解决方案，学术界没有，Google 没有，百度没有

长 query 理解的难点：

+ 从文本 query->structure query 的转化非常难，一点语言的变化就会带来新的挑战。
+ 长 query 往往对应着高阶关系，这里面的 noise 是乘数关系，每一部一点小小的失误累加起来就会使得结果完全不靠谱

而搜索是一个对精确度要求特别高的应用，如何在保证召回说得过去的情况下（例如 1%的网页 query），达到足够高的 precision？

这是学术界和工业界都没能解决的问题

2，但是，我可以做到对直接的命名实体的查询，以及一部分命名实体 + 关系的查询。

前者是：

    query：苹果公司
    返回：[苹果公司] 是 一家消费电子公司

后者是：

    query：苹果公司首席执行官
    structured：苹果公司 ->首席执行官
    返回：库克

2.a，首先解决单个命名实体的查询

这个是可行的，也是学术界工业界已经做到了的。

这个问题可以定义为 entity linking 问题，既，给定一个文本，如何找出其中的 entity，然后 match 到知识图谱中的对象，目前学术界最新的结果大概是 50-60%的 F 值

大概做法是收集足够的别名，给定一个 query 后做一次精准匹配，然后把匹配到的名称的对象拿出来，然后用上下文进行消歧。

这个任务主要有三个问题：

+ 消除歧义：苹果是水果，还是电脑？
+ 别名，（Alias，surface form）的处理：既如何做到“春哥”->李宇春的识别
+ 覆盖率：知识图谱够不够大，够不够全。

我个人感觉这应该是最先解决的问题。

surface form 和覆盖率已然要靠足够大和全的知识图谱。而目前已知的做法离自动构建超过人工编辑的知识图谱距离很远。

2.b， 然后是基于命名实体的结果之后的一阶关系查询

这个通过关键词来 hit 到知识图谱中的关系即可，对于明显的类似两个词的 query 是做得到的。

先做了命名实体匹配，然后用剩下的部分做一次对关系的几种名称的 exact match 之类的方法。离能够解决自然语言多种 variance，然后给出和关键词匹配一样级别的模糊查找，相距甚远。当然这是整个学术界工业界都尚未解决的问题。

---

## 知识图谱：大数据语义链接的基石 - 李涓子

**知识图谱基础**

+ DBpedia
    + 250 概念, 4M 实例, 6000 属性, 500M 三元组, 在线更新
+ yago
    + 350K 概念, 10M 实例, 100 属性, 120M 三元组
+ XLORE
    + 850K 概念, 8M 实例, 70K 属性
+ Freebase
    + 15K 概念, 40M 实例, 4000 属性, 1B 三元组, Google KB 核心
+ Google KG
    + 15K 概念, 600M 实例, 20B 三元组
+ BabelNet
    + 50M 义项, 50+ 种语言, 262M 三元组
+ WordNet
    + 7 种欧洲语言, 跨语言链接

**知识图谱类型**

+ 领域无关知识图谱
    + 人工构建
        + ResearchCyc, www.cyc.com/platform/researchcyc
        + WordNet, wordnet.princeton.edu
    + 基于维基百科
        + DBPedia, dbpedia.org
        + YAGO, yago-knowledge.org
        + Freebase, freebase.com
        + WikiTaxonomy
        + BabelNet, babelnet.org
    + 开放知识抽取
        + KnowItAll, openie.cs.washington.edu
        + NELL, rtw.ml.cmu.edu
        + Probase, research.microsoft.com/en-us/projects/probase/
    + 中文知识图谱
        + 百度知心
        + 搜狗知立方
+ 特定领域知识图谱
    + FOAF, Geonames, Linked Movie Database, etc
+ 跨语言知识图谱
    + DBPedia, Yago, Freebase, XLORE, etc

**万维网信息描述语言塔**

![kg1](./_resources/kg1.jpg)

### 概念三角形

![kg2](./_resources/kg2.jpg)

哲学的本体定义: Ontology is the philosohpical study of the nature of **being, becoming, existence** or **reality**, as well as the **basic categories** of being and their relations.

计算机领域本体定义: An **ontology** is a formal, explicit specification of a shared conceptualization - Gruber 1993

+ **Conceptualization**: **an abstract model** of phenomena in the world by having identified the relevant concepts of those phenomena.
+ **Explicit**: the type of concepts used, and the constraints on their use are explicitly defined.
+ **Formal**: the fact that the ontology should be machine readable.
+ **Shared**: ontology should capture consensual knowledge accepted by the communities.

### 本体形式化

五元组表示 O = {C, R, F, A, I}

+ **C** - concepts
    + 概念集合，通常以 Taxonomy 形式组织
    + 球星，球迷
+ **R** - relations
    + 描述概念或者实例之间语义关系的集合
    + subClassOf, birthplace
+ **F** - functions
    + 一组特殊的关系，关系中第 n 个元素的值由其他 n-1 个元素的值确定
    + Price-of-a-used-car 由 the car-model, manufacturing data 和 kilometers 确定
+ **A** - axioms
    + 公理
    + 如果 A 是 B 的子女，B 是 C 的子女，则 A 是 C 的子孙
+ **I** - instances
    + 描述具体的**个体**
    + 如：Peter 是概念**学生**的实例

### 本体的描述方法

+ 资源描述框架 RDF
    + Resource Description Framework
+ RDF 数据模式
    + **资源 Resource**
        + 使用 **URI** 唯一标示一个资源
        + 一个资源通常标示一个事物(Thing)
    + **属性 Property**
        + 一种特殊类型的资源，用以描述资源与资源间的关系
    + **语句 Statement**
        + 由 3 种资源组成的三元组(Triple)
        + 主语 rdf:subject，谓语 rdf:predicate 以及宾语 rdf:object

**本体的简化形式**

O = {C, I, T, P}

+ **C** - concepts
    + 描述领域或任务中的**抽象概念**，通常以 Taxonomy 形式组织
    + 如描述世界知识的本体中，**学生**和**老师**是两个概念
+ **I** - instances
    + 描述具体的**实例**
    + **学生Peter**是概念学生的实例
+ **T** - ISA
    + 概念与概念之间、实例与概念之间的关系
    + `subClassOf关系`和`instanceOf关系`
+ **P** - properties
    + 本体中用于描述实例信息的**其他语义关系**
    + 如: **instance-attribute-value**(AVP)

**总结**

+ 知识图谱实现对客观世界从字符串描述到结构化语义描述，是对客观世界的知识映射(mapping world knowledge)
+ 本体可以作为知识图谱表示的概念模型和逻辑基础
+ 知识图谱可以描述不同层次和粒度的概念抽象
+ 知识图谱可以作为互联网资源组织的基础

---

## Knowledge-Based Application Design Patterns - Google

Freebase, Google Refine, Schema.org, Knowledge Graph

**Freebase Knowledge Graph**

+ www.freebase.com
+ Open, crowd-sourced, knowledge graph
+ 23M+ topics, 2k+ commons types
+ RESTful APIs
+ Creative Commons Attribution License
+ Data dumps
+ A source for Google's Knowledge Graph

+ Discovery Patterns
    + **Turning Strings into things**
+ Visualization Patterns
    + Trees, Maps, Timelines, Graphs, Bubble Charts
    + [Thinkbase](http://thinkbase.cs.auckland.ac.nz)


## 论文

+ http://cpfd.cnki.com.cn/Article/CPFDTOTAL-LNSL201008001095.htm
+ http://www.cqvip.com/qk/93202x/200803/27479992.html
+ [In Papers]Building, Maintaining, and Using Knowledge Baese: A Report from the Trenches
+ [In papers]国内知识图谱应用研究综述
