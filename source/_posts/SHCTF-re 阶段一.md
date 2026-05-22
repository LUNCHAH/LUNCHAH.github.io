title: SHCTF-re 阶段一
date: 2026-2-7 
category: GAME
cover: /source/passagecover/12.jpg
---

## SHCTF-a_cup_of_tea
一道简单的tea算法题目，核心伪代码如下

```c
void __fastcall sub_1241(__int64 a1, __int64 p_src)
{
    int n15; // [rsp+1Ch] [rbp-4h]

    for ( n15 = 0; n15 <= 15; ++n15 )
        *(4LL * (n15 / 4) + a1) += *(n15 + p_src) << (8 * (n15 % 4));
}
```

```c
_BOOL8 __fastcall sub_1439(_DWORD *a1)
{
    sub_134E(a1, aWelcomeToShctf_0);              // "welcome_to_SHCTF"
    if ( *a1 != -1699360031 || a1[1] != -1120419751 )
        return 0;
    sub_134E(a1 + 2, aWelcomeToShctf_0);          // "welcome_to_SHCTF"
    return a1[2] == -1515845715 && a1[3] == -1804683212;
}
```

```c
__int64 __fastcall sub_134E(unsigned int *a1, _DWORD *p_welcome_to_SHCTF)
{
    __int64 result; // rax
    unsigned int v3; // [rsp+1Ch] [rbp-24h]
    unsigned int v4; // [rsp+20h] [rbp-20h]
    int v5; // [rsp+24h] [rbp-1Ch]
    unsigned int n0x1F; // [rsp+28h] [rbp-18h]

    v3 = *a1;
    v4 = a1[1];
    v5 = 0;
    for ( n0x1F = 0; n0x1F <= 0x1F; ++n0x1F )
    {
        v5 -= 1640531527;
        v3 += (v4 + v5) ^ (16 * v4 + *p_welcome_to_SHCTF) ^ ((v4 >> 5) + p_welcome_to_SHCTF[1]);
        v4 += (v3 + v5) ^ (16 * v3 + p_welcome_to_SHCTF[2]) ^ ((v3 >> 5) + p_welcome_to_SHCTF[3]);
    }
    *a1 = v3;
    result = v4;
    a1[1] = v4;
    return result;
}
```

第一个代码在对密钥进行字符转ASCII并且进行小端序排序，第二个和第三个则是正常的tea算法加密，逆向代码如下

```c
#include <bits/stdc++.h>
int main()
{
    char enc[]="welcome_to_SHCTF";
    uint32_t enc0[4]={0};
    for(int i=0;i<=15;i++)
    {
        enc0[i/4]+=enc[i]<<((i%4)*8);
    }
    for(int i=0;i<4;i++)
    {
        printf("0x%x,",enc0[i]);
    }
    return 0;
}

```

```c
#include <bits/stdc++.h>
using namespace std;

int main()
{
    uint32_t enc[4]={0x9ab5d2e1,0xbd37c059,0xA5A607AD,0x946EB834};
    uint32_t key[4]={0x636c6577,0x5f656d6f,0x535f6f74,0x46544348};
    uint32_t delta=0x9e3779b9;

    for(int n=0;n<4;n+=2)
    {
        uint32_t sum = delta << 5;
        for(int i=0;i<32;i++)
        {
            enc[n+1]-=(enc[n]+sum)^((enc[n]<<4)+key[2])^((enc[n]>>5)+key[3]);
            enc[n]  -=(enc[n+1]+sum)^((enc[n+1]<<4)+key[0])^((enc[n+1]>>5)+key[1]);
            sum -= delta;
        }
    }

    unsigned char *p = (unsigned char*)enc;
    for(int i=0;i<16;i++)
        putchar(p[i]);

    puts("");
}

```

<font style="background-color:#FBDE28;">依旧需要注意：</font>

**<font style="color:#DF2A3F;">TEA只能处理32位数据，所以在IDA中不要忘了shift+e</font>**

## SHCTF-where are you
一道很有意思的题目，藏了很多东西，也有很多假的东西和指引。

首先查找字符串定位到main逻辑

![](/blog_essay_picture/SHCTF-re阶段一/1.png)

函数sub_401FE0中找到

![](/blog_essay_picture/SHCTF-re阶段一/2.png)

逆向后得到假flag。

![](/blog_essay_picture/SHCTF-re阶段一/3.png)

找到可疑数据byte_404090，x 定位，找到真正逻辑

![](/blog_essay_picture/SHCTF-re阶段一/4.png)

这里需要下断点，在第二个virtualprotect处下断点

动调前的ipaddress

![](/blog_essay_picture/SHCTF-re阶段一/5.png)

动调后的ipaddress

![](/blog_essay_picture/SHCTF-re阶段一/6.png)

很明显发生变化了。下面就是一点点脑洞了，类似于处理花指令，这里也是CUP处理，然后可以得到真实逻辑

```c
int __cdecl sub_402090(const char *Buf1)
{
  unsigned int n24; // [esp+4h] [ebp-84h]
  int n36; // [esp+Ch] [ebp-7Ch]
  int n24_1; // [esp+10h] [ebp-78h]
  _BYTE p_Buffer[100]; // [esp+20h] [ebp-68h] BYREF

  n24 = strlen(Buf1);
  if ( n24 != 24 )
    goto LABEL_7;
  for ( n24_1 = 0; n24_1 < 24; ++n24_1 )
    Buf1[n24_1] ^= 0x22u;
  if ( !memcmp(Buf1, fakeflag, 0x18u) )
    return printf("%s", byte_404090);
LABEL_7:
  if ( n24 != 36 )
    // "Try again!\n"
    return printf("%s", aTryAgain);
  memset(p_Buffer, 0, sizeof(p_Buffer));
  memcpy(p_Buffer, Buf1, 0x24u);
  sub_4023A0(p_Buffer, 36, byte_404430, 16);
  for ( n36 = 0; n36 < 36; ++n36 )
  {
    if ( p_Buffer[n36] != byte_4031D4[n36] )
      // "Wrong Flag!\n"
      return printf("%s", aWrongFlag);
  }
  // "Congratulations! The flag is what you input.\n"
  return printf("%s", aCongratulation);
}
```

查看过后是RC4，在上面可以看到密钥及其处理，那么这题就出了。逆向脚本如下

```python
def rc4(data, key):
    S = list(range(256))
    j = 0

    # KSA
    for i in range(256):
        j = (j + S[i] + key[i % len(key)]) % 256
        S[i], S[j] = S[j], S[i]

    # PRGA
    i = j = 0
    out = bytearray()

    for byte in data:
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]

        K = S[(S[i] + S[j]) % 256]
        out.append(byte ^ K)

    return bytes(out)


cipher = bytes([
    0xea,0x64,0x65,0x15,0xff,0x0a,0xad,0x41,
    0x6f,0x81,0xa1,0x7b,0xa8,0xd0,0x5e,0x69,
    0x74,0x92,0x6a,0xe3,0xbd,0x6b,0x33,0x97,
    0x2d,0xc2,0xb5,0xfa,0xd0,0x8f,0x6d,0x3f,
    0xad,0x00,0xd0,0x91
])

key = bytes([
    0xE7,0xDA,0x07,0xAE,
    0xE5,0xFB,0xC3,0x0F,
    0x31,0xD8,0xDF,0x1B,
    0x3B,0x2E,0x5B,0x02
])

plain = rc4(cipher, key)

print("明文:", plain)
print("flag:", plain.decode(errors="ignore"))

```

## SHCTF-damagePE
题干如下

![](/blog_essay_picture/SHCTF-re阶段一/7.png)

附件丢进010中之后如下

![](/blog_essay_picture/SHCTF-re阶段一/8.png)

可以看到最大的问题在于PE被出题人改成了SH，改完后我们尝试运行并丢进IDA中

![](/blog_essay_picture/SHCTF-re阶段一/9.png)

得到第一部分flag

![](/blog_essay_picture/SHCTF-re阶段一/10.png)

得到第二部分flag和提示。提示说找到第二个IAT，我们该怎么找导入函数呢？IDA中有一个强大的功能叫Imports，这个视窗中可以找到所有的导入函数

![](/blog_essay_picture/SHCTF-re阶段一/11.png)

那答案就很明显了，两个都拿去试一下就能拿到flag了。

## SHCTF-Safe Image Encryption
一道图像加密逆向问题，原伪代码如下

```c
__int64 __fastcall main(int n3, char **a2, char **a3)
{
    int v3; // r13d
    __int64 v5; // r14
    unsigned int v6; // ebx
    int v7; // eax
    int v8; // r13d
    int v9; // r15d
    unsigned __int64 v10; // rcx
    char v11; // di
    char v12; // si
    unsigned __int64 v13; // rtt
    char v14; // r9
    unsigned __int16 v15; // bx
    char v16; // di
    unsigned __int8 v17; // si
    char v18; // dl
    int v19; // ebx
    int v20; // r12d
    char v22; // [rsp+15h] [rbp-2A3h]
    char v23; // [rsp+16h] [rbp-2A2h]
    char v24; // [rsp+17h] [rbp-2A1h]
    char v25; // [rsp+23h] [rbp-295h] BYREF
    unsigned int v26; // [rsp+24h] [rbp-294h] BYREF
    unsigned int v27; // [rsp+28h] [rbp-290h] BYREF
    char v28[4]; // [rsp+2Ch] [rbp-28Ch] BYREF
    _QWORD v29[4]; // [rsp+30h] [rbp-288h] BYREF
    __int64 v30; // [rsp+50h] [rbp-268h] BYREF
    unsigned __int64 n1003; // [rsp+58h] [rbp-260h]
    _QWORD v32[73]; // [rsp+70h] [rbp-248h] BYREF

    v32[65] = __readfsqword(0x28u);
    if ( n3 <= 3 )
    {
        __printf_chk(2, "Usage: %s <original.png> <key_file> <encrypted.png>\n", *a2);
        return 1;
    }
    else
    {
        v5 = sub_EB59(a2[1], &v26, &v27, v28, 4);
        if ( v5 )
        {
            std::ifstream::basic_ifstream(v32, a2[2], 8);
            sub_FE54(&v30, *(&v32[29] + *(v32[0] - 24LL)), 0xFFFFFFFFLL, 0, 0xFFFFFFFFLL, v29);
            if ( n1003 )
            {
                if ( n1003 == 1003 )
                {
                    sub_FF74(v29, (4 * v27 * v26), &v25);
                    v7 = v3;
                    v8 = 0;
                    v9 = v7;
                    while ( v27 > v8 )
                    {
                        v20 = 0;
                        v19 = v9;
                        while ( v26 > v20 )
                        {
                            v10 = (4 * (v20 + v8 * v26));
                            v22 = *(v5 + v10 + 1);
                            v23 = *(v5 + v10 + 2);
                            v24 = *(v5 + v10 + 3);
                            v11 = *(v30 + (v10 % n1003 + 1) % n1003);
                            v12 = *(v30 + (v10 % n1003 + 2) % n1003);
                            v13 = v10 % n1003 + 3;
                            v14 = v8 * v8 + v11;
                            LOBYTE(v15) = v20 * v20 + *(v30 + v10 % n1003) + (*(v30 + v10 % n1003) ^ 0xAA);
                            v16 = v12 ^ (v20 * v8) ^ (3 * v11);
                            HIBYTE(v15) = v16;
                            v17 = v14 + ((2 * v12) ^ 0x66);
                            v18 = (*(v30 + v13 % n1003) ^ 0x55) - 16;
                            v19 = (((*(v30 + v13 % n1003) ^ 0x55) - 16) << 24) | (v17 << 16) & 0xFFFFFF | v15;
                            *(v29[0] + v10) = *(v5 + v10) ^ (v20 * v20 + *(v30 + v10 % n1003) + (*(v30 + v10 % n1003) ^ 0xAA));
                            *(v29[0] + v10 + 1) = v22 ^ v16;
                            *(v29[0] + v10 + 2) = v23 ^ v17;
                            *(v29[0] + v10 + 3) = v24 ^ v18;
                            ++v20;
                        }
                        v9 = v19;
                        ++v8;
                    }
                    sub_FBD3(a2[3], v26, v27, 4, v29[0], 4 * v26);
                    puts("Encryption completed.");
                    sub_DB6D(v5);
                    sub_FEB0(v29);
                    v6 = 0;
                }
                else
                {
                    puts("Hint: key length is 1003 characters.");
                    v6 = 1;
                }
            }
            else
            {
                puts("Key text is empty!");
                v6 = 1;
            }
      std::string::_M_dispose(&v30);
      std::ifstream::~ifstream(v32);
    }
    else
    {
      puts("Error loading image.");
      return 1;
    }
  }
  return v6;
}
```

说实话我不是很懂这是什么玩意儿，逆向脚本如下

```python
from PIL import Image
import os

KEYLEN = 1003

# 常见背景候选（你也可以继续加）
BACKGROUND_CANDIDATES = [
    ("white",   (255, 255, 255, 255)),
    ("black",   (0, 0, 0, 255)),
    ("transparent", (0, 0, 0, 0)),
    ("gray128", (128, 128, 128, 255)),
    ("gray200", (200, 200, 200, 255)),
    ("blue",    (0, 0, 255, 255)),
    ("red",     (255, 0, 0, 255)),
    ("green",   (0, 255, 0, 255)),
]

def recover_key(img, known_pixel):
    """
    利用假设背景 known_pixel 来恢复 key
    """
    w, h = img.size
    data = bytearray(img.tobytes())

    key = [None] * KEYLEN

    for y in range(h):
        for x in range(w):

            idx = 4 * (x + y * w)

            c0, c1, c2, c3 = data[idx:idx+4]
            p0, p1, p2, p3 = known_pixel

            # 推 mask
            m0 = c0 ^ p0
            m3 = c3 ^ p3

            pos0 = idx % KEYLEN
            pos3 = (idx + 3) % KEYLEN

            # 爆破 k0
            if key[pos0] is None:
                for guess in range(256):
                    calc = (x*x + guess + (guess ^ 0xAA)) & 0xFF
                    if calc == m0:
                        key[pos0] = guess
                        break

            # 爆破 k3
            if key[pos3] is None:
                for guess in range(256):
                    calc = ((guess ^ 0x55) - 16) & 0xFF
                    if calc == m3:
                        key[pos3] = guess
                        break

    recovered = sum(k is not None for k in key)

    # 未恢复补0
    for i in range(KEYLEN):
        if key[i] is None:
            key[i] = 0

    return bytes(key), recovered


def decrypt(img, key):
    """
    完整解密 ELF 的 XOR mask
    """
    w, h = img.size
    data = bytearray(img.tobytes())

    for y in range(h):
        for x in range(w):

            idx = 4 * (x + y * w)

            k0 = key[idx % KEYLEN]
            k1 = key[(idx + 1) % KEYLEN]
            k2 = key[(idx + 2) % KEYLEN]
            k3 = key[(idx + 3) % KEYLEN]

            m0 = (x*x + k0 + (k0 ^ 0xAA)) & 0xFF
            m1 = (k2 ^ (x*y) ^ (3*k1)) & 0xFF
            m2 = (y*y + k1 + ((2*k2) ^ 0x66)) & 0xFF
            m3 = ((k3 ^ 0x55) - 16) & 0xFF

            data[idx]   ^= m0
            data[idx+1] ^= m1
            data[idx+2] ^= m2
            data[idx+3] ^= m3

    return Image.frombytes("RGBA", (w, h), bytes(data))


def score_image(img):
    """
    给解密结果打分：
    正常图片颜色分布不会像随机噪声一样均匀
    用简单方差统计判断
    """
    pixels = list(img.getdata())
    total = 0
    for r, g, b, a in pixels[::500]:  # 抽样
        total += abs(r - g) + abs(g - b)
    return total


if __name__ == "__main__":
    img = Image.open("encrypt.png").convert("RGBA")

    os.makedirs("outputs", exist_ok=True)

    best = None

    for name, bg in BACKGROUND_CANDIDATES:
        print(f"\n[*] Trying background: {name} {bg}")

        key, recovered = recover_key(img, bg)
        print(f"[+] Key recovered bytes: {recovered}/{KEYLEN}")

        dec = decrypt(img, key)

        out_file = f"outputs/decrypted_{name}.png"
        dec.save(out_file)

        sc = score_image(dec)
        print(f"[+] Score = {sc}")

        if best is None or sc > best[0]:
            best = (sc, out_file)

    print("\n==============================")
    print("[★] Best candidate output:")
    print("File:", best[1])
    print("==============================")
    print("Open outputs/ folder and check the decrypted images!")

```

