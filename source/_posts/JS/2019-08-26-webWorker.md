---
title: webWorker
date: 2019-08-26 17:07:54
tags: JS
gitment: webWorker
description: 让JS实现多线程，把大量计算的数据“托管”到新开的线程
---

### 浏览器进程 ###
*** Browser进程 ***
浏览器的主进程，负责协调、主控
*** 第三方插件进程 ***
每种类型的插件对应一个进程，仅当使用该插件时才创建
*** GPU进程 ***
最多一个，用于3D绘制等
*** 浏览器渲染进程（浏览器内核） ***
Renderer进程，内部是多线程的，也就是我们每个标签页所拥有的进程，互不影响，负责页面渲染，脚本执行，事件处理等

### 浏览器渲染进程（浏览器内核） ###
*** GUI渲染线程 ***
负责渲染浏览器界面，包括解析HTML、CSS、构建DOM树、Render树、布局与绘制等   
当界面需要重绘（Repaint）或由于某种操作引发回流(reflow)时，该线程就会执行
*** JS引擎线程 ***
JS内核，也称JS引擎，负责处理执行javascript脚本   
等待任务队列的任务的到来，然后加以处理，浏览器无论什么时候都只有一个JS引擎在运行JS程序
*** 事件触发线程 ***
听起来像JS的执行，但是其实归属于浏览器，而不是JS引擎，用来控制时间循环   
当JS引擎执行代码块如setTimeout时（也可来自浏览器内核的其他线程,如鼠标点击、AJAX异步请求等），会将对应任务添加到事件线程中   
当对应的事件符合触发条件被触发时，该线程会把事件添加到待处理队列的队尾，等待JS引擎的处理   
注意：由于JS的单线程关系，所以这些待处理队列中的事件都得排队等待JS引擎处理（当JS引擎空闲时才会去执行）
*** 定时触发器线程 ***
setInterval与setTimeout所在线程   
定时计时器并不是由JS引擎计时的，因为如果JS引擎是单线程的，如果JS引擎处于堵塞状态，那会影响到计时的准确   
当计时完成被触发，事件会被添加到事件队列，等待JS引擎空闲了执行   
注意：W3C的HTML标准中规定，setTimeout中低与4ms的时间间隔算为4ms
*** 异步HTTP请求线程 ***
在XMLHttpRequest在连接后新启动的一个线程   
线程如果检测到请求的状态变更，如果设置有回调函数，该线程会把回调函数添加到事件队列，同理，等待JS引擎空闲了执行

### GUI渲染线程与JS引擎线程互斥 ###
因为JS引擎可以修改DOM树，那么如果JS引擎在执行修改了DOM结构的同时，GUI线程也在渲染页面，那么这样就会导致渲染线程获取的DOM的元素信息可能与JS引擎操作DOM后的结果不一致。为了防止这种现象，GUI线程与JS线程需要设计为互斥关系，当JS引擎执行的时候，GUI线程需要被冻结，但是GUI的渲染会被保存在一个队列当中，等待JS引擎空闲的时候执行渲染。    
因此，如果JS引擎正在进行CPU密集型计算，那么JS引擎将会阻塞，长时间不空闲，导致渲染进程一直不能执行渲染，页面就会看起来卡顿卡顿的，渲染不连贯，所以，要尽量避免JS执行时间过长。

### webWorker ###
JS引擎是单线程的，当JS执行时间过长会导致页面阻塞，对于一些需要CPU密集型计算场景，可以使用webWorker，webWorker可以向浏览器申请一个子线程，该子线程服务于主线程，完全受主线程控制。   
JS引擎线程与worker线程间通过 `postMessage` API通信

```js
// 主线程
let worker = new Worker('./web.worker.js')
worker.addEventListener('message', e => {
  console.log(`%cwebWorker执行完毕，耗时：${e.data.toLocaleString()}ms`, 'color: #52c41a;')
}, false)
worker.postMessage(count)
```

```js
// worker线程
addEventListener('message', e => {
  postMessage(delay(e.data))
}, false)
```

webWorker 不能访问 dom

使用 `postMessage` 传递数据时：   
无论是对象还是基本类型的值，都拷贝数据的副本，可传递[ArrayBuffer](https://segmentfault.com/a/1190000009878632)（最快，序列化->传递->反序列化->内存）、base64、string等多种格式，但是如果需要传递的数据量很大时，传递值还是会很耗时（完整把任务复制给别的线程）。   
还有一种方式是直接转移，这种方式不会随着数据量的变大而变慢，大概有点移交所有权的感觉，用这种方式在worker之间转移数据，会导致始终只有一个worker能访问数据，也是不理想的（把某个特定区域的内存移过去）

由于存在以上两个问题，诞生了共享内存[SharedArrayBuffer](https://segmentfault.com/a/1190000009878699)，但是会有 竞态 问题，可以用 Atomics 解决。