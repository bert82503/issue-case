

《唯品会Java开发手册》－与阿里手册的比较文学
==================
> 江南白衣，2018.07.25


## 1. 前言
很早很早以前，只有Sun写于1999年的[《Java编码规范》](http://www.oracle.com/technetwork/java/javase/documentation/codeconvtoc-136057.html)，
里面多是命名和格式的规定。后来万能的[Google也出了一份](https://segmentfault.com/a/1190000002761014)，但内容仍以这两者为主。
直到最近的[《阿里巴巴Java开发手册》](https://github.com/alibaba/p3c)，
作为首个对外公布的企业级Java开发手册，**里面整理的大量`最佳实践`，对整个业界都有大的意义。**

受益于阿里手册开放的Apache 2.0 License，我们**在其基础上结合唯品会的内部经验定制了一个《唯品会Java开发手册》，
并随[VJTools](https://github.com/vipshop/vjtools)开源。**

**[前人栽树](https://baike.baidu.com/item/前人栽树)**，讲解一次规范本来是件很苦很累的事情，
但有了阿里手册大量详尽的周边介绍文章，我们就可以偷个懒，只说说自己定制的部分好了。修改主要有三：
1. [《Clean Code》](https://book.douban.com/subject/4199741/)、[《Effective Java》](https://book.douban.com/subject/3360807/)，
   佳篇难弃，一并补充。
2. [《SEI CERT Oracle Coding Standard for Java》](https://www.securecoding.cert.org/confluence/display/java/SEI+CERT+Oracle+Coding+Standard+for+Java)
   和 Sonar 的[零零碎碎的规则](https://rules.sonarsource.com/java/)。
3. 规则尽量精简，期望**所有规则都真正影响x代码质量和可读性，让开发人员看到修改建议项都觉得是服气的**，
   而不觉得是学院派在唠唠叨叨，浪费大家时间。


## 2. 规范落地的代码检查工具
**一切规范，都得自动化的代码检查工具保证其落地。**

一切工具，都不能自动化检查规范里的所有规则。

### 小历史
1. 最早是简单的CheckStyle，后来有同样基于文本分析的PMD
2. 再后来又有了基于字节码分析的FindBugs
3. 再后来出现了Sonar，集成了三者
4. 又后来，Facebook基于FindBugs推出自己**实战派的规则[fb-contrib](http://fb-contrib.sourceforge.net/)**，再次被Sonar集成
5. 最后，Sonar发展了自己的语法分析体系，改进并替换了 CheckStyle 和 PMD 的规则

### 选择
阿里基于PMD写了自己的规则集和IDE插件P3C，而我们则仍然选择Sonar。

一来，Sonar既有IDE Plugin，又有服务侧的DashBoard给所有的项目相关者共享，还可与其他系统打通**质量数据**。

二来，**Sonar + FindBugs + FB Contrib 体系已包含了大量的现成规则**。

三来，PMD的规则极度难读难改，只有阿里同学能玩得转。对于我们来说，Sonar用Java编写的规则亲切得多。


## 3. 第一章 《命名规约》
### 3.1 拼音命名
阿里手册禁止了拼音与英文的混合：
> 【强制】代码中的命名严禁使用拼音与英文混合的方式，更不允许直接使用中文的方式。

而我们考虑到一些中国式业务词汇，可能缺乏通用好懂的英文对应，用词霸硬翻出生僻的英文，反而给阅读者制造了困难，所以拼音还是不能禁止。

既然允许了拼音，那混合似乎也不能禁止，比如例子里的getPingFenByName()，如果PingFen找不到英文对应，那总比GenJuMingZiHuoQuPingFen好。

又**新增了拼音缩写的禁止，对新来的阅读者这个是真的很难猜**，所以规则重写成：
> 禁止拼音缩写，避免阅读者费劲猜测；尽量不用拼音, 除非中国式业务词汇没有通用易懂的英文对应。

### 3.2 命名的其“模糊度”
来自《Clean Code》的精华浓缩：

1. 如果上下文很清晰，局部变量可以使用 `list` 这种简略命名，否则应该使用 `userList` 这种更清晰的命名。
2. 禁止 `a1, a2, a3` 这种带编号的没诚意的命名方式。
3. 方法的参数名叫 `bookList`，方法里的局部变量名叫 `theBookList` 也是很没诚意。
4. 如果一个应用里同时存在 `Account、AccountInfo、AccountData` 类，
   或者一个类里同时有 `getAccountInfo()、getAccountData()`, `save()、 store()` 的函数，阅读者将非常困惑。
5. `callerId` 与 `calleeId`， `myDearFriendsWithA` 与 `myDearFriendsWithB` 这种拼写极度接近，考验阅读者眼力的。

### 3.3 避免Java中并不严谨地允许的变量重名
比如父类有个foo的成员变量，而Java允许子类再定义一个foo的成员变量。
同理，有了bar的成员变量，但它的函数参数，或者局部变量，仍然可以叫bar，能把人看晕了。

好在，Sonar有规则兜了底。

### 3.4 常量命名
Sonar很聪明，即使是static final的变量，如果不是基础类型就不当成是常量，也不会提醒你大写它，比如`static final Logger logger`。

### 3.5 其他
比如一些不属于命名规范的规则移除了，看[定制记录](https://vipshop.github.io/vjtools/#/standard/ali)，不一一啰嗦。


## 4. 第二章《格式规约》
### 4.1 项目组统一的代码格式模板
所有规范中，关于空格，括号等等的描述是最无趣难读难记的，所以强烈建议用项目组统一的代码格式模板来代替这些描述。

而到底行宽是80,100还是120字符，缩进是Tab或二空格 ,四空格不重要，大家统一就好。

**接手项目时，先统一格式化一次**，避免后续改动时，代码格式的变化淹没了真正逻辑的变化。

### 4.2 一个类就是一篇文章
**这是《Clean Code》的一个重要思想，你要按一篇文章来布局你的代码，想象一个阅读者的存在。**

方法要排列，而不是总把新增方法放在最后面。

1. 顺序依次是：构造函数 >分组的 (公有方法/保护方法/私有方法) > getter/setter方法。
2. 当一个类有多个构造方法，或者多个同名的重载方法，这些方法应该放置在一起。其中参数较多的方法在后面。
3. 作为调用者的方法，尽量放在被调用的方法前面。

同理，**一段代码也是一段文章，需要合理的分段**，而不是一口气读到尾。
**不同组的变量之间，不同业务逻辑的代码行之间，有时候插入一个空行，起逻辑分段的作用**，等于一句无言的注释。

### 4.3 其他
我们没有理由假设读者能记住整个Java运算符优先级表。用小括号来明确优先级，除非作者和Reviewer都认为去掉小括号也不会使代码被误解，甚至更易于阅读。

对于一些特殊场景（如使用大量的字符串拼接成一段文字，或者想把大量的枚举值排成一列），
为了避免IDE自动格式化，可以使用@formatter: off和@formatter: on来包装这段代码。

其他看[定制记录](https://vipshop.github.io/vjtools/#/standard/ali)，不一一啰嗦。


## 5. 注释规约
### 5.1 注释覆盖率，与更清晰的代码来减少注释；删除无意义的注释，空标签
有些规范过度强调了所有方法都必须有注释，而我们特别说明了如果代码命名与结构足够清晰，的确可以不写注释。

大爱《Clean Code》，也提倡**删掉下面这些无意义的注释和空标签，让代码更整洁。**
```java
/**
 * put elephant into fridge.
 *
 * @param elephant
 * @param fridge
 * @return
 */
public void put(Elephant elephant, Fridge fridge);
```

### 5.2 JavaDoc中不要为了HTML格式化而大量使用HTML标签和转义字符
在那些老式规范中，提倡大家必须使用\<\p\>， \<\pre\>这样的html标签，以及\&lt ，\&quot这样的转义字符来格式化注释，它们只对生成HTML版的JavaDoc有用，
而在我们直接阅读代码时造成了很大的困扰。

**鉴于实际场景里我们很少阅读HTML的JavaDoc，多数是直接阅读代码**，所以我们提倡不使用它们，并禁止IDE对JavaDoc的格式化。

### 5.3 避免创建人，创建日期，及更新日志的注释
阿里手册里：【强制】所有的类都必须添加创建者和创建日期。

但代码后续还会有多人多次维护，创建人也可能会离职，所以让我们相信**源码版本控制系统对更新记录的管理，能做得比这更好吧。**


## 小结
看似不累，码完三章还是很累，后面的章节下回再续了。

规范还是初版，欢迎大家去GitHub留言，一起改进。

全文见 https://vipshop.github.io/vjtools/#/


[原文](http://calvin1978.blogcn.com/?p=1771)

