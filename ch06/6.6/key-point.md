# 6.6控制流的翻译
## 6.6.1布尔表达式
  B1 || B2 如果B1为真，则不用再计算B2，整个表达式为真
  
  B1 && B2 如果B1为假，则不用再计算B2，整个表达式为假

## 6.6.2短路代码
  语句
  
      if(x<100||x>200 && x!=y) x=0;
  代码
  
      if x < 100 gotoL2
      
      if False x>200 goto L1
      
      if False x!=y goto L1
     
      L2: x = 0
 
      L1: 接下来的代码
  
  在短路跳转中，布尔运算符&&、||和！被翻译成跳转指令，运算符本身并不出现在代码中，布尔表达式的值是通过代码序列中的位置来表示的。

## 6.6.3控制流语句
### 翻译
文法：B表示布尔表达式，S代表语句

    S->if(B) S1
    
    S->if(B) S1 else S2
    
    s->while(B) S1
    
继承属性

    B.true: B为真的跳转目标
    
    B.false: B为假的跳转目标
    
    B.next： S执行完毕时的跳转目标
    
### 语法制导的定义
产生式|语义规则
-----|-------
P->S | S.next = newlabel()<br>P.code = S.code &#124;&#124; label(s.next)|
S->assign|S.code = assign.code|
S->if(B)S1|B.true = newlabel()<br>B.false = newlabel()<br>S1.next = S2.next = s.next<br>S.code = B.code &#124; &#124;label(B.true)  <br>&#124; &#124; S1.code  &#124; &#124;gen('goto'S.next)<br> &#124; &#124;label(B.false) &#124; &#124;S2.code|
S->while(B) S1|begin = newlabel()<br>B.true = newlabel()<br>B.false = S.next<br>S1.next = begin<br>S.code = label(begin) &#124; &#124;B.code<br> &#124; &#124;label(B.true) &#124; &#124;S1.code<br> &#124; &#124;gen('goto'begin)|
S->S1S2|S1.next = newlabel()<br>S2.next = S.next<br>S.code = S1.code &#124; &#124;label(S.next) &#124; &#124;S2.code

## 6.6.4布尔表达式的控制流翻译
### 生成的代码执行时跳转到两个标号之一
1、表达式的值为真时，跳转到B.true
2、表达式的值为假时，跳转到B.false

### B.true和B.false 是两个继承属性，根据B所在的上下文指向不同的位置
1、如果B是if语句的条件表达式，分别指向then分支和else分支；如果没有else分支，则指向if语句的下一条指令
2、如果B是while语句的条件表达式，分别指向循环体的开头和循环出口处

### 布尔表达式的代码SDD
产生式|语义规则
--------|------------
B->B1 &#124; &#124;B2|B1.true = B.true<br>B1.false = newlabel()<br>B2.true = B.true<br>B2.false = B.false<br>B.code = B1.code &#124; &#124;label(B1.true) &#124; &#124;B2.code
B->B1&&B2|B1.true = newlabel()<br>B1.false = B.false<br>B2.true = B.true<br>B2.false = B.false<br>B.code = B1.code &#124; &#124;label(B1.true) &#124; &#124;B2.code
B->!B1|B1.true = B.false<br>B1.false = B.true <br> B.code = B1.code
B->E1 rel E2|B.code = E1.code &#124; &#124;E2.code<br> &#124; &#124;gen('if' E1.addr rel.op E2.addr 'goto' B.true)
B->true|B.code = gen('goto' B.true)
B->false|B.code = gen('goto' B.false)

## 6.6.5避免生成冗余的goto指令
## 6.6.6布尔值和跳转代码
1、程序中出现布尔表达式的目的可能就是求出它的值，比如x = a<b
2、处理办法 首先建立表达式的语法树，然后根据表达式的不同角色来处理
3、文法

	S->id = E; | if (E) S | while(E) S | SS
	E->E || E | E && E | E rel E |.....
	
4、根据E的语法树结点所在的位置
	
	S->while(E) S1中的E，生成跳转代码
	对于S->id = E,生成计算右值的代码
	
