title: FridaNativeHook(持续更新版)
date: 2026-8-11
category: LEARN
cover: /passagecover/14.jpg
---

## 前言
按理来说这篇文章可以放在FridaHook里，但由于是一个难点，变式多样，很多地方和FridaHook的Java层hook不一样，所以单开一篇，方便知识点的查阅学习。<font style="color:#DF2A3F;">由于笔者也在逐渐学习，所以文章会逐步丰富更新，同时也难免出现一些错误遗落，还请读者斧正和批判性阅读</font>。

## 什么是NativeHook？
在了解native前我们需要深入了解一下什么是so文件

### 什么是so文件？
正如前面的初入门所说，so文件是必定包含在apk中的一个文件，是app的本地功能模块，它主要的作用在于用C或C++语法来撰写一些底层的验证或算法逻辑，因为C语言更接近CPU，编译运行速度比Java更快，也更节约资源。它为程序提供运行时调用和加载的二进制库，是底层功能，性能优化与核心功能实现的基础

### so文件参与的执行流
那么so文件在程序运行时所参与的执行流是怎么样的？如下图所示

```plain
                Android App 启动
                       │
                       ▼
              Application / Activity
                       │
                       ▼
              Java / Kotlin 代码
                       │
                       │
             System.loadLibrary()
                       │
                       ▼
               Android Linker
               (linker / linker64)
                       │
                       ▼
              加载 libxxx.so
                       │
                       ▼
              so 映射到进程内存
                       │
              ┌────────┴────────┐
              │                 │
              ▼                 ▼
        初始化函数            JNI注册
      .init / .init_array       │
              │                 │
              └────────┬────────┘
                       ▼
                so处于可执行状态
                       │
                       ▼
              Java 调用 native 方法
                       │
                       ▼
              JNI 函数 / Native函数
                       │
                       ▼
             so 内部真实代码执行
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
           算法       逻辑      系统API
             │         │         │
             └─────────┼─────────┘
                       ▼
                 返回结果
                       │
                       ▼
                Java / Kotlin
                       │
                       ▼
                  App继续运行
```

## Hook？
现在我们来说说该如何进行nativehook。

程序在正常跑的情况下会将我们的输入传入so，so接收之后经过处理再传出，而我们hook的点主要就在于传入so和so处理这两个地方。

首先我们会用到新的FridaAPI：Interceptor.attach，onEnter和 Module.findGlobalExportByName() 

### FridaAPI
首先来讲讲Interceptor.attach这个API，用法为

```javascript
Interceptor.attach(targer_address,{
    ......
})
```

这是nativehook中较为核心的API之一，意义为将大括号内的hook代码插入替代掉目标地址中的代码。相当于一个Frida拦截器，运行脚本后，当程序运行到我们指定地址时就会触发拦截机制，将函数入口从源码改为我们Frida中的代码，但是Interceptor.attach无法识别函数名，所以我们需要填具体的地址。

如何找到具体地址呢？Module.findGlobalExportByName() 就是一个很好的帮手。这个API的作用在于遍历全进程的到处函数表，找到指定函数的地址然后返回地址，这个API比较适用于找strcmp，malloc，memcpy，memset等全局函数。

与Module.findGlobalExportByName()的全进程搜索相反，Process.getModuleByName与xxx.getExportByName的组合就是指定导出表来搜索，示例用法如下

```javascript
var libc = Process.getModuleByName("libc.so");
var addr = libc.getExportByName("strcmp");
```

以上示例用法相当于指定在libc.so中找到strcmp这个函数并返回其地址。

值得注意的是，在nativehook中，除去找到hook点，最重要的第一步就是找到hook点的地址。

接下来说说onEnter(args)这个API

onEnter的作用在于在函数进入时执行其内部函数，而args就是我们所有hook函数的参数，因为这里hook了函数的参数，所以后面我们需要将args来具体赋值，示例如下

```javascript
onEnter(args)
{
    var str1=args[0].readCString();
    var str2=args[1].readCString();

    if(str1=="Hello")
        {
        console.log("input:",str1);
        console.log("\n");
        console.log("flag:",str2);
        }
}
```

与onEnter相反，onLeave(retval)就是在函数返回前执行，笔者暂时还未遇到就不多赘述。

### 方法论
那我们在看到apk时该如何判断是否要nativehook，要的话又该hook哪个点呢？

看是否要native分析的关键点在于apk中是否调用了本地so文件，可以通过JADX查看，而是否要nativehook则要看so文件中的具体代码逻辑。而找到hook点的关键在于看so逻辑中的返回和比较。因为一般来说so文件的逻辑都是入口->处理逻辑->储存赋值->比较->返回，其中比较和返回是最有可能出现flag或与flag相关的地方。

