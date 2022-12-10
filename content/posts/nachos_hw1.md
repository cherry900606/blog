---
title: "Nachos hw1"
date: 2021-10-13T10:48:33+08:00
draft: false
toc: true
images:
tags: 
  - 作業系統
---

## 安裝

```
sudo apt install csh g++
git clone https://gitlab.com/connlabtw21/nachos2021.git
cd NachOS2021
sudo cp -r usr /
cd code
make
```
make下去後會噴出很多訊息，這是正常的，等待幾分鐘後即結束。

## Testing

測試是否下載成功。

```
cd userprog
./nachos -e ../test/test1
```
![](https://i.imgur.com/sExYdU0.png)

## 範例──實作system call

1. 在 /code/userprog/syscall.h 加入 `#define SC_Example 13`
2. 在 /code/userprog/syscall.h 加入 `void Example(int number);`
3. 在 /code/userprog/exception.cc 加入
```
case SC_Example:
    val=kernel->machine->ReadRegister(4);
    cout<<"Example value:"<<val<<endl;
    return;
```
4. 在 /code/test/start.s 加入
```
    .globl Example
    .ent Example
Example:
    addiu   $2,$0,SC_Example
    syscall
    j   $31
    .end Example
```
5. 在 /code/test 新增 example.c
```
#include "syscall.h"
main(){
    int n;
    for(n=1;n<5;n++)
        Example(n);
}
```
6. 修改 /code/test/Makefile
![](https://i.imgur.com/2XcVrW8.png)

7. 回到 /code 目錄底下， 輸入 `make` 指令recomplie
8. 
```
cd userprog
./nachos -e ../test/example
```
![](https://i.imgur.com/AtrB6tr.png)

## 實作ThreadYield

`ThreadYield()` 顧名思義，就是強制讓CPU去執行下一個thread。

根據助教提供的提示，到nachos的code翻翻看，會看到thread.h這個檔案，裡面定義了與thread相關的一些功能。其中就有看起來能用到的`Yield()`。

現在的問題是: 該怎麼調用這個函式?回到 `exception.cc` 中，發現其他幾個case如果要呼叫的話，都是用 `kernel->xx->yy()` 的形式。以SC_PrintInt為例，它有一行 `kernel->machine->ReadRegister()`。如果看machine.h，會發現底下也有這麼一個函式。

既然如此，那就模仿這個寫法試試看好了。

照著範例的步驟操作，其中1、2、4、6步已經被寫好了。不過記得，要先找到是哪個變數掌管當前的thread，才能用它來呼叫yield。因此需要trace code。

在main.cc中可以觀察到kernel被Intialize，因此回過頭檢視kernel.h，恰好在這裡發現了 Thread *currentThread 這行，同時註解也說明這個pointer掌握CPU，看來就是關鍵。

在 /code/userprog/exception.cc 加入
```
case SC_ThreadYield:
    cout<<"Call ThreadYield"<<endl;
    kernel->currentThread->Yield();
    return
```

修改 /code/test/example.c
```
#include "syscall.h"
main(){
    PrintInt(88);
    ThreadYield();
    PrintInt(99);
}
```
重新recompile後，在 ./code/userprog 底下執行指令 `./nachos -e ../test/example -e ../test/test1`，結果如下:
![](https://i.imgur.com/Odz67TL.png)

## 實作Log

題目要求每當呼叫Log時，要把字元參數以特定格式寫入NachOS.log這個檔案。此外，若遇到特定的字母，就要改寫"error"。

整理後可知需要能:
1. 接收字元參數
2. 判斷字元
3. 建立檔案並以特定名稱命名(若檔案不存在)
4. 寫入\打開檔案

前兩項要求不難，基本上跟範例沒兩樣，只是從整數改成字元而已。後兩項就需要進一步思考如何達成。

上一個練習需要探索/code/thread底下的文件，這個則可以從/code/filesys找到線索。

在filesys.h中，發現裡面的Create(char*)、Open(char*)好像能用來建立檔案。同時，openfile.h中的WriteAt(char*,int,int)也是需要的。

在 syscall.h 中加入 `#define SC_Log 14` 跟 `void Log(char);`

在 start.s 加入
```
        .globl Log
        .ent Log
Log:
        addiu   $2,$0,SC_Log
        syscall
        j       $31
        .end    Log
```

在 exception.cc 加入
```
		case SC_Log:
			//cout<<"call log"<<endl;
			file = kernel->fileSystem->Open((char*)"NachOS.log");
			if(!file){
				cout<<"Create log file"<<endl;
				kernel->fileSystem->Create((char *)"NachOS.log",0);
				file = kernel->fileSystem->Open((char*)"NachOS.log");
			}
			c = kernel->machine->ReadRegister(4);
			if(c=='t'||c=='T'){
				cout<<"[B10832019_Log]error"<<endl;
				file->WriteAt((char*)"[B10832019_Log]error\n",21,file->Length());
			}
			else{
				cout<<"[B10832019_Log]"<<c<<endl;
				file->WriteAt((char*)"[B10832019_Log]",15,file->Length());
				file->WriteAt(&c,1,file->Length());
				file->WriteAt((char*)"\n",1,file->Length());
			}
			return;
```

新增 mytest.c
```c
#include "syscall.h"
main(){
	Log('a');
	Log('t');
	Log('b');
	Log('T');
}
```

修改Makefile
```
all: halt shell matmult sort test1 test2 example mytest 

(中略)

mytest: mytest.o start.o
	$(LD) $(LDFLAGS) start.o mytest.o -o mytest.coff
	../bin/coff2noff mytest.coff mytest
```
![](https://i.imgur.com/GsBRuBW.png)

![](https://i.imgur.com/9A1O42z.png)

## 遇到的錯誤訊息

1. 沒寫SC_xx 會有error
2. exception.cc  variable error in switch case

好像是case中若要宣告變數，需要在{}中，不然就是在外面宣告

3. tab vs 空格 in makefile
空格會報錯。

4. 網路文件大小寫未必正確 ex: openfile的length()

5. bool Create(char *name, int initialSize)

實作跟宣告的參數不一樣，一直有error。