---
title: fix: onDrop 在新标签页打开文件，而不是触发drop事件 
tags: ["js"]
date: 2021-12-24
---

遇到一个问题，在拖动文件到绑定 drop 事件的元素时，drop 事件没有触发，而是直接在新标签页打开了文件，

## 解决方案

同时绑定 DragOver，Drop 事件，同时 preventDefault

```js
const handleDragOver: DragEventHandler<HTMLButtonElement> = event => {
    // 跳过drop区域判断
    event.preventDefault();
};

const handleDrop: DragEventHandler<HTMLButtonElement> = event => {
    event.preventDefault();
};
```

## 解释

A listener for the dragenter and dragover events are used to indicate valid drop targets, that is, places where dragged items may be dropped. Most areas of a web page or application are not valid places to drop data. Thus, the default handling of these events is not to allow a drop.

## extra link

-   https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API/Drag_operations#specifying_drop_targets
