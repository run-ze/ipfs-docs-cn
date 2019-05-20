---
title: 创建自己的 IPFS 服务
---

ipfs 会运行一些默认服务，比如 dht、bitswap 和诊断程序。
它们都只是在 ipfs PeerHost 上注册了一个处理程序，用来监听新连接。
对于这个功能，`corenet` 包有着非常简洁的接口。
所以让我们尝试构建一个简单的演示服务来尝试一下！

让我们从构建服务主机开始：

```
package main

import (
	"fmt"

	core "github.com/ipfs/go-ipfs/core"
	corenet "github.com/ipfs/go-ipfs/core/corenet"
	fsrepo "github.com/ipfs/go-ipfs/repo/fsrepo"

	"code.google.com/p/go.net/context"
)
```

我们不需要为此导入太多依赖。
现在，我们只需要一个 main 函数：

启动一个 ipfs 节点：

```
func main() {
	// Basic ipfsnode setup
	r, err := fsrepo.Open("~/.ipfs")
	if err != nil {
		panic(err)
	}

	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	cfg := &core.BuildCfg{
		Repo:   r,
		Online: true,
	}

	nd, err := core.NewNode(ctx, cfg)

	if err != nil {
		panic(err)
	}
```

这只是使用用户的 `~/.ipfs` 目录中的配置启动默认 ipfs 节点的基本代码模板。

接下来，我们将开始构建我们的服务。

```
	list, err := corenet.Listen(nd, "/app/whyrusleeping")
	if err != nil {
		panic(err)
	}

	fmt.Printf("I am peer: %s\n", nd.Identity.Pretty())

	for {
		con, err := list.Accept()
		if err != nil {
			fmt.Println(err)
			return
		}

		defer con.Close()

		fmt.Fprintln(con, "Hello! This is whyrusleepings awesome ipfs service")
		fmt.Printf("Connection from: %s\n", con.Conn().RemotePeer())
	}
}
```

这就是在 ipfs 之上编写服务所需的全部代码。
当一个客户端连接时，我们向它们发送问候，打印出它的节点 ID，然后关闭回话。
这是最简单的服务，你可以按照自己的想法写代码处理连接。

现在我们需要一个客户端来连接我们：

```
package main

import (
	"fmt"
	"io"
	"os"

	core "github.com/ipfs/go-ipfs/core"
	corenet "github.com/ipfs/go-ipfs/core/corenet"
	peer "github.com/ipfs/go-ipfs/p2p/peer"
	fsrepo "github.com/ipfs/go-ipfs/repo/fsrepo"

	"golang.org/x/net/context"
)

func main() {
	if len(os.Args) < 2 {
		fmt.Println("Please give a peer ID as an argument")
		return
	}
	target, err := peer.IDB58Decode(os.Args[1])
	if err != nil {
		panic(err)
	}

	// Basic ipfsnode setup
	r, err := fsrepo.Open("~/.ipfs")
	if err != nil {
		panic(err)
	}

	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	cfg := &core.BuildCfg{
		Repo:   r,
		Online: true,
	}

	nd, err := core.NewNode(ctx, cfg)

	if err != nil {
		panic(err)
	}

	fmt.Printf("I am peer %s dialing %s\n", nd.Identity, target)

	con, err := corenet.Dial(nd, target, "/app/whyrusleeping")
	if err != nil {
		fmt.Println(err)
		return
	}

	io.Copy(os.Stdout, con)
}
```

这个客户端会建立它的 ipfs 节点（注意：建立节点的成本有些高，你一般无需为每次连接建立一个 ipfs 实例），
然后连接到我们刚刚建立的节点。

要尝试一下，在一个电脑上运行下面的命令：
```
$ ipfs init # 如果你没有已经启动了一个 ipfs 实例
$ go run host.go
```

这应该会打印出它的节点 ID，复制它以便在另一台机子上使用：
```
$ ipfs init # 如果你没有已经启动了一个 ipfs 实例
$ go run client.go <peerID>
```

It should print out `Hello! This is whyrusleepings awesome ipfs service`
这会打印出 `Hello! This is whyrusleepings awesome ipfs service`。

现在，你可能会疑惑：“我为什么要用这个？它比 `net` 包好在哪儿？”
下面是它的好处：

1. 不管一个节点的 IP 如何改变，你都能连接到它。
2. 你利用了我们的软件包的 NAT 穿透的优点。
3. 你得到了一个更有意义的协议 ID 文字而非一个端口数。

By [whyrusleeping](http://github.com/whyrusleeping)
