---
title: "Compiler 作業一：lex"
date: 2022-05-14T11:21:12+08:00
draft: false
toc: true
---

## lex 的基本架構
```lex
%{
// part 1

%}
// part 2

%%
// part 3

%%
// part 4

```

* part1 可以 include 用到的 library
* part2 用 regular grammar 定義 token
* part3 寫對 token 要做什麼動作
* part4 是 optional，可以自定義 main()並呼叫 yylex()，也可以不寫

## part1
```
#define LIST     strcat(buf,yytext)
#define token(t) {LIST; printf("<%s>\n",#t); return t;};
#define tokenInteger(t,i) {LIST; printf("<%s:%d>\n",#t,i);}
#define tokenString(t,s) {LIST; printf("<%s:%s>\n",#t,s);}

char buf[MAX_LINE_LENG];

```
先 define 一些東西以減少重複性 code。
另外，原本老師提供的檔案是沒有 #t， 只有 t而已，但我在本地端用 cygwin 跑不過才改的。

## part2
```
digit [0-9]
integer {digit}+
letter [a-zA-Z]
identifier {letter}({digit}|{letter})*
real [-+]?{integer}\.{integer}?([Ee][-+]?{digit})?
string \"(\"\"|.)*\"
```
ID必須是以字母開頭，後面由數字或字母的 closure 組成。
字串則是以 " 開頭跟結尾，中間如果有出現 "，那就必須多加一個 " 作為標示。例如 "aa""bb" 會被當作 aa"bb。

## part3
如果是operator，像是+-*%之類的token，就直接寫出他們的樣子，並寫出相對應的動作即可。
像是：
```
"+"     {token('+');}
"-"     {token('-');}
"*"     {token('*');}
"/"     {token('/');}
"%"     {token('%');}
```

如果是 keyword，那就寫出該字並寫出相對應的動作。作業有提到大小寫字母的組合都要能辨識。
例：
```
[bB][oO][oO][lL]                {token(BOOL);}
[bB][rR][eE][aA][kK]    {token(BREAK);}
[cC][hH][aA][rR]                {token(CHAR);}
[cC][aA][sS][eE]                {token(CASE);}
[cC][lL][aA][sS][sS]    {token(CLASS);}
```

再來就是在 part2 定義過規則的那些token。

為了讓 lex 除了辨識是哪種 token 外，也能得到其值，要用之前寫好的token()系列。
例如：
```
{integer} {
        tokenString(INTEGER, yytext);
 }
 {identifier} {
        tokenString(IDENTIFIER, yytext);
}
```

至於 string 的處理又再麻煩一點，因為要把前後的 " 拿掉，並且中間若有 "" 也要處理。
```
{string} {
        char s[MAX_LINE_LENG];
        int index = 0;
        for(int i=1;i<yyleng-1;i++)
        {
                if(yytext[i]=='"')
                        i+=1;
                s[index++]=yytext[i];
        }
        s[index]='\0';
        tokenString(STRING, s);
}
```
我原本沒有加上 `s[index]='\0';`，在 server 上執行時字串一直輸出多餘內容，找好久才發現 bug。

## part4
```
main()
{
    yylex();
    // 視情況加進想要的東西
}
```

## Others
### comments
分為單行註解與多行註解。

首先要在 part2 加上 `%x CMNT`

單行註解
`"//"[^\n]*      {LIST;}`

多行註解
```
"/*"    {
        LIST;
        BEGIN CMNT;
}
<CMNT>\n        {
        LIST;
		        printf("%d: %s", linenum++, buf);
        buf[0] = '\0';
}
<CMNT>. {LIST;}
<CMNT>"*/"      {
        LIST;
        BEGIN INITIAL;
}
```

### symbol table
如果有寫 symbol table，就在辨識到 ID 的時候 insert 進去。

可以把 part4 改一下，最後把 symbol table 給 dump 出來。