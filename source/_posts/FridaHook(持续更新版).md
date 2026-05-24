title: FridaHook(持续更新版)
date: 2026-5-24
category: LEARN
cover: /passagecover/8.jpg
---

## 前言
在使用frida时，我们常使用一些命令行来hook上虚拟机或真机，同时在我们写hook脚本时，我们也会常用到一些frida的API来为我们服务，这篇文章的主要目的就是来详细讲讲fridahook中的一些注意事项，命令和脚本书写，脚本API等。<font style="color:#DF2A3F;">由于作者本人也在逐渐学习，目前也处于处于入门阶段，所以本篇文章会逐渐更新完善和更改修正，在完结之前请批判性阅读。</font>

## 工具类
一般来说我们在做安卓题时需要用到虚拟机，而我个人推荐studio64，当然用uu模拟器也行，看个人习惯。但我还是想说安卓studio用习惯其实感觉还不错，很纯净很方便，跟真机更加接近，方便后面逐渐接触真机的学习。

安卓studio的官方下载网址为：[https://developer.android.google.cn/studio?hl=zh-cn](https://developer.android.google.cn/studio?hl=zh-cn)

## 脚本类
安卓逆向中我们常撰写JS脚本来hookapp，而其中的接口和API则是需要用frida所提供的，倘若脚本不符合frida所提供的API或有本质上的JS语法错误frida会直接报错，下面讲讲常见的fridaAPI

```javascript
Java.perform(function(){
  ...
})
```

这个命令非常重要，这是fridahook脚本的必要开头，主要作用在于当与JVM连接成功后才会启动内部代码，否则将不会执行其内部代码，。基本上写fridahook脚本记死这个开头就行。但值得注意的是在hooknative层时是不需要用这个开头的，毕竟native层不依赖JVM。

```javascript
Java.use(...)
```

这个的主要作用就是调用我们所需要的特定的类，但值得注意的是，当我们都调用某一个特定的函数时要注意是否为public static，倘若不是那么我们就要创造一个实例即a.$new()才能正常hook

```javascript
。。。.。。。.implementation=function(...){
  ...
}
```

这个语句的主要作用在于当我们需要将某一函数内部逻辑替换掉或强行返回某一特定的值(诸如此类的功能)时我们可以使用这个API，那么当我们hook上app后进程中的这个函数就会被API内的逻辑所替换。

```javascript
console.log()
```

输出逻辑

```javascript
Java.perform(function () {
    Java.choose("xxx", {
        onMatch: function(instance) {
            instance.get_flag(1337);
        },
        onComplete: function() {
          console.log("done");
        }
    });
});
```

当我们遇到一些对象无法正常hook时，比如mainactivity中的对象，我们就要使用这套API。它会进入内存中搜索choose后的进程名称或具体函数，一旦有符合的就进入onmatch中，执行onmatch中的逻辑，搜索过程中每有一次符合就会调用一次onmatch，当所有都完成后就会调用oncomplete，终止搜索和执行。

## 命令类
命令类有两种类别，一种是adb命令，一种是frida命令。

### adb命令
adb命令更像是上帝视角，使用adb时我们更多的是在以外部向内看，主要是用来获取权限，查看日志，安装，删除之类的事，更多的是对系统的管理和处理。

常见的adb命令如下

```shell
adb devices
```

这个命令的作用在于查看已上线并且能够连接的设备数量及名称。

```shell
adb root
```

这个命令的作用在于直接获取目标设备的最高权限。

```shell
adb shell ps
```

作用在于查找已连接设备上所有进程的PID

```shell
adb push [推送文件] [目标地址]
```

这个命令用于将一些在Windows上的文件推送到安卓的系统中，一般来说我们直接推送进去的文件会被存放在/data/local/tmp/中。我们一般用这个命令将frida-server推送到安卓设备中。

```shell
adb shell
```

这个命令的作用在于进入安卓的Linux shell，用来获取一些程序的运行权限，管理设备权限以及cd到各个文件夹管理文件。

当使用adb shell命令后我们进入安卓的内部系统中，如下图所示

![](/blog_essay_picture/FridaHookpicture.png)

当你输入su之后也能获取设备的最高权限，其作用与adb root等价。

进入shell后直接cd /data/local/tmp/然后ls查看fridaserver是否推送成功，然后执行

```shell
chmod 755 xxx
```

为fridaserver赋权，然后./fridaserver即可。

### frida命令
frida的命令结构一般为

```shell
frida [操作] [目标]
```

像我们常见的frida -U -f xxx -l xxx.js就是这种格式。frida还有很多其他的操作系数，下面来详细讲讲

-U：连接USB设备，包括但不限于真机和模拟器

-f：启动某个具体的app进程并注入

-l：load

--no-pause：较新版本的frida基本不需要这个命令了，其作用是是注入后继续运行，防止进程注入后停止

-n：也就是attach，即附加到正在运行的进程中

-p：与上面-f不同，-f后加具体的app名称，-p后加的是某一进程的PID

