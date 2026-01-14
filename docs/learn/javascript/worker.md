# mid

## promise

```txt
var x, y = 2
console.log( x + y ) // NaN <-- 因为x还没有设定
```

把x和y加起来，但如果它们中的任何一个还没有准备好，就等待两者都准备好。一但可以就马上执行加运算。

```javascript
function add(getX, getY, cb) {
    var x, y;
    getX( function(xVal) {
        x = xVal

        if( y != undefined) {
            cb( x + y )
        }
    })
    getY( function(yVal) {
        y = yVal

        if(x != undefined) {
            cb( x + y)
        }
    })
}

add( fetchX, fetchY, function(sum) {
    console.log(sum)
})
```

这样的处理将他们都变成了将来值

### Promise值

```javascript
function add(xPromise, yPromise) {
    return Promise.all([xPromise,yPromise])
    .then((values)=>{
        return values[0] + values[1]
    })
}

add(fetchX, fetchY)
.then((sum)=>{
    console.log( sum )
})
```

## web worker
像浏览器环境，很容易提供多个Javascript引擎实例，各自运行在自己的线程上，这样你可以在每个线程上运行不同的程序。程序中每一个这样的独立的多线程部分被称为一个web workder。这种类型的并行化被称为任务并行，因为器重点在于把程序划分为多个块来并发运行。

Worker之间以及它们和主程序之间，不会共享任何作用域或资源，那会把所有多线程编程的噩梦带到前端领域，而是通过一个基本的事件消息机制互相联系。

Worker wl 对象是一个事件侦听者和触发者，可以通过订阅它来获得这个worker发出的事件以及发送事件给这个worker。

以下是如何侦听事件
wl.addEventListener("message", function(evt){
    // evt.data
})
也可以发送message事件给这个worker
wl.postMessage("something cool to say")
在这个worker内部，收发消息是完全对称的：
// mycoolworker.js
addEventListener("message", function(evt){
    // evt.data
})

postMessage("a really cool reply")


### worker环境
在worker内部是无法访问主程序的任何资源的。这意味着你不能访问它的任何全局变量，也不能访问页面的DOM或者其他资源。记住这是一个完全独立的线程。
但是，你可以执行网络操作（Ajax，WebSockects）以及设定定时器。还有worker可以访问几个重要的全局变量和功能的本地副本，包括navigator、location、JSON和applicationCache。
你还可以通过importScripts(...)向worker加载额外的JavaScript脚本。
// 在worker内部
importScripts('foo.js', 'bar.js');
