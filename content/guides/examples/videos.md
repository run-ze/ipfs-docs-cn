---
title: 播放视频
---

ipfs 可以用来存储和播放视频，假设我们添加一个视频：

```
ipfs add -q sintel.mp4 | tail -n1
```

以得到的散列为例，你可以用几种不同的方式来查看它：

通过命令行：
```
ipfs cat $vidhash | mplayer -vo xv -
```

通过本地网关：
```
mplayer http://localhost:8080/ipfs/$vidhash

# 或者在chrome或firefox的标签中打开

chromium http://localhost:8080/ipfs/$vidhash
```
(Note: the gateway method works with most video players and browsers)
（注意：网关方式适用于大多数视频播放器和浏览器）

By [whyrusleeping](http://github.com/whyrusleeping)
