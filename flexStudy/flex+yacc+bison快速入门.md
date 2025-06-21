为什么我们要使用flex和bison？
1. 为了高效、结构化的实现词法分析和语法分析

本学期的课程主要集中在前端，而最开始学习的词法分析和语法分析是最简单且相对容易实现的部分。所以发展出了成熟的工具可以根据给定规则实现一个词法和语法分析器。

> Flex/Bison 类似于“自动切菜机”，可以帮你把食材切成均匀的块，但要做出一道菜（编译器），你仍需决定菜谱（语言设计）、调味（语义逻辑）、火候（优化）、摆盘（用户体验）。**工具简化了流程，但无法替代你对“烹饪逻辑”的思考**。
# flex
一个flex程序长什么样子：
```flex
%{
	/定义段/
	/*这里放着一些文件头引入、宏定义、函数声明。因为最后得到的词法分析器其实是一个c程序，这里面的东西会直接复制到生成的lex.yy.c文件中*/
%}

	/*正则表达式宏定义*/
	/*在这个区域可以对相应的正则表达式加以定义，这样就可以避免重复写复杂的正则表达式*/

%%
	/*规则段*/
	/*这个区域规定了遇到相应的正则式应该做出那些动作
		为了使用之前定义的变量这里可以使用{}实现变量的使用
	*/
%%
	/*用户代码段*/
```
其中第一部分和第三部分并不是必须的。但是如果没有这两个部分的话，这个程序编译得到的程序就无法独立使用而已。

## 从最简单到初具人形
一个最简单的flex程序
```flex
%%
[0-9]+ {return 1;}
.      {return 0;}
```

这是一个编译的flex程序，他可以有flex得到一个.c，得到的.c也可以使用gcc编译得到可执行程序。但是由于他没有main，所以无法独立执行，只可以被调用。

如果要得到一个独立执行的词法分析器。则需要完善第三个部分，以及我们需要输出结果以及引入标准输入输出，所以需要添加头文件（完善第一部分）。

于是完善后的flex程序如下：

```flex
%{ // 官方定义definitions段
#include<stdio.h>
%}

%% // rules段
[0-9]+ { printf("数字"); }
.      { printf("其他"); }
%%

// code段
int main() {
	yylex();
	return 0;
}
```

文件核心部分讲解：
- yylex()函数，这个函数就相当于状态机
> 理解：
> 在词法分析阶段其实就是实现一个有限自动机，而由于我们手工实现一个有限自动机很麻烦并且会遗漏状态，于是我们通过设计flex帮助我们实现。
> 实现的逻辑就是
> 1. 正规式是容易理解并且功能强大
> 2. 发现了正规式-有限自动机定理（我称之为）
> 所以我们只需要给定正规式就可以设计软件，通过Thompson算法和子集构造法得到有限自动机。加之我们规则的动作就实现了匹配到相应的字符串之后做什么动作。
> 而这个yylex就是flex帮助我们实现的自动机调用函数

> yylex()函数，他内部是一个持续的循环，在循环内会不断读取yyin的字符。同时进行有限状态机的状态移动，他遵从优先级和最长匹配原则，所以在rules段的规则先后顺序应该要注意。一直匹配，他会记录下最长匹配的状态，一直当匹配不下去了之后就会返回最长匹配状态并且执行相应的状态。直到循环结束或者是没有字符串了（也就是读到了EOF）。

他的内部大致如下：
```c
int yylex() {
    // 初始化匹配器、缓冲区等
    while (true) {
        int which = match_next_token(); // 扫描输入，返回匹配到的模式编号
        switch (which) {
            case 0: // 默认规则：没有任何模式匹配
                if (EOF) return 0;
                ECHO;   // 打印当前字符，消耗它
                break;
            case 1: // 第一条规则匹配
                /* 执行你在 .l 文件中写的第1条 action */
                break;
            case 2: // 第二条规则匹配
                /* 执行第2条 action */
                break;
            // …
        }
        // 如果 action 中有 return，则 yylex() 立即返回
    }
}
```
所以这里也就可以解释，为什么我之前写的哪个字符串匹配程序，遇到不匹配的字符串他就会直接输出。

为了实现共能更加强大的词法分析器（编译器要求）我们就有需要完善我们的程序

## 逐渐完善功能

为了完善功能，也就是
1. 匹配到更多更准确并且符合要求的串
2. 对相应的串执行相应的操作

那么就分别对应了
1. 按要求完善正则规则区域
2. 完善code区域（虽然这个功能在规则区的动作里就能实现，但是为了模块化并且实现更复杂的功能）
## 写一个用来测试的东西
一个测试demo
```c
%{  
#include<stdio.h>  
#define TEST 1  
int installTEST();  
%}  
  
// 想要测什么正则表达式子直接在这里写就行了  
test    (\\.)*  
%%  
  
{test}  { installTEST(); return TEST; }  
.               { return yytext[0]; }  
  
%%  
int installTEST() {  
       return TEST;  
}  
  
int yywrap() {  
       return 1;  
}  
  
int main() {  
       int c;  
       printf("开始\n");  
       while((c = yylex())) {  
               printf("yylex() 返回值: %d\n", c);  
               switch(c) {  
                       case TEST:  
                               printf("匹配到了: %s\n", yytext); break;  
                       default:  
                               printf("没有匹配到相应的规则, 字符: %c (ASCII=%d)\n", c, c); break;  
               }  
       }  
       return 0;  
}
```

## 逐渐完善
### 一些行业黑话

1. ws - whitespace空白符，空白符的动作一般都是 {;}
2. delim - delimiter分隔符，一般用来表示文本中用来分开两个词法单元的符号，例如空格，tab，换行等
3. space - space空格，胆指空格
4. letter - letter字母，用来指代字母，所以定义常常使用到字符类
5. digit - digit 0-9的数字中的一个，所以定义也用到字符类
6. relop - relational operators，关系运算符

### 一些行业规定
1. 保留字一般往前放，因为这样才能确保被识别为保留字而不是标识符
2. return (token)，token一般用括号包起来
### 一些行业技巧
#### 状态
> 在 Flex 中，**状态（Start Conditions）** 是用于控制词法分析器在不同上下文中切换规则的机制。它允许你根据当前解析的上下文环境，选择性地启用或禁用特定的词法规则。这在处理需要跨行或嵌套的结构（如注释、字符串、条件编译等）时非常有用。

有两种状态（严格来说三种）
1. 包含
   %s
2. 独占
   %x
3. 默认（INITIAL）

在动作中使用BEGIN就可以实现状态的转换
```flex
%s COMMENT // 定义了一个注释状态

"/*" { BEGIN(COMMENT);}
<COMMENT>"*/" { BEGIN(INITIAL);} // 回到初始状态
<COMMENT>.|\n {;} //这里面只要是合法的c语言就可以了
```

前缀表示状态下应该做什么
#### 一些宏

ECHO，把读到的输出出去，这也是默认的行为。所以如果一些情况下没有对一些token行为进行定义的话就会输出出来

# YACC/BISON

我愿意称之为文法定义文件。文法规定以及语法语义设计。
通过文法自动化得到语法分析步骤，并设计语义动作，使得成功实现语法分析和语义执行

一个.y文件大致是这样的
```cpp
%{
#include<xxx>
%}

%token xxx

%%
exp: 文法 {语义动作}

辅助函数
%%
```

## 如何使用和编译

关于有什么用？actually yacc表示的是：
Yet Another Compiler Compiler（又一个编译器编译器）挺好玩的定义

yacc能根据我们给的一套上下文无关文法自动生成对应的语法分析器代码。

我们应该主动的去阅读一些软件的文档。：
```shell
其作用在一开始就给我们揭示了：
Generate a deterministic LR or generalized LR (GLR) parser employing  
LALR(1), IELR(1), or canonical LR(1) parser tables.

简而言之，我们可以选择
-H
-d
-v

这三个是最重要的。
```

如果我们现在有一个.y文件，我们可以通过
```shell
yacc -d logic.y # 
```

下一步就是阅读程序给我们的makefile文件：
```Makefile
test5-1: test5-1.tab.o lex.yy.o  
	gcc -o test5-1 test5-1.tab.o lex.yy.o -ly

lex.yy.o: lex.yy.c test5-1.tab.h
	gcc -c lex.yy.c

test5-1.tab.o: test5-1.tab.c
	gcc -c test5-1.tab.c

lex.yy.c: test5-1.l
	flex test5-1.l

test5-1.tab.c: test5-1.y
	bison -dv test5-1.y

test5-1.tab.h: test5-1.y
	echo "test5-1.tab.h was created at the same time as test5-1.tab.c."

clean:
	rm -f test5-1 lex.yy.o test5-1.tab.o lex.yy.c test5-1.tab.c test5-1.tab.h test5-1.stackdump test5-1.output

```

从这里就应该可以看出来了，这个程序就是借助词法分析器的.o文件和语法分析文件.o连接而成的文件。
文件逻辑是
- .l->.yy.c->.yy.o
- .y->.tab.c->.tab.o

而需要连接起来就需要内容上有一点点规定：

## 文件间的关联

首先确认：
1. 词法分析器和语法分析器都来来自于文法规定，且文法在.y文件中定义实现。所以核心先实现.y文件。
2. 文法中规定了语法规则。所以是语法分析器的核心。文法符号中规定了文法符号，文法符号最后基本上要对应到终结符，终结符来源就是词法分析器。于是就是在这里定义Token。
   如果二义文法甚至要定义好优先级和结合性
3. 然后词法分析器的核心其实是什么？actually是yylex函数
   [[实验1学习#yylex()]]。
   同理.y文件其实也就是我们设计文法并设计实现，最后yyac自动编译得到一个yyparse函数而已

所以如果是一个很复杂的问题的时候，我们完全是可以自己写一个.c文件来实现更强大的功能，然后调用yyparse函数。

而我们前面也注意到了，其实yacc会得到一个.h文件，这个文件一个重要的功能其实就是定义很多，诸如token的预定义等。而此时我们词法分析器需要保持一致，此时也就不再需要自己#define了，可以直接#include

