---
title: "Nand2tetris 筆記"
date: 2021-08-07T20:47:40+08:00
draft: false
toc: true
images:
tags: 
  - note
  - coursera
---

課名: [Build a Modern Computer from First Principles: Nand to Tetris Part I](https://www.coursera.org/learn/build-a-computer?)

我的github作業: [連結](https://github.com/cherry900606/nand2tetris)
## project0
進入課程網站下載課程教材(https://www.nand2tetris.org/software)，點選Download the Nand2tetris Software Suite。
裡面有兩個資料夾projects跟tools，選擇projects/00，把裡面的file.txt壓縮成project0.zip，上傳到作業繳交區即可。

## project1
實作projects/01裡面的15個.hdl檔，把它們打包成project1.zip，上傳到作業繳交區。

使用tools資料夾裡面的HardwareSimulator.bat，可以先用projects/demo裡面的Xor.hdl測試功能。

如果語法不正確會無法成功load chip，錯誤訊息會顯示在Hardware Simulator下方，值得注意的是有時候該區域會被擋住，如圖紅框所示:
![](https://i.imgur.com/g3oKsGE.png)
如果實作結果與比對不符，會在比對過程中止，同樣會在紅框顯示第幾筆測試資料有誤。

如果自己測試.tst檔都沒問題，那麼繳交上去也能拿到全部分數。

可使用notepad++實作chip。過程中若要用到其他的chip，可以參考Hack Chip Set(https://drive.google.com/file/d/1IsDnH0t7q_Im491LQ7_5_ajV0CokRbwR/view)提供的API。

## project2

要怎麼用HDL實現if-else語法呢?可以用上週作業寫過的Mux16。

Mux16會有兩個16-bit input，一個是原本的值，一個是符合條件後會變成的值，sel則是判斷的依據。

基本上可以想成:
```c
if(sel)
    then b
else
    then a
```

zr要怎麼判斷呢?可以用上週作業的Or8Way兩次，如果兩個結果都為0就代表out為零。

幾個實作ALU會需要的語法:

```c
b[0..15]=false // 輸入的第0~15 bit都為0
```
```c
out[0..7]=low8, out[8..15]=high8, out[15]=signBit
// 把out的某些bit另外拿出來，之後要用來做其他判斷
```
## project3
之前幾次作業都是combinational circuit，本周的sequential circuit會有out又接回來in的情形。

in=out是不被允許的，可能會有類似:「circle in connections」的錯誤訊息。需要另外拉一條線接回in，像是:out=feedback, in=feedback之類的。

Bit Chip會需要用到bulit-in 的DFF。

RAM要先用DMux把load訊號展開給更小的單元，最後在把資料Mux過。丟進去的address長度參考給定的規格，都是幾個高位元bit自己DMux&Mux會用到，剩下來的是給更小單元用的。

PC的實作順序，看coursera討論區有人提到似乎要以:inc, load, reset的順去去寫，不確定是否必要。最後要用Register來存資料，不能直接在reset的Mux16就寫out=out。討論區有助教留言說:「A common problem with this chip is that out=out should be from the Register chip.」([連結](https://www.coursera.org/learn/build-a-computer/discussions/forums/aHveBSj3Eea8jw6UvTi2Tw/threads/tuzNlXAyEeuL8hI1GgNn1w))

作業也是把a, b兩個資料夾的.hdl都壓縮成project3.zip上傳。

## project4
{{< figure src="https://i.imgur.com/Uu8ob9T.png" width="70%" height="70%" >}}

{{< figure src="https://i.imgur.com/3Tg5tw6.png" width="70%" height="70%" >}}

{{< figure src="https://i.imgur.com/G4bJ6Is.png" width="70%" height="70%" >}}

{{< figure src="https://i.imgur.com/z5cWgv9.png" width="70%" height="70%" >}}

D: data register
A: address / data register
M: the currently selected memory register: M = RAM[A]

結束程式:
```python
(END)
    @END
    0;JMP
```

@+lowercase letter = variable
@+uppercase letter = label

virtual registers: R0, R1, R2......->大寫！

作業部分:
SCREEN大小為8192
用一個while(true)的loop去跑，有按就跑到WHITE(設值為0)，不然就跑到BLACK(設值為1)
測fill記得要選no amination，不然怎麼跑都不會有結果。

## project5
### Memory
![](https://i.imgur.com/FHH31Xi.png)

16384 -> 0100 0000 0000 0000
24576 -> 0110 0000 0000 0000
觀察可知，要區分RAM, Screen與Keyboard，可由13~14bit判斷:
00, 01 -> RAM
10 -> Screen
11 -> Keyboard
跟project3很像，先DMux分配load，之後再Mux
RAM, Screen, Keyboard的規格可以查Hack Chip Set，address根據容量大小而異。

測試時注意:
> To anyone facing the same problem... on the Hardware Simulator user interface, right above where you see the script executing the tests, there are three drop down boxes. The one furthest to the right which is labeled "View" is probably currently set to "Script." Click the drop down and select "Screen," and you will see an interface that has a keyboard icon. Click it and then hit the corresponding key to complete the test. [連結](https://www.coursera.org/learn/build-a-computer/discussions/weeks/5/threads/VGOSrER6EeezGhJSvzqk_g)

**CPU**


這部分最困難的是自行分析c，也就是各個元件的控制訊號該怎麼生成。



首先先從**Mux16**的c開始。我們知道instruction分為A instruction與C instruction，前者是把數字存放進A register，後者是進行運算。因此此處若為A instruction，Mux16就選擇instruction；若為C instruction，Mux16就選擇ALU output。

判斷指令直接看instruction[15]。

同理**A register**的c也是看instruction[15]。

**writeM**訊號指的是需不需要write back to memory。


觀察表格後，會發現d1是關鍵。因此若為C instruction且d1為1，則writeM為1，反之為0。

**D register**同樣觀察表格，發現與d2有關。因此若為C instruction且d2為1，則c為1，反之為0。

第二個**Mux16**是要選擇A register output或者inM。A instruction選擇前者，C instruction選擇後者這點應該不難理解。這裡需要參考下圖，發現要選A還是M由a，也就是instruction[12]控制。


**ALU**的c就是C instruction中c的部分。由第 11 bit 依序填入到第 6 bit即可。

最後是**PC**。沒想法的話就回去看課程網站提供的chip API或是chip的描述:

reset訊號已經有了，inc恆為true，怎麼生成load才是問題。當load==1，代表要把in讀進來，也就是說要jump。那甚麼時候要jump? 當然是條件有達成的時候。

觀察表格會發現，j1代表out < 0條件為真要jump，j2代表out=0條件為真要jump，j3代表out>0條件為真要jump。因此就看: (j1&ng) || (j2&zr) || (j3&(! (ng || zr)) ) 。

參考:
https://zhuanlan.zhihu.com/p/343850636

## project6
依照投影片提供的API去寫，最後再接起來就好。
![](https://i.imgur.com/Bn4kvns.png)
![](https://i.imgur.com/5JVoDrp.png)
![](https://i.imgur.com/jJSFbm8.png)

程式邏輯:
![](https://i.imgur.com/Y1aK4vu.png)

pre-defined symbols:
![](https://i.imgur.com/lFnZHtW.png)

如果另外三個.hack比對都沒問題，那麼Pong.asm可以不用比對直接上傳，不然很浪費時間。