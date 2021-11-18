---
title: ProseMirror Resolved Position
tags: [prosemirror]
date: 2021-11-18
---

## resolvedPos 是做什么的？

> You can resolve a position to get more information about it. Objects of this class represent such a resolved position, providing various pieces of context information, and some helper methods.

`resolvedPos` 将一个 doc 中的位置信息转换成`class ResolvedPos`的实例，通过实例的方法，用户可以方便的获取各种信息

## prerequisite

- `$pos`代表经过`class ResolvedPos`转换的实例, `pos`代表`integer`位置

### 生成$pos

`resolvedPos` 提供了一个 `resolve` 静态方法，将位置信息转化为 `ResolvedPos`

```js
static resolve(doc, pos) {
    if (!(pos >= 0 && pos <= doc.content.size)) throw new RangeError("Position " + pos + " out of range")
    let path = []

    // node-bofore <p> node-start XXXXXX node-end </p> node-after
    // parentOffset 相对当前node开始位置(node-start)的偏移量，剩余可偏移
    // doc 是没有标签的特殊node, 开始位置不需要 +1 直接为pos
    let start = 0, parentOffset = pos
    for (let node = doc;;) {
        // 返回parentOffset 所在 child 在 node.content中的 index, 和相对 node.content start 位置的 offset
        let {index, offset} = node.content.findIndex(parentOffset)
        // 剩余的偏移量
        let rem = parentOffset - offset

        // 当前层级的 node 节点，pos 所在 child 的 index， pos 所在 child 相对整体 doc 的 offset
        path.push(node, index, start + offset)
        if (!rem) break

        node = node.child(index)
        if (node.isText) break

        // 将 parentOffset 变为相对 node 开始位置的偏移
        parentOffset = rem - 1

        // node 开始位置相对 doc 的偏移
        start += offset + 1
    }
    return new ResolvedPos(pos, path, parentOffset)
}

constructor(pos, path, parentOffset) {
    // :: number The position that was resolved.
    this.pos = pos
    this.path = path
    // :: number
    // The number of levels the parent node is from the root. If this
    // position points directly into the root node, it is 0. If it
    // points into a top-level paragraph, 1, and so on.
    this.depth = path.length / 3 - 1
    // :: number The offset this position has into its parent node.
    this.parentOffset = parentOffset
}
```

从这段代码可以看到 `$pos.pos`就是传入的原位置信息

再来看看 prosemirror 如何表示 pos

```html
<p>One</p>
<blockquote>
  <p>Two</p>
</blockquote>
```

```
0   1 2 3 4    5
 <p> O n e </p>

5            6   7 8 9 10    11            12
 <blockquote> <p> T w o  </p> </blockquote>

```

其对应的内部 prosemirror json 结构为

```json
{
  "type": "doc",
  "content": [
    {
      "type": "paragraph",
      "content": [
        {
          "type": "text",
          "text": "one"
        }
      ]
    },
    {
      "type": "blockquote",
      "content": [
        {
          "type": "paragraph",
          "content": [
            {
              "type": "text",
              "text": "two"
            }
          ]
        }
      ]
    }
  ]
}
```

则 `pos 2` `resolve` 后的的`$pos`结构为

```json
{
  "pos": 2,
  "path": [
    {
      "type": "doc",
      "content": [
        ...
      ]
    },
    0,
    0,
    {
      "type": "paragraph",
      "content": [
        {
          "type": "text",
          "text": "one"
        }
      ]
    },
    0,
    1
  ],
  "depth": 1,
  "parentOffset": 1
}
```

### resolveDepth

```js
resolveDepth(val) {
    if (val == null) return this.depth
    if (val < 0) return this.depth + val
    return val
}
```
