title: furryCTF
date: 2026-01-30
cover: https://s41.ax1x.com/2026/05/22/pmpCQBR.jpg
category: GAME
---

## furryCTF-Lua
Lua编码

```c
local b = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
local function dec(data)
    data = string.gsub(data, '[^' .. b .. '=]', '')
    return (data:gsub('.', function(x)
        if (x == '=') then return '' end
        local r, f = '', (b:find(x) - 1)
        for i = 6, 1, -1 do r = r .. (f % 2 ^ i - f % 2 ^ (i - 1) > 0 and '1' or '0') end
        return r;
    end):gsub('%d%d%d?%d?%d?%d?%d?%d?', function(x)
        if (#x ~= 8) then return '' end
        local c = 0
        for i = 1, 8 do c = c + (x:sub(i, i) == '1' and 2 ^ (8 - i) or 0) end
        return string.char(c)
    end))
end

local args = {...}

if #args ~= 1 then
    print("[-] use `lua hello.lua flag{fake_flag}`")
    return
end

print(load(dec("G0x1YVQAGZMNChoKBAgIeFYAAAAAAAAAAAAAACh3QAGAoa4BAA6gkwAAAFIAAAABgf9/tAEAAJUBA36vAYAHAQIAgEqBCQALAwAADgMGAYADAQAVBAWArwKABosEAAKOBAkDCwUAAg4FCgSABQAAFQYFgK8CgAaVBgWArwKABkQFBADEBAACnwQJBbAEBQ9EAwQBSQEKAE8BAABFgQEARoEAAEaBAQCGBIZ0YWJsZQSHaW5zZXJ0BIdzdHJpbmcEhWJ5dGUEhHN1YgNyAAAAAAAAAIEAAACBgKetAAADjQsAAAAOAAABiQABAAMBAQBEAAMCPAADADgBAIADAAIASAACALgAAIADgAIASAACAEcAAQCGBIZ0YWJsZQSHY29uY2F0BIItFL0yMC0zMC0xOS0yMS05LTM5LTQ1LTAtNDUtNjItNy03MC0zOC00NS02My03MC0xLTYtNjUtMzItODMtMTUEj1lvdSBBcmUgUmlnaHQhBIdXcm9uZyGCAAAAAQEAgICAgICAgICA"))(args[1]))
```

最底下解base64可得

![](/blog_essay_picture/furryCTF/1.png)

Lua码就可以看到了，我们使用unluac可得以下代码

```c
local L1_1, L2_1, L3_1, L4_1, L5_1, L6_1, L7_1, L8_1, L9_1, L10_1, L11_1, L12_1, L13_1
L1_1 = {}
L2_1 = 0
L3_1 = #A0_1
L3_1 = L3_1 - 1
L4_1 = 1
for L5_1 = L2_1, L3_1, L4_1 do
  L6_1 = table
  L6_1 = L6_1.insert
  L7_1 = L1_1
  L8_1 = L5_1 + 1
  L9_1 = string
  L9_1 = L9_1.byte
  L10_1 = string
  L10_1 = L10_1.sub
  L11_1 = A0_1
  L12_1 = L5_1 + 1
  L13_1 = L5_1 + 1
  L10_1, L11_1, L12_1, L13_1 = L10_1(L11_1, L12_1, L13_1)
  L9_1 = L9_1(L10_1, L11_1, L12_1, L13_1)
  L9_1 = L9_1 ~ 114
  L6_1(L7_1, L8_1, L9_1)
end

function L2_1()
  local L0_2, L1_2, L2_2
  L0_2 = table
  L0_2 = L0_2.concat
  L1_2 = L1_1
  L2_2 = "-"
  L0_2 = L0_2(L1_2, L2_2)
  if "20-30-19-21-9-39-45-0-45-62-7-70-38-45-63-70-1-6-65-32-83-15" == L0_2 then
    L0_2 = "You Are Right!"
    return L0_2
  else
    L0_2 = "Wrong!"
    return L0_2
  end
end

return L2_1()

```

概括就是异或114。

## furryCTF-分组密码
期待了很久的AES算法题。

进入main逻辑

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  FILE *Stream; // eax
  size_t n0x100; // eax
  char n4; // cl
  char *v6; // edx
  unsigned __int8 v7; // ah
  char v8; // ch
  char v9; // ah
  char v10; // al
  unsigned int n0x20; // eax
  __m128 *v12; // edx
  __m128 *v13; // esi
  char *v14; // eax
  int v15; // edx
  int n16; // edi
  char v17; // cl
  _OWORD *v18; // ecx
  _OWORD *v19; // edx
  unsigned int n28; // esi
  int v21; // edx
  bool v22; // cf
  unsigned __int8 v23; // al
  unsigned __int8 v24; // al
  unsigned __int8 v25; // al
  int n32; // eax
  char *p_fake_flag; // eax
  char ArgList; // [esp+0h] [ebp-220h]
  char ArgList_1; // [esp+0h] [ebp-220h]
  char ArgList_2; // [esp+0h] [ebp-220h]
  unsigned __int8 v32; // [esp+8h] [ebp-218h]
  unsigned __int8 v33; // [esp+9h] [ebp-217h]
  unsigned __int8 v34; // [esp+Ah] [ebp-216h]
  unsigned int n4_1; // [esp+Ch] [ebp-214h]
  unsigned int n0x20_1; // [esp+Ch] [ebp-214h]
  _DWORD v37[3]; // [esp+10h] [ebp-210h] BYREF
  unsigned int v38; // [esp+1Ch] [ebp-204h] BYREF
  _DWORD v39[4]; // [esp+C0h] [ebp-160h] BYREF
  _OWORD v40[2]; // [esp+D0h] [ebp-150h] BYREF
  _OWORD v41[2]; // [esp+F8h] [ebp-128h] BYREF
  char Buffer[16]; // [esp+118h] [ebp-108h] BYREF
  __int128 v43; // [esp+128h] [ebp-F8h]

  printf("input your flag:\n", ArgList);
  Stream = _acrt_iob_func(0);
  fgets(Buffer, 256, Stream);
  n0x100 = strcspn(Buffer, "\r\n");
  if ( n0x100 >= 0x100 )
    sub_401784();
  Buffer[n0x100] = 0;
  v40[0] = *Buffer;
  v40[1] = v43;
  if ( strlen(v40) < 0x20
    || *Buffer != 'OP'
    || Buffer[2] != 'F'
    || Buffer[3] != 'P'
    || *&Buffer[4] != 'TC'
    || Buffer[6] != 'F'
    || Buffer[7] != '{'
    || HIBYTE(v43) != '}' )
  {
    printf("flag length error", ArgList_1);
    exit(0);
  }
  n4 = 4;
  v39[0] = 0x278CF13A;
  v39[1] = 0xE2609BD4;
  v6 = &v38 + 1;
  v39[2] = 0xC3A75D11;
  v7 = 0xCD;
  v39[3] = 0x4EB8097F;
  v37[0] = 0xF3022201;
  v37[1] = 0xF7E6F544;
  v37[2] = 0xB0AB9A8;
  v38 = 0xFFEECDAC;
  for ( n4_1 = 4; n4_1 < 44; ++n4_1 )
  {
    v32 = *(v6 - 1);
    v33 = v6[1];
    v34 = v6[2];
    if ( (n4 & 3) != 0 )
    {
      v8 = v7;
    }
    else
    {
      v8 = byte_403158[v33];
      v33 = byte_403158[v34];
      v34 = byte_403158[v32];
      v32 = byte_403158[v7] ^ byte_403258[n4_1 >> 2];
    }
    v9 = *(v6 - 12);
    v6[3] = v32 ^ *(v6 - 13);
    v7 = v8 ^ v9;
    n4 = n4_1 + 1;
    v6[5] = v33 ^ *(v6 - 11);
    v10 = v34 ^ *(v6 - 10);
    v6[4] = v7;
    v6[6] = v10;
    v6 += 4;
  }
  n0x20 = 0;
  v12 = v39;
  n0x20_1 = 0;
  do
  {
    v13 = (v40 + n0x20);
    if ( v40 + n0x20 > &v12->m128_u32[3] + 3 || (&v13->m128_u32[3] + 3) < v12 )
    {
      *v13 = _mm_xor_ps(*v13, *v12);
    }
    else
    {
      v14 = v40 + n0x20;
      v15 = v12 - v13;
      n16 = 16;
      do
      {
        v17 = v14[v15];
        *v14++ ^= v17;
        --n16;
      }
      while ( n16 );
    }
    sub_4010B0(v13, v37);
    v12 = v13;
    n0x20 = n0x20_1 + 16;
    n0x20_1 = n0x20;
  }
  while ( n0x20 < 0x20 );
  v18 = v41;
  v41[0] = xmmword_403270;
  v19 = v40;
  n28 = 28;
  v41[1] = xmmword_403280;
  while ( *v18 == *v19 )
  {
    v18 = (v18 + 4);
    v19 = (v19 + 4);
    v22 = n28 < 4;
    n28 -= 4;
    if ( v22 )
    {
      v21 = 0;
      goto LABEL_33;
    }
  }
  v22 = *v18 < *v19;
  if ( *v18 == *v19
    && (v23 = *(v18 + 1), v22 = v23 < *(v19 + 1), v23 == *(v19 + 1))
    && (v24 = *(v18 + 2), v22 = v24 < *(v19 + 2), v24 == *(v19 + 2))
    && (v25 = *(v18 + 3), v22 = v25 < *(v19 + 3), v25 == *(v19 + 3)) )
  {
    v21 = 0;
  }
  else
  {
    v21 = v22 ? -1 : 1;
  }
LABEL_33:
  n32 = 32;
  do
    --n32;
  while ( n32 );
  p_fake_flag = "yes";
  if ( v21 )
    p_fake_flag = "fake flag";
  printf(p_fake_flag, ArgList_1);
  printf("\n", ArgList_2);
  return 0;
}
```

首先说明了flag格式，遍历所有函数后确认这是略有魔改的AES算法128位密钥模式，那么main逻辑里面那个地方就是密钥扩展混淆逻辑。

可以确认xmmword_403270和xmmword_403280是目标密文

```c
int __fastcall sub_4010B0(char *a1, int a2)
{
  int n4; // edx
  char *v4; // edi
  int v5; // esi
  int v6; // ebx
  char *v7; // ecx
  char v8; // al
  char *v9; // ebx
  int n4_1; // esi
  char v11; // cl
  char v12; // al
  char v13; // cl
  char v14; // al
  char v15; // cl
  char v16; // al
  char v17; // cl
  char *v18; // edi
  char v19; // bh
  char v20; // ch
  char v21; // dl
  char v22; // dh
  int n4_2; // edx
  _BYTE *v24; // esi
  char *v25; // eax
  char v26; // cl
  bool v27; // zf
  char v28; // cl
  int n4_3; // edx
  char v30; // al
  char v31; // cl
  char v32; // al
  char v33; // cl
  char v34; // al
  char v35; // cl
  char v36; // al
  int v37; // ecx
  char v38; // al
  int result; // eax
  int v41; // [esp+10h] [ebp-10h]
  int n9; // [esp+14h] [ebp-Ch]
  _BYTE *v43; // [esp+18h] [ebp-8h]
  char v44; // [esp+1Fh] [ebp-1h]

  n4 = 4;
  v4 = a1;
  v41 = a2;
  v5 = a2 + 3;
  v6 = a2 - a1;
  v7 = a1 + 1;
  do
  {
    v8 = *(v5 - 3);
    v5 += 4;
    *(v7 - 1) ^= v8;
    v7 += 4;
    *(v7 - 4) ^= v7[v6 - 4];
    *(v7 - 3) ^= *(v5 - 5);
    *(v7 - 2) ^= *(v5 - 4);
    --n4;
  }
  while ( n4 );
  v9 = v4 + 2;
  n9 = 9;
  v43 = (v41 + 18);
  do
  {
    sub_401050(v4);
    n4_1 = 4;
    v11 = v4[1];
    v4[1] = v4[5];
    v4[5] = v4[9];
    v4[9] = v4[13];
    v12 = v4[10];
    v4[13] = v11;
    v13 = *v9;
    *v9 = v12;
    v14 = v4[14];
    v4[10] = v13;
    v15 = v4[6];
    v4[6] = v14;
    v16 = v4[15];
    v4[14] = v15;
    v17 = v4[3];
    v4[3] = v16;
    v4[15] = v4[11];
    v4[11] = v4[7];
    v4[7] = v17 ^ 0x66;
    v18 = v9;
    do
    {
      v19 = v18[1];
      v18 += 4;
      v20 = *(v18 - 4);
      v21 = *(v18 - 5);
      v44 = *(v18 - 6);
      v22 = v21 ^ v44 ^ v20 ^ v19;
      *(v18 - 6) = v22 ^ v44 ^ (2 * (v21 ^ v44)) ^ (0x1B * ((v21 ^ v44) >> 7));
      *(v18 - 5) = v22 ^ v21 ^ (2 * (v20 ^ v21)) ^ (0x1B * ((v20 ^ v21) >> 7));
      *(v18 - 4) = v22 ^ v20 ^ (2 * (v20 ^ v19)) ^ (0x1B * ((v20 ^ v19) >> 7));
      *(v18 - 3) = v22 ^ v19 ^ (2 * (v19 ^ v44)) ^ (0x1B * ((v19 ^ v44) >> 7));
      --n4_1;
    }
    while ( n4_1 );
    v4 = a1;
    n4_2 = 4;
    v24 = v43;
    v9 = a1 + 2;
    v25 = a1 + 2;
    do
    {
      v25 += 4;
      *(v25 - 6) ^= *(v24 - 2);
      *(v25 - 5) ^= *(v24 - 1);
      *(v25 - 4) ^= *v24;
      v26 = v24[1];
      v24 += 4;
      *(v25 - 3) ^= v26;
      --n4_2;
    }
    while ( n4_2 );
    v27 = n9-- == 1;
    v43 = v24;
  }
  while ( !v27 );
  sub_401050(a1);
  v28 = a1[1];
  n4_3 = 4;
  a1[1] = a1[5];
  a1[5] = a1[9];
  a1[9] = a1[13];
  v30 = a1[10];
  a1[13] = v28;
  v31 = *v9;
  *v9 = v30;
  v32 = a1[14];
  a1[10] = v31;
  v33 = a1[6];
  a1[6] = v32;
  v34 = a1[15];
  a1[14] = v33;
  v35 = a1[3];
  a1[3] = v34;
  a1[15] = a1[11];
  v36 = a1[7];
  a1[7] = v35 ^ 0x66;
  a1[11] = v36;
  v37 = v41 + 161;
  do
  {
    v38 = *(v37 - 1);
    v37 += 4;
    *(v9 - 2) ^= v38;
    v9 += 4;
    *(v9 - 5) ^= *(v37 - 4);
    *(v9 - 4) ^= *(v37 - 3);
    result = *(v37 - 2);
    *(v9 - 3) ^= result;
    --n4_3;
  }
  while ( n4_3 );
  return result;
}
```

可知sub_4010B0为AES算法中的轮换和列混淆函数。现在来看看传参，这个函数传入了v13和v37，由于main逻辑存在以下代码

```c
  n0x20 = 0;
  v12 = v39;
  n0x20_1 = 0;
  do
  {
    v13 = (v40 + n0x20);
    if ( v40 + n0x20 > &v12->m128_u32[3] + 3 || (&v13->m128_u32[3] + 3) < v12 )
    {
      *v13 = _mm_xor_ps(*v13, *v12);
    }
    else
    {
      v14 = v40 + n0x20;
      v15 = v12 - v13;
      n16 = 16;
      do
      {
        v17 = v14[v15];
        *v14++ ^= v17;
        --n16;
      }
      while ( n16 );
```

可以推断v13为输入明文，<font style="color:#DF2A3F;">而由于这个mm_xor_ps()是SSE指令，作用是将128位寄存器按位异或存储到另一个寄存器上，即v13^=v12</font>，所以v12，即v39，就是初始向量。if的条件判断是放溢出，保证数据安全，无需过多在意。那么此时就可以确定v37是密钥。

那么现在我们密钥，初始向量，密文都有了，由于密钥混淆扩展函数写在main逻辑里面，那么那里的byte_403258就无疑是轮常数了。好，现在数据全齐，逻辑也大多摸清楚。检查魔改。

发现在轮换逻辑中

```c
    sub_401050(v4);
    n4_1 = 4;
    v11 = v4[1];
    v4[1] = v4[5];
    v4[5] = v4[9];
    v4[9] = v4[13];
    v12 = v4[10];
    v4[13] = v11;
    v13 = *v9;
    *v9 = v12;
    v14 = v4[14];
    v4[10] = v13;
    v15 = v4[6];
    v4[6] = v14;
    v16 = v4[15];
    v4[14] = v15;
    v17 = v4[3];
    v4[3] = v16;
    v4[15] = v4[11];
    v4[11] = v4[7];
    v4[7] = v17 ^ 0x66;
    v18 = v9;
```

与普通的AES轮换不同，这里异或了一个常数0x66。函数sub_401050为

```c
char *__thiscall sub_401050(char *this)
{
  char *result; // eax
  int n4; // edx
  int v3; // ecx

  result = this + 2;
  n4 = 4;
  do
  {
    v3 = *(result - 2);
    result += 4;
    *(result - 6) = byte_403158[v3];
    *(result - 5) = byte_403158[*(result - 5)];
    *(result - 4) = byte_403158[*(result - 4)];
    *(result - 3) = byte_403158[*(result - 3)];
    --n4;
  }
  while ( n4 );
  return result;
}
```

那么很明显这个就是标准的S盒查表替换混淆。

最终逆向脚本如下

```c
#include <stdio.h>
#include <stdint.h>
#include <string.h>

/* ===========================
   AES-128 CBC Decrypt (Magic)
   =========================== */

static const uint8_t sbox[256] = {
    0x63,0x1E,0x77,0x7B,0xF2,0x6B,0x6F,0xC5,0x30,0x01,0x67,0x2B,0xFE,0xD7,0xAB,0x76,
    0xCA,0x82,0xC9,0x7D,0xFA,0x59,0x47,0xF0,0xAD,0xD4,0xA2,0xAF,0x9C,0xA4,0x72,0xC0,
    0xB7,0xFD,0x93,0x26,0x36,0x3F,0xF7,0xCC,0x34,0xA5,0xE5,0xF1,0x71,0xD8,0x31,0x15,
    0x04,0xC7,0x23,0xC3,0x18,0x96,0x05,0x9A,0x07,0x12,0x80,0xE2,0xEB,0x27,0xB2,0x75,
    0x09,0x83,0x2C,0x1A,0x1B,0x6E,0x5A,0xA0,0x52,0x3B,0xD6,0xB3,0x29,0xE3,0x2F,0x84,
    0x53,0xD1,0x00,0xED,0x20,0xFC,0xB1,0x5B,0x6A,0xCB,0xBE,0x39,0x4A,0x4C,0x58,0xCF,
    0xD0,0xEF,0xAA,0xFB,0x43,0x4D,0x33,0x85,0x45,0xF9,0x02,0x7F,0x50,0x3C,0x9F,0xA8,
    0x51,0xA3,0x40,0x8F,0x92,0x9D,0x38,0xF5,0xBC,0xB6,0xDA,0x21,0x10,0xFF,0xF3,0xD2,
    0xCD,0x0C,0x13,0xEC,0x5F,0x97,0x44,0x17,0xC4,0xA7,0x7E,0x3D,0x64,0x5D,0x19,0x73,
    0x60,0x81,0x4F,0xDC,0x22,0x2A,0x90,0x88,0x46,0xEE,0xB8,0x14,0xDE,0x5E,0x0B,0xDB,
    0xE0,0x32,0x3A,0x0A,0x49,0x06,0x24,0x5C,0xC2,0xD3,0xAC,0x62,0x91,0x95,0xE4,0x79,
    0xE7,0xC8,0x37,0x6D,0x8D,0xD5,0x4E,0xA9,0x6C,0x56,0xF4,0xEA,0x65,0x7A,0xAE,0x08,
    0xBA,0x78,0x25,0x2E,0x1C,0xA6,0xB4,0xC6,0xE8,0xDD,0x74,0x1F,0x4B,0xBD,0x8B,0x8A,
    0x70,0x3E,0xB5,0x66,0x48,0x03,0xF6,0x0E,0x61,0x35,0x57,0xB9,0x86,0xC1,0x1D,0x9E,
    0xE1,0xF8,0x98,0x11,0x69,0xD9,0x8E,0x94,0x9B,0x7C,0x87,0xE9,0xCE,0x55,0x28,0xDF,
    0x8C,0xA1,0x89,0x0D,0xBF,0xE6,0x42,0x68,0x41,0x99,0x2D,0x0F,0xB0,0x54,0xBB,0x16
};

static uint8_t inv_sbox[256];
static const uint8_t Rcon[11] = {0x07,0x09,0x12,0x04,0x08,0x10,0x21,0x40,0x88,0x1B,0x36};

/* ===== Init InvSbox ===== */
static void InitInvSBox() {
    for(int i=0;i<256;i++)
        inv_sbox[sbox[i]] = i;
}

static uint8_t xtime(uint8_t x) {
    return (x << 1) ^ ((x >> 7) * 0x1B);
}


/* ===== GF Multiply ===== */
static uint8_t gmul(uint8_t x, uint8_t m) {

    uint8_t x2 = xtime(x);      // x * 2
    uint8_t x4 = xtime(x2);     // x * 4
    uint8_t x8 = xtime(x4);     // x * 8

    if (m == 0x09) return x8 ^ x;           // 8x + x
    if (m == 0x0B) return x8 ^ x2 ^ x;      // 8x + 2x + x
    if (m == 0x0D) return x8 ^ x4 ^ x;      // 8x + 4x + x
    if (m == 0x0E) return x8 ^ x4 ^ x2;     // 8x + 4x + 2x

    return 0;
}


/* ===== KeyExpansion ===== */
static void KeyExpansion(const uint8_t *key, uint8_t *RoundKey) {
    memcpy(RoundKey, key, 16);

    uint8_t temp[4];
    int rcon_idx = 1;

    for(int i=16;i<176;i+=4) {
        memcpy(temp, RoundKey+i-4, 4);

        if(i % 16 == 0) {
            uint8_t t=temp[0];
            temp[0]=temp[1];
            temp[1]=temp[2];
            temp[2]=temp[3];
            temp[3]=t;

            temp[0]=sbox[temp[0]];
            temp[1]=sbox[temp[1]];
            temp[2]=sbox[temp[2]];
            temp[3]=sbox[temp[3]];

            temp[0] ^= Rcon[rcon_idx++];
        }

        for(int j=0;j<4;j++)
            RoundKey[i+j] = RoundKey[i-16+j] ^ temp[j];
    }
}

/* ===== AddRoundKey ===== */
static void AddRoundKey(uint8_t *state, uint8_t *rk) {
    for(int i=0;i<16;i++)
        state[i] ^= rk[i];
}

/* ===== InvSubBytes ===== */
static void InvSubBytes(uint8_t *state) {
    for(int i=0;i<16;i++)
        state[i] = inv_sbox[state[i]];
}

/* ===== InvShiftRows + Magic ===== */
static void InvShiftRows_Magic(uint8_t *s) {
    uint8_t tmp[16];
    memcpy(tmp,s,16);

    s[1]=tmp[13]; s[5]=tmp[1]; s[9]=tmp[5]; s[13]=tmp[9];
    s[2]=tmp[10]; s[6]=tmp[14]; s[10]=tmp[2]; s[14]=tmp[6];

    s[3]=tmp[7]^0x66;
    s[7]=tmp[11];
    s[11]=tmp[15];
    s[15]=tmp[3];
}

/* ===== InvMixColumns ===== */
static void InvMixColumns(uint8_t *s) {
    uint8_t t[16];
    for(int c=0;c<4;c++) {
        int i=c*4;
        t[i+0]=gmul(s[i],0x0e)^gmul(s[i+1],0x0b)^gmul(s[i+2],0x0d)^gmul(s[i+3],0x09);
        t[i+1]=gmul(s[i],0x09)^gmul(s[i+1],0x0e)^gmul(s[i+2],0x0b)^gmul(s[i+3],0x0d);
        t[i+2]=gmul(s[i],0x0d)^gmul(s[i+1],0x09)^gmul(s[i+2],0x0e)^gmul(s[i+3],0x0b);
        t[i+3]=gmul(s[i],0x0b)^gmul(s[i+1],0x0d)^gmul(s[i+2],0x09)^gmul(s[i+3],0x0e);
    }
    memcpy(s,t,16);
}

/* ===== AES Block Decrypt ===== */
static void AES_decrypt_block(uint8_t *block, uint8_t *rk) {
    AddRoundKey(block, rk+160);

    for(int round=9; round>=1; round--) {
        InvShiftRows_Magic(block);
        InvSubBytes(block);
        AddRoundKey(block, rk+round*16);
        InvMixColumns(block);
    }

    InvShiftRows_Magic(block);
    InvSubBytes(block);
    AddRoundKey(block, rk);
}

/* ===========================
   Main CBC Decrypt
   =========================== */
int main() {
    InitInvSBox();

    uint8_t key[16] = {
        0x01,0x22,0x02,0xF3,
        0x44,0xF5,0xE6,0xF7,
        0xA8,0xB9,0x0A,0x0B,
        0xAC,0xCD,0xEE,0xFF
    };

    uint8_t iv[16] = {
        0x3A,0xF1,0x8C,0x27,
        0xD4,0x9B,0x60,0xE2,
        0x11,0x5D,0xA7,0xC3,
        0x7F,0x09,0xB8,0x4E
    };

    uint8_t ciphertext[32] = {
        0x2B,0x1B,0xC9,0x99,0xBE,0xBD,0xE6,0x85,
        0x30,0xC9,0x09,0x10,0x26,0x3C,0xF3,0x26,
        0x62,0xE7,0xD0,0xED,0xE0,0x9F,0x07,0xCF,
        0x3E,0x7E,0x21,0xBD,0xF7,0x29,0x11,0x9E
    };

    uint8_t RoundKey[176];
    KeyExpansion(key, RoundKey);

    uint8_t flag[33]={0};
    uint8_t block[16];

    /* Block1 */
    memcpy(block,ciphertext,16);
    AES_decrypt_block(block,RoundKey);
    for(int i=0;i<16;i++)
        flag[i]=block[i]^iv[i];

    /* Block2 */
    memcpy(block,ciphertext+16,16);
    AES_decrypt_block(block,RoundKey);
    for(int i=0;i<16;i++)
        flag[16+i]=block[i]^ciphertext[i];

    printf("Flag: %s\n",flag);
    return 0;
}

```

噢噢对，值得注意的一点是，<font style="color:#DF2A3F;">这里的轮换函数与标准的AES也不同，所以在逆向的时候需要按着IDA里面的逻辑来，以及异或的位置不要放错了</font>

## furryCTF-TimeManager
一道蛮有意思的题目，不算特别难，main逻辑如下

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  int n; // [rsp+Ch] [rbp-34h]
  time_t v5; // [rsp+10h] [rbp-30h]
  time_t v6; // [rsp+20h] [rbp-20h]
  time_t v7; // [rsp+28h] [rbp-18h]

  v6 = time(0);
  v5 = v6;
  puts("Welcome to the Wired, Lain.");
  puts("Your NAVI is ready to assist you.");
  puts("Just wait 3 hours, and you will see the flag.");
  for ( n = 0; n <= 10799; ++n )
  {
    sleep(1u);
    puts((&mystr)[n % 116]);                    // "The Wired is the upper directory of the real world."
    v7 = time(0);
    if ( v7 != v5 + 1 )
      exit(2);
    srand(v7 + dword_6043 - v6);
    cipher[n % 128] ^= rand();                  // "!q"
    cipher[n % 17] ^= rand();                   // "!q"
    v5 = v7;
  }
  puts("\nWow, u can really do it");
  puts(cipher);                                 // "!q"
  return 0;
}
```

摸清楚几个地方

v6和v7是不同的，即使它们都被赋值为time(0),但是由于所在位置的不同，所以它们触发的时间也就不同，那么v6和v7就是不一样的。由于程序进入循环后会sleep一秒，所以srand里面表达式的意义就在于循环次数加dword_6043，然后就是常规异或

```c
// attributes: thunk
unsigned int sleep(unsigned int seconds)
{
  return sleep(seconds);
}
```

但是值得注意，<font style="color:#DF2A3F;">这题是在Linux环境下产出的，所以在运行解密脚本的时候也要在Linux环境下运行，否则就会出现一大堆乱码。</font>

逆向脚本如下

```c
import ctypes
import sys

# 加载 C 标准库 (Linux)
try:
    libc = ctypes.CDLL("libc.so.6")
except:
    # 如果是在 Windows 上尝试 (可能会失败，因为 rand 实现不同)
    try:
        libc = ctypes.cdll.msvcrt
    except:
        print("无法加载 libc")
        sys.exit()

# 原始数据
raw_data = [
    0x21, 0x71, 0xD8, 0xED, 0xDD, 0xA9, 0xCB, 0x02, 0xFB, 0x3E, 0x77, 0xDF, 0x96, 0x6D, 0x6D, 0x29,
    0x69, 0xCF, 0xDC, 0xC1, 0xEA, 0xBE, 0x23, 0xAA, 0x1D, 0xE4, 0x25, 0xD4, 0x9D, 0x3A, 0x8A, 0x50,
    0xCA, 0xD6, 0x86, 0x48, 0x21, 0xFB, 0xD5, 0x75, 0x44, 0x49, 0x63, 0x1B, 0x30, 0xB8, 0x18, 0x39,
    0x22, 0xB2, 0x43, 0xC8, 0x82, 0x06, 0xDC, 0x1D, 0x88, 0xBF, 0x1A, 0xB8, 0x0C, 0xFB, 0x54, 0xC9,
    0x57, 0x7A, 0xB3, 0xDD, 0x94, 0x70, 0x06, 0xAD, 0x41, 0x8F, 0x13, 0x7B, 0x66, 0x31, 0x90, 0xF7,
    0xEC, 0xDC, 0xB7, 0xE8, 0xC4, 0x60, 0x3C, 0x69, 0xBD, 0xD8, 0x8E, 0x9B, 0xAB, 0xA0, 0x50, 0x07,
    0xCD, 0x40, 0x7C, 0xFE, 0x30, 0xF2, 0xCA, 0x45, 0xE2, 0x53, 0x7D, 0x19, 0xD8, 0x16, 0x79, 0xBD,
    0x47, 0xD3, 0x93, 0x33, 0xCD, 0xCB, 0xD4, 0xCA, 0xDE, 0x38, 0xB5, 0xC5, 0x36, 0xFF, 0xA3, 0x87
]

cipher = bytearray(raw_data)
dword_6043 = 0xBEADDEEF

# 模拟循环
for n in range(10800):
    # 计算种子
    # ctypes.c_uint 确保溢出行为与 C 语言一致
    seed = ctypes.c_uint((n + 1) + dword_6043).value

    libc.srand(seed)

    # 第一次 XOR
    r1 = libc.rand()
    cipher[n % 128] ^= (r1 & 0xFF)  # 只取低8位

    # 第二次 XOR
    r2 = libc.rand()
    cipher[n % 17] ^= (r2 & 0xFF)

print("Flag:", cipher.decode('utf-8', errors='ignore'))
```

C语言逆向代码如下

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <string.h>

// 从题目 .data 段提取的原始字节数据
unsigned char cipher[] = {
    0x21, 0x71, 0xD8, 0xED, 0xDD, 0xA9, 0xCB, 0x02, 0xFB, 0x3E, 0x77, 0xDF, 0x96, 0x6D, 0x6D, 0x29,
    0x69, 0xCF, 0xDC, 0xC1, 0xEA, 0xBE, 0x23, 0xAA, 0x1D, 0xE4, 0x25, 0xD4, 0x9D, 0x3A, 0x8A, 0x50,
    0xCA, 0xD6, 0x86, 0x48, 0x21, 0xFB, 0xD5, 0x75, 0x44, 0x49, 0x63, 0x1B, 0x30, 0xB8, 0x18, 0x39,
    0x22, 0xB2, 0x43, 0xC8, 0x82, 0x06, 0xDC, 0x1D, 0x88, 0xBF, 0x1A, 0xB8, 0x0C, 0xFB, 0x54, 0xC9,
    0x57, 0x7A, 0xB3, 0xDD, 0x94, 0x70, 0x06, 0xAD, 0x41, 0x8F, 0x13, 0x7B, 0x66, 0x31, 0x90, 0xF7,
    0xEC, 0xDC, 0xB7, 0xE8, 0xC4, 0x60, 0x3C, 0x69, 0xBD, 0xD8, 0x8E, 0x9B, 0xAB, 0xA0, 0x50, 0x07,
    0xCD, 0x40, 0x7C, 0xFE, 0x30, 0xF2, 0xCA, 0x45, 0xE2, 0x53, 0x7D, 0x19, 0xD8, 0x16, 0x79, 0xBD,
    0x47, 0xD3, 0x93, 0x33, 0xCD, 0xCB, 0xD4, 0xCA, 0xDE, 0x38, 0xB5, 0xC5, 0x36, 0xFF, 0xA3, 0x87
};

int main() {
    int n;
    // 题目中的 dword_6043
    unsigned int dword_6043 = 0xBEADDEEF; 

    printf("Starting decryption...\n");

    // 模拟循环，范围 0 到 10799
    for (n = 0; n <= 10799; ++n) {
        // 核心逻辑还原：
        // 在原程序中：srand(v7 + dword_6043 - v6);
        // 因为 v7 - v6 = n + 1 (流逝的时间秒数)
        // 所以 seed = (n + 1) + dword_6043
        unsigned int seed = (n + 1) + dword_6043;
        srand(seed);

        // 执行两次异或操作
        // rand() 返回 int，但在 C 中与 char 异或时只取低位有效
        cipher[n % 128] ^= rand();
        cipher[n % 17] ^= rand();
    }

    printf("Decryption complete.\nFlag: %s\n", cipher);
    
    return 0;
}
```

