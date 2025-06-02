首先这里我们可以分析下给我们的几个calculator的程序：

actually，如果看懂这几个程序的递进和关系，也就理解这个.y文件了。

给的四个程序都是能实现这个目标的：实现一个计算器。这里只是简单的语法分析器吗？其实不是，这里的其实已经是一个语义分析器了。他将计算器的语义提取出来并且我们可以认为将val这个属性计算出来，并且最后输出。

题目以有无词法分析器和是否二义性文法，为我们创造了四种写法。

如果是二义文法，我们就需要手动声明下。在中间区标明%left %right

如果没有词法分析器，表示yylex函数没有实现，所以要么我们自己实现，要么写一个.l文件，让flex实现。

calculator0:
```cpp
%{
	#include <ctype.h>
	#include <stdio.h>
	int yylex();
	int yyerror(char* s);  /* yyerror 声明，使用库中的实现，为避免出现warning */
  #define YYSTYPE double /* 将Yacc栈定义为double类型 */
  #define YYDEBUG 1      /* 允许debug模式 */
%}

%token NUM LPAREN RPAREN ENTER PLUS MINUS TIMES DIVIDE

%%

  /* 这样写prog可以让分析器每次读入一行进行分析，下一行重新分析expr */

prog : prog expln
		 | expln
		 ;

expln 	: expr ENTER {printf("\nThe value of the expression is %lf.\n", $1);}
			 	;
		 	
expr  : expr PLUS term	{$$ = $1 + $3;}
			| expr MINUS term {$$ = $1 - $3;}
			| term	
			;

term	:	term TIMES factor  {$$ = $1 * $3;}
			| term DIVIDE factor {$$ = $1 / $3;}
			| factor
			;
			
factor  : LPAREN expr RPAREN {$$ = $2;}
				| MINUS factor 	{$$ = - $2;}
				| NUM						{$$ = $1;}
				;
 
%%

/* 用c写的识别算术表达式的词法分析器。
 * 忽略空格和制表符
 * 能够识别+，-，*，/，(，)，换行符
 * 还能够识别浮点数（整数也识别为浮点数）
 */

int yylex(){
  int c;
  do{
    c=getchar();
  } while(c==' ' || c=='\t');
  switch(c){
  case '+': return PLUS;
  case '-': return MINUS;
  case '*': return TIMES;
  case '/': return DIVIDE;
  case '(': return LPAREN;
  case ')': return RPAREN;
  case '\n': return ENTER;
  default: 
    if ((c=='.')||(isdigit(c))){
      ungetc(c,stdin);
      scanf("%lf", &yylval);
      return NUM;
    } else {
      printf("\nLEX:ERROR! c=%c\n", c);
  	  return -1;}
  }
}

int main(){
  // yydebug = 1;
	yyparse();
	return 0;
}
```

那么最后一切的一切就会推到如何实现一个合适的文法。

所以仿照这种手法我们可以写出一个不需要词法.l文件的语义分析文件。
其实我倒是觉得老师这个实验的这里语法分析都是次要的，他并没有完全的体现语法分析的含义，倒是很注重语义分析的感觉。

文件如下：
```cpp
%{
#include<string.h>
#include<ctype.h>
#include<stdio.h>
int yylex();
int yyerror(const char* s);

#define YYSTYPE int
%}

%token OR AND NOT LPAREN RPAREN TRUE FALSE ENTER
%left OR
%left AND
%right NOT

%%

input:
	 expr ENTER 			{ printf("%s\n", $1 ? "true" : "false"); }
	 | input expr ENTER 	{ printf("%s\n", $2 ? "true" : "false"); }
	 ;

expr:
	expr OR expr			{ $$ = $1 || $3; }
	| expr AND expr			{ $$ = $1 && $3; }
	| NOT expr				{ $$ = !$1; }
	| LPAREN expr RPAREN	{ $$ = $2; }
	| TRUE					{ $$ = 1; }
	| FALSE					{ $$ = 0; }
	;

%%


int yylex() {
    int c;
    do {
        c = getchar();
    } while (c == ' ' || c == '\t');

    // 先读取单词，再比较
    if (isalpha(c)) {
        char buf[32];
        int i = 0;
        do {
            buf[i++] = c;
            c = getchar();
        } while (isalpha(c) && i < sizeof(buf)-1);
        ungetc(c, stdin);  // 放回非字母字符
        buf[i] = '\0';

        if (strcmp(buf, "or") == 0) return OR;
        if (strcmp(buf, "and") == 0) return AND;
        if (strcmp(buf, "not") == 0) return NOT;
        if (strcmp(buf, "true") == 0) return TRUE;
        if (strcmp(buf, "false") == 0) return FALSE;
    }

    switch (c) {
        case '(': return LPAREN;
        case ')': return RPAREN;
        case '\n': return ENTER;
        default:
            printf("LEX ERROR: Invalid char '%c'\n", c);
            return -1;
    }
}

int yyerror(const char *s) {
    fprintf(stderr, "Syntax Error: %s\n", s);
    return 0;
}

int main() {
	yyparse();
	return 0;
}

```