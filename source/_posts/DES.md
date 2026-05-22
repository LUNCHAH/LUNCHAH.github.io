title: DES
date: 2026-01-28
category: LEARN
cover: https://s41.ax1x.com/2026/05/22/pmp94fO.jpg
---

## 前言
DES是一种非常强大的加密逻辑，其代码复杂性与数据混淆性堪称一绝，尽管整体来说仍然是较为线性的算法，虽然出题出得少但是我们仍然要进行一定的了解，毕竟在VM的题或者一些密码题当中仍然有可能出现。

与TEA家族相同，DES也分为三种模式：ECB，CBC，CTR，其实现逻辑大同小异；与AES不同，DES的密钥不会分为128，192和256位，就是固定的8字节密钥。

三种模式一览

1.ECB（电子密码本模式）：明文和密文必须符合某一长度，如果不足时就会自动填充。明文和密文在加解密过程中被分为数块，在加解密的执行流中每一块数据的加解密互不干扰并且可以并行运行，即使某一块发生错误也不会影响到其他的数据块，只会影响到自身的加解密。

2.CBC（密码分组链接模式）：	明文和密文必须符合某一长度或其倍数，如果不足时就会自动填充。加密时程序会首先随机生成一个初始向量与第一组密文异或，后面的所有数据都会与前一个数据相异或，如果发生错误就会进而影响到整一个程序出错。

3.CTR（计数器模式）：这种模式下引入了一个计数器，计数器的值在执行过程中是绝对绝对绝对不可能重复的，意味着每一组数据对应计数器中一个特定的值，这个值会与数据进行异或用来使数据更为混淆。但这种模式可以处理任意长度的数据，无需关注数据长度，也无需对数据进行填充。

![](/blog_essay_picture/DES/1.png)

## DES的核心组件
### 总览
无论在哪一种模式下的DES它们的盒与表都是不会发生变化的(故曰核心组件)，各种表与盒如下

```c
//定义IP表
const int IP[64]=
{
    58, 50, 42, 34, 26, 18, 10, 2,
    60, 52, 44, 36, 28, 20, 12, 4,
    62, 54, 46, 38, 30, 22, 14, 6,
    64, 56, 48, 40, 32, 24, 16, 8,
    57, 49, 41, 33, 25, 17,  9, 1,
    59, 51, 43, 35, 27, 19, 11, 3,
    61, 53, 45, 37, 29, 21, 13, 5,
    63, 55, 47, 39, 31, 23, 15, 7
    };

//定义IP逆表
const int FP[64]=
{
    40, 8, 48, 16, 56, 24, 64, 32,
    39, 7, 47, 15, 55, 23, 63, 31,
    38, 6, 46, 14, 54, 22, 62, 30,
    37, 5, 45, 13, 53, 21, 61, 29,
    36, 4, 44, 12, 52, 20, 60, 28,
    35, 3, 43, 11, 51, 19, 59, 27,
    34, 2, 42, 10, 50, 18, 58, 26,
    33, 1, 41,  9, 49, 17, 57, 25
    };

//定义置换的E表
const int E[48]=
{
    32,  1,  2,  3,  4,  5,
    4,  5,  6,  7,  8,  9,
    8,  9, 10, 11, 12, 13,
    12, 13, 14, 15, 16, 17,
    16, 17, 18, 19, 20, 21,
    20, 21, 22, 23, 24, 25,
    24, 25, 26, 27, 28, 29,
    28, 29, 30, 31, 32,  1
    };

//定义P盒置换表
const int P[32]=
{
    16,  7, 20, 21,
    29, 12, 28, 17,
    1, 15, 23, 26,
    5, 18, 31, 10,
    2,  8, 24, 14,
    32, 27,  3,  9,
    19, 13, 30,  6,
    22, 11,  4, 25
    };

/*以上是对密文的处理
接下来定义的PC1和PC2是对密钥的处理*/
const int PC1[56]=
{
    57, 49, 41, 33, 25, 17,  9,
    1, 58, 50, 42, 34, 26, 18,
    10,  2, 59, 51, 43, 35, 27,
    19, 11,  3, 60, 52, 44, 36,
    63, 55, 47, 39, 31, 23, 15,
    7, 62, 54, 46, 38, 30, 22,
    14,  6, 61, 53, 45, 37, 29,
    21, 13,  5, 28, 20, 12,  4
    };

const int PC2[48]=
{
    14, 17, 11, 24,  1,  5,
    3, 28, 15,  6, 21, 10,
    23, 19, 12,  4, 26,  8,
    16,  7, 27, 20, 13,  2,
    41, 52, 31, 37, 47, 55,
    30, 40, 51, 45, 33, 48,
    44, 49, 39, 56, 34, 53,
    46, 42, 50, 36, 29, 32
    };

//定义左位移数表
const int left_shift[16]=
{1, 1, 2, 2, 2, 2, 2, 2, 1, 2, 2, 2, 2, 2, 2, 1};

//开始定义8个S盒
const uint8_t S[8][4][16]=
{
    {
        {14,4,13,1,2,15,11,8,3,10,6,12,5,9,0,7},
        {0,15,7,4,14,2,13,1,10,6,12,11,9,5,3,8},
        {4,1,14,8,13,6,2,11,15,12,9,7,3,10,5,0},
        {15,12,8,2,4,9,1,7,5,11,3,14,10,0,6,13}
    },
    {
        {15,1,8,14,6,11,3,4,9,7,2,13,12,0,5,10},
        {3,13,4,7,15,2,8,14,12,0,1,10,6,9,11,5},
        {0,14,7,11,10,4,13,1,5,8,12,6,9,3,2,15},
        {13,8,10,1,3,15,4,2,11,6,7,12,0,5,14,9}
    },
    {
        {10,0,9,14,6,3,15,5,1,13,12,7,11,4,2,8},
        {13,7,0,9,3,4,6,10,2,8,5,14,12,11,15,1},
        {13,6,4,9,8,15,3,0,11,1,2,12,5,10,14,7},
        {1,10,13,0,6,9,8,7,4,15,14,3,11,5,2,12}
    },
    {
        {7,13,14,3,0,6,9,10,1,2,8,5,11,12,4,15},
        {13,8,11,5,6,15,0,3,4,7,2,12,1,10,14,9},
        {10,6,9,0,12,11,7,13,15,1,3,14,5,2,8,4},
        {3,15,0,6,10,1,13,8,9,4,5,11,12,7,2,14}
    },
    {
        {2,12,4,1,7,10,11,6,8,5,3,15,13,0,14,9},
        {14,11,2,12,4,7,13,1,5,0,15,10,3,9,8,6},
        {4,2,1,11,10,13,7,8,15,9,12,5,6,3,0,14},
        {11,8,12,7,1,14,2,13,6,15,0,9,10,4,5,3}
    },
    {
        {12,1,10,15,9,2,6,8,0,13,3,4,14,7,5,11},
        {10,15,4,2,7,12,9,5,6,1,13,14,0,11,3,8},
        {9,14,15,5,2,8,12,3,7,0,4,10,1,13,11,6},
        {4,3,2,12,9,5,15,10,11,14,1,7,6,0,8,13}
    },
    {
        {4,11,2,14,15,0,8,13,3,12,9,7,5,10,6,1},
        {13,0,11,7,4,9,1,10,14,3,5,12,2,15,8,6},
        {1,4,11,13,12,3,7,14,10,15,6,8,0,5,9,2},
        {6,11,13,8,1,4,10,7,9,5,0,15,14,2,3,12}
    },
    {
        {13,2,8,4,6,15,11,1,10,9,3,14,5,0,12,7},
        {1,15,13,8,10,3,7,4,12,5,6,11,0,14,9,2},
        {7,11,4,1,9,12,14,2,0,6,10,13,15,3,5,8},
        {2,1,14,7,4,10,8,13,15,12,9,0,3,5,6,11}
    }
};
```

### IP表**<font style="color:#000000;">(Initial Permutation-初始置换)</font>**
```c
const int IP[64]=
{
    58, 50, 42, 34, 26, 18, 10, 2,
    60, 52, 44, 36, 28, 20, 12, 4,
    62, 54, 46, 38, 30, 22, 14, 6,
    64, 56, 48, 40, 32, 24, 16, 8,
    57, 49, 41, 33, 25, 17,  9, 1,
    59, 51, 43, 35, 27, 19, 11, 3,
    61, 53, 45, 37, 29, 21, 13, 5,
    63, 55, 47, 39, 31, 23, 15, 7
    };
```

IP表的作用在于对输入明文进行扩散混淆：

将输入的第 1 位变为输出的第58位，输入的第 2 位变为输出的第50位......(诸如此类以此类推)

### 逆IP表(<font style="color:#000000;">Final Permutation-最终置换</font>)
(这里的命名大抵是翻译的问题)

```c
const int FP[64]=
{
    40, 8, 48, 16, 56, 24, 64, 32,
    39, 7, 47, 15, 55, 23, 63, 31,
    38, 6, 46, 14, 54, 22, 62, 30,
    37, 5, 45, 13, 53, 21, 61, 29,
    36, 4, 44, 12, 52, 20, 60, 28,
    35, 3, 43, 11, 51, 19, 59, 27,
    34, 2, 42, 10, 50, 18, 58, 26,
    33, 1, 41,  9, 49, 17, 57, 25
    };
```

这里就是对输入明文进行二次扩散混淆，运行逻辑与IP表相同。

注意：在加密时输入的明文先在IP表中进行处理再在FP表中进行处理，那么在解密时密文就应该先在FP表中进行处理再在IP表中处理

### E表
```c
const int E[48]=
{
    32,  1,  2,  3,  4,  5,
    4,  5,  6,  7,  8,  9,
    8,  9, 10, 11, 12, 13,
    12, 13, 14, 15, 16, 17,
    16, 17, 18, 19, 20, 21,
    20, 21, 22, 23, 24, 25,
    24, 25, 26, 27, 28, 29,
    28, 29, 30, 31, 32,  1
    };
```

由于我们输入的数据是32位数据，这里E表的作用就是将输入的32位数据通过重复某几位上的数据转化为48位数据。

设计目的在于使某一位数据的变化能影响到多个S盒(后文会讲到)，同时与48位的子密钥(后文会讲到)相匹配。

### P盒
```c
const int P[32]=
{
    16,  7, 20, 21,
    29, 12, 28, 17,
    1, 15, 23, 26,
    5, 18, 31, 10,
    2,  8, 24, 14,
    32, 27,  3,  9,
    19, 13, 30,  6,
    22, 11,  4, 25
    };
```

P盒的存在主要是为了处理从S盒出来后的数据，从S盒出来的数据的第 1 位是输出的第16位(诸如此类以此类推)，这样的设计可以大大增强数据的扩展性和混淆性

### PC-1表
```c
const int PC1[56]=
{
    57, 49, 41, 33, 25, 17,  9,
    1, 58, 50, 42, 34, 26, 18,
    10,  2, 59, 51, 43, 35, 27,
    19, 11,  3, 60, 52, 44, 36,
    63, 55, 47, 39, 31, 23, 15,
    7, 62, 54, 46, 38, 30, 22,
    14,  6, 61, 53, 45, 37, 29,
    21, 13,  5, 28, 20, 12,  4
    };
```

首先，DES使用64位密钥，其中<font style="color:#000000;">第8、16、24、32、40、48、56、64位是奇偶效验位，在进行处理时会被丢弃。P盒对密钥的处理方式与别的盒相同。最终会按照表中的结构输出56位的密钥。(与后面生成28位半密钥相关)</font>

### <font style="color:#000000;">PC-2表</font>
```plain
const int PC2[48]=
{
    14, 17, 11, 24,  1,  5,
    3, 28, 15,  6, 21, 10,
    23, 19, 12,  4, 26,  8,
    16,  7, 27, 20, 13,  2,
    41, 52, 31, 37, 47, 55,
    30, 40, 51, 45, 33, 48,
    44, 49, 39, 56, 34, 53,
    46, 42, 50, 36, 29, 32
};
```

PC-2表主要是针对已经处理好的，从PC-1表中出来并且完成位移和拼接的密钥，将它们混淆为48位子密钥，并且由于左位移的存在导致每次从PC-2表中出来的子密钥都不相同，且<font style="color:rgb(15, 17, 21);">丢弃的位：PC-1输出中的第9、18、22、25、35、38、43、54位。</font>

### <font style="color:rgb(15, 17, 21);">左移位数表</font>
```c
const int left_shift[16]=
{1, 1, 2, 2, 2, 2, 2, 2, 1, 2, 2, 2, 2, 2, 2, 1};
```

定义了一个左移位数的数组。

### S盒
```c
const uint8_t S[8][4][16]=
{
    {
        {14,4,13,1,2,15,11,8,3,10,6,12,5,9,0,7},
        {0,15,7,4,14,2,13,1,10,6,12,11,9,5,3,8},
        {4,1,14,8,13,6,2,11,15,12,9,7,3,10,5,0},
        {15,12,8,2,4,9,1,7,5,11,3,14,10,0,6,13}
    },
    {
        {15,1,8,14,6,11,3,4,9,7,2,13,12,0,5,10},
        {3,13,4,7,15,2,8,14,12,0,1,10,6,9,11,5},
        {0,14,7,11,10,4,13,1,5,8,12,6,9,3,2,15},
        {13,8,10,1,3,15,4,2,11,6,7,12,0,5,14,9}
    },
    {
        {10,0,9,14,6,3,15,5,1,13,12,7,11,4,2,8},
        {13,7,0,9,3,4,6,10,2,8,5,14,12,11,15,1},
        {13,6,4,9,8,15,3,0,11,1,2,12,5,10,14,7},
        {1,10,13,0,6,9,8,7,4,15,14,3,11,5,2,12}
    },
    {
        {7,13,14,3,0,6,9,10,1,2,8,5,11,12,4,15},
        {13,8,11,5,6,15,0,3,4,7,2,12,1,10,14,9},
        {10,6,9,0,12,11,7,13,15,1,3,14,5,2,8,4},
        {3,15,0,6,10,1,13,8,9,4,5,11,12,7,2,14}
    },
    {
        {2,12,4,1,7,10,11,6,8,5,3,15,13,0,14,9},
        {14,11,2,12,4,7,13,1,5,0,15,10,3,9,8,6},
        {4,2,1,11,10,13,7,8,15,9,12,5,6,3,0,14},
        {11,8,12,7,1,14,2,13,6,15,0,9,10,4,5,3}
    },
    {
        {12,1,10,15,9,2,6,8,0,13,3,4,14,7,5,11},
        {10,15,4,2,7,12,9,5,6,1,13,14,0,11,3,8},
        {9,14,15,5,2,8,12,3,7,0,4,10,1,13,11,6},
        {4,3,2,12,9,5,15,10,11,14,1,7,6,0,8,13}
    },
    {
        {4,11,2,14,15,0,8,13,3,12,9,7,5,10,6,1},
        {13,0,11,7,4,9,1,10,14,3,5,12,2,15,8,6},
        {1,4,11,13,12,3,7,14,10,15,6,8,0,5,9,2},
        {6,11,13,8,1,4,10,7,9,5,0,15,14,2,3,12}
    },
    {
        {13,2,8,4,6,15,11,1,10,9,3,14,5,0,12,7},
        {1,15,13,8,10,3,7,4,12,5,6,11,0,14,9,2},
        {7,11,4,1,9,12,14,2,0,6,10,13,15,3,5,8},
        {2,1,14,7,4,10,8,13,15,12,9,0,3,5,6,11}
    }
};
```

S盒的主要作用在于将一个 6 位数据转为 4 位数据。S[a][b][c]的意思是将第 a 个S盒中第 b 行第 c 列的数据当作最终数据输出。至于为什么是 4 位数据：因为S盒中最大的值为15，在二进制中的表示为1111，所以这样就能得到一个 4 位数据。(再具体a，b，c的求法后文会讲到)

## DES的核心逻辑
在DES算法中有几个逻辑是固定不变的

### 置换函数
```c
uint64_t permute(uint64_t input,const int *table,int n)
{
    uint64_t result=0;
    for(int i=0;i<n;i++)
    {
        int bit_pos=table[i]-1;
        if(input & (1ULL<<(64-1-bit_pos)))
            result |= (1ULL<<(n-1-i));
    }
    return result;
}
```

首先看轮函数的传参：传入input，table和n。input就是我们的输入，table就是表，一般来说就是各种表与盒，而 n 则是某一个表或盒的元素具体数量。

由于在数学定义中，算法是从 1 开始计算索引的，这里的table[i]-1就是计算出相应的C语言索引(毕竟C语言的索引是从 0 开始的)。后面的if是在检测第64-1-bit_pos位是否为1，如果是 1 就在该位置上放一个1，否则跳过。

### 循环位移函数
```c
uint32_t left_rotate28(uint32_t value,int shift)
{
    return ((value<<shift)|(value>>(28-shift)))&0x0FFFFFFF;
}
```

作用就在于处理密钥，对密钥进行位移。解密时反向位移即可。

### 子密钥生成函数
```c
void generate_subkeys(uint64_t key,uint64_t subkeys[16])
{
    uint64_t pc1_key=permute(key,PC1,56);
    uint32_t left=(pc1_key>>28)&0x0FFFFFFF;
    uint32_t right=pc1_key&0x0FFFFFFF;
    for(int i=0;i<16;i++)
    {
        left=left_rotate28(left,left_shift[i]);
        right=left_rotate28(right,left_shift[i]);
        uint64_t combined=((uint64_t)left<<28)|right;
        subkeys[i]=permute(combined << 8, PC2, 48); 
    }
}
```

子密钥生成逻辑在DES中也是不变的核心函数逻辑，很清晰就不赘述了。

### 轮函数
```c
uint32_t f_function(uint32_t right,uint64_t subkey)
{
    uint64_t expanded=permute((uint64_t)right<<32,E,48);
    uint64_t xored=expanded^subkey;
    uint32_t out=0;

    for(int i=0;i<8;i++)
    {
        uint8_t six=(xored>>(42-i*6))&0x3F;
        uint8_t row=((six&0x20)>>4)|(six&0x01);
        uint8_t col=(six>>1)&0x0F;
        uint8_t val=S[i][row][col];
        out|=(uint32_t)val<<(28-i*4);
    }
    return (uint32_t)permute((uint64_t)out<<32,P,32);
}
```

轮函数的主要作用在于将 6 位数据通过S盒的替换，变成 4 位数据(上文S盒处有讲)。



**接下来将分别展示ECB，CBC，CTR的完整代码，梳理其执行流并比较其不同。**

## ECB模式
```c
#include <bits/stdc++.h>
using namespace std;

//定义IP表
const int IP[64]=
{
    58, 50, 42, 34, 26, 18, 10, 2,
    60, 52, 44, 36, 28, 20, 12, 4,
    62, 54, 46, 38, 30, 22, 14, 6,
    64, 56, 48, 40, 32, 24, 16, 8,
    57, 49, 41, 33, 25, 17,  9, 1,
    59, 51, 43, 35, 27, 19, 11, 3,
    61, 53, 45, 37, 29, 21, 13, 5,
    63, 55, 47, 39, 31, 23, 15, 7
};

//定义IP逆表
const int FP[64]=
{
    40, 8, 48, 16, 56, 24, 64, 32,
    39, 7, 47, 15, 55, 23, 63, 31,
    38, 6, 46, 14, 54, 22, 62, 30,
    37, 5, 45, 13, 53, 21, 61, 29,
    36, 4, 44, 12, 52, 20, 60, 28,
    35, 3, 43, 11, 51, 19, 59, 27,
    34, 2, 42, 10, 50, 18, 58, 26,
    33, 1, 41,  9, 49, 17, 57, 25
};

//定义置换的E表
const int E[48]=
{
    32,  1,  2,  3,  4,  5,
    4,  5,  6,  7,  8,  9,
    8,  9, 10, 11, 12, 13,
    12, 13, 14, 15, 16, 17,
    16, 17, 18, 19, 20, 21,
    20, 21, 22, 23, 24, 25,
    24, 25, 26, 27, 28, 29,
    28, 29, 30, 31, 32,  1
};

//定义P盒置换表
const int P[32]=
{
    16,  7, 20, 21,
    29, 12, 28, 17,
    1, 15, 23, 26,
    5, 18, 31, 10,
    2,  8, 24, 14,
    32, 27,  3,  9,
    19, 13, 30,  6,
    22, 11,  4, 25
};

/*以上是对密文的处理
接下来定义的PC1和PC2是对密钥的处理*/
const int PC1[56]=
{
    57, 49, 41, 33, 25, 17,  9,
    1, 58, 50, 42, 34, 26, 18,
    10,  2, 59, 51, 43, 35, 27,
    19, 11,  3, 60, 52, 44, 36,
    63, 55, 47, 39, 31, 23, 15,
    7, 62, 54, 46, 38, 30, 22,
    14,  6, 61, 53, 45, 37, 29,
    21, 13,  5, 28, 20, 12,  4
};

const int PC2[48]=
{
    14, 17, 11, 24,  1,  5,
    3, 28, 15,  6, 21, 10,
    23, 19, 12,  4, 26,  8,
    16,  7, 27, 20, 13,  2,
    41, 52, 31, 37, 47, 55,
    30, 40, 51, 45, 33, 48,
    44, 49, 39, 56, 34, 53,
    46, 42, 50, 36, 29, 32
};

//定义左位移数表
const int left_shift[16]=
{1, 1, 2, 2, 2, 2, 2, 2, 1, 2, 2, 2, 2, 2, 2, 1};

//开始定义8个S盒
const uint8_t S[8][4][16]=
{
    {
        {14,4,13,1,2,15,11,8,3,10,6,12,5,9,0,7},
        {0,15,7,4,14,2,13,1,10,6,12,11,9,5,3,8},
        {4,1,14,8,13,6,2,11,15,12,9,7,3,10,5,0},
        {15,12,8,2,4,9,1,7,5,11,3,14,10,0,6,13}
    },
    {
        {15,1,8,14,6,11,3,4,9,7,2,13,12,0,5,10},
        {3,13,4,7,15,2,8,14,12,0,1,10,6,9,11,5},
        {0,14,7,11,10,4,13,1,5,8,12,6,9,3,2,15},
        {13,8,10,1,3,15,4,2,11,6,7,12,0,5,14,9}
    },
    {
        {10,0,9,14,6,3,15,5,1,13,12,7,11,4,2,8},
        {13,7,0,9,3,4,6,10,2,8,5,14,12,11,15,1},
        {13,6,4,9,8,15,3,0,11,1,2,12,5,10,14,7},
        {1,10,13,0,6,9,8,7,4,15,14,3,11,5,2,12}
    },
    {
        {7,13,14,3,0,6,9,10,1,2,8,5,11,12,4,15},
        {13,8,11,5,6,15,0,3,4,7,2,12,1,10,14,9},
        {10,6,9,0,12,11,7,13,15,1,3,14,5,2,8,4},
        {3,15,0,6,10,1,13,8,9,4,5,11,12,7,2,14}
    },
    {
        {2,12,4,1,7,10,11,6,8,5,3,15,13,0,14,9},
        {14,11,2,12,4,7,13,1,5,0,15,10,3,9,8,6},
        {4,2,1,11,10,13,7,8,15,9,12,5,6,3,0,14},
        {11,8,12,7,1,14,2,13,6,15,0,9,10,4,5,3}
    },
    {
        {12,1,10,15,9,2,6,8,0,13,3,4,14,7,5,11},
        {10,15,4,2,7,12,9,5,6,1,13,14,0,11,3,8},
        {9,14,15,5,2,8,12,3,7,0,4,10,1,13,11,6},
        {4,3,2,12,9,5,15,10,11,14,1,7,6,0,8,13}
    },
    {
        {4,11,2,14,15,0,8,13,3,12,9,7,5,10,6,1},
        {13,0,11,7,4,9,1,10,14,3,5,12,2,15,8,6},
        {1,4,11,13,12,3,7,14,10,15,6,8,0,5,9,2},
        {6,11,13,8,1,4,10,7,9,5,0,15,14,2,3,12}
    },
    {
        {13,2,8,4,6,15,11,1,10,9,3,14,5,0,12,7},
        {1,15,13,8,10,3,7,4,12,5,6,11,0,14,9,2},
        {7,11,4,1,9,12,14,2,0,6,10,13,15,3,5,8},
        {2,1,14,7,4,10,8,13,15,12,9,0,3,5,6,11}
    }
};

//字节数组转64位整数 (Big Endian)
uint64_t bytes_to_u64(const uint8_t *b)
{
    uint64_t v=0;
    for(int i=0;i<8;i++)
        v = (v<<8)|b[i];
    return v;
}

//64位整数转字节数组 (Big Endian)
void u64_to_bytes(uint64_t v,uint8_t *b)
{
    for(int i=7;i>=0;i--)
    {
        b[i]=v&0xFF;
        v>>=8;
    }
}

//定义置换函数，支持任意长度
//注意：此函数假定输入数据的“第1位”对应 uint64_t 的 MSB (第63位)
uint64_t permute(uint64_t input,const int *table,int n)
{
    uint64_t result=0;
    for(int i=0;i<n;i++)
    {
        int bit_pos=table[i]-1;
        // 检查input的第table[i]位是否为1
        if(input & (1ULL<<(64-1-bit_pos)))
            result |= (1ULL<<(n-1-i));
    }
    return result;
}

//循环左移28位
uint32_t left_rotate28(uint32_t value,int shift)
{
    return ((value<<shift)|(value>>(28-shift)))&0x0FFFFFFF;
}

//生成16个子密钥
void generate_subkeys(uint64_t key,uint64_t subkeys[16])
{
    uint64_t pc1_key=permute(key,PC1,56);
    uint32_t left=(pc1_key>>28)&0x0FFFFFFF;
    uint32_t right=pc1_key&0x0FFFFFFF;

    for(int i=0;i<16;i++)
    {
        left=left_rotate28(left,left_shift[i]);
        right=left_rotate28(right,left_shift[i]);
        
        // 组合左右两部分，共56位
        uint64_t combined=((uint64_t)left<<28)|right;
        
        /* FIX: permute函数默认输入是左对齐(MSB对齐)的。
           combined当前只有低56位有效(0-55)，高8位是0。
           必须左移8位，让有效数据占据高位(63-8)，permute才能正确读取。
        */
        subkeys[i]=permute(combined << 8, PC2, 48); 
    }
}

//轮函数F
uint32_t f_function(uint32_t right,uint64_t subkey)
{
    // right是32位，左移32位使其对齐到64位MSB
    uint64_t expanded=permute((uint64_t)right<<32,E,48);
    uint64_t xored=expanded^subkey;
    uint32_t out=0;

    for(int i=0;i<8;i++)
    {
        // 每次取6位
        uint8_t six=(xored>>(42-i*6))&0x3F;
        uint8_t row=((six&0x20)>>4)|(six&0x01);
        uint8_t col=(six>>1)&0x0F;
        uint8_t val=S[i][row][col];
        out|=(uint32_t)val<<(28-i*4);
    }
    // out是32位，左移32位使其对齐到64位MSB
    return (uint32_t)permute((uint64_t)out<<32,P,32);
}

//DES加密8字节块
uint64_t DES_encrypt(uint8_t *plaintext,uint8_t *key)
{
    uint64_t subkeys[16];
    generate_subkeys(bytes_to_u64(key),subkeys);

    uint64_t data=permute(bytes_to_u64(plaintext),IP,64);
    uint32_t left=(data>>32)&0xFFFFFFFF;
    uint32_t right=data&0xFFFFFFFF;

    for(int i=0;i<16;i++)
    {
        uint32_t temp=right;
        right=left^f_function(right,subkeys[i]);
        left=temp;
    }

    // DES最后一轮后，是先R16再L16 (即无需再swap，直接拼接)
    // right作为高位，left作为低位
    uint64_t combined=((uint64_t)right<<32)|left;
    return permute(combined,FP,64);
}

int main()
{
    // 输入明文和密钥
    uint8_t plaintext[]=" ";
    uint8_t key_str[]=" "; // 密钥必须是8字节

    int len=strlen((char*)plaintext);
    // PKCS5/7 Padding
    int pad=8-(len%8);
    int total=len+pad;

    // 准备缓冲区
    uint8_t buf[256]={0}; 
    memcpy(buf,plaintext,len);
    memset(buf+len,pad,pad); 

    printf("Ciphertext (Hex): \n");
    for(int i=0;i<total;i+=8)
    {
        uint64_t enc=DES_encrypt(buf+i,key_str);

        uint8_t out[8];
        u64_to_bytes(enc,out);

        for(int j=0;j<8;j++)
        {
            printf("%02X",out[j]);
            if(j%4==3)
            {
                printf("\n");
            }
        }
    }
    return 0;
}
```

大部分函数上文都已提及，所以这里我们把main函数和DES_encrypt函数单拎出来(毕竟主要执行流就在这两个函数中了)

```c
int main()
{
    // 输入明文和密钥
    uint8_t plaintext[]=" ";
    uint8_t key_str[]=" "; // 密钥必须是8字节

    int len=strlen((char*)plaintext);
    // PKCS5/7 Padding
    int pad=8-(len%8);
    int total=len+pad;

    // 准备缓冲区
    uint8_t buf[256]={0}; 
    memcpy(buf,plaintext,len);
    memset(buf+len,pad,pad); 

    printf("Ciphertext (Hex): \n");
    for(int i=0;i<total;i+=8)
    {
        uint64_t enc=DES_encrypt(buf+i,key_str);

        uint8_t out[8];
        u64_to_bytes(enc,out);

        for(int j=0;j<8;j++)
        {
            printf("%02X",out[j]);
            if(j%4==3)
            {
                printf("\n");
            }
        }
    }
    return 0;
}
```

可以看到main函数一开始就对明文的长度进行检索，并将其填充到 8 的倍数长度(因为DES除CTR模式以外都只能加密长度为 8 的倍数长度的明文)。将处于缓冲区中的数据和密钥传入DES_encrypt函数中。后面的u64_to_bytes是为了方便输出才有的。

```c
uint64_t DES_encrypt(uint8_t *plaintext,uint8_t *key)
{
    uint64_t subkeys[16];
    generate_subkeys(bytes_to_u64(key),subkeys);

    uint64_t data=permute(bytes_to_u64(plaintext),IP,64);
    uint32_t left=(data>>32)&0xFFFFFFFF;
    uint32_t right=data&0xFFFFFFFF;

    for(int i=0;i<16;i++)
    {
        uint32_t temp=right;
        right=left^f_function(right,subkeys[i]);
        left=temp;
    }

    // DES最后一轮后，是先R16再L16 (即无需再swap，直接拼接)
    // right作为高位，left作为低位
    uint64_t combined=((uint64_t)right<<32)|left;
    return permute(combined,FP,64);
}
```

代码很清晰，就不多赘述了。但有两点需要注意

**第一**

```c
for(int i=0;i<16;i++)
    {
        uint32_t temp=right;
        right=left^f_function(right,subkeys[i]);
        left=temp;
    }
```

这个代码在每一种模式下的DES都是不变的。

**第二**

```c
for(int i = 15; i >= 0; i--) {
    uint32_t temp = left;
    left = right ^ f_function(left, subkeys[i]);
    right = temp;
}
```

解密时改代码逆向逻辑如上

## CBC模式
```c
#include <bits/stdc++.h>
using namespace std;

//定义IP表
const int IP[64]=
{
    58, 50, 42, 34, 26, 18, 10, 2,
    60, 52, 44, 36, 28, 20, 12, 4,
    62, 54, 46, 38, 30, 22, 14, 6,
    64, 56, 48, 40, 32, 24, 16, 8,
    57, 49, 41, 33, 25, 17,  9, 1,
    59, 51, 43, 35, 27, 19, 11, 3,
    61, 53, 45, 37, 29, 21, 13, 5,
    63, 55, 47, 39, 31, 23, 15, 7
};

//定义IP逆表
const int FP[64]=
{
    40, 8, 48, 16, 56, 24, 64, 32,
    39, 7, 47, 15, 55, 23, 63, 31,
    38, 6, 46, 14, 54, 22, 62, 30,
    37, 5, 45, 13, 53, 21, 61, 29,
    36, 4, 44, 12, 52, 20, 60, 28,
    35, 3, 43, 11, 51, 19, 59, 27,
    34, 2, 42, 10, 50, 18, 58, 26,
    33, 1, 41,  9, 49, 17, 57, 25
};

//定义置换的E表
const int E[48]=
{
    32,  1,  2,  3,  4,  5,
    4,  5,  6,  7,  8,  9,
    8,  9, 10, 11, 12, 13,
    12, 13, 14, 15, 16, 17,
    16, 17, 18, 19, 20, 21,
    20, 21, 22, 23, 24, 25,
    24, 25, 26, 27, 28, 29,
    28, 29, 30, 31, 32,  1
};

//定义P盒置换表
const int P[32]=
{
    16,  7, 20, 21,
    29, 12, 28, 17,
    1, 15, 23, 26,
    5, 18, 31, 10,
    2,  8, 24, 14,
    32, 27,  3,  9,
    19, 13, 30,  6,
    22, 11,  4, 25
};

/*以上是对密文的处理
接下来定义的PC1和PC2是对密钥的处理*/
const int PC1[56]=
{
    57, 49, 41, 33, 25, 17,  9,
    1, 58, 50, 42, 34, 26, 18,
    10,  2, 59, 51, 43, 35, 27,
    19, 11,  3, 60, 52, 44, 36,
    63, 55, 47, 39, 31, 23, 15,
    7, 62, 54, 46, 38, 30, 22,
    14,  6, 61, 53, 45, 37, 29,
    21, 13,  5, 28, 20, 12,  4
};

const int PC2[48]=
{
    14, 17, 11, 24,  1,  5,
    3, 28, 15,  6, 21, 10,
    23, 19, 12,  4, 26,  8,
    16,  7, 27, 20, 13,  2,
    41, 52, 31, 37, 47, 55,
    30, 40, 51, 45, 33, 48,
    44, 49, 39, 56, 34, 53,
    46, 42, 50, 36, 29, 32
};

//定义左位移数表
const int left_shift[16]=
{1, 1, 2, 2, 2, 2, 2, 2, 1, 2, 2, 2, 2, 2, 2, 1};

//开始定义8个S盒
const uint8_t S[8][4][16]=
{
    {
        {14,4,13,1,2,15,11,8,3,10,6,12,5,9,0,7},
        {0,15,7,4,14,2,13,1,10,6,12,11,9,5,3,8},
        {4,1,14,8,13,6,2,11,15,12,9,7,3,10,5,0},
        {15,12,8,2,4,9,1,7,5,11,3,14,10,0,6,13}
    },
    {
        {15,1,8,14,6,11,3,4,9,7,2,13,12,0,5,10},
        {3,13,4,7,15,2,8,14,12,0,1,10,6,9,11,5},
        {0,14,7,11,10,4,13,1,5,8,12,6,9,3,2,15},
        {13,8,10,1,3,15,4,2,11,6,7,12,0,5,14,9}
    },
    {
        {10,0,9,14,6,3,15,5,1,13,12,7,11,4,2,8},
        {13,7,0,9,3,4,6,10,2,8,5,14,12,11,15,1},
        {13,6,4,9,8,15,3,0,11,1,2,12,5,10,14,7},
        {1,10,13,0,6,9,8,7,4,15,14,3,11,5,2,12}
    },
    {
        {7,13,14,3,0,6,9,10,1,2,8,5,11,12,4,15},
        {13,8,11,5,6,15,0,3,4,7,2,12,1,10,14,9},
        {10,6,9,0,12,11,7,13,15,1,3,14,5,2,8,4},
        {3,15,0,6,10,1,13,8,9,4,5,11,12,7,2,14}
    },
    {
        {2,12,4,1,7,10,11,6,8,5,3,15,13,0,14,9},
        {14,11,2,12,4,7,13,1,5,0,15,10,3,9,8,6},
        {4,2,1,11,10,13,7,8,15,9,12,5,6,3,0,14},
        {11,8,12,7,1,14,2,13,6,15,0,9,10,4,5,3}
    },
    {
        {12,1,10,15,9,2,6,8,0,13,3,4,14,7,5,11},
        {10,15,4,2,7,12,9,5,6,1,13,14,0,11,3,8},
        {9,14,15,5,2,8,12,3,7,0,4,10,1,13,11,6},
        {4,3,2,12,9,5,15,10,11,14,1,7,6,0,8,13}
    },
    {
        {4,11,2,14,15,0,8,13,3,12,9,7,5,10,6,1},
        {13,0,11,7,4,9,1,10,14,3,5,12,2,15,8,6},
        {1,4,11,13,12,3,7,14,10,15,6,8,0,5,9,2},
        {6,11,13,8,1,4,10,7,9,5,0,15,14,2,3,12}
    },
    {
        {13,2,8,4,6,15,11,1,10,9,3,14,5,0,12,7},
        {1,15,13,8,10,3,7,4,12,5,6,11,0,14,9,2},
        {7,11,4,1,9,12,14,2,0,6,10,13,15,3,5,8},
        {2,1,14,7,4,10,8,13,15,12,9,0,3,5,6,11}
    }
};

//字节数组转64位整数 (Big Endian)
uint64_t bytes_to_u64(const uint8_t *b)
{
    uint64_t v=0;
    for(int i=0;i<8;i++)
        v = (v<<8)|b[i];
    return v;
}

//64位整数转字节数组 (Big Endian)
void u64_to_bytes(uint64_t v,uint8_t *b)
{
    for(int i=7;i>=0;i--)
    {
        b[i]=v&0xFF;
        v>>=8;
    }
}

//定义置换函数，支持任意长度
//注意：此函数假定输入数据的"第1位"对应 uint64_t 的 MSB (第63位)
uint64_t permute(uint64_t input,const int *table,int n)
{
    uint64_t result=0;
    for(int i=0;i<n;i++)
    {
        int bit_pos=table[i]-1;
        // 检查input的第table[i]位是否为1
        if(input & (1ULL<<(64-1-bit_pos)))
            result |= (1ULL<<(n-1-i));
    }
    return result;
}

//循环左移28位
uint32_t left_rotate28(uint32_t value,int shift)
{
    return ((value<<shift)|(value>>(28-shift)))&0x0FFFFFFF;
}

//生成16个子密钥
void generate_subkeys(uint64_t key,uint64_t subkeys[16])
{
    uint64_t pc1_key=permute(key,PC1,56);
    uint32_t left=(pc1_key>>28)&0x0FFFFFFF;
    uint32_t right=pc1_key&0x0FFFFFFF;

    for(int i=0;i<16;i++)
    {
        left=left_rotate28(left,left_shift[i]);
        right=left_rotate28(right,left_shift[i]);
        
        // 组合左右两部分，共56位
        uint64_t combined=((uint64_t)left<<28)|right;
        
        /* FIX: permute函数默认输入是左对齐(MSB对齐)的。
           combined当前只有低56位有效(0-55)，高8位是0。
           必须左移8位，让有效数据占据高位(63-8)，permute才能正确读取。
        */
        subkeys[i]=permute(combined << 8, PC2, 48); 
    }
}

//轮函数F
uint32_t f_function(uint32_t right,uint64_t subkey)
{
    // right是32位，左移32位使其对齐到64位MSB
    uint64_t expanded=permute((uint64_t)right<<32,E,48);
    uint64_t xored=expanded^subkey;
    uint32_t out=0;

    for(int i=0;i<8;i++)
    {
        // 每次取6位
        uint8_t six=(xored>>(42-i*6))&0x3F;
        uint8_t row=((six&0x20)>>4)|(six&0x01);
        uint8_t col=(six>>1)&0x0F;
        uint8_t val=S[i][row][col];
        out|=(uint32_t)val<<(28-i*4);
    }
    // out是32位，左移32位使其对齐到64位MSB
    return (uint32_t)permute((uint64_t)out<<32,P,32);
}

//基本DES加密8字节块
uint64_t DES_encrypt_block(uint64_t plaintext, uint64_t key)
{
    uint64_t subkeys[16];
    generate_subkeys(key, subkeys);

    uint64_t data = permute(plaintext, IP, 64);
    uint32_t left = (data >> 32) & 0xFFFFFFFF;
    uint32_t right = data & 0xFFFFFFFF;

    for(int i=0;i<16;i++)
    {
        uint32_t temp = right;
        right = left ^ f_function(right, subkeys[i]);
        left = temp;
    }

    // DES最后一轮后，是先R16再L16 (即无需再swap，直接拼接)
    // right作为高位，left作为低位
    uint64_t combined = ((uint64_t)right << 32) | left;
    return permute(combined, FP, 64);
}

//CBC模式DES加密
void DES_CBC_encrypt(uint8_t *plaintext, uint8_t *key, uint8_t *iv, int len, uint8_t *ciphertext)
{
    uint64_t key64 = bytes_to_u64(key);
    uint64_t iv64 = bytes_to_u64(iv);
    
    int blocks = (len + 7) / 8;  // 计算需要的块数
    uint64_t prev_block = iv64;
    
    for(int i = 0; i < blocks; i++)
    {
        // 准备当前明文块
        uint8_t block[8] = {0};
        int copy_len = min(8, len - i*8);
        memcpy(block, plaintext + i*8, copy_len);
        
        // PKCS7填充
        if(copy_len < 8)
        {
            uint8_t pad = 8 - copy_len;
            for(int j = copy_len; j < 8; j++)
                block[j] = pad;
        }
        
        uint64_t plaintext64 = bytes_to_u64(block);
        
        // CBC模式：先与前一个密文块异或
        uint64_t xored = plaintext64 ^ prev_block;
        
        // DES加密
        uint64_t encrypted = DES_encrypt_block(xored, key64);
        
        // 保存密文块
        u64_to_bytes(encrypted, ciphertext + i*8);
        
        // 更新前一个密文块
        prev_block = encrypted;
    }
}

int main()
{
    // 输入明文和密钥
    uint8_t plaintext[] = "GWHT{R3Verse_15_BeAu71Ful!!!}";
    uint8_t key_str[] = "10831k0m"; // 密钥必须是8字节
    uint8_t iv[] = "initvec0";      // IV必须是8字节
    
    int len = strlen((char*)plaintext);
    int blocks = (len + 7) / 8;  // 计算需要的块数
    int padded_len = blocks * 8;
    
    // 准备缓冲区
    uint8_t ciphertext[256] = {0};
    
    // CBC模式加密
    DES_CBC_encrypt(plaintext, key_str, iv, len, ciphertext);
    
    printf("CBC Mode Ciphertext (Hex): \n");
    for(int i = 0; i < padded_len; i++)
    {
        printf("%02X", ciphertext[i]);
        if((i+1) % 16 == 0) printf("\n");
        else if((i+1) % 4 == 0) printf(" ");
    }
    printf("\n");
    
    return 0;
}
```

可以看到，CBC模式下的代码与ECB模式下的代码几乎完全相同，唯二不同的地方就在于前一个数据与后一个数据相异或，且定义了一个初始化向量(详情请见前言)。但总体上与ECB模式相同。

## CTR模式
```c
#include <bits/stdc++.h>
using namespace std;

//定义IP表
const int IP[64]=
{
    58, 50, 42, 34, 26, 18, 10, 2,
    60, 52, 44, 36, 28, 20, 12, 4,
    62, 54, 46, 38, 30, 22, 14, 6,
    64, 56, 48, 40, 32, 24, 16, 8,
    57, 49, 41, 33, 25, 17,  9, 1,
    59, 51, 43, 35, 27, 19, 11, 3,
    61, 53, 45, 37, 29, 21, 13, 5,
    63, 55, 47, 39, 31, 23, 15, 7
};

//定义IP逆表
const int FP[64]=
{
    40, 8, 48, 16, 56, 24, 64, 32,
    39, 7, 47, 15, 55, 23, 63, 31,
    38, 6, 46, 14, 54, 22, 62, 30,
    37, 5, 45, 13, 53, 21, 61, 29,
    36, 4, 44, 12, 52, 20, 60, 28,
    35, 3, 43, 11, 51, 19, 59, 27,
    34, 2, 42, 10, 50, 18, 58, 26,
    33, 1, 41,  9, 49, 17, 57, 25
};

//定义置换的E表
const int E[48]=
{
    32,  1,  2,  3,  4,  5,
    4,  5,  6,  7,  8,  9,
    8,  9, 10, 11, 12, 13,
    12, 13, 14, 15, 16, 17,
    16, 17, 18, 19, 20, 21,
    20, 21, 22, 23, 24, 25,
    24, 25, 26, 27, 28, 29,
    28, 29, 30, 31, 32,  1
};

//定义P盒置换表
const int P[32]=
{
    16,  7, 20, 21,
    29, 12, 28, 17,
    1, 15, 23, 26,
    5, 18, 31, 10,
    2,  8, 24, 14,
    32, 27,  3,  9,
    19, 13, 30,  6,
    22, 11,  4, 25
};

/*以上是对密文的处理
接下来定义的PC1和PC2是对密钥的处理*/
const int PC1[56]=
{
    57, 49, 41, 33, 25, 17,  9,
    1, 58, 50, 42, 34, 26, 18,
    10,  2, 59, 51, 43, 35, 27,
    19, 11,  3, 60, 52, 44, 36,
    63, 55, 47, 39, 31, 23, 15,
    7, 62, 54, 46, 38, 30, 22,
    14,  6, 61, 53, 45, 37, 29,
    21, 13,  5, 28, 20, 12,  4
};

const int PC2[48]=
{
    14, 17, 11, 24,  1,  5,
    3, 28, 15,  6, 21, 10,
    23, 19, 12,  4, 26,  8,
    16,  7, 27, 20, 13,  2,
    41, 52, 31, 37, 47, 55,
    30, 40, 51, 45, 33, 48,
    44, 49, 39, 56, 34, 53,
    46, 42, 50, 36, 29, 32
};

//定义左位移数表
const int left_shift[16]=
{1, 1, 2, 2, 2, 2, 2, 2, 1, 2, 2, 2, 2, 2, 2, 1};

//开始定义8个S盒
const uint8_t S[8][4][16]=
{
    {
        {14,4,13,1,2,15,11,8,3,10,6,12,5,9,0,7},
        {0,15,7,4,14,2,13,1,10,6,12,11,9,5,3,8},
        {4,1,14,8,13,6,2,11,15,12,9,7,3,10,5,0},
        {15,12,8,2,4,9,1,7,5,11,3,14,10,0,6,13}
    },
    {
        {15,1,8,14,6,11,3,4,9,7,2,13,12,0,5,10},
        {3,13,4,7,15,2,8,14,12,0,1,10,6,9,11,5},
        {0,14,7,11,10,4,13,1,5,8,12,6,9,3,2,15},
        {13,8,10,1,3,15,4,2,11,6,7,12,0,5,14,9}
    },
    {
        {10,0,9,14,6,3,15,5,1,13,12,7,11,4,2,8},
        {13,7,0,9,3,4,6,10,2,8,5,14,12,11,15,1},
        {13,6,4,9,8,15,3,0,11,1,2,12,5,10,14,7},
        {1,10,13,0,6,9,8,7,4,15,14,3,11,5,2,12}
    },
    {
        {7,13,14,3,0,6,9,10,1,2,8,5,11,12,4,15},
        {13,8,11,5,6,15,0,3,4,7,2,12,1,10,14,9},
        {10,6,9,0,12,11,7,13,15,1,3,14,5,2,8,4},
        {3,15,0,6,10,1,13,8,9,4,5,11,12,7,2,14}
    },
    {
        {2,12,4,1,7,10,11,6,8,5,3,15,13,0,14,9},
        {14,11,2,12,4,7,13,1,5,0,15,10,3,9,8,6},
        {4,2,1,11,10,13,7,8,15,9,12,5,6,3,0,14},
        {11,8,12,7,1,14,2,13,6,15,0,9,10,4,5,3}
    },
    {
        {12,1,10,15,9,2,6,8,0,13,3,4,14,7,5,11},
        {10,15,4,2,7,12,9,5,6,1,13,14,0,11,3,8},
        {9,14,15,5,2,8,12,3,7,0,4,10,1,13,11,6},
        {4,3,2,12,9,5,15,10,11,14,1,7,6,0,8,13}
    },
    {
        {4,11,2,14,15,0,8,13,3,12,9,7,5,10,6,1},
        {13,0,11,7,4,9,1,10,14,3,5,12,2,15,8,6},
        {1,4,11,13,12,3,7,14,10,15,6,8,0,5,9,2},
        {6,11,13,8,1,4,10,7,9,5,0,15,14,2,3,12}
    },
    {
        {13,2,8,4,6,15,11,1,10,9,3,14,5,0,12,7},
        {1,15,13,8,10,3,7,4,12,5,6,11,0,14,9,2},
        {7,11,4,1,9,12,14,2,0,6,10,13,15,3,5,8},
        {2,1,14,7,4,10,8,13,15,12,9,0,3,5,6,11}
    }
};

//字节数组转64位整数 (Big Endian)
uint64_t bytes_to_u64(const uint8_t *b)
{
    uint64_t v=0;
    for(int i=0;i<8;i++)
        v = (v<<8)|b[i];
    return v;
}

//64位整数转字节数组 (Big Endian)
void u64_to_bytes(uint64_t v,uint8_t *b)
{
    for(int i=7;i>=0;i--)
    {
        b[i]=v&0xFF;
        v>>=8;
    }
}

//定义置换函数，支持任意长度
//注意：此函数假定输入数据的"第1位"对应 uint64_t 的 MSB (第63位)
uint64_t permute(uint64_t input,const int *table,int n)
{
    uint64_t result=0;
    for(int i=0;i<n;i++)
    {
        int bit_pos=table[i]-1;
        // 检查input的第table[i]位是否为1
        if(input & (1ULL<<(64-1-bit_pos)))
            result |= (1ULL<<(n-1-i));
    }
    return result;
}

//循环左移28位
uint32_t left_rotate28(uint32_t value,int shift)
{
    return ((value<<shift)|(value>>(28-shift)))&0x0FFFFFFF;
}

//生成16个子密钥
void generate_subkeys(uint64_t key,uint64_t subkeys[16])
{
    uint64_t pc1_key=permute(key,PC1,56);
    uint32_t left=(pc1_key>>28)&0x0FFFFFFF;
    uint32_t right=pc1_key&0x0FFFFFFF;

    for(int i=0;i<16;i++)
    {
        left=left_rotate28(left,left_shift[i]);
        right=left_rotate28(right,left_shift[i]);
        
        // 组合左右两部分，共56位
        uint64_t combined=((uint64_t)left<<28)|right;
        
        /* FIX: permute函数默认输入是左对齐(MSB对齐)的。
           combined当前只有低56位有效(0-55)，高8位是0。
           必须左移8位，让有效数据占据高位(63-8)，permute才能正确读取。
        */
        subkeys[i]=permute(combined << 8, PC2, 48); 
    }
}

//轮函数F
uint32_t f_function(uint32_t right,uint64_t subkey)
{
    // right是32位，左移32位使其对齐到64位MSB
    uint64_t expanded=permute((uint64_t)right<<32,E,48);
    uint64_t xored=expanded^subkey;
    uint32_t out=0;

    for(int i=0;i<8;i++)
    {
        // 每次取6位
        uint8_t six=(xored>>(42-i*6))&0x3F;
        uint8_t row=((six&0x20)>>4)|(six&0x01);
        uint8_t col=(six>>1)&0x0F;
        uint8_t val=S[i][row][col];
        out|=(uint32_t)val<<(28-i*4);
    }
    // out是32位，左移32位使其对齐到64位MSB
    return (uint32_t)permute((uint64_t)out<<32,P,32);
}

//基本DES加密8字节块
uint64_t DES_encrypt_block(uint64_t plaintext, uint64_t key)
{
    uint64_t subkeys[16];
    generate_subkeys(key, subkeys);

    uint64_t data = permute(plaintext, IP, 64);
    uint32_t left = (data >> 32) & 0xFFFFFFFF;
    uint32_t right = data & 0xFFFFFFFF;

    for(int i=0;i<16;i++)
    {
        uint32_t temp = right;
        right = left ^ f_function(right, subkeys[i]);
        left = temp;
    }

    // DES最后一轮后，是先R16再L16 (即无需再swap，直接拼接)
    // right作为高位，left作为低位
    uint64_t combined = ((uint64_t)right << 32) | left;
    return permute(combined, FP, 64);
}
uint64_t generate_nonce()
{
    // 在实际应用中，应该使用真随机数生成器
    // 这里为了演示，使用时间戳和随机数
    
    uint64_t timestamp = static_cast<uint64_t>(time(nullptr));
    uint64_t random_part = static_cast<uint64_t>(rand());
    
    // 组合成一个64位的nonce
    return (timestamp << 32) | random_part;
}

//CTR模式DES加密
void DES_CTR_encrypt(uint8_t *plaintext, uint8_t *key, uint8_t *nonce, int len, uint8_t *ciphertext)
{
    uint64_t key64 = bytes_to_u64(key);
    uint64_t counter = bytes_to_u64(nonce);  // 使用nonce作为初始计数器
    
    int blocks = (len + 7) / 8;  // 计算需要的块数
    
    for(int i = 0; i < blocks; i++)
    {
        // 加密计数器
        uint64_t encrypted_counter = DES_encrypt_block(counter, key64);
        
        // 准备当前明文块
        uint8_t block[8] = {0};
        int copy_len = min(8, len - i*8);
        memcpy(block, plaintext + i*8, copy_len);
        
        uint64_t plaintext64 = bytes_to_u64(block);
        
        // CTR模式：加密流与明文异或
        uint64_t encrypted = plaintext64 ^ encrypted_counter;
        
        // 保存密文块
        u64_to_bytes(encrypted, ciphertext + i*8);
        
        // 计数器递增
        counter++;
    }
}

int main()
{
    // 输入明文和密钥
    uint8_t plaintext[] = "GWHT{R3Verse_15_BeAu71Ful!!!}";
    uint8_t key_str[] = "10831k0m"; // 密钥必须是8字节
    uint64_t nonce = generate_nonce();
    
    int len = strlen((char*)plaintext);
    int blocks = (len + 7) / 8;  // 计算需要的块数
    
    // 准备缓冲区
    uint8_t ciphertext[256] = {0};
    
    // CTR模式加密
    DES_CTR_encrypt(plaintext, key_str, nonce, len, ciphertext);
    
    printf("CTR Mode Ciphertext (Hex): \n");
    for(int i = 0; i < len; i++)
    {
        printf("%02X", ciphertext[i]);
        if((i+1) % 16 == 0) printf("\n");
        else if((i+1) % 4 == 0) printf(" ");
    }
    printf("\n");
    
    return 0;
}
```

与上面两种方式都不同，CTR模式需要定义一个计数器nonce。CTR模式下密钥替代明文进入DES加密逻辑中与密钥进行加密生成密钥流，而明文就只是在最后与密钥流进行异或生成最终密文。

## <font style="color:rgb(15, 17, 21);">DES加密模式详细对比表</font>
| **<font style="color:rgb(15, 17, 21);">对比维度</font>** | **<font style="color:rgb(15, 17, 21);">ECB模式 (Electronic Codebook)</font>** | **<font style="color:rgb(15, 17, 21);">CBC模式 (Cipher Block Chaining)</font>** | **<font style="color:rgb(15, 17, 21);">CTR模式 (Counter)</font>** |
| --- | --- | --- | --- |
| **<font style="color:rgb(15, 17, 21);">全称</font>** | <font style="color:rgb(15, 17, 21);">Electronic Codebook（电子密码本）</font> | <font style="color:rgb(15, 17, 21);">Cipher Block Chaining（密码块链接）</font> | <font style="color:rgb(15, 17, 21);">Counter（计数器）</font> |
| **<font style="color:rgb(15, 17, 21);">工作原理</font>** | <font style="color:rgb(15, 17, 21);">每个明文块独立加密，互不影响</font> | <font style="color:rgb(15, 17, 21);">每个明文块先与前一个密文块异或，再加密</font> | <font style="color:rgb(15, 17, 21);">将分组密码转换为流密码，加密计数器生成密钥流</font> |
| **<font style="color:rgb(15, 17, 21);">加密公式</font>** | <font style="color:rgb(15, 17, 21);">Cᵢ = Eₖ(Pᵢ)</font> | <font style="color:rgb(15, 17, 21);">C₀ = Eₖ(P₀ ⊕ IV)</font><font style="color:rgb(15, 17, 21);">   </font><font style="color:rgb(15, 17, 21);">Cᵢ = Eₖ(Pᵢ ⊕ Cᵢ₋₁)</font> | <font style="color:rgb(15, 17, 21);">Cᵢ = Pᵢ ⊕ Eₖ(Counterᵢ)</font> |
| **<font style="color:rgb(15, 17, 21);">解密公式</font>** | <font style="color:rgb(15, 17, 21);">Pᵢ = Dₖ(Cᵢ)</font> | <font style="color:rgb(15, 17, 21);">P₀ = Dₖ(C₀) ⊕ IV</font><font style="color:rgb(15, 17, 21);">   </font><font style="color:rgb(15, 17, 21);">Pᵢ = Dₖ(Cᵢ) ⊕ Cᵢ₋₁</font> | <font style="color:rgb(15, 17, 21);">Pᵢ = Cᵢ ⊕ Eₖ(Counterᵢ)</font> |
| **<font style="color:rgb(15, 17, 21);">是否需要IV/Nonce</font>** | <font style="color:rgb(15, 17, 21);">❌</font><font style="color:rgb(15, 17, 21);"> 不需要</font> | <font style="color:rgb(15, 17, 21);">✅</font><font style="color:rgb(15, 17, 21);"> 需要8字节IV</font> | <font style="color:rgb(15, 17, 21);">✅</font><font style="color:rgb(15, 17, 21);"> 需要Nonce（通常8字节）</font> |
| **<font style="color:rgb(15, 17, 21);">IV/Nonce要求</font>** | <font style="color:rgb(15, 17, 21);">无</font> | <font style="color:rgb(15, 17, 21);">必须随机且不可预测</font> | <font style="color:rgb(15, 17, 21);">必须唯一（通常随机生成）</font> |
| **<font style="color:rgb(15, 17, 21);">是否需要填充</font>** | <font style="color:rgb(15, 17, 21);">✅</font><font style="color:rgb(15, 17, 21);"> 需要（PKCS#5/7）</font> | <font style="color:rgb(15, 17, 21);">✅</font><font style="color:rgb(15, 17, 21);"> 需要（PKCS#5/7）</font> | <font style="color:rgb(15, 17, 21);">❌</font><font style="color:rgb(15, 17, 21);"> 不需要（流密码特性）</font> |
| **<font style="color:rgb(15, 17, 21);">填充方式</font>** | <font style="color:rgb(15, 17, 21);">不足8字节用填充值补齐</font> | <font style="color:rgb(15, 17, 21);">不足8字节用填充值补齐</font> | <font style="color:rgb(15, 17, 21);">无填充，最后块可截断</font> |
| **<font style="color:rgb(15, 17, 21);">并行加密</font>** | <font style="color:rgb(15, 17, 21);">✅</font><font style="color:rgb(15, 17, 21);"> 可以（各块独立）</font> | <font style="color:rgb(15, 17, 21);">❌</font><font style="color:rgb(15, 17, 21);"> 不可以（依赖前块）</font> | <font style="color:rgb(15, 17, 21);">✅</font><font style="color:rgb(15, 17, 21);"> 可以（计数器可预计算）</font> |
| **<font style="color:rgb(15, 17, 21);">并行解密</font>** | <font style="color:rgb(15, 17, 21);">✅</font><font style="color:rgb(15, 17, 21);"> 可以（各块独立）</font> | <font style="color:rgb(15, 17, 21);">❌</font><font style="color:rgb(15, 17, 21);"> 不可以（依赖前块）</font> | <font style="color:rgb(15, 17, 21);">✅</font><font style="color:rgb(15, 17, 21);"> 可以（计数器可预计算）</font> |
| **<font style="color:rgb(15, 17, 21);">相同明文块结果</font>** | <font style="color:rgb(15, 17, 21);">相同（安全性缺陷）</font> | <font style="color:rgb(15, 17, 21);">不同（与位置相关）</font> | <font style="color:rgb(15, 17, 21);">不同（计数器变化）</font> |
| **<font style="color:rgb(15, 17, 21);">错误传播</font>** | <font style="color:rgb(15, 17, 21);">只影响当前块</font> | <font style="color:rgb(15, 17, 21);">影响当前及后续块</font> | <font style="color:rgb(15, 17, 21);">只影响当前位（流密码）</font> |
| **<font style="color:rgb(15, 17, 21);">自同步能力</font>** | <font style="color:rgb(15, 17, 21);">❌</font><font style="color:rgb(15, 17, 21);"> 无</font> | <font style="color:rgb(15, 17, 21);">✅</font><font style="color:rgb(15, 17, 21);"> 有（可从错误恢复）</font> | <font style="color:rgb(15, 17, 21);">❌</font><font style="color:rgb(15, 17, 21);"> 无</font> |
| **<font style="color:rgb(15, 17, 21);">实现复杂度</font>** | <font style="color:rgb(15, 17, 21);">简单</font> | <font style="color:rgb(15, 17, 21);">中等</font> | <font style="color:rgb(15, 17, 21);">中等</font> |
| **<font style="color:rgb(15, 17, 21);">安全性等级</font>** | <font style="color:rgb(15, 17, 21);">⭐</font><font style="color:rgb(15, 17, 21);">☆☆☆☆（最低）</font> | <font style="color:rgb(15, 17, 21);">⭐⭐⭐</font><font style="color:rgb(15, 17, 21);">☆☆（中等）</font> | <font style="color:rgb(15, 17, 21);">⭐⭐⭐⭐</font><font style="color:rgb(15, 17, 21);">☆（较高）</font> |
| **<font style="color:rgb(15, 17, 21);">加密速度</font>** | <font style="color:rgb(15, 17, 21);">快</font> | <font style="color:rgb(15, 17, 21);">中等</font> | <font style="color:rgb(15, 17, 21);">快（可并行）</font> |
| **<font style="color:rgb(15, 17, 21);">内存需求</font>** | <font style="color:rgb(15, 17, 21);">低</font> | <font style="color:rgb(15, 17, 21);">低</font> | <font style="color:rgb(15, 17, 21);">低</font> |
| **<font style="color:rgb(15, 17, 21);">适用场景</font>** | <font style="color:rgb(15, 17, 21);">随机数据加密</font><font style="color:rgb(15, 17, 21);">   </font><font style="color:rgb(15, 17, 21);">单个数据块加密</font> | <font style="color:rgb(15, 17, 21);">通用数据加密</font><font style="color:rgb(15, 17, 21);">   </font><font style="color:rgb(15, 17, 21);">文件、消息加密</font> | <font style="color:rgb(15, 17, 21);">实时数据流</font><font style="color:rgb(15, 17, 21);">   </font><font style="color:rgb(15, 17, 21);">随机访问加密</font> |
| **<font style="color:rgb(15, 17, 21);">不适用场景</font>** | <font style="color:rgb(15, 17, 21);">结构化数据</font><font style="color:rgb(15, 17, 21);">   </font><font style="color:rgb(15, 17, 21);">重复模式数据</font> | <font style="color:rgb(15, 17, 21);">实时流数据（错误传播）</font> | <font style="color:rgb(15, 17, 21);">需要认证的场合</font> |
| **<font style="color:rgb(15, 17, 21);">标准化</font>** | <font style="color:rgb(15, 17, 21);">ISO 10116</font> | <font style="color:rgb(15, 17, 21);">ISO 10116</font> | <font style="color:rgb(15, 17, 21);">NIST SP 800-38A</font> |
| **<font style="color:rgb(15, 17, 21);">常见应用</font>** | <font style="color:rgb(15, 17, 21);">早期系统</font><font style="color:rgb(15, 17, 21);">   </font><font style="color:rgb(15, 17, 21);">简单协议</font> | <font style="color:rgb(15, 17, 21);">SSL/TLS（早期）</font><font style="color:rgb(15, 17, 21);">   </font><font style="color:rgb(15, 17, 21);">IPSec</font><font style="color:rgb(15, 17, 21);">   </font><font style="color:rgb(15, 17, 21);">文件加密</font> | <font style="color:rgb(15, 17, 21);">WiFi WPA2</font><font style="color:rgb(15, 17, 21);">   </font><font style="color:rgb(15, 17, 21);">磁盘加密</font><font style="color:rgb(15, 17, 21);">   </font><font style="color:rgb(15, 17, 21);">实时通信</font> |
| **<font style="color:rgb(15, 17, 21);">优缺点总结</font>** | **<font style="color:rgb(15, 17, 21);">优点</font>**<font style="color:rgb(15, 17, 21);">：简单、并行</font><font style="color:rgb(15, 17, 21);">   </font>**<font style="color:rgb(15, 17, 21);">缺点</font>**<font style="color:rgb(15, 17, 21);">：模式暴露、不安全</font> | **<font style="color:rgb(15, 17, 21);">优点</font>**<font style="color:rgb(15, 17, 21);">：隐藏模式、较安全</font><font style="color:rgb(15, 17, 21);">   </font>**<font style="color:rgb(15, 17, 21);">缺点</font>**<font style="color:rgb(15, 17, 21);">：串行、错误传播</font> | **<font style="color:rgb(15, 17, 21);">优点</font>**<font style="color:rgb(15, 17, 21);">：并行、无需填充</font><font style="color:rgb(15, 17, 21);">   </font>**<font style="color:rgb(15, 17, 21);">缺点</font>**<font style="color:rgb(15, 17, 21);">：需唯一nonce</font> |
| **<font style="color:rgb(15, 17, 21);">推荐程度</font>** | <font style="color:rgb(15, 17, 21);">❌</font><font style="color:rgb(15, 17, 21);"> 不推荐（已淘汰）</font> | <font style="color:rgb(15, 17, 21);">⚠️</font><font style="color:rgb(15, 17, 21);"> 谨慎使用（逐渐淘汰）</font> | <font style="color:rgb(15, 17, 21);">✅</font><font style="color:rgb(15, 17, 21);"> 推荐使用（现代应用）</font> |
## <font style="color:rgb(15, 17, 21);">DES解密</font>
DES的解密相比于AES来说更为简单，其表，盒，函数和执行流基本上都是不变的，唯一变的东西是密钥的输入方式，即加密时子密钥从索引第0个开始调度使用，解密时则刚好相反，从索引第15个开始调度使用。其他的流程全部与加密相同。

