# Java--ooclass--BUAA
ooclass homework Unit 1

## OO第一单元总结

### 一、概况

本单元三次作业的整体目的是对一个含有括号的表达式（包括三角函数）进行**去括号**操作，并在可能的情况下对性能优化（**表达式化简**）。在构建整个项目的过程中，主要是学习到了正则表达式，面向对象构建类的思路，如何产生优秀的代码风格，类继承和父类引用表示对象的简便性，递归下降的思维方式等等。

由于第一周不仅事情多而且还坐大牢的缘故，在第一次作业中我采用了*预解析模式读入*，甚至在这个模式下我也“四处碰壁，无路可通”，印证了“<u>万事开头难</u>”这一俗语。第二周的我奋发图强，采用*一般读入模式直接推倒重构*，这一周的我在开动时可以说是“家徒四壁，无路可走”，这谁想得到开头难的后头是“<u>难上加难</u>”呢。第三周就很仁慈了，只用改一些细枝末节的函数，本以为“<u>难于上青天</u>”的难，却因为第二次作业的“万丈高楼平地起”而变得“手可摘星辰”。这也是我这次作业中体会最深的一个点，一个好的架构带给后续开发的可扩展是多么亲切啊。

#### 复杂度分析各参数含义

##### 以每个方法为单位进行分析：

**圈复杂度（Cyclomatic Complexity）**是衡量**计算机程序复杂程度**的一种措施。它根据程序从开始到结束的**线性独立路径**的数量计算得来的。（可以理解为每遇到一个分支，圈复杂度就增加）

**ev(G)** 基本复杂度是用来衡量**程序非结构化程度**的，非结构成分降低了程序的质量，增加了代码的维护难度，使程序难于理解。因此，基本复杂度高意味着非结构化程度高，难以模块化和维护。实际上，消除了一个错误有时会引起其他的错误。（大致类似于代码风格吧）

**v(G)** 是用来衡量一个模块**判定结构的复杂程度**，数量上表现为独立路径的条数，即合理的预防错误所需测试的最少路径条数，圈复杂度大说明程序代码可能质量低且难于测试和维护，经验表明，程序的可能错误和高的圈复杂度有着很大关系。

**iv(G)** 模块设计复杂度是用来衡量模块判定结构，即**模块和其他模块的调用关系**。软件模块设计复杂度高意味模块耦合度高，这将导致模块难于隔离、维护和复用。模块设计复杂度是从模块流程图中移去那些不包含调用子模块的判定和循环结构后得出的圈复杂度，因此模块设计复杂度不能大于圈复杂度，通常是远小于圈复杂度。（高内聚，低耦合的设计理念）

##### 以每个类为单位进行分析：

- **OCavg**代表类的方法的平均循环复杂度。
- **OCmax**代表类的方法的最高循环复杂度。
- **WMC**代表类的总循环复杂度。

原文链接：[IDEA圈复杂度插件（MetricsReload）下载与使用](https://blog.csdn.net/Dkangel/article/details/106279052)

### 二、第一次作业

#### 1、题目要求及思路分析

第一次作业大致要求为：对一个只含有一个自变量x的多项式进行去括号和化简，此时输入多项式括号层数最多为1层。表达式由项相互加减构成，项由因子的乘积构成。因子有三类：**常量因子**、**变量因子**、**表达式因子**。

由于第一次作业采用了预解析模式，所以对表达式、项、因子的逻辑关系以及如何分别解析他们都没有深入研究。我也是在第一次课上实验中找到了研究的方法，理清了一些思路。并把这个思路带到了我的第一次作业实现中。

#### 2、基于度量的程序结构分析

##### 2.1 UML类图分析（仅显示主要属性和方法）



![1648200962725](C:\Users\L\AppData\Roaming\Typora\typora-user-images\1648200962725.png)

在第一次作业中，我用了*HashMap<BigInteger,BigInteger>*数据结构来存储每一步的操作数。第一个BigInteger表示指数，第二个BigInteger表示系数。可以看到主要是对预解析模式下的六个操作分别建类，他们又继承于Operator这个父类，拥有**初始化方法和计算方法**。值得一提的是在Pos和Neg类中，操作数只有一个，故在初始化这两个类时，将right变量初始化为null。Operation类储存了每一步操作的结果，这是在初始化方法中就完成的。调用了Operator类的*calculate*方法，可以直接得到该操作的最终结果并储存在result变量中。每一个Operation对象又都有一个*printResult*方法，可以打印出结果，这样只用打印出最后一步所产生的结果就能得到输出。Lexer类用于解析每个步骤的字符串，并储存所有的操作operations。

这个架构在现在的我看来逻辑也算比较清晰了，但缺点也是有的。其一是可以考虑将Operator类设置为**接口**，因为我从来没生成Operator父类对象，都是生成的子类对象，而且两个方法也可以不用具体写出来。其二是我的数据结构，因为没注意到第一次作业要求的指数最大不超过8，所以也是用的BigInteger保存指数，增加了不必要的开销，而且自己对BigInteger使用的不熟练也导致了后面被hack到bug。最后一点就是Lexer类写的有点复杂了，直接用一个Lexer对象对所有的操作进行处理会显得很臃肿，这里面我把所有操作存了下来，因为以后的每一步都有可能调用前面步骤的结果。然后再加上对六类操作的解析，使得我的这个方法的行数处于危险边缘，多次合并才达到了代码风格的要求。

##### 2.2 方法复杂度分析

| Method                              | CogC   | ev(G) | iv(G) | v(G)   |
| ----------------------------------- | ------ | ----- | ----- | ------ |
| src.Add.calculate()                 | 3      | 1     | 2     | 2      |
| src.Lexer.split(List<String>, int)  | **18** | 1     | 6     | **11** |
| src.Main.main(String[])             | 7      | 1     | 9     | 9      |
| src.Mul.calculate()                 | 4      | 1     | 2     | 2      |
| src.Neg.calculate()                 | 0      | 1     | 1     | 1      |
| src.Operation.Operation(Operator)   | 3      | 1     | 2     | 2      |
| src.Operation.getResult()           | 0      | 1     | 1     | 1      |
| src.Operation.printResult()         | **14** | 1     | 4     | 7      |
| src.Pow.calculate()                 | 6      | 4     | 5     | 5      |
| src.StringExchange.exchange(String) | 4      | 1     | 5     | 5      |
| src.Sub.calculate()                 | 3      | 1     | 2     | 2      |
| ...                                 |        |       |       |        |
| **Total**                           | 62     | 25    | 50    | 58     |
| **Average**                         | 2.82   | 1.14  | 2.27  | 2.64   |

在表中可以看到针对字符串的split方法和最后打印化简后的表达式printResult方法复杂度较高。split复杂度较高的原因是一方面一些操作只需要一个操作数，这表明不能直接用split**一一对应**的取到对应操作符和操作数。但现在看来可以使用try（catch）方法进行判断（本质上这也就是一个if-else语句）。另一方面要识别出fi（1<=i<=n-1)，然后将fi的结果作为新操作数进行计算。printResult方法主要复杂在化简，用到了很多条件语句，比如系数为1且有x时省略系数，指数为2时将x\*\*2化简为x\*x等等。实际上在化简的步骤中我也有相应的错误，找到了别人的bug，发现自己也有……我的化简部分基本在printResult中，这也是我第一次作业中方法复杂度最大的部分，的确复杂了就很容易错呀。

##### 2.3 类复杂度分析

| Class              | OCavg    | OCmax | WMC  |
| ------------------ | -------- | ----- | ---- |
| src.Add            | 1.5      | 2     | 3    |
| src.Lexer          | **12**   | 12    | 12   |
| src.Main           | **4**    | 4     | 4    |
| src.Mul            | 1.5      | 2     | 3    |
| src.Neg            | 1        | 1     | 2    |
| src.Operation      | **3.33** | 7     | 10   |
| src.Operator       | 1        | 1     | 4    |
| src.Pos            | 1        | 1     | 2    |
| src.Pow            | 3        | 5     | 6    |
| src.StringExchange | **4**    | 4     | 4    |
| src.Sub            | 1.5      | 2     | 3    |

可以看到Lexer的类复杂度最高，大部分Operator子类复杂度最低。各操作子类比较好的体现了低耦合的特性，但用于解析的Lexer的作用其实不仅包括解析了，还包括新建Operator类和Operation类，多功能杂糅使得其变得十分繁琐。

#### 3、公测互测bug分析

在强测中被找到一个bug，在互测中也被找到了bug，但通过一次提交全部修改。bug产生的原因是BigInteger类的方法用得不好。之前写代码的时候是以long类型读入，在计算过程中再转换到BigInteger类型。这就导致了一个*999999999999999999999*的样例就可以把我hack掉。

在互测期间，我主要目的是参考其他人的一般解析代码，为第二次作业使用一般解析做准备。也正是对同学代码的分析，找到了一些可能的bug。总结来说正则表达式是一个挺容易出错的点，**虽然强大，但也致命**。尤其是在使用replaceAll方法时，很有可能考虑情况不全面。有同学在对字符串预处理时用replaceAll加了前导0，比如将“（-”变成了“（0-”，在第一次作业中没问题，但在第二次作业的三角函数中添了0就不符合三角函数因子定义了。也有同学在最后化简时用replaceAll将“x\*\*2”硬生生替换成“x\*x”，第一次互测由于条件限制，无法hack这个bug因为“x\*\*21”也会被替换。所以replaceAll很方便，但很难通过逻辑推理发现这个bug。

### 三、第二次作业

#### 1、题目要求及思路分析

新增了**三角函数因子**，**自定义函数因子**和**求和函数因子**，同时输入可以产生**括号嵌套**。

因为要重构，并且是用一般解析模式，在周四之前，我都觉得这次作业难以下手，用什么数据结构就很困扰我，让我很难展开进一步的思考。但在周四研讨课上，主讲同学讲述了他自己所理解的数据结构和进一步的递归下降解析法。这使得我产生了一丝明悟。正准备下手时，逛了下讨论区，发现了又一个**新颖的数据结构**，并且我越思考越觉得这种数据结构更可行且便于统一操作。我也采用了这种数据结构*HashMap<HashMap<String,BigInteger>,BigInteger>*。内层HashMap表示x和三角函数及其指数，外层的value值表示整体的系数。这样一来，无论是表达式还是项亦或是因子都可以用这个通用的数据结构来表示。这样子减少了思维的难度，但增加了空间的占用率。毕竟一个常数因子都要以一个HashMap来存。但只有周五周六两天了，再空想就真的要凉凉了。这也是我第一单元作业写代码写得（真）“天昏地暗”的时候。

#### 2、基于度量的程序结构分析

##### 2.1 UML类图分析（仅显示主要属性和方法）

![1648215683683](C:\Users\L\AppData\Roaming\Typora\typora-user-images\1648215683683.png)

这次作业使用了*HashMap<HashMap<String,BigInteger>,BigInteger>*作为数据存储的对象，所以很显然，表达式、项和因子几乎都有这个类型的属性。Lexer类用于词法分析，每次调用都会得到一个符号或者因子。Parser类是将Lexer分析到的东西解析成表达式、项、因子。在思路上是按照的**递归下降思路**进行词法分析并解析出各表达式、项、因子。总共六类因子，除了自定义函数外，我都将其继承在Factor类下。原因是自定义函数在我看来只需要一个整体替换，当然，是在解析过程中替换掉而不是预处理中替换掉，因为预处理用replaceAll之类的方法基本都是基于正则表达式，但他解决不了括号的嵌套。本次作业为图方便，直接把三角函数当做字符串进行处理，这样子就表明了这种架构几乎不太可能会对三角函数进行有效的化简，也为第三次作业的架构埋下了伏笔。重写了.equals()和.hashCode()方法，在性能上基本只进行了合并同类项和对最终字符串答案的化简。

##### 2.2 方法复杂度分析

| Method                                    | CogC   | ev(G)  | iv(G)  | v(G)   |
| ----------------------------------------- | ------ | ------ | ------ | ------ |
| DiyFunction.DiyFunction(String)           | 3      | 1      | 3      | 3      |
| DiyFunction.applyFunction(String, String) | 3      | 1      | 3      | 3      |
| DiyFunction.simplify(String)              | 0      | 1      | 1      | 1      |
| Expr.Expr()                               | 0      | 1      | 1      | 1      |
| Expr.addTerm(Term)                        | 4      | 1      | 3      | 3      |
| Expr.addTermNeg(Term)                     | 4      | 1      | 3      | 3      |
| Expr.equals(Object)                       | 3      | 3      | 2      | 4      |
| Expr.getFactors()                         | 0      | 1      | 1      | 1      |
| Expr.hashCode()                           | 0      | 1      | 1      | 1      |
| Expr.toString()                           | **19** | 1      | 7      | 7      |
| Factor.Factor()                           | 0      | 1      | 1      | 1      |
| Factor.getFactors()                       | 0      | 1      | 1      | 1      |
| Lexer.Lexer(String)                       | 0      | 1      | 1      | 1      |
| Lexer.getBrackets()                       | 6      | 1      | 3      | 5      |
| Lexer.getCurToken()                       | 0      | 1      | 1      | 1      |
| Lexer.getType()                           | 0      | 1      | 1      | 1      |
| Lexer.readNext()                          | 8      | 2      | 7      | 8      |
| Main.getFunction1()                       | 0      | 1      | 1      | 1      |
| Main.getFunction2()                       | 0      | 1      | 1      | 1      |
| Main.getFunction3()                       | 0      | 1      | 1      | 1      |
| Main.main(String[])                       | 5      | 1      | 4      | 5      |
| Main.simplify(String)                     | 0      | 1      | 1      | 1      |
| Num.Num(BigInteger, String)               | 2      | 1      | 2      | 2      |
| Num.getFactors()                          | 0      | 1      | 1      | 1      |
| Parser.Parser(Lexer)                      | 0      | 1      | 1      | 1      |
| Parser.parserExpr(String)                 | **22** | 5      | **18** | **19** |
| Parser.parserFactor(String)               | **14** | **10** | **11** | **11** |
| Parser.parserTerm()                       | **29** | 1      | **12** | **12** |
| Sum.Sum(String, String)                   | 2      | 2      | 2      | 3      |
| Sum.applySum()                            | 0      | 1      | 1      | 1      |
| Sum.simplify(String)                      | 0      | 1      | 1      | 1      |
| Term.Term()                               | 0      | 1      | 1      | 1      |
| Term.addFactor(Factor)                    | **23** | 6      | 8      | 8      |
| Term.equals(Object)                       | 3      | 3      | 2      | 4      |
| Term.getTerms()                           | 0      | 1      | 1      | 1      |
| Term.hashCode()                           | 0      | 1      | 1      | 1      |
| Term.toString()                           | 0      | 1      | 1      | 1      |
| Tri.Tri(String, BigInteger, String)       | 2      | 1      | 2      | 2      |
| Tri.getFactors()                          | 0      | 1      | 1      | 1      |
| Var.Var(BigInteger, String)               | 2      | 1      | 2      | 2      |
| Var.getFactors()                          | 0      | 1      | 1      | 1      |
| Total                                     | 154    | 65     | 117    | 127    |
| Average                                   | 3.76   | 1.59   | 2.85   | 3.1    |

因为要考虑到最后字符串的化简，表达式类的toString方法很难降低复杂度。针对表达式、项、因子的Parser类中的方法复杂度十分显眼。在整体架构中，也确实是递归下降的整个逻辑有点模糊吧，很多注释的代码摆在那儿，最终自己是反复试探n多次找到了一个看似合理的递归。Term类的addFactor方法也有很高的复杂度，但Expr类的addTerm方法复杂度却不高。原因应该是因为Term类的addFactor方法实际上是乘法，而Expr类的addTerm方法实际上是加法。由于我采用的数据结构内外嵌套了两层HashMap，所以循环的会比较多，当初写乘法和加法时，都是在计算过程中进行简化和合并同类项的，所以这部分花时间，也费脑……

##### 2.3 类复杂度分析

| Class       | OCavg   | OCmax | WMC   |
| ----------- | ------- | ----- | ----- |
| DiyFunction | 2.33    | 3     | 7     |
| Expr        | 2.71    | 7     | 19    |
| Factor      | 1       | 1     | 2     |
| Lexer       | **3.2** | 8     | 16    |
| Main        | 1.8     | 5     | 9     |
| Num         | 1.5     | 2     | 3     |
| Parser      | **7.5** | 12    | 30    |
| Sum         | 1.67    | 3     | 5     |
| Term        | 2.5     | 8     | 15    |
| Tri         | 1.5     | 2     | 3     |
| Var         | 1.5     | 2     | 3     |
| Total       |         |       | 112   |
| Average     | 2.73    | 4.82  | 10.18 |

没有任何意外，Parser类独占鳌头，因为表达式、项、因子在这个类里混战，各种符号在里面飘来飘去。当时写完后已经周六下午了，在中测debug的时候脑子很不清醒，也很难全面分析这个递归关系。当然，在最后还是基本完成了解析的任务。现在在看的话，其实逻辑还是蛮清晰的，但**循环复杂度**飚的太高了，可能是因为代码实现太笨重了吧。

#### 3、公测互测bug分析

这次的强测是惨不忍睹，错了很多个，加上互测的hack，一共相当于是改了两个bug，且都是因为replaceAll没考虑全面，自己的三角函数是有括号的，而在最终化简时完全没有意识到这个点，比如还是很自然的把x\*\*2替换成x\*x，这就不符合形式化表述了，此外，无脑将“+”提到前面来也是有可能动到三角函数里的“+”导致答案直接错误。

在互测中，也是因为方便后续符合化简形式，我将“(-”直接化为“(0-”，这种前期的预处理到了后面完全就忘得一干二净了。当我在沾沾自喜这个bug能hack到互测房的小粗心们的时候，还没意识到我也有这个bug，最后才暗道小丑竟是我自己。。。事实上在对符号判断试验了很多次后，完全没必要这样子使得整个式子变得易于理解，在解析过程中已经把符号给带上了。本次bug都出自于一些化简中，看起来和最复杂的Lexer和Parser并无太大关系，但实际上却正是因为在写它们时，没有一个**明确的思路**，想到哪儿，写到哪儿，忘记了自己之前的规划，从而导致前期的预处理与正式的解析产生了矛盾，事实上应该是先想好怎么预处理或者怎么解析，两者选其一后再做，而不是瞻前顾后，容易赔了性能又不对。

### 四、第三次作业

#### 1、题目要求及思路分析

第三次作业允许三角函数中有**嵌套因子**，**自定义函数**也可以嵌套了，**求和函数**也可以嵌套了。

由于采用递归下降方法，括号的嵌套对我而言基本都不用变，我唯一要做的就是处理三角函数的表达式。这可感动到我了，OO终于在这周重见天日了。由于第二次作业采用的纯粹的字符串表示三角函数，而这已经很难再更改了，所以第二次作业的伏笔使我想到，这次还是用字符串表示三角函数，只不过再提前处理一下就可以了。将本来就是字符串的三角函数内部重新当成一个表达式，对其进行解析，使用toString方法直接再返回化简好后的字符串带入到三角函数括号内。当然，得考虑到这是三角函数内部，x\*\*2就不要再化简成x\*x了，不然还要多套层括号，性能反而降低。HashMap本来是无序的，但HashMap是通过hash算法把key映射到table数组的某个位置，也就是说化成字符串后，相等的三角函数字符串顺序基本上也是相同的，这对于合并同类项而言是一个莫大的简化，甚至合并同类项的部分根本不需要修改。即使没有针对三角函数的公式化简，也依然能有一个较高的强测分数。

#### 2、基于度量的程序结构分析

##### 2.1 UML类图分析（仅显示主要属性和方法）

![1648220820295](C:\Users\L\AppData\Roaming\Typora\typora-user-images\1648220820295.png)

类的数量和功能基本不变。对自定义函数类增加了getParamemters方法，用于详细拆分每个表达式因子，而非直接以“，”进行分割。而表达式中的新增reverse方法是将一个正系数项移到前面，方法内部通过追踪当前所在括号层数判断这个“+”是否是**最外层表达式的一个项**。事实上真正做的事大部分都是蕴含在第二次作业的框架里的，三角函数的字符串转化为表达式，化简后再从表达式转化为字符串，看起来工程量很大，但实际上有已有的代码支撑，寥寥数十行就能解决这个问题，知道整个表达式化简的逻辑是对的，根本就不用继续再追踪整个过程是怎么实现的，就好像只是提供了接口，剩下的工作就交给已有的代码了。

##### 2.2 方法复杂度分析

| Method                                    | CogC   | ev(G)  | iv(G)  | v(G)   |
| ----------------------------------------- | ------ | ------ | ------ | ------ |
| DiyFunction.DiyFunction(String)           | 3      | 1      | 3      | 3      |
| DiyFunction.applyFunction(String, String) | 3      | 1      | 3      | 3      |
| DiyFunction.getParameters(String, int)    | 6      | 1      | 5      | 6      |
| DiyFunction.simplify(String)              | 0      | 1      | 1      | 1      |
| Expr.Expr()                               | 0      | 1      | 1      | 1      |
| Expr.addTerm(Term)                        | 4      | 1      | 3      | 3      |
| Expr.addTermNeg(Term)                     | 4      | 1      | 3      | 3      |
| Expr.equals(Object)                       | 3      | 3      | 2      | 4      |
| Expr.getFactors()                         | 0      | 1      | 1      | 1      |
| Expr.hashCode()                           | 0      | 1      | 1      | 1      |
| Expr.reverse(String)                      | 8      | 6      | 6      | 6      |
| Expr.toString()                           | **20** | 1      | 8      | 8      |
| Factor.Factor()                           | 0      | 1      | 1      | 1      |
| Factor.getFactors()                       | 0      | 1      | 1      | 1      |
| Lexer.Lexer(String)                       | 0      | 1      | 1      | 1      |
| Lexer.getBrackets()                       | 6      | 1      | 3      | 5      |
| Lexer.getCurToken()                       | 0      | 1      | 1      | 1      |
| Lexer.getType()                           | 0      | 1      | 1      | 1      |
| Lexer.readNext()                          | **10** | 2      | 8      | 9      |
| Main.getFunction1()                       | 0      | 1      | 1      | 1      |
| Main.getFunction2()                       | 0      | 1      | 1      | 1      |
| Main.getFunction3()                       | 0      | 1      | 1      | 1      |
| Main.main(String[])                       | 5      | 1      | 4      | 5      |
| Main.simplify(String)                     | 0      | 1      | 1      | 1      |
| Num.Num(BigInteger, String)               | 2      | 1      | 2      | 2      |
| Num.getFactors()                          | 0      | 1      | 1      | 1      |
| Parser.Parser(Lexer)                      | 0      | 1      | 1      | 1      |
| Parser.parserExpr(String)                 | **22** | 5      | **18** | **19** |
| Parser.parserFactor(String)               | **14** | **10** | **11** | **11** |
| Parser.parserTerm()                       | **29** | 1      | **12** | **12** |
| Sum.Sum(String, String)                   | 2      | 2      | 2      | 3      |
| Sum.applySum()                            | 0      | 1      | 1      | 1      |
| Sum.simplify(String)                      | 0      | 1      | 1      | 1      |
| Term.Term()                               | 0      | 1      | 1      | 1      |
| Term.addFactor(Factor)                    | **23** | 6      | 8      | 8      |
| Term.equals(Object)                       | 3      | 3      | 2      | 4      |
| Term.getTerms()                           | 0      | 1      | 1      | 1      |
| Term.hashCode()                           | 0      | 1      | 1      | 1      |
| Term.toString()                           | 0      | 1      | 1      | 1      |
| Tri.Tri(String, BigInteger, String)       | **12** | 3      | 9      | **10** |
| Tri.getFactors()                          | 0      | 1      | 1      | 1      |
| Tri.simplify(String)                      | 0      | 1      | 1      | 1      |
| Var.Var(BigInteger, String)               | 2      | 1      | 2      | 2      |
| Var.getFactors()                          | 0      | 1      | 1      | 1      |
| **Total**                                 | 181    | 75     | 138    | 150    |
| **Average**                               | 4.11   | 1.70   | 3.14   | 3.41   |

大部分与第二次作业无明显差异，这次重点主要是在三角函数Tri的构造方法中，其方法复杂度明显上升，主要是多了一些条件判断用于化简和正则匹配，个人认为无伤大雅。但也正是因为其他地方不用改才导致后面有被测到bug。

##### 2.3 类复杂度分析

| Class       | OCavg    | OCmax | WMC  |
| ----------- | -------- | ----- | ---- |
| DiyFunction | 3        | 5     | 12   |
| Expr        | **3.25** | 8     | 26   |
| Factor      | 1        | 1     | 2    |
| Lexer       | **3.4**  | 9     | 17   |
| Main        | 1.8      | 5     | 9    |
| Num         | 1.5      | 2     | 3    |
| Parser      | **7.5**  | 12    | 30   |
| Sum         | 1.67     | 3     | 5    |
| Term        | 2.5      | 8     | 15   |
| Tri         | **3.33** | 8     | 10   |
| Var         | 1.5      | 2     | 3    |
| Total       |          |       | 132  |
| Average     | 3        | 5.73  | 12   |

Expr类和Tri类的循环复杂度增加了，我主要也是改的这两个类。正常现象吧，我对我这次的提交充满信心。

#### 3、公测互测bug分析

打脸的时刻到来了，强测没有被hack到，但互测有被冒犯到。sum在替换i时没有打括号，这本该是第二次作业的bug。。。bug遗留问题果然还是无法避免。这次的bug可以说是一个另类了，没有出现在复杂度高的部分中，但也提醒了我括号的重要性，还是replaceAll这个方法一定得谨慎再谨慎。

### 五、心得体会

初遇OO，日思夜想，再谈OO，夜不能寐，三见OO，日落而息。第一次作业，几乎毫无头绪，让人头大，连pre都还没做完呢……第二次作业嘛，三角控制强，化简秒全场；括号输出高，hack刀刀爆。可以说是只做到了基本的东西，剩下的优化实在是力不从心啊，感觉以后每周五都可能在淦一场小美赛。第三次作业就很开心啦，终于能做到周五不熬夜了555~

当然，我不是一个人在战斗，这次的任务能够完成还是要靠大家的无私分享。讨论区也好，研讨课也好，甚至那备受吐槽的互测都是对我帮助很大的。要说的话最难受的还是在第一周，pre明确的告诉了我们应该建一个什么类，而作业就直接把这个问题抛给了我们，所以起步的困难是我感触最大的。由于我既用了预解析，又用到了一般解析，所以第一二周我都在这个问题上挣扎。大量的代码复用是面向对象的一个优点，或许第三次作业相对于第二次作业的难度梯度上涨不大就是让我们明白这个道理？一个好的架构在于拒绝修改，接受扩展。这对于面向对象来说十分好实现。只要预判出来了以后可能需要扩展什么功能，如果在做第二次作业时就意识到三角函数内部其实可扩展成表达式因子，那么第三次作业几乎都可以直接push了。我也曾了解过同为面向对象一员的c++，与之相比，Java没有运算符的重载（除开String类的“+”吧），也不需要写析构函数，每次也只能继承一个父类，简化了许多部分，而且可以直接查看底层代码的这个功能会使得我们在用一个几乎从没用过的方法时方便很多。而且还有这么强大的IDEA可以使用，我还有什么理由不能学会呢。
