---
title: JS Object 基本概念
date: 2025-11-23 16:51:10 +0800
categories: [技术, 浏览器]
tags: [JavaScript, V8, 浏览器引擎]
math: true
mermaid: true
---


https://source.chromium.org/

以 `7.6.303.28` 版本进行学习。高版本引入了沙箱。

# V8 低版本编译

```bash
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```

```bash
export PATH=$PATH:<path to depot_tools>

git reset --hard 138bff28
export DEPOT_TOOLS_UPDATE=0
gclient

fetch v8
如果失败需要：
fetch --force v8
```

下载之后，我们需要根据题目，或者cve的版本来对其进行checkout，因为我们在这里学习，直接checkout到7.6.303.28 进行学习。

> 如果题目给的是一个 Chrome 浏览器那么首先安装浏览器然后再网址栏中输入 `chrome://version` 查看版本，例如
> 

```bash
cd v8
git checkout  7.6.303.28
gclient sync -D
```

如果题目给了 `diff` 文件需要将 patch 到项目中。

```bash
git apply ./oob.diff
```

接着安装相关依赖

```bash
./build/install-build-deps.sh --no-chromeos-fonts
```

编译 `v8` ，这里选的 `release` 版本。`debug` 版本改为 `x64.debug` ，32 为版本将 `x64` 改为 `ia32` 。如果调试漏洞的话, 最好选择 `release` 版本 因为 `debug` 版本可能会有很多检查。

```bash
./tools/dev/gm.py x64.release
```

安装`turbolizer`

```jsx
sudo apt install npm
cd /path/to/v8/tools/turbolizer
sudo npm install n -g
sudo n 16.20.0 # sudo n latest
sudo npm i
sudo npm run-script build

```

# 调试V8

在 `~/.gdbinit` 添加 `v8` 的调试插件：

```bash
source /path/to/v8/tools/gdbinit
source /path/to/v8/tools/gdb-v8-support.py
```

- `-allow-natives-syntax` 开启原生API (用的比较多)
- `-trace-turbo` 跟踪生成TurboFan IR
- `--trace-turbo-filter`，限制到特定的函数。
- `-print-bytecode` 打印生成的bytecode
- `-shell` 运行脚本后切入交互模式
- 更多参数可以参考 `-help`

调试 `js` 脚本可以采用如下命令：

```bash
gdb ./d8
r --allow-natives-syntax --shell ./exp.js
```

接着就可以开始调试了

# 存储形式

![image.png](/assets/img/2025-11-23-basic-concepts-of-js-object/image.png)

### Smi 形式

所有不超过 0x7FFFFFFF 的整数都以 Smi 的形式存储。

- 在32为上可以表示有符号的 31 位的整数，通过右移一位可以获得原始值

![image.png](/assets/img/2025-11-23-basic-concepts-of-js-object/image_1.png)

- 在64位上可以表示有符号的32位的整数，通过右移32位可以获得原始值

![image.png](/assets/img/2025-11-23-basic-concepts-of-js-object/image_2.png)

### HeapObject 指针

最低位为 1 表示指向 HeapObject 的指针。

- 32位

![image.png](/assets/img/2025-11-23-basic-concepts-of-js-object/image_3.png)

- 64位

![image.png](/assets/img/2025-11-23-basic-concepts-of-js-object/image_4.png)

### Heap Number

表示不能在 `Smi` 范围内表示的整数，均以 `double` 值的形式保存在 `Heap Number` 的 `Value`里。

![image.png](/assets/img/2025-11-23-basic-concepts-of-js-object/image_5.png)

![image.png](/assets/img/2025-11-23-basic-concepts-of-js-object/image_6.png)

![image.png](/assets/img/2025-11-23-basic-concepts-of-js-object/image_7.png)

`b = [1.1,3, 2.2, {'aaa':'bbb'}] ;` 

如果既有 `Object` 和 `double` 类型的话。【因为double也可以表示为指针，这样是为了区分。】

这种情况下， `double` 都变为了 `HeapNumber`。 `Mao` 表示其类型。

 

V8 的所有结构的关系。

```bash
// Most object types in the V8 JavaScript are described in this file.
//
// Inheritance hierarchy:
// - Object
//   - Smi          (immediate small integer)
//   - HeapObject   (superclass for everything allocated in the heap)
//     - JSReceiver  (suitable for property access)
//       - JSObject
//         - JSArray
//         - JSArrayBuffer
//         - JSArrayBufferView
//           - JSTypedArray
//           - JSDataView
//         - JSBoundFunction
//         - JSCollection
//           - JSSet
//           - JSMap
//         - JSStringIterator
//         - JSSetIterator
//         - JSMapIterator
//         - JSWeakCollection
//           - JSWeakMap
//           - JSWeakSet
//         - JSRegExp
//         - JSFunction
//         - JSGeneratorObject
//         - JSGlobalObject
//         - JSGlobalProxy
//         - JSValue
//           - JSDate
//         - JSMessageObject
//         - JSModuleNamespace
//         - JSV8BreakIterator     // If V8_INTL_SUPPORT enabled.
//         - JSCollator            // If V8_INTL_SUPPORT enabled.
//         - JSDateTimeFormat      // If V8_INTL_SUPPORT enabled.
//         - JSListFormat          // If V8_INTL_SUPPORT enabled.
//         - JSLocale              // If V8_INTL_SUPPORT enabled.
//         - JSNumberFormat        // If V8_INTL_SUPPORT enabled.
//         - JSPluralRules         // If V8_INTL_SUPPORT enabled.
//         - JSRelativeTimeFormat  // If V8_INTL_SUPPORT enabled.
//         - JSSegmentIterator     // If V8_INTL_SUPPORT enabled.
//         - JSSegmenter           // If V8_INTL_SUPPORT enabled.
//         - WasmExceptionObject
//         - WasmGlobalObject
//         - WasmInstanceObject
//         - WasmMemoryObject
//         - WasmModuleObject
//         - WasmTableObject
//       - JSProxy
//     - FixedArrayBase
//       - ByteArray
//       - BytecodeArray
//       - FixedArray
//         - FrameArray
//         - HashTable
//           - Dictionary
//           - StringTable
//           - StringSet
//           - CompilationCacheTable
//           - MapCache
//         - OrderedHashTable
//           - OrderedHashSet
//           - OrderedHashMap
//         - FeedbackMetadata
//         - TemplateList
//         - TransitionArray
//         - ScopeInfo
//         - ModuleInfo
//         - ScriptContextTable
//         - ClosureFeedbackCellArray
//       - FixedDoubleArray
//     - Name
//       - String
//         - SeqString
//           - SeqOneByteString
//           - SeqTwoByteString
//         - SlicedString
//         - ConsString
//         - ThinString
//         - ExternalString
//           - ExternalOneByteString
//           - ExternalTwoByteString
//         - InternalizedString
//           - SeqInternalizedString
//             - SeqOneByteInternalizedString
//             - SeqTwoByteInternalizedString
//           - ConsInternalizedString
//           - ExternalInternalizedString
//             - ExternalOneByteInternalizedString
//             - ExternalTwoByteInternalizedString
//       - Symbol
//     - Context
//       - NativeContext
//     - HeapNumber
//     - BigInt
//     - Cell
//     - DescriptorArray
//     - PropertyCell
//     - PropertyArray
//     - Code
//     - AbstractCode, a wrapper around Code or BytecodeArray
//     - Map
//     - Oddball
//     - Foreign
//     - SmallOrderedHashTable
//       - SmallOrderedHashMap
//       - SmallOrderedHashSet
//     - SharedFunctionInfo
//     - Struct
//       - AccessorInfo
//       - AsmWasmData
//       - PromiseReaction
//       - PromiseCapability
//       - AccessorPair
//       - AccessCheckInfo
//       - InterceptorInfo
//       - CallHandlerInfo
//       - EnumCache
//       - TemplateInfo
//         - FunctionTemplateInfo
//         - ObjectTemplateInfo
//       - Script
//       - DebugInfo
//       - BreakPoint
//       - BreakPointInfo
//       - StackFrameInfo
//       - StackTraceFrame
//       - SourcePositionTableWithFrameCache
//       - CodeCache
//       - PrototypeInfo
//       - Microtask
//         - CallbackTask
//         - CallableTask
//         - PromiseReactionJobTask
//           - PromiseFulfillReactionJobTask
//           - PromiseRejectReactionJobTask
//         - PromiseResolveThenableJobTask
//       - Module
//       - ModuleInfoEntry
//     - FeedbackCell
//     - FeedbackVector
//     - PreparseData
//     - UncompiledData
//       - UncompiledDataWithoutPreparseData
//       - UncompiledDataWithPreparseData
//
// Formats of Object::ptr_:
//  Smi:        [31 bit signed int] 0
//  HeapObject: [32 bit direct pointer] (4 byte aligned) | 01
```

![image.png](/assets/img/2025-11-23-basic-concepts-of-js-object/image_8.png)