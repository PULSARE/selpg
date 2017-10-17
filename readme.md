# Golang - selpg 
### 简介
· 该实用程序从标准输入或从作为命令行参数给出的文件名读取文本输入。它允许用户指定来自该输入并随后将被输出的页面范围。除了包含Linux实用程序现实的示例外，本文还有以下特性：
- 它用实例说明了 Linux 软件开发环境的能力。
- 它演示了对一些系统调用和 C 库函数的适当使用，其中包括 fopen、fclose、access、setvbuf、perror、strerror 和 popen。
- 它实现了打算用于通用目的的实用程序（而不是一次性程序）所应有的那种彻底的错误检查。
- 它对潜在的问题提出警告，如在 C 中编程时可能出现的缓冲区溢出，并就如何预防这些问题提供了建议。
- 它演示了如何进行手工编码的命令行参数解析。
- 它演示了如何在管道中以及在输入、输出和错误流重定向的情况下使用该工具。


----------


### 命令行准则
    $ command mandatory_opts [ optional_opts ] [ other_args ]


----------


### 参数处理
1. “-sNumber”和“-eNumber”强制选项：selpg要求用户用两个命令行参数“-sNumber”（例如，“-s10”表示从第10页开始）和“-eNumber”（例如，“-e20”表示在第20页结束）指定要抽取的页面范围的起始页和结束页。
2. “-lNumber”和“-f”可选选项：selpg可以处理两种输入文本：页行数固定的文本和页数由ASCII码确定的换页字符定界的文本。
3. “-dDestination”可选选项：selpg还允许用户使用“-dDestination”选项将选定的页直接发送至打印机。


----------


### 输入处理    
· selpg 通过以下方法记住当前页号：如果输入是每页行数固定的，则 selpg 统计新行数，直到达到页长度后增加页计数器。如果输入是换页定界的，则 selpg 改为统计换页符。这两种情况下，只要页计数器的值在起始页和结束页之间这一条件保持为真，selpg就会输出文本（逐行或逐字）。当那个条件为假（也就是说，页计数器的值小于起始页或大于结束页）时，则 selpg 不再写任何输出。


----------


### 数据结构
　　```type selpgArgs struct{
　　　　　	startPage int
　　　　　	endPage int
　　　　　	inFilename string
　　　　　	pageLen int
　　　　　	printDest string
　　　　　	pageType int
　　　}```

* pageLen 表示每页的行数，可以被“-l”指令修改
* pageType 表示页的类型，l为确定页行数的文本，f为页数由ASCII码确定的换页字符定界的文本

函数分别为main函数，processArgs函数和processProceed函数。processProceed用来读写文件和判断文件的分页类型（固定或者根据换页符分页），processArgs用来判断指令的具体要求和传递相应参数。

----------


### 测试
* ```$ selpg -s1 -e2 -l10 input.txt```
将第一页和第二页输出。此时每一页有10行。因此输出为line1~line20
```
//input.txt中的，屏幕显示的
line1
line2
line3
line4
line5
line6
line7
line8
line9
line10
line11
line12
line13
line14
line15
line16
line17
line18
line19
line20
```  


----------
*  ```$ selpg -s1 -e2 < input.txt```
该命令与示例 1 所做的工作相同，但在本例中，selpg 读取标准输入，而标准输入已被 shell／内核重定向为来自“input_file”而不是显式命名的文件名参数。


----------

* ```selpg -s1 -e4 -l10 input.txt >output.txt```
selpg 将第 1 页到第 4 页写至标准输出；标准输出被 shell／内核重定向至“output.txt”。也就是输出前40行至output.txt。原先output.txt只有44~80行，现在为line1~line80.
```
//output.txt中的，屏幕不显示，只执行操作
line1
line2
line3
line4
line5
line6
line7
line8
line9
line10
line11
line12
line13
line14
line15
line16
line17
line18
line19
line20
```  

* ```selpg -s10 -e20 input_file 2>error_file```
不符合标准的信息将被输出至错误信息文件error.txt。

* ```selpg -s1 -e2 -f input.txt```
假定页由换页符定界。第 1 页到第 2 页被写至 selpg 的标准输出（屏幕）。
```
//output.txt中的，屏幕显示
line1
line2
line3
line4
line5 // /f换页符
line6
line7
line8
line9
line10
line11
line12
line13
line14  // /f换页符
```
* ```selpg -s1 -e2 -dlp1 input_file```
输出错误信息，因为没有打印机。
