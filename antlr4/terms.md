
下面介绍很多重要的与语言识别相关的术语。

**语言(`Language`)**

- A language is a set of **valid sentences**  
一门语言是一个有效语句的集合。


- Sentences are composed of **phrases**, which are composed of subphrases, and so on  
语句由词组组成，词组由子词组组成，子词组又由更小的子词组组成，依此类推。

**语法(`Grammar`)**

- A grammar formally defines the **syntax rules** of a language  
语法定义了语言的语义规则。

- Each rule in a grammar expresses the **structure** of a subphrase  
语法中的每条规则定义了一种词组结构。

**语义树或语法分析树(`Syntax tree/parse tree`)**

- This represents the structure of the sentence where each subtree root gives an abstract name to the elements beneath it.  
代表了语句的结构，其中每个子树的根节点都使用一个抽象的名字给其包含的元素命名。


- The **subtree roots** correspond to **grammar rule** names.  
子树的根节点对应了语法规则的名字。


- The **leaves** of the tree are symbols or **tokens**s of the sentence.  
树的叶子节点是语句中的符号或者词汇。

**词法符号(`Token`)**

- A token is a **vocabulary symbol** in a language;  
词法符号就是一门语言的基本词汇符号


- these can represent a category of symbols such as "identifier" or can represent s single operator or keyword.  
它们可以代表像是“标识符”这样的一类符号，也可以代表一个单一的运算符，或者代表一个关键字。

**词法分析器或者分词器(`Lexer or tokenizer`)**

- This breaks up an **input character stream** into **tokens**;  
分词器将输入的字符序列分解成一系列词汇。


- A lexer performs lexical analysis.  
词法分析器执行词法分析。


**语法分析器(`Parser`)**

- A parser checks sentences for membership in a specific language by checking the sentence's structure against the rules of a grammar.  
语法分析器通过检查语句的结构是否符合语法规则的定义来验证该语句在特定语言中是否合法。


- ANTLR generates top-down parsers called `ALL(*)` that can **use all remaining input symbols to make decisions**.  
`ANTLR`能够生成被称为`ALL(*)`的自顶向下的语法分析器，`ALL(*)`是指它可以利用剩余的所有输入文本来进行决策。


- Top-down parser are **goal-oriented** and start matching at the rule associated with the coarsest, such as program or inputFile.  
自顶向下的语法分析器以结果为导向，首先匹配最粗粒度的规则，这样的规则通常命名为`program`或者`inputFile`。

**递归下降的语法分析器(`Recursive-descent parser`)**

- This is a specific kind of top-down parser implemented with **a function** for **each rule** in the grammar.  
这是自顶向下的语法分析器的一种实现，每条规则都对应语法分析器中的一个函数。

**前向预测(`Lookahead`)**

- Parsers use lookahead to **make decisions** by comparing the symbols that begin each alternatives.  
语法分析器使用前向预测来进行决策，具体方法是将输入的符号与每个备选分支的起始符号进行比较。
