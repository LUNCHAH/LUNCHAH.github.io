## 前言
在使用frida时，我们常使用一些命令行来hook上虚拟机或真机，同时在我们写hook脚本时，我们也会常用到一些frida的API来为我们服务，这篇文章的主要目的就是来详细讲讲fridahook中的一些注意事项，命令和脚本书写，脚本API等。<font style="color:#DF2A3F;">由于作者本人也在逐渐学习，目前也处于处于入门阶段，所以本篇文章会逐渐更新完善和更改修正，在完结之前请批判性阅读。</font>

## 工具类
一般来说我们在做安卓题时需要用到虚拟机，而我个人推荐studio64，当然用uu模拟器也行，看个人习惯。但我还是想说安卓studio用习惯其实感觉还不错，很纯净很方便，跟真机更加接近，方便后面逐渐接触真机的学习。

安卓studio的官方下载网址为：[https://developer.android.google.cn/studio?hl=zh-cn](https://developer.android.google.cn/studio?hl=zh-cn)

## adb命令
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

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62008266/1779450002408-c7f4a3c2-74b9-4079-aec9-4e039101c81a.png)

当你输入su之后也能获取设备的最高权限，其作用与adb root等价。

进入shell后直接cd /data/local/tmp/然后ls查看fridaserver是否推送成功，然后执行

```shell
chmod 755 xxx
```

为fridaserver赋权，然后./fridaserver即可。

## 环境安装类
frida的正常运行需要安装两个，一个是pip下载frida，一个是对应版本的frida-server。

frida-server的github下载地址为：[https://github.com/frida/frida/releases](https://github.com/frida/frida/releases)，在这里你可以根据你的模拟器型号，例如x86_64，arm64，来下载frida-server，然后运行命令adb push [你的frida-server] [目标地址]，这个目标地址一般是/data/local/tmp/。

frida的安装需要依靠python环境，笔者推荐frida17.5.2，搭配使用python3.12.4和frida-tools14.2.x，比较适合新手学习，frida的安装命令为

```python
pip install frida==你要的版本
pip install frida-tools==你要的版本
```

安装好后可以输入命令

```shell
frida --version
```

来查看自己是否安装成功。

## frida命令
frida的命令结构一般为

```shell
frida [操作] [目标]
```

像我们常见的frida -U -f xxx -l xxx.js和frida -U -f xxx就是这种格式。frida还有很多其他的操作系数，下面来详细讲讲

-U：连接USB设备，包括但不限于真机和模拟器

-f：启动某个具体的app进程并注入

-l：load

--no-pause：较新版本的frida基本不需要这个命令了，其作用是是注入后继续运行，防止进程注入后停止

-n：也就是attach，即附加到正在运行的进程中

-p：与上面-f不同，-f后加具体的app名称，-p后加的是某一进程的PID

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

需要注意的是，当我们在使用choose时，我们应当先frida -U -f进入程序，然后再把我们的复制到shell中跑，如果直接frida -U -f xxx -l xxx.js的话大概率会因为mainactivity还未初始化，导致choose无法找到mainactivity的实例无法正常hook。

在撰写hook脚本时我们常会遇到某个函数是非static，这是我们就需要去new一个实例给这个函数来方便我们hook，即

```javascript
var example=xxx.$new(...)
```

这个语句在使用时需要看具体函数逻辑，譬如我需要hook某个如下的checker函数

```java
public class Checker {
    int num1;
    int num2;
}
```

那么我在撰写脚本时就需要写

```javascript
var obj=a.$new();
obj.num1.value(...);
obj.num2.value(...);
```

因为这个checker逻辑没有规定任何传参及其形式，所以我可以直接new一个实例

但是倘若像如下checker规定了传参

```java
public class Checker {
    int num1;
    int num2;

    Checker(int a, int b) {
        this.num1 = a;
        this.num2 = b;
    }
}
```

那么我们在写hook脚本时就不能直接空new，而是在符合这个函数的特定需求的情况下去new，像如下语句

```javascript
var obj=a.$new(... , ...);
```

至于上文所提及的无法正常hookmainactivity的情况其实就是在mainactivity中我们不能随意进行new，那么这是我们就需要利用Java.choose去搜索进程再使用Java.use然后才去new，而不是直接new一个实例出来。

