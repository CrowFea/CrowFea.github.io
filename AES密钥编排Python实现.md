---
title: AES密钥编排Python实现
date: 2019-04-16 18:38:45
categories:
    - 密码学
tags: 
    - 密码学
mp3: http://domain.com/awesome.mp3
cover: http://domain.com/awesome.jpg
---


### 引言
对想出AES的前辈大写的佩服，光是写了密钥编排我就写了一下午…这里把AES讲述一下，再把代码过程总结一下

### AES密钥扩展原理
AES加密算法涉及4种操作：字节替代（SubBytes）、行移位（ShiftRows）、列混淆（MixColumns）和轮密钥加（AddRoundKey）。
我们这里提及的是构造密钥的方法，密钥扩展，原理如下：
![](https://s2.ax1x.com/2019/04/26/EnZj1A.png)

### 代码构造
首先我们构造RotWord的函数，他的目的在于增加扩散性。这里实际上我们要干的事情就是完成一个置换，做的方式是一个字符串的拼接。注意python内部不能使用word[2:4]之类的方法，只能是word[2:]表示从第2个往后。
```python
def RotWord(word):
    return word[2:]+word[:2]
``
接下来构造SubBytes方法。这个方法实现的就是S盒。我们这里的方法是输入字符串，转换为相应的十六进制的字符。Python里不能使用switch方法，只能用字典进行搜索。
对于字典来说，我们可以用key来搜索value。如果没搜索到就默认为None。
```python
numbers = {
         '0' : '0000','1' : '0001','2' : '0010','3' : '0011','4' : '0100','5' : '0101','6' : '0110',
         '7' : '0111','8' : '1000','9' : '1001','A' : '1010','B' : '1011','C' : '1100','D' : '1101',
         'E' : '1110','F' : '1111'
    }

def hex2Bin(key):
    return numbers.get(key[0], None)+numbers.get(key[1], None)
```
当然我们这里还需要的方法，要从value来搜索key，要自定义一个get_key方法：
```python
def get_key (dict, value):
    for k, v in dict.items():
        if v == value:
            return k
```
接下来实现SubBytes方法，注意原方法中，要先求key在域中的乘法逆，在进行计算。但是计算机内的0指的是第一个元素，所以不必求逆。
```python
def SubBytes(key):
    #print(key)
    key=hex2Bin(key)
    c='11000110'
    b=''
    for i in range(8):
        temp=(int(key[i])+int(key[(i+4)%8])+int(key[(i+5)%8])+int(key[(i+6)%8])+int(key[(i+7)%8])+int(c[i]))%2
       # print(temp)
        temp=str(temp)
        b+=temp
    b=b[::-1]
    #print(b)
    return get_key(numbers,b[:4])+get_key(numbers,b[4:])
```
再者进行构造SubWord方法，这个方法的目的在于混淆。使用S盒对原序列进行S置换。
```python
def SubWord(word):
    temp=''
    i=0
    while i<len(word):
        temp+=SubBytes(word[i]+word[i+1])
        i+=2
    return temp
    ```
最后我们来实现KeyExpansion方法。这里注意在循环中，temp是一个字符串，binnum是一个二进制数。注意不能将二者混用，因为不是每一次temp的值是更新的，在实践中就会出现问题。
```python
def KeyExpansion(m):
    RCon=['01000000','02000000','04000000','08000000','10000000','20000000','40000000','80000000','1B000000','36000000']
    i=0
    key=[]
    while i<16:
        key.append(m[i*2]+m[i*2+1])
        i+=1
    w=[]
    for i in range(0,4):
        w.append(key[4*i]+key[4*i+1]+key[4*i+2]+key[4*i+3])
        #print(w[i])
    #temp=SubWord(RotWord(w[0]))
    #temp=hex2Bin(temp)
    #temp=w[0]
    #temp=int(hex2Bin8(SubWord(RotWord(temp))),2)|int(hex2Bin8(RCon[i//4-1]),2)
    for i in range(4,44):
        temp=w[i-1]
        
        binnum=int(hex2Bin8(temp),2)
        if(i%4==0):
            binnum=int(hex2Bin8(SubWord(RotWord(temp))),2)^int(hex2Bin8(RCon[i//4-1]),2)
        wapp=hex((int(hex2Bin8(w[i-4]),2)^binnum))
        wapp=wapp[2:].upper()
        zernum=8-len(wapp)
        while zernum:
            wapp='0'+wapp
            zernum-=1
        w.append(wapp)
    return w
    ```
最后我们将全部的代码放在下面：
```python
def RotWord(word):
    return word[2:]+word[:2]

numbers = {
         '0' : '0000','1' : '0001','2' : '0010','3' : '0011','4' : '0100','5' : '0101','6' : '0110',
         '7' : '0111','8' : '1000','9' : '1001','A' : '1010','B' : '1011','C' : '1100','D' : '1101',
         'E' : '1110','F' : '1111'
    }

def get_key (dict, value):
    for k, v in dict.items():
        if v == value:
            return k

def hex2Bin(key):
    return numbers.get(key[0], None)+numbers.get(key[1], None)

def hex2Bin8(key):
    str=''
    zernum=8-len(key)
    while zernum:
        key='0'+key
        zernum-=1
    for i in range(8):
        str+=numbers.get(key[i], None)
    #print(str)
    return str
    
def SubBytes(key):
    #print(key)
    key=hex2Bin(key)
    c='11000110'
    b=''
    for i in range(8):
        temp=(int(key[i])+int(key[(i+4)%8])+int(key[(i+5)%8])+int(key[(i+6)%8])+int(key[(i+7)%8])+int(c[i]))%2
       # print(temp)
        temp=str(temp)
        b+=temp
    b=b[::-1]
    #print(b)
    return get_key(numbers,b[:4])+get_key(numbers,b[4:])

def SubWord(word):
    temp=''
    i=0
    while i<len(word):
        temp+=SubBytes(word[i]+word[i+1])
        i+=2
    return temp

def KeyExpansion(m):
    RCon=['01000000','02000000','04000000','08000000','10000000','20000000','40000000','80000000','1B000000','36000000']
    i=0
    key=[]
    while i<16:
        key.append(m[i*2]+m[i*2+1])
        i+=1
    w=[]
    for i in range(0,4):
        w.append(key[4*i]+key[4*i+1]+key[4*i+2]+key[4*i+3])
        #print(w[i])
    #temp=SubWord(RotWord(w[0]))
    #temp=hex2Bin(temp)
    #temp=w[0]
    #temp=int(hex2Bin8(SubWord(RotWord(temp))),2)|int(hex2Bin8(RCon[i//4-1]),2)
    for i in range(4,44):
        temp=w[i-1]
        
        binnum=int(hex2Bin8(temp),2)
        if(i%4==0):
            binnum=int(hex2Bin8(SubWord(RotWord(temp))),2)^int(hex2Bin8(RCon[i//4-1]),2)
        wapp=hex((int(hex2Bin8(w[i-4]),2)^binnum))
        wapp=wapp[2:].upper()
        zernum=8-len(wapp)
        while zernum:
            wapp='0'+wapp
            zernum-=1
        w.append(wapp)
    return w
m='2B7E151628AED2A6ABF7158809CF4F3C'
key=KeyExpansion(m)
key
```
最后的结果如下：

![](https://s2.ax1x.com/2019/04/26/EnZ7TO.png)