---
title: 操作网络
---

IPFS 的一切都与网络相关！它包含了一组有用的命令，用以观察网络。

看看你直接连接到了谁：

```sh
ipfs swarm peers
```

如果下面例子中的节点不工作，从 `ipfs swarm peers` 的输出中挑一个。

```sh
ipfs swarm connect /ip4/104.236.176.52/tcp/4001/ipfs/QmSoLnSGccFuZQJzRadHn95W2CrSFmZuTdDWP8HXaHca9z
```

在网络中查找指定的节点：

```sh
ipfs dht findpeer QmSoLnSGccFuZQJzRadHn95W2CrSFmZuTdDWP8HXaHca9z
```

By [whyrusleeping](http://github.com/whyrusleeping)
