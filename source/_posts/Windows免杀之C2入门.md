title: Windows免杀之C2入门
date: 2026-09-24
category: LEARN
cover: /passagecover/16.jpg
---

# 前言
由于笔者是第一次接触windows免杀方面的知识，所整理的知识点难免有错漏，再加上Windows方面本身就繁复错杂，所以很多地方笔者的研究和理解都不够，望读者海涵斧正。本篇博客会随着笔者研究和学习的深入而更新。

# 正文
## 1.Adaptix C2定义及框架
Adaptix C2指的是高度可扩展和适应渗透工具，其框架主要由GUI Client，Teamserver，Agent，Listener和BOF组成。

GUI Client：操作员所使用的图形化客户端

Teamserver：位于攻击侧的，用于提供管理服务和控制的组件

Agent：是一个泛称，而Beacon则是Agent的一种。Agent是宿主机上的代理程序，负责接收，回传和主动回连证明在线

Listener：通信出入口，用于接收Agent的回传并发送给Teamserver和将Teamserver的指令下发给Agent

COFF：C源码编译器编译完成但还未生成可执行文件的文件，在windows上.obj就是COFF文件，.lib是多个COFF的归档集合，PE文件则是从COFF扩展而来

BOF：一个可被Agent加载运行的COFF文件

而在一般的攻防逻辑中运作方式如下

```plain
操作员 GUI
    │ 任务/参数
    ▼
Adaptix Teamserver ── Listener（HTTP/DNS/SMB/TCP）
    │                         ▲
    │ 任务队列/结果            │ 加密 C2 流量
    ▼                         │
Beacon Agent（目标主机）  ─────┘
    │
    ├─ 内置命令
    └─ BOF/Async BOF
```

我们可以在攻防中实时修改，上传BOF文件给Agent进行加载和运行从而达到多样的目的

## 2.BOF
如上文所言，BOF是一个可被宿主加载的COFF对象，但并未连接生成为正常的PE文件，所以并不具有标准PE入口，导入目录等所以无法调用常规的IAT。而由于BOF未经过链接器，所以并没有正常PE文件所拥有的基地址，标准库，绝对地址等，在内存中其代码和数据是随机分配在一块空间中的，所以这个时候PIC就出现了。

值得一提的是，BOF中是不存在main逻辑的，BOF的入口是go函数，COFF加载器在运行BOF时会定位到其go函数的相对地址来执行其内的代码。

### 那么PIC是什么？
PIC称为位置无关代码，由于BOF的代码在内存中是随机存放的，所以在读取和运行时是依靠相对地址，而BOF的代码天生就带有随机性，即BOF在编写和编译时就采用了相对地址读取的方式，CPU在读取代码时会根据PIC中的相对地址跳转到下一个地址的代码处执行。所以PIC是BOF代码本身的属性。

COFF加载器把BOF加载到内存后复制相对应的节区(也就是我们常说的段)，通过解析重定位表修正部分地址，绑定beacon API，最后调用go函数。

## 3.BOF与C2通信
如上文所言，BOF是未经过链接器的文件，不具有exe等文件一样双击启动自动运行的能力，所以必须通过Beacon来进行与C2之间的通信。其中比较重要的就是Beacon API，必须有能够给BOF调用的Beacon API BOF才能够实现正常的功能如输出，回显等，常见的Beacon API如下表所示

| <font style="color:rgb(15, 17, 21);">BeaconPrintf</font> | | | | <font style="color:rgb(15, 17, 21);">向操作员输出信息，结果回传到 C2 客户端</font> |
| --- | --- | --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">BeaconDataParse / BeaconDataExtract</font> | | | | <font style="color:rgb(15, 17, 21);">解析操作员传给 BOF 的参数</font> |
| <font style="color:rgb(15, 17, 21);">BeaconInjectProcess</font> | | | | <font style="color:rgb(15, 17, 21);">请求 Beacon 向指定进程注入载荷</font> |
| <font style="color:rgb(15, 17, 21);">BeaconUseToken / BeaconRevertToken</font> | | | | <font style="color:rgb(15, 17, 21);">操作令牌</font> |
| <font style="color:rgb(15, 17, 21);">BeaconIsAdmin</font> | | | | <font style="color:rgb(15, 17, 21);">判断当前是否管理员</font> |
| <font style="color:rgb(15, 17, 21);">BeaconSpawnTemporaryProcess</font> | | | | <font style="color:rgb(15, 17, 21);">创建临时进程</font> |
| <font style="color:rgb(15, 17, 21);">BeaconCleanupProcess</font> | | | | <font style="color:rgb(15, 17, 21);">清理进程资源</font> |


## 4.OPSEC
<font style="color:rgb(51, 51, 51);">OPSEC（Operations Security，行动安全）是识别、分析和控制行动中可能泄露任务、身份、能力、时间线和基础设施的信息。</font>

<font style="color:rgb(51, 51, 51);">在授权红队或对抗仿真中，它首先关注：</font>

+ <font style="color:rgb(51, 51, 51);">是否只触碰授权资产；</font>
+ <font style="color:rgb(51, 51, 51);">是否使用专用身份和最小权限；</font>
+ <font style="color:rgb(51, 51, 51);">是否保护客户数据和实验样本；</font>
+ <font style="color:rgb(51, 51, 51);">是否所有动作可记录、可停止、可解释；</font>
+ <font style="color:rgb(51, 51, 51);">是否可以在结束后撤销和清理。</font>

<font style="color:rgb(51, 51, 51);">它不等于：</font>

+ <font style="color:rgb(51, 51, 51);">永远不被发现；</font>
+ <font style="color:rgb(51, 51, 51);">关闭日志；</font>
+ <font style="color:rgb(51, 51, 51);">删除证据；</font>
+ <font style="color:rgb(51, 51, 51);">绕过 EDR；</font>
+ <font style="color:rgb(51, 51, 51);">把复杂的隐藏技术堆叠在不必要的实验上。</font>

OPSEC可以分为五步执行

### <font style="color:rgb(51, 51, 51);">第一步：识别资产与任务</font>
<font style="color:rgb(51, 51, 51);">明确：</font>

+ <font style="color:rgb(51, 51, 51);">哪些 CIDR、主机、账号和租户在范围内；</font>
+ <font style="color:rgb(51, 51, 51);">哪些时间段允许操作；</font>
+ <font style="color:rgb(51, 51, 51);">哪些命令被允许；</font>
+ <font style="color:rgb(51, 51, 51);">哪些数据绝不能读取或外传；</font>
+ <font style="color:rgb(51, 51, 51);">什么条件下必须停止。</font>

### <font style="color:rgb(51, 51, 51);">第二步：建模观察者</font>
<font style="color:rgb(51, 51, 51);">至少考虑：</font>

+ <font style="color:rgb(51, 51, 51);">蓝队和 SOC；</font>
+ <font style="color:rgb(51, 51, 51);">EDR/AV；</font>
+ <font style="color:rgb(51, 51, 51);">网络检测；</font>
+ <font style="color:rgb(51, 51, 51);">云审计；</font>
+ <font style="color:rgb(51, 51, 51);">身份系统；</font>
+ <font style="color:rgb(51, 51, 51);">目标主机上的其他租户或业务方；</font>
+ <font style="color:rgb(51, 51, 51);">你自己的 Teamserver、工作站和日志系统。</font>

### <font style="color:rgb(51, 51, 51);">第三步：分析暴露面</font>
<font style="color:rgb(51, 51, 51);">常见暴露面包括：</font>

```plain
进程树、命令行、文件落地、内存权限、跨进程句柄、
线程起始地址、调用栈、模块变化、DNS/HTTP 元数据、
凭据使用、GUI 操作记录、样本 hash、Git 历史和聊天记录
```

### <font style="color:rgb(51, 51, 51);">第四步：设计控制措施</font>
<font style="color:rgb(51, 51, 51);">控制措施可包括：</font>

+ <font style="color:rgb(51, 51, 51);">最小权限；</font>
+ <font style="color:rgb(51, 51, 51);">专用且可过期的账号；</font>
+ <font style="color:rgb(51, 51, 51);">MFA；</font>
+ <font style="color:rgb(51, 51, 51);">固定 kill date；</font>
+ <font style="color:rgb(51, 51, 51);">低并发和有限输出；</font>
+ <font style="color:rgb(51, 51, 51);">隔离 Listener；</font>
+ <font style="color:rgb(51, 51, 51);">样本版本号、hash 和用途标签；</font>
+ <font style="color:rgb(51, 51, 51);">加密存储；</font>
+ <font style="color:rgb(51, 51, 51);">全量审计；</font>
+ <font style="color:rgb(51, 51, 51);">明确停止和回滚流程。</font>

### <font style="color:rgb(51, 51, 51);">第五步：复盘和销毁</font>
<font style="color:rgb(51, 51, 51);">结束后：</font>

1. <font style="color:rgb(51, 51, 51);">终止实验任务；</font>
2. <font style="color:rgb(51, 51, 51);">撤销临时凭据和令牌；</font>
3. <font style="color:rgb(51, 51, 51);">清理实验文件和会话；</font>
4. <font style="color:rgb(51, 51, 51);">恢复虚拟机快照；</font>
5. <font style="color:rgb(51, 51, 51);">核对 GUI、Teamserver、Agent、主机和检测平台时间线；</font>
6. <font style="color:rgb(51, 51, 51);">保留必要证据，不篡改客户日志；</font>
7. <font style="color:rgb(51, 51, 51);">形成蓝队可以复现和验证的报告。</font>

