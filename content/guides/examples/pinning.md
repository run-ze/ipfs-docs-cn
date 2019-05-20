---
title: 固定文件
---

固定是 ipfs 中非常重要的概念。ipfs 的语义试图让所有对象都感觉像是本地对象，没有“从远程服务器获取这个文件”，
只有 `ipfs cat` 或 `ipfs get` 这样不管对象实际在哪里，都有相同行为的操作。
虽然这样很好，但有时你会希望能控制哪些东西会留在身边。
固定是一种让你能告诉 ipfs 永远把指定对象保留在本地的机制。
ipfs 有着相当激进的缓存机制，在你对一个 ipfs 对象操作后，会把它保留在本地很短的时间，
但是这些对象可能会经常被垃圾回收。要避免被垃圾回收，只需要简单的固定住你在意的散列。
通过 `ipfs add` 命令添加的对象会默认被递归地固定。
```
echo "ipfs rocks" > foo
ipfs add foo
ipfs pin ls --type=all
ipfs pin rm <foo hash>
ipfs pin rm -r <foo hash>
ipfs pin ls --type=all
```

你可能注意到，第一个 `ipfs pin rm` 命令不起作用，它应该会警告你：给定的散列被“递归固定”了。
ipfs 世界里有三种固定方式：
直接固定：只固定单个分块，与其他块没有关系。
递归固定：固定给定的块，和它所有的子块。
还有间接固定：它的父块被递归固定的结果。

一个被固定的对象不会被垃圾回收，不信的话，试试这个：
```
ipfs add foo
ipfs repo gc
ipfs cat <foo hash>
```

但如果 foo 被取消固定了……
```
ipfs pin rm -r <foo hash>
ipfs repo gc
ipfs cat <foo hash>
```

By [whyrusleeping](http://github.com/whyrusleeping)
