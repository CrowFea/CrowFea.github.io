---
title: C++避免TLE问题
date: 2019-07-13 20:02:45
categories:
    - C++
tags: 
    - C++
mp3: http://domain.com/awesome.mp3
cover: http://domain.com/awesome.jpg
---

C++采用的是输入输出流的方式，但是cin，cout速度并不快，造成在一些题目中经常有TLE的情况。

但实际上不是因为cin慢！！！是因为c++兼容c的方法，将cin的流绑定到scanf中，所以会变慢。
<!--more-->
可以将这两个流解绑1
```c++
std::ios::sync_with_stdio(false);
std::cin.tie(0);
sync_with_stdio
```
这个函数是一个“是否兼容stdio”的开关，C++为了兼容C，保证程序在使用了std::printf和std::cout的时候不发生混乱，将输出流绑到了一起。

若出现数据集超大造成 cin TLE的情况。这时候大部分人（包括原来我也是）认为这是cin的效率不及scanf的错。其实像上文所说，这只是C++为了兼容而采取的保守措施。我们可以在IO之前将stdio解除绑定，这样做了之后要注意不要同时混用cout和printf之类。

在默认的情况下cin绑定的是cout，每次执行 << 操作符的时候都要调用flush，这样会增加IO负担。可以通过tie(0)（0表示NULL）来解除cin与cout的绑定，进一步加快执行效率。

我们这里看一看tie方法

tie是将两个stream绑定的函数，空参数的话返回当前的输出流指针。
```c++
#include <iostream>
#include <fstream>
 
int main(int argc, char *argv[])
{
	std::ostream *prevstr;
	std::ofstream ofs;
	ofs.open("test.txt");
 
	std::cout << "tie example:\n";	// 直接输出到屏幕
 
	*std::cin.tie() << "This is inserted into cout\n";	// 空参数调用返回默认的output stream，也就是cout
	prevstr = std::cin.tie(&ofs);						// cin绑定ofs，返回原来的output stream
	*std::cin.tie() << "This is inserted into the file\n";	// ofs，输出到文件
	std::cin.tie(prevstr);									// 恢复
 
	ofs.close();
	system("pause");
	return 0;
}
```
输出：
```
tie example:
This is inserted into cout
```

同时当前目录下的test.txt输出：
```
This is inserted into the file
```