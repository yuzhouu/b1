---
title: 如何阻止浏览器选区更换
tags: [prosemirror]
date: 2021-11-24
---

在做 prosemirror 编辑器的时候常常遇到这样的场景，点击 toolbar 时，编辑区内的选区丢失。

解决办法，监听 toolbar 的 mousedown 事件，e.preventDefault(), 可以让选区不丢失，同时编辑区的 blur 事件也不会触发

```js
onMouseDown={e => {
    e.preventDefault()
}}
```

这里看一下 mousedown 的 defaultAction
https://w3c.github.io/uievents/#event-type-mousedown

> Varies: Start a drag/drop operation; start a text selection; start a scroll/pan interaction (in combination with the middle mouse button, if supported)

## 其他

如果 button 不改变 prosemirror 内部状态，选区也不会丢失

css `user-select：none` ， 和外部普通 button 在 click 的情况下，如果没有 dispatch action, 编辑区的 blur 事件会触发，但是选取不会丢失。
