---
title: "Compiler 作業二：yacc"
date: 2022-05-14T11:58:18+08:00
draft: false
toc: true
images:
tags: 
  - complier
---

## yacc 基本架構
```
%{
// part 1

%}
// part 2

%%
// part 3

%%
// part 4
```

* part1 跟 lex 一樣，include 需要用到的 file，以及一些宣告
* part2 列出所有 termial 的 token、non-terminal 的 type，跟 operator 的優先順序等等
* part3 寫 grammar ，以及需要的動作
* part4 跟 lex 一樣，可寫可不寫 main()

## Grammar

因為在實作的過程中，主要是先寫好整個 grammar，最後在補上 sementic check 所需要的東西，所以分這兩塊來寫。

grammar 的部分就是把該語言的語法寫清楚，這部分我是跟著 project 的文件去寫。

紀錄兩個小技巧：
如果是像 statment 這類可能有一行，也可能有多行的可以這樣寫
```
statements:
	statement {Trace("Reducing to statements\n");}
	| statement statements  {Trace("Reducing to statements\n");}
;
```
如果是可有可無的部分，像是函式宣告可能沒有參數，那可以這樣寫
```
optional_parameters:
	parameters {Trace("Reducing to optional_parameters\n");}
	|  {Trace("Reducing to optional_parameters\n");}
;
```

## sementic
為了讓 yacc 能夠判斷常見的錯誤，像是雖然符合文法，但該變數沒有宣告、型態不符等等，會需要額外寫東西輔助。

這時候就需要用到 `%union` ，讓 yylval 或 yytext 知道現在是甚麼型態。

我這樣寫
```
%union
{
	struct  e{
		int ival;
		float fval;
		char *sval;
		bool bval;
		int dtype;
	}Element;
}
```
以及將某些會需要紀錄參數型態、數值的token修改，最後搭配symbol table。

這部分直接實作或看code感覺比較容易，就不贅述。