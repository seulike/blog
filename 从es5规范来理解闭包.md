## 前言
 网上有很多对闭包的解释，本文希望从es5规范出发对闭包进行探讨。其中所有对概念的解释都来自es5规范并注明了对应章节。
 如若对细节不清楚，可以详读es5规范进行理解。
 
## 主要概念
 涉及到闭包的主要有三个概念:可执行代码（ Executable Code ），执行上下文（Execution Context） 及词法环境（Lexical Environment）。
 ECMAScript 中有三种类型的可执行代码（10.1）：全局代码，eval代码，函数代码。本文只关注于函数代码来解释闭包，并只给出关键步骤。
 代码的执行过程中，当控制转移到一个新的可执行代码（例如调用函数），会进入一个执行上下文（Execution Context）（10.3）。
 所有的执行上下文会形成一个栈。栈底是全局上下文，而栈顶是当前正在运行的执行上下文。
 10.2节介绍了词法环境。词法环境定义了标识符与具体变量和函数之间的联系。当函数被执行时，会创建一个新的词法环境。
 词法环境包含一个环境记录（Environment Record ）和一个指向外部词法环境（outer Lexical Environment）的引用。其中环境记录记录标识符的绑定。
 
## 创建执行上下文
  一个执行上下文主要包含LexicalEnvironment，VariableEnvironment，this绑定（ThisBinding ）。当进入一个执行上下文。会设置其内部值。
  10.4.3描述了进入函数代码(设其函数对象为F)时如何设置执行上下文。首先创建一个新的Lexical Environment env，并设置其outer lexical environment reference 
为函数对象F的[[Scope]]属性。之后设置执行上下文的LexicalEnvironment = VariableEnvironment  = env。根据相应规则设置this。
之后执行声明绑定实例化（declaration binding instantiation10.5）。这一步，函数代码的变量和函数声明会被当做绑定添加到执行上下文的
的VariableEnvironment的环境记录中。  
 其中对于每个函数声明f。设置其标识符为fn。执行13.2创建函数对象fo，其中传入当前运行中的执行上下文中的VariableEnvironment 作为Scope。
13.2中的第9步会设置fo的[[Scope]]属性为传入的Scope。设定fn = fo。

## 变量的查找及闭包的产生
  如果一个函数访问了它的外部变量，它就是一个闭包。在10.3.1节标识符（Identifier）的解析中给出了访问闭包的原理。 
  1. 设置env=运行的执行上下文的LexicalEnvironment。 
2. 调用10.2.2.1节中定义的GetIdentifierReference(lex,Identifier,strict)
GetIdentifierReference主要就是根据lex的环境记录找到对应标识符的值。如若没有，则将lex设置为outer environment reference继续递归调用自身。
如若lex == null，return undefined。
  闭包就是由此产生的。在当前的LexicalEnvironment中无法找到变量时就会递归的从outer environment reference继续寻找。因而就可以访问了它的外部变量。
## 浏览器实现
  闭包产生的关键就在于生成函数对象时会设置其[[Scope]]属性为正在运行的执行上下文的词法环境。因而形成作用域链。可以沿着作用域链查询变量。 
  chrome中执行一段测试代码： 
  ```
  function test(){
    var str = '';
    var data = 23;
    function inner(){
        print(data + 23);
    }
    function print(para){
        console.log(para);
    }
    return inner;
  }
  test();
  ```
  在inner处断点调试，查看变量值，根据图可以看到函数对象上都有[[Scopes]],里面保存了闭包变量。inner与print的值相同。这应该是chrome的v8引擎的实现特点。   
  ![在chrome中的实现](https://github.com/seulike/blog/blob/master/img/closure1.png)
  
 
