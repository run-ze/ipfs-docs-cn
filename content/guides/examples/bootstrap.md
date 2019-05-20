---
title: 更改引导节点列表
---

IPFS 会通过一些节点了解网络上的其它节点，这些节点的列表叫做 IPFS 引导列表。
IPFS 默认自带一些可信节点的列表，但你也可以按照需要更改这个列表。
自定义引导列表的一个常见用途是创建私人 IPFS 网络。

首先，让我们列出节点的引导列表：

```
> ipfs bootstrap list
/ip4/104.131.131.82/tcp/4001/ipfs/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ
/ip4/104.236.151.122/tcp/4001/ipfs/QmSoLju6m7xTh3DuokvT3886QRYqxAzb1kShaanJgW36yx
/ip4/104.236.176.52/tcp/4001/ipfs/QmSoLnSGccFuZQJzRadHn95W2CrSFmZuTdDWP8HXaHca9z
/ip4/104.236.179.241/tcp/4001/ipfs/QmSoLpPVmHKQ4XTPdz8tjDFgdeRFkpV8JgYq8JVJ69RrZm
/ip4/104.236.76.40/tcp/4001/ipfs/QmSoLV4Bbm51jM9C4gDYZQ9Cy3U6aXMJDAbzgu2fzaDs64
/ip4/128.199.219.111/tcp/4001/ipfs/QmSoLSafTMBsPKadTEgaXctDQVcqN88CNLHXMkTNwMKPnu
/ip4/162.243.248.213/tcp/4001/ipfs/QmSoLueR4xBeUbY9WZ9xGUUxunbKWcrNFTDAadQJmocnWm
/ip4/178.62.158.247/tcp/4001/ipfs/QmSoLer265NRgSp2LA3dPaeykiS1J6DifTC88f5uVQKNAd
/ip4/178.62.61.185/tcp/4001/ipfs/QmSoLMeWqB7YGVLJN3pNLQpmmEk35v6wYtsMGLzSr5QBU3
```

上面这些行是 IPFS 默认引导节点的地址——它们由 IPFS 开发团队运行。
这些地址全部由 [multiaddr](https://github.com/jbenet/multiaddr) 格式定义和解析，所有的协议都是显式指定的。
这样，你的节点明确地知道怎么连接到这些引导节点——它们的地址是明确的。

除非你知道这样做意味着什么，否则不要改变这个列表。
引导是分布式系统中的安全要点，恶意的引导节点只会告诉你其他恶意节点。
建议保留 IPFS 开发团队提供的默认列表，或者——在建立私人网络时——只保留在你掌控下的节点列表。
不要把你不信任的节点添加到列表里。

这里我们往引导列表里添加一个节点：
```
> ipfs bootstrap add /ip4/25.196.147.100/tcp/4001/ipfs/QmaMqSwWShsPg2RbredZtoneFjXhim7AQkqbLxib45Lx4S
```

这里我们从引导列表里删除一个节点：
```
> ipfs bootstrap rm /ip4/128.199.219.111/tcp/4001/ipfs/QmSoLSafTMBsPKadTEgaXctDQVcqN88CNLHXMkTNwMKPnu
```

假如我们想为新的引导列表创建备份，可以通过把 `ipfs bootstrap list` 的输出重定向到一个文件来做到：
```
> ipfs bootstrap list >save
```

如果我们想从头开始，我们可以一次删掉全部引导列表：
```
> ipfs bootstrap rm --all
```

如果引导列表是空的，我们可以恢复默认的列表：
```
> ipfs bootstrap add --default
```

再次删除全部引导列表，然后通过把我们之前保存的列表用管道传给 `ipfs bootstrap add` 来恢复它：
```
> ipfs bootstrap rm --all
> cat save | ipfs bootstrap add
```


By [jbenet](http://github.com/jbenet) and [insanity54](http://github.com/insanity54)
