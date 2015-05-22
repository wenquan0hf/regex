# 如何写出高效率的正则表达式

如果纯粹是为了挑战自己的正则水平，用来实现一些特效（例如使用正则表达式计算质数、解线性方程），效率不是问题；如果所写的正则表达式只是为了满足一两次、几十次的运行，优化与否区别也不太大。但是，如果所写的正则表达式会百万次、千万次地运行，效率就是很大的问题了。我这里总结了几条提升正则表达式运行效率的经验（工作中学到的，看书学来的，自己的体会），贴在这里。如果您有其它的经验而这里没有提及，欢迎赐教。

为行文方便，先定义两个概念。

**误匹配**：指正则表达式所匹配的内容范围超出了所需要范围，有些文本明明不符合要求，但是被所写的正则式“击中了”。例如，如果使用\d{11}来匹配 11 位的手机号，\d{11}不单能匹配正确的手机号，它还会匹配 98765432100 这样的明显不是手机号的字符串。我们把这样的匹配称之为误匹配。

**漏匹配**：指正则表达式所匹配的内容所规定的范围太狭窄，有些文本确实是所需要的，但是所写的正则没有将这种情况囊括在内。例如，使用\d{18}来匹配 18 位的身份证号码，就会漏掉结尾是字母 X 的情况。

写出一条正则表达式，既可能**只出现**误匹配（条件写得极宽松，其范围大于目标文本），也可能**只出现**漏匹配（只描述了目标文本中多种情况种的一种），还可能**既有误匹配又有漏匹配**。例如，使用`\w+\.com`来匹配`.com`结尾的域名，既会误匹配 abc_.com 这样的字串（合法的域名中不含下划线，\w包含了下划线这种情况），又会漏掉 ab-c.com 这样的域名（合法域名中可以含中划线，但是\w不匹配中划线）。

精准的正则表达式意味着既无误匹配且无漏匹配。当然，现实中存在这样的情况：只能看到有限数量的文本，根据这些文本写规则，但是这些规则将会用到海量的文本中。这种情况下，尽可能地（如果不是完全地）消除误匹配以及漏匹配，并提升运行效率，就是我们的目标。本文所提出的经验，主要是针对这种情况。

**掌握语法细节**。正则表达式在各种语言中，其语法大致相同，细节各有千秋。明确所使用语言的正则的语法的细节，是写出正确、高效正则表达式的基础。例如，perl 中与 \w 等效的匹配范围是`[a-zA-Z0-9_]`；perl 正则式不支持肯定逆序环视中使用可变的重复（variable repetition inside lookbehind，例如(?<=.*)abc），但是 .Net 语法是支持这一特性的；又如，JavaScript连逆序环视（Lookbehind,如(?<=ab)c）都不支持，而 Perl 和 Python 是支持的。《精通正则表达式》第3章《正则表达式的特性和流派概览》明确地列出了各大派系正则的异同，这篇文章也简要地列出了几种常用语言、工具中正则的比较。对于具体使用者而言，至少应该详细了解正在使用的那种工作语言里正则的语法细节。

**先粗后精，****先加后减**。使用正则表达式语法对于目标文本进行描述和界定，可以像画素描一样，先大致勾勒出框架，再逐步在局步实现细节。仍举刚才的手机号的例子，先界定\d{11}，总不会错；再细化为1[358]\d{9}，就向前迈了一大步（至于第二位是不是3、5、8，这里无意深究，只举这样一个例子，说明逐步细化的过程）。这样做的目的是先消除漏匹配（刚开始先尽可能多地匹配，做加法），然后再一点一点地消除误匹配（做减法）。这样有先有后，在考虑时才不易出错，从而向“不误不漏”这个目标迈进。

**留有余地**。所能看到的文本sample是有限的，而待匹配检验的文本是海量的，暂时不可见的。对于这样的情况，在写正则表达式时要跳出所能见到的文本的圈子，开拓思路，作出“战略性前瞻”。例如，经常收到这样的垃圾短信：“发*票”、“发#漂”。如果要写规则屏蔽这样烦人的垃圾短信，不但要能写出可以匹配当前文本的正则表达式 发[*#](?:票|漂)，还要能够想到 发.(?:票|漂|飘)之类可能出现的“变种”。这在具体的领域或许会有针对性的规则，不多言。这样做的目的是消除漏匹配，延长正则表达式的生命周期。

**明确**。具体说来，就是**谨慎**用点号这样的元字符，**尽可能**不用星号和加号这样的任意量词。只要能确定范围的，例如\w，就不要用点号；只要能够预测重复次数的，就不要用任意量词。例如，写析取twitter消息的脚本，假设一条消息的 xml 正文部分结构是<span class=”msg”>…</span>且正文中无尖括号，那么<span class=”msg”>[^<]{1,480}</span>这种写法**的思路**要好于<span class=”msg”>.*</span>，原因有二：一是使用[^<]，它保证了文本的范围不会超出下一个小于号所在的位置；二是明确长度范围，{1,480}，其依据是一条twitter消息大致能的字符长度范围。当然，480这个长度是否正确还可推敲，但是这种思路是值得借鉴的。说得狠一点，“滥用点号、星号和加号是不环保、不负责任的做法”。

**不要让稻草压死骆驼**。每使用一个普通括号()而不是非捕获型括号(?:…)，就会保留一部分内存等着你再次访问。这样的正则表达式、无限次地运行次数，无异于一根根稻草的堆加，终于能将骆驼压死。养成合理使用(?:…)括号的习惯。

**宁简勿繁**。将一条复杂的正则表达式拆分为两条或多条简单的正则表达式，编程难度会降低，运行效率会提升。例如用来消除行首和行尾空白字符的正则表达式`s/^\s+|\s+$//g`;，其运行效率理论上要低于`s/^\s+//g`; `s/\s+$//g`; 。这个例子出自《精通正则表达式》第五章，书中对它的评论是“它几乎总是最快的，而且显然最容易理解”。既快又容易理解，何乐而不为？工作中我们还有其它的理由要将C==(A|B)这样的正则表达式拆为A和B两条表达式分别执行。例如，虽然A和B这两种情况只要有一种能够击中所需要的文本模式就会成功匹配，但是如果只要有一条子表达式（例如A）会产生误匹配，那么不论其它的子表达式（例如B）效率如何之高，范围如何精准，C的总体精准度也会因A而受到影响。

**巧妙定位**。有时候，我们需要匹配的 the，是作为单词的 the（两边有空格），而不是作为单词一部分的 t-h-e 的有序排列（例如 together 中的 the）。在适当的时候用上^，$，\b 等等定位锚点，能有效提升找到成功匹配、淘汰不成功匹配的效率。
