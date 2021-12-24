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
    event.preventDefault();
    // event.stopPropagation()
};

const handleDrop: DragEventHandler<HTMLButtonElement> = event => {
    event.preventDefault();
    // event.stopPropagation()
};
```

## extra link

https://stackoverflow.com/questions/6756583/prevent-browser-from-loading-a-drag-and-dropped-file
