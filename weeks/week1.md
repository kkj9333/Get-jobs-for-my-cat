# Day 1 了解Clangformat用法并配置成功生成项目
## **什么是clang-format?**
Clang-Format可用于格式化（排版）多种不同语言的代码。
其自带的排版格式主要有：LLVM, Google, Chromium, Mozilla, WebKit等
## **安装**
clang-format本质是一个命令行工具
- window：https://releases.llvm.org/download.html
- ubuntu：sudo apt install clang-format
## **基本使用方式**
```javascript
clang-format [options] [<file> ...]
clang-format --help //建议至少浏览一遍帮助信息。
```
## **Clang-format导出格式**
在工程项目中，可以使用.clang-format or _clang-format格式的文件来自定义规范格式，将file放入项目里代码文件相同的根目录下：<br>
- 1.目录运行cmd 输入clang-format -style=file.
- 2.如果是VS等支持clang-format的，则不需要额外设置，VS会自动检查项目根目录下的clangformat文件
一个创建clang-format文件的简单方式是：
```javascript
clang-format -style=可选格式名 -dump-config > .clang-format
```
**可选格式最好写预设那那几个写最接近你想要的格式. 比如我想要接近google C++ style的。 我就写-style=google**

## **Clang-format配置文件说明**
Documents: https://clang.llvm.org/docs/ClangFormatStyleOptions.html <br>
注意： 一般不全部重定义规则, 提供了BasedOnStyle标识让我们来重定义部分格式 <br>
```javascript
# 语言: None, Cpp, Java, JavaScript, ObjC, Proto, TableGen, TextProto
Language: Cpp
# BasedOnStyle: LLVM
# 访问说明符(public、private等)的偏移
AccessModifierOffset: -4
# 开括号(开圆括号、开尖括号、开方括号)后的对齐: Align, DontAlign, AlwaysBreak(总是在开括号后换行)
AlignAfterOpenBracket: Align
# 连续赋值时，对齐所有等号
AlignConsecutiveAssignments: true
# 连续声明时，对齐所有声明的变量名
AlignConsecutiveDeclarations: true
# 左对齐逃脱换行(使用反斜杠换行)的反斜杠
AlignEscapedNewlinesLeft: true
# 水平对齐二元和三元表达式的操作数
AlignOperands: true
# 对齐连续的尾随的注释
AlignTrailingComments: true
# 允许函数声明的所有参数在放在下一行
AllowAllParametersOfDeclarationOnNextLine: true
# 允许短的块放在同一行
AllowShortBlocksOnASingleLine: false
# 允许短的case标签放在同一行
AllowShortCaseLabelsOnASingleLine: false
# 允许短的函数放在同一行: None, InlineOnly(定义在类中), Empty(空函数), Inline(定义在类中，空函数), All
AllowShortFunctionsOnASingleLine: Empty
# 允许短的if语句保持在同一行
AllowShortIfStatementsOnASingleLine: false
# 允许短的循环保持在同一行
AllowShortLoopsOnASingleLine: false
# 总是在定义返回类型后换行(deprecated)
AlwaysBreakAfterDefinitionReturnType: None
# 总是在返回类型后换行: None, All, TopLevel(顶级函数，不包括在类中的函数), 
#   AllDefinitions(所有的定义，不包括声明), TopLevelDefinitions(所有的顶级函数的定义)
AlwaysBreakAfterReturnType: None
# 总是在多行string字面量前换行
AlwaysBreakBeforeMultilineStrings: false
# 总是在template声明后换行
AlwaysBreakTemplateDeclarations: false
# false表示函数实参要么都在同一行，要么都各自一行
BinPackArguments: true
# false表示所有形参要么都在同一行，要么都各自一行
BinPackParameters: true
# 大括号换行，只有当BreakBeforeBraces设置为Custom时才有效
BraceWrapping:
  # case 语句后面
  AfterCaseLabel: false
  # class定义后面
  AfterClass: false
  # enum定义后面
  AfterEnum: false
  # 函数定义后面
  AfterFunction: false
  # 命名空间定义后面
  AfterNamespace: false
  # struct定义后面
  AfterStruct: false
  # union定义后面
  AfterUnion: false
  # extern导出块后面
  AfterExternBlock: false
  # catch之前
  BeforeCatch: false
  # else之前
  BeforeElse: false
  # 缩进大括号(整个大括号框起来的部分都缩进)
  IndentBraces: false
  # 空函数的大括号是否可以在一行
  SplitEmptyFunction: true
  # 空记录体(struct/class/union)的大括号是否可以在一行
  SplitEmptyRecord: true
  # 空namespace的大括号是否可以在一行
  SplitEmptyNamespace: true
```
## **Clang-format实战**

在开源项目根目录下放好配置文件，我采用的IDE是Visual Studio 2022，会自动检查；<br><br>
![image](https://user-images.githubusercontent.com/51207072/210161978-9e6408ac-7382-43b0-aee7-60fc38374867.png#pic_left)<br>
然后尝试修改代码,结果出现错误：<br><br>
<div align="left">
<img src=https://user-images.githubusercontent.com/51207072/210162029-cf66fccb-214d-47c5-abdc-e80e3aca4444.png width=35% />
</div>
查找原因ing...<br>
查找博客发现原因是原来Visual Studio的C++代码格式化可选使用clang-format, 但它只提供默认样式！！！, 如果想使用自定义样式则需要在每个项目目录下放一个.clang-format或_clang-format文件, 没有对全部项目通用的可自定义样式（放在sln同级目录下）详见：https://blog.csdn.net/Liuqz2009/article/details/118677784<br>
但事实是即便每个放置了由于VS自动clang-format解析不了，也会报错，所以只能尝试自定义样式，我实验时反正是报错，如有其他方案还请指出。<br>
<div align="left">
<img src=https://user-images.githubusercontent.com/51207072/210165922-1d0f094c-2765-4e1c-9508-c424de5a0694.png />
</div>
我在cmake构建项目时，将源代码和VS项目分离了，只需在sln下面放置一个就行。<br>
然后就没有报错了,开始愉快改写代码...<br>

# Day 2-7 阅读论文 Investigating Graph Embedding Neural Networks with Unsupervised Features Extraction for Binary Analysis 
## **论文出处**
NDSS
## **研究背景**
二进制分析技术即通过利用可执行的机器代码来分析应用程序的控制结构和运行方式，有助于研究人员更好地分析软件运行原理，协助查找漏洞以及防范非预期行为等，从而保障软件和系统安全。随着人工智能技术的发展，机器学习和智能化的方法也应用在了二进制分析领域。不幸的是，传统代码分析技术通常是为源代码的分析而定制的，不能立即用于二进制代码。例如，源代码的编译过程就会破坏了变量和函数的命名，以及变量的类型，从而阻碍了二进制代码技术的适用性。
## **研究动机**
传统代码分析技术不能立即用于二进制代码，我们迫切需要一种能将源代码分析广阔领域中的知识转移到在二进制分析的方法，这些知识可以成为设计新的、有效的，易于使用的二进制分析工具的宝贵帮助。这些知识可以成为设计新的、有效的，易于使用的二进制分析工具的宝贵帮助。<br>
二进制分析另外两个主要的限制是基本的块嵌入包含有限的语义信息和通常仅适用于单一的指令集体系结构（ISA）。通常一份源码可以编译为X86和ARM不同指令集下的两份可执行机器代码，他们应当是等价的，然而传统机器学习在跨架构二进制分析目前并没有得到很好的应用，因为他们通常只适用于单一指令集，我们需要改良这种现状。<br>
## **相关工作**
- Bao T, Burket J, Woo M, et al. {BYTEWEIGHT}: Learning to recognize functions in binary code
- Guo W, Mu D, Xu J, et al. Lemna: Explaining deep learning based security applications[C]
- Shin E C R, Song D, Moazzezi R. Recognizing functions in binaries with neural networks[C]
- Mikolov T, Sutskever I, Chen K, et al. Distributed representations of words and phrases and their compositionality[J].
- Feng Q, Zhou R, Xu C, et al. Scalable graph-based bug search for firmware images[C]
- X. Xu, C. Liu, Q. Feng, H. Yin, L. Song, and D. Song, “Neural network based graph embedding for cross-platform binary code similarity detection,” in Proceedings of - - the 24th ACM SIGSAC Conference on Computer and Communications Security, (CCS), 2017, pp. 363–376.
- Zuo F, Li X, Young P, et al. Neural machine translation inspired binary code similarity comparison beyond function pairs[J]. arXiv preprint arXiv:1808.04706, 2018.


