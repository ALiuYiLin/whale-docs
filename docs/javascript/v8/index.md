# V8 引擎

3. 老生代内存管理（标记-清除/标记-整理）
   老生代对象存活时间长、数量多，无法用Scavenge（复制成本搞），因此采用标记-清除（Mark-Sweep）和标记-整理（Mark-Compact）结合的策略。
    （1）标记-清除（Mark-Sweep）是老生代GC的基础，分为两个阶段：
   
        1. 标记阶段： 从根对象（如window、全局变量）出发，遍历所有可达对象，标记为存活；
   
    （2）清除阶段：遍历佬生代内存，清除未被标记的对象，释放内存。
   
   - 优点：无需复制对象，适合大量存活对象；
   - 缺点：清除后产生内存碎片（空闲内存不连续），后续分配大对象时可能触发频繁GC。

### 标记整理（mark-compact）

为解决内存碎片问题，再标记阶段后增加整理步骤：

1. 标记存活对象；
2. 整理阶段：将所有存活对象向内存一端移动，使内存地址连续；
3. 清除阶段：清除边界外的所有内存。
- 优点：无内存碎片；
- 缺点：整理阶段耗时较长（需移动对象）。

### V8的优化策略

为减少GC导致的JS执行暂停（Stop-The-World），V8引入：

- 增量标记：将标记阶段拆分为多个小步骤，穿插在JS执行中；
- 并发标记：GC线程与JS线程并行执行（仅在标记阶段）；
- 惰性清理：仅在内存不足时才清理未标记对象。

## 四、内存泄漏排查与修复

### 1. 内存泄漏的定义

不再使用的对象仍被引用，导致GC无法回收其内存，最终引发内存占用率持续升高、应用卡顿甚至崩溃。

### 2. 常见内存泄漏场景

| 场景       | 示例代码                                                       | 原因               |
| -------- | ---------------------------------------------------------- | ---------------- |
| 意外全局变量   | `function foo(){a=1;}`                                     | 未声明的变量挂载到window  |
| 未清理的定时器  | `setInterval(()=>{},1000)`                                 | 定时器持有回调函数引用      |
| 未移除的事件监听 | `elem.addEventListener('click',fn)`                        | 事件监听持有elem/fn 引用 |
| 闭包泛滥     | function foo(){let a = 1; return ()=>a;}                   | 闭包持有外层变量引用       |
| 无效DOM引用  | `let div = document.getElementById('div');div.remove()`    | 变量仍持有DOM引用       |
| 无限制缓存    | `const cache = {};function set(key,var){cache[key] = val}` | 缓存过期/清理机制        |

### 3.排查工具与步骤

#### （1）核心工具

- Chrome DevTools: Memory面板（内存快照）、Performance面板（内存趋势）；

- Node.js：`node --inspect`（调试）、`chinic.js`（性能分析）。

#### （2）排查步骤（以Chrome为例）

1. 复现问题：打开DevTools->Performance->勾选"Memory"->点击录制按钮→操作触发泄漏场景→停止录制；

2. 分析趋势：查看内存曲线是否持续上升（无下降）；

3. 生成快照：Memory→选择“Heap snapshot”→点击“Take snapshot”

4. 对比快照：多次操作后生成第二个快照→切换到“Comparison”模式→对比两个快照的对象数量/大小；

5. 定位代码：在快照中找到增长的对象→查看“Retainers”（引用链）→定位到持有引用的代码；

6. 修复验证：修改代码后，重新录制验证内存是否正常。

### 4.修复方法

针对常见场景的修复示例“

```javascript
// 1. 避免意外全局变量（使用let/const，或严格模式）
'use strict';
function foo() {
    let a = 1; // 而非a=1
}


// 2.清理定时器
let timer = setInterval(() => {},1000);
clearInterval(timer); // 不再使用时清理

// 3.移除事件监听
function fn(){}
elem.addEventListener('click',fn);
elem.removeEventListener('click',fn); // 移除监听


// 4.解除DOM引用
let div = document.getElementById('div');
div.remove();

div = null; // 清空引用


// 5.限制缓存大小

const cache = new Map();
function setCache(key, val) {
    if(cache.size > 100) {
        // 删除最早的缓存
        const firstKey = cache.keys().next().value;
        cache.delete(firstKey)
    }
    cache.set(key,val)
}
```

## 五、总结

### 核心关键点回顾

1. 执行上下文与调用栈：执行上下文是JS执行的环境容器（分全局/函数/eval），调用栈以栈结构管理执行上下文的入栈/出栈，栈溢出是调用栈的深度超出限制导致；

2. 垃圾回收机制：V8按对象生命周期分新生代（Scavenge复制算法）和老生代（标记-清除/标记-整理），通过增量标记、并发标记优化GC暂停时间；

3. 内存泄漏修复：核心是切断无用对象的引用链，常见手段包括清理定时器/事件监听、避免意外全局变量、解除DOM引用、限制缓存大小等。

### 关键提示

- V8的内存限制（64位约1.4GB）是为了控制GC暂停时间，Node.js中处理大文件需避免一次加载；

- 内存泄漏排查的核心是找到“无用但仍被引用”的对象，通过快照对比可快速定位；

- 闭包本身不导致泄漏，滥用闭包（如长期持有大对象引用）才会引发问题。
