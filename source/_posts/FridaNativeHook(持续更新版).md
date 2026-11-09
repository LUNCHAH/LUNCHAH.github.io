title: FridaNativeHook(持续更新版)
date: 2026-8-11
category: LEARN
cover: /passagecover/14.jpg
---

## 前言
按理来说这篇文章可以放在FridaHook里，但由于是一个难点，变式多样，很多地方和FridaHook的Java层hook不一样，所以单开一篇，方便知识点的查阅学习。<font style="color:#DF2A3F;">由于笔者也在逐渐学习，所以文章会逐步丰富更新，同时也难免出现一些错误遗落，还请读者斧正和批判性阅读</font>。最近更新于2026.11.9。

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
首先来讲讲**Interceptor.attach**这个API，用法为

```javascript
Interceptor.attach(targer_address,{
    ......
})
```

这是nativehook中较为核心的API之一，意义为将大括号内的hook代码插入替代掉目标地址中的代码。相当于一个Frida拦截器，运行脚本后，当程序运行到我们指定地址时就会触发拦截机制，将函数入口从源码改为我们Frida中的代码，但是Interceptor.attach无法识别函数名，所以我们需要填具体的地址。

如何找到具体地址呢？**Module.findGlobalExportByName() **就是一个很好的帮手。这个API的作用在于遍历全进程的到处函数表，找到指定函数的地址然后返回地址，这个API比较适用于找strcmp，malloc，memcpy，memset等全局函数。

与Module.findGlobalExportByName()的全进程搜索相反，**Process.getModuleByName**与**xxx.getExportByName**的组合就是在指定文件的模块中进行搜索，示例用法如下

```javascript
var libc = Process.getModuleByName("libc.so");
var addr = libc.getExportByName("strcmp");
```

以上示例用法相当于指定在libc.so中找到strcmp这个函数并返回其地址。

值得注意的是，在nativehook中，除去找到hook点，最重要的第一步就是找到hook点的地址。

接下来说说**onEnter(args)**这个API

onEnter的作用在于在函数进入时执行其内部函数，而args就是我们所要hook函数的参数，因为这里hook了函数的参数，所以后面我们需要将args来具体赋值，示例如下

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

与onEnter相反，**onLeave(retval)**就是在函数返回前执行，而onEnter和onLeave都是可以单独使用的。以下是onLeave的具体示例

```javascript
var a=Process.getModuleByName("liba0x9.so");
var addr=a.getExportByName("Java_com_ad2001_a0x9_MainActivity_check_1flag");
Interceptor.attach(addr,{
    onLeave(retval)
    {
        retval.replace(1337);
    }
});
```

主要作用就是在程序退出时返回1337。

当我们需要调用so文件中某一个自定义函数而不是strcmp，strcpy等封装好的函数时我们就需要使用**xxx.base.add(偏移)**和**new NativeFunction(地址 , '函数类型' , ['输入类型'])**

示例代码如下

```javascript
var baseaddr=Process.getModuleByName("libfrida0xa.so");
var addr=baseaddr.base.add(0x206B0);
var get_flag= new NativeFunction(addr,'void',['int' , 'int']);
get_flag(1,2);
```

而在nativehook中很难不遇到死代码或者花指令此时可以手动nop或者进行hook，在利用hook进行nop时主要使用的API为**new X86(根据实际情况替换架构)Writer(nop起始地址)**和**Memory.protect(nop地址 , 设置可读写内存大小 , '读写权限')**

示例代码如下

```javascript
Process.attachModuleObserver({
  onAdded:function(module){
    var a=Process.getModuleByName("libfrida0xb.so");
    var addr=a.getExportByName("Java_com_ad2001_frida0xb_MainActivity_getFlag");
    var nopaddr=addr.add(0x1E);
    var write=new X86Writer(nopaddr);
    Memory.protect(nopaddr, 0x1000, 'rwx');
    try{
      write.putNop();
      write.putNop();
      write.putNop();
      write.putNop();
      write.putNop();
      write.putNop();
      write.flush();
    }
    finally{
      write.dispose()
    }
  }
})
```

**Process.attachModuleObserver**这个API的主要作用是监听模块加载事件，但是只负责监听，不负责回调调用等

**onAdded**的作用则着重在于回调调用，进程中已经存在的模块这个API会先回调调用一遍，新模块加载时此API则会继续被调用。这个API能够有效防止so和frida调用异步导致调用失败的问题。

**putNop**的作用就是nop，但是一次只能nop掉一个字节，所以在进行hooknop时需要注意要nop掉的字节数

**flush**的作用则是将nop指令真正写入内存中，不调用则会导致putNop指令只存在于write的缓冲区中无法起到nop的作用

**dispose**则是进行资源释放防止Writer执行失败而资源未释放而造成的内存泄漏或状态残留

解释一下Memory.protect中的rwx，这个rwx并不是我们所认知的寄存器，而是读写权限，在安卓中.text段的代码权限默认是 r-x 并不包含write权限，所以我们就需要增加write权限以保证Writer能够将指令正常写入进内存中方式错误
