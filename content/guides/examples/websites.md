---
title: 将 IPFS 用于网站
---

### 把网站托管到 IPFS 的简短指南

#### 建立你的网站

假设你在 `mysite` 目录有一个静态网站。

为了把它作为网站发布，请确保你的 IPFS 守护程序正在运行：

```bash
$ ipfs daemon
```

然后添加你网站的目录：

```bash
$ ls mysite
img index.html
$ ipfs add -r mysite
added QmcMN2wqoun88SVF5own7D5LUpnHwDA6ALZnVdFXhnYhAs mysite/img/spacecat.jpg
added QmS8tC5NJqajBB5qFhcA1auav14iHMnoMZJWfmr4k3EY6w mysite/img
added QmYh6HbZhHABQXrkQZ4aRRSoSa6bb9vaKoHeumWex6HRsT mysite/index.html
added QmYeAiiK1UfB8MGLRefok1N7vBTyX8hGPuMXZ4Xq1DPyt7 mysite/
```

文件夹名称 `mysite/` 旁边的最后一个散列需要记住，先把它叫做 `$SITE_CID`。

你可以通过使用浏览器、`wget` 或 `curl` 打开 `http://localhost:8080/ipfs/$SITE_CID` 来测试它。

要从另外一个 IPFS 节点查看它，你可以在浏览器中尝试 `http://gateway.ipfs.io/ipfs/$SITE_CID`。
这也可以在你添加网站文件的网络之内或之外的另一个设备上的浏览器中进行。

那些散列很难记住。让我们看看摆脱它们的方法。

#### 使用 DNS 记录作为快捷方式

假设你有一个域名 `your.domain`，并且可以通过注册商的管理面板管理它的 DNS 条目。

为 `your.domain.` 创建一个 DNS TXT 记录（[DNSLink](https://docs.ipfs.io/guides/concepts/dnslink/)），
值为 `dnslink=/ipfs/$SITE_CID`，`$SITE_CID` 是上一节获得的值。

一旦你创建了这条记录，并且它已经被传播开了，你应该能获取它。

```bash
$ dig +noall +answer TXT your.domain
your.domain.            60      IN      TXT     "dnslink=/ipfs/$SITE_CID"
```
现在你可以到 `http://localhost:8080/ipns/your.domain` 浏览你的网站。

你也可以在 `http://gateway.ipfs.io/ipns/your.domain` 网关尝试一下。

#### 使用星际命名系统

每次改变你的网站，你都不得不重新发布它，用新的 `$SITE_CID` 更新 DNS TXT 记录并等待它传播开。

你可以使用 IPNS，[星际命名系统](https://docs.ipfs.io/guides/concepts/ipns/)来绕过这个限制。

你可能注意到上一节的链接中使用的是 `/ipns/` 而非 `/ipfs/`。

IPNS 用于 ipfs 网络中的可变内容。
它相对易用，并且让你无须每次改变网站内容都要更新 dns 记录。

要为你的内容启用 IPNS，请运行以下命令，其中的 `$SITE_CID` 是第一步中获取的散列值。

```bash
$ ipfs name publish $SITE_CID
Published to $PEER_ID: /ipfs/$SITE_CID
```
你需要留意并保存 `$PEER_ID` 的值以便后续使用。

加载 `http://localhost:8080/ipns/$PEER_ID` 和 `http://gateway.ipfs.io/ipns/$PEER_ID` 来确认此步骤成功。

返回注册商的管理面板，把 `your.domain` 的 DNS TXT 记录改为 `dnslink=/ipns/$PEER_ID`，
等待这个记录传播开，然后尝试 `http://localhost:8080/ipns/your.domain`
和 `http://gateway.ipfs.io/ipns/your.domain`。

**注意：** 当使用 IPNS 更新网站时，在更新传播过程中，可能会从获取到的两个不同散列加载网站资源。
这会导致过时的 URL 或缺失的资源，直到更新传播完成。

#### 把你的域名指向 IPFS

现在你在 ipfs/ipns 上有了个网站，但访客无法通过 `http://your.domain` 访问它。

解决方法是让一个 IPFS 网关来解析对 `http://your.domain` 的请求。

返回注册商的管理面板，为 `your.domain` 添加一条 A 记录，
值为一个监听 80 端口的 HTTP 请求的 IPFS 守护程序（比如 `gateway.ipfs.io`）的 IP 地址。
如果你不知道想用的守护程序的 IP 地址，可以用这样的命令找到它：

```bash
$ nslookup gateway.ipfs.io
```
注意返回的 IPv4 地址，你应该为每个地址创建一条 A 记录。

访客的浏览器会在请求头的 Host 字段发送 `your.domain`。
ipfs 网关会识别 `your.domain`，获取你域名的 DNS TXT 记录的值，
然后提供 `/ipns/your.domain/` 而非 `/` 的文件。

如果你把 `your.domain` 的 A 记录指向 `gateway.ipfs.io` 的 IPv4地址，
并且 DNS 已经传播开了，
那么任何人都能不需要额外配置就访问你托管到 ipfs 的网站 `http://your.domain`。

#### 使用 CNAME

此外，也可以使用 CNAME 记录来指向网关的 DNS 记录。
这种情况下，网关的 IP 地址会自动更新。

但你需要更改的是 `_dnslink.your.domain` 而不是 `your.domain` 的 TXT 记录。

所以在为 `your.domain` 创建指向 `gateway.ipfs.io` 的 CNAME 记录，
并且为 `_dnslink.your.domain` 添加记录  `dnslink=/ipns/<your peer id>` 之后，
你可以不需要明确指定 ipfs 网关的 IP 地址就托管你的网站。

希望你玩得开心！

By
[Whyrusleeping](https://github.com/whyrusleeping) and
[Emma Humphries](https://github.com/emceeaich)
