title: ROT家族
date: 2026-1-20 
category: LEARN
cover: /passagecover/13.jpg
---

ROT函数，意味循环位移函数，用于将特定数据进行特定方向和位数的旋转位移

核心性质有三：

1.<font style="color:#DF2A3F;">循环性</font>：数据进行循环位移后不会发生溢出，而是从数据的低位重新进入

2.<font style="color:#DF2A3F;">可逆性</font>：进行相反方向相同位数的操作可以得到原数据

3.<font style="color:#DF2A3F;">保位性</font>：数据的位数和性质不会发生改变

ROT函数分为ROR(右移)和ROL(左移)，在ROR和ROL后面接上的数字就是要位移的位数

<font style="color:#DF2A3F;">核心公式</font>有二

```c
ROL(x, k) = (x << k) | (x >> (n - k))
```

```c
ROR(x, k) = (x >> k) | (x << (n - k))
```

buu的IgniteMe中有一个函数就涉及这种函数

![](/blog_essay_picture/ROT1.png)

这里涉及两种操作，分别是ROL，循环左移四位和位移后的结果向右位移一位。

这种ROT函数是在<font style="color:#DF2A3F;">二进制的基础</font>上进行处理(如下)

```c
原始: 1000 0000 0000 0111 0000 0000 0000 0000
ROL4: 0000 0000 0000 0111 0000 0000 0000 0000 1000
简化为32位: 0000 0000 0111 0000 0000 0000 0000 1000
十六进制: 0x00700008
十进制: 7340040
```

简洁明了地体现了ROT函数的三大性质和两大核心公式

然后我们再向右位移一位

```c
原始:  0x80070000 (1000 0000 0000 0111 0000 0000 0000 0000)
ROL4:  0x00700008 (0000 0000 0111 0000 0000 0000 0000 1000)
>>1:   0x00380004 (0000 0000 0011 1000 0000 0000 0000 0100)
```

由于这里返回的是16位数据，所以从低位到高位进行截取十六位可得

```c
0000 0000 0000 0100
```

那么可得最终结果为4

而ROT函数在调用的时候一般都是ROL/ROR(待处理数据 , 位移位数)，但大部分的题目都会标注出来。

========================================================

再来看看在web中的ROT的三目运算判断，取材于buu的login

题目逻辑如下

![](source/blog_essay_picture/ROT2.png)

可以看到其中已经显示出了密文和加密逻辑

```c
(c <= "Z" ? 90 : 122) >= (c = c.charCodeAt(0) + 13) ? c : c - 26);})
```

这里就是一个ROT13的字母表位移，倘若密文数据比Z小，就取90，否则取122，然后取密文的ASCII码+13，接着将90(或122)与后面的数据进行比较，如果符合条件那么密文数据不变，否则密文数据的ASCII-26得到新的数据，逆向脚本如下

```c
#include <bits/stdc++.h>
int main()
{
    char enc[37]="PyvragFvqrYbtvafNerRnfl@syner-ba.pbz";
    for(int i=0;i<37;i++)
    {
        if(isupper(enc[i]))
        {
            enc[i]=((enc[i]+13-'A')%26)+'A';
        }
        else if(islower(enc[i]))
        {
            enc[i]=((enc[i]+13-'a')%26)+'a';
        }
        else
        {
            enc[i]=enc[i];
        }
    }
    for(int i=0;i<37;i++)
    {
        printf("%c",enc[i]);
    }
    return 0;
}
```

这里的逆向是ROT家族里面<font style="color:#DF2A3F;">最特殊的</font>ROT13，我们来详细讲讲这个ROT13(因为其他的ROT都是可以直接计算的，唯独ROT13可以进行字母的计算——类似于凯撒)

ROT13有两层含义，一是常规的ROT，即将数据在二进制的层面上进行计算；二是<font style="color:#DF2A3F;">在字母的层面上(非字母不处理)</font>进行计算，相当于凯撒密码那种偏移，这道login就很好地体现了——可以看到原JS代码中的-13(类似于凯撒密码的偏移量)

ROT13的逆向逻辑很简单，就是将字母表分成两部分，每份十三个，然后按序号进行处理，最后加回来以保证最终结果的正确。类似于

```c
A B C D E F G H I J K L M 
N O P Q R S T U V W X Y Z

A ↔ N
B ↔ O
C ↔ P
...
M ↔ Z
```

在我的逆向脚本中的-'A'和-'a'和%26就是为了将字母化为从0~13的序号，然后在序号的基础上进行处理，最后再加回来，即

<font style="color:#DF2A3F;">将字母空间数组化</font>

以达到简化索引和计算的目的

ROT13还有很多变式，诸如ROT5，即只对数字进行处理；ROT18，即ROT13+ROT5；ROT47，即对所有可打印ASCII进行处理

```c
def rot5(text):
    result = []
    for char in text:
        if '0' <= char <= '9':
            result.append(chr((ord(char) - ord('0') + 5) % 10 + ord('0')))
        else:
            result.append(char)
    return ''.join(result)

# 示例：12345 → 67890
# 特点：数字0-9的ROT5也是自逆的
```

```c
def rot18(text):
    # 字母ROT13 + 数字ROT5
    result = []
    for char in text:
        if 'A' <= char <= 'Z':
            result.append(chr((ord(char) - ord('A') + 13) % 26 + ord('A')))
        elif 'a' <= char <= 'z':
            result.append(chr((ord(char) - ord('a') + 13) % 26 + ord('a')))
        elif '0' <= char <= '9':
            result.append(chr((ord(char) - ord('0') + 5) % 10 + ord('0')))
        else:
            result.append(char)
    return ''.join(result)

# 示例：Hello123 → Uryyb678
```

```c
def rot47(text):
    # ASCII 33-126 (94个可打印字符)
    result = []
    for char in text:
        if 33 <= ord(char) <= 126:
            # 47是94的一半，所以也是自逆的
            result.append(chr(33 + (ord(char) - 33 + 47) % 94))
        else:
            result.append(char)
    return ''.join(result)

# 示例：Hello! → w6==@
# 特点：可以加密符号、数字、大小写字母
```

至于其他更复杂的变式基本都是嵌套异或，base64等逻辑，掌握以上基本ROT是可以很快破解的。

<font style="color:#DF2A3F;">tips：值得注意的是ROT算法中其内部的某一元素是能提出来计算的，跟普通的算法逻辑逆向时一个方法，例如a=ROR(b , c)，则有b=ROL(a , c)</font>

