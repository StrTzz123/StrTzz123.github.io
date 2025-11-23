---
title: 主流浏览器及其JavaScript Engine介绍
date: 2025-11-23 12:00:00 +0800
categories: [技术, 浏览器]
tags: [JavaScript, 浏览器引擎, V8, Webkit]
pin: false
---

## 浏览器通用架构

- 有各种不同的浏览器：chrome、Edge、Safari
- 一个浏览器就是一个OS
- 架构类似：多进程+IPC（进程间通信）

![image.png](/assets/img/2025-11-23-browser-js-engine/image.png)

- `Brower Process` ：负责控制浏览器的主题部分，比如地址栏，书签等等。也负责整个浏览器的权限控制，比如网络访问和文件读写。一个浏览器只有一个 `brower process` 进程
    
    ![image.png](/assets/img/2025-11-23-browser-js-engine/image%201.png)
    
- `randerer process` ： 渲染进程负责控制一个网页显示的内容， `Javascript Engine` (比如chome的v9)就是其中一部分。一个web页面就对应一个进程。 **有沙箱保护。.**
    
    ![image.png](/assets/img/2025-11-23-browser-js-engine/image%202.png)
    
- `GPU process` : 负责渲染内容，**唯一**。
- `utility process` : 负责浏览器的一些设置功能的进程。
- `plugin process` ： 负责管理运行**浏览器的插件**，每个插件都有自己的**单独的进程**。

设计的目的：

- 耦合性低
- 简化权限控制
- 稳定性好，一个进程崩溃不影响其他进程。

## JS 引擎

不同的浏览器有着不同的引擎

- `chrome` 的引擎是 `v8` , 同时 `v8` 也是 `Node.js` 的 `JS Engine` 。 `V8` 调试接口非常丰富，基本上可以给你任何你想要的信息。
- `safari` 的 `js` 引擎是 `webkit` , 很多的 `appstore` 的程序也使用 `webkit`。
- `edge` 以前用的是 `chakracore` 现在使用 `v8` 了。 `chakracore` 几乎已经被淘汰了（代码量小，适合学习）
- `firefox` 使用的是 `spidermonkey`

## JS 引擎流水线机制

`js` 引擎： 处理 `js` 语言时，通常先把网页代码下载下来，浏览器进行解析。

![image.png](/assets/img/2025-11-23-browser-js-engine/image%203.png)

> profiling data： 收集这些参数信息。
deeptimize：去优化。如果又有其他参数输入给这个函数，就会转化为字节码，目的是为了保证函数执行的正确性。
> 

### parse

将js代码翻译为语法树。

![image.png](/assets/img/2025-11-23-browser-js-engine/image%204.png)

目的：

- 检查错误的语法
- 为生成 `bytecode` 字节码做好准备。

### interpreter

这个可以理解为一个内置的虚拟机。

![image.png](/assets/img/2025-11-23-browser-js-engine/image%205.png)

对每个case有着不同的操作。将其翻译为字节码

![image.png](/assets/img/2025-11-23-browser-js-engine/image%206.png)

- 将AST转化为字节码
- 解析执行 `Bytecode`
- 和 `parse` 可以组成一个完整的 `JS Engine`

### JIT Compiler( 编译器优化)

- 编译器执行的字节码会很慢，JIT编译器用于优化 `Hot Function` 【执行次数多的函数】
    - 将弱类型转化为强类型【可以理解为将python代码转化为C代码】

```jsx
function simple_add(a, b){
    reutrn a + b;
}
 
for(let i = 0; i < 0x100000; i++){
    simple_add(i, i + 1);
}
```

考虑一下上面的代码：

因为 JavaScript 是弱类型的语言，因此如果执行 Bytecode，我们需要考虑 a 和 b 的类型是什么。

- case 1: a 和 b 都是整型
- case 2: a 和 b 都是字符串
- case 3: a，b是字符串及整型的搭配
- …

根据 simple_add 函数的实际参数情况，我就只希望它把 a ，b 考虑成整型，并对其做优化。

```nasm
simple_add:
    res = add a, b
    return res
```

如果后面调用 `simple_add("AA", "BB")` 怎么办？

```nasm
simeple add:
    check a
    jmp to bail_out if not integer
    check b
    jmp to bail_out if not integer
    res = add a, b
    return res

bail_out:  # [也就是进行去优化]
    # Go back to interpreter
```

## 常见 `JS` 引擎架构

- `v8` (`chrome`)
    
    ![image.png](/assets/img/2025-11-23-browser-js-engine/image%207.png)
    
    - google开发的浏览器，全平台通用，并且开源。
    - 插件多，速度快，最安全。
    - `v8` 调试接口最丰富，基本上可以给你任何你想要的信息。
- `SpiderMonket` FireFOx
    
    ![image.png](/assets/img/2025-11-23-browser-js-engine/image%208.png)
    
    - 火狐开发的一个浏览器，全平台通用。
- Chakra Core(Edge)
    
    ![image.png](/assets/img/2025-11-23-browser-js-engine/image%209.png)
    
    - `Edge` 是 `Windows` 专用浏览器，不开源。
    - ChakraCore 是 Edge 的 JS 引擎，代码量少，便于学习。
    - 后面微软又转化为chromium浏览器
- `Webkit` （safari）
    
    ![image.png](/assets/img/2025-11-23-browser-js-engine/image%2010.png)
    
    - 苹果专用，不开源
    - webkit 同时被苹果的 app Store Mail 以及它很多应用程序内嵌使用