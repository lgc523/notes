---
title: "TCP数据传输-分片、滑动窗口、重发、流控制"
date: 2021-09-01T01:28:08+08:00
draft: true
toc: true
images:
tags: 
  - tcp
---

## 通信问题

自身不包含可靠传递数据机制的协议，他们可能会使用用一种像数据校验和或者CRC这样的数学函数来检测收到的有差错的数据，但是他们不会去纠正差错。以太网和基于其上的其他协议，协议提供了一定次数的重试，如果还是不成功则放弃。

使用差错校正码(基本是添加一些冗余的比特，使得即使某些比特被毁，真实的信息也可以被恢复回来)来纠正通信问题是处理差错的一种非常重要的方法。

另外一种方法是简单的 “**尝试重新发送**" ,直到信息最终被接收，这种方法被称为**自动重复请求**(Automatic Repeat Request，ARQ)，构成了许多通信协议的基础，包括TCP在内。

客户端可服务器之间的通信是一个数据传输过程，通信的消息将以数据包形式进行传输，为了更有效的进行通信，TCP协议在数据进行数据传输时，使用滑动窗口机制来同时发送多个数据包，当数据包丢失时，TCP协议利用数据重发功能重新发送数据包，因接收端接收数据包的能力不同，TCP流控制会根据接收能力发送适当数量的数据包。

## 多跳通信信道问题

差错类型(分组比特差错)

分组重新排序

分组复制

分组泯灭(丢失)

## 通过重发分组处理分组丢失的问题

重发分组直到它被正确的接收来解决分组丢失、比特差错，需要判断

1. 接收方是否收到分组
2. 接受方接收到的分组是否与之前的发送方发送的一样。

接收方给发送方发信号以确定自己已经接收到一个分组，称为确认或者**ACK**(acknowledgement)。就是发送方发送一个分组，然后等待一个ACK。接收方收到这个分组时，发送对应的ACK。方发送发收到这个ACK，再发送另一个分组，一直继续。

1. 发送方对一个ACK  应该等待多长时间?

2. ACK 丢失怎么办？

   ACK 丢失，发送方不能轻易地把这种情况与原分组丢失的情况区分出来，所以它简单的再次发送原分组。这样接收方就可能会收到两个或更多的拷贝，因此必须准备好处理这种重复分组的情况。

   接受方收到被传送分组的重复(duplicate)副本，可以通过**序列号**(sequence number) 来处理。基本上在源端发送时，每个唯一的分组都有一个新的序列号，这个序列号由分组自身一直携带着。接受方可以使用这个唯一的序列号来判断它是否已经见过这个分组，如果见过就丢弃它。

3. 分组被接收到了，里面有错怎么办？

   使用编码来检测一个大的分组中的差错一般都很简单，仅使用比其自身小很多的一些比特即可纠正，更简单的编码一般不能纠正差错，但是能检测他们(这就是校验和与CRC受欢迎的原因)，为了检测分组里的差错，使用一种**检验和**形式。当一个接受方收到一个含有差错的分组时，它不发送ACK，最后发送方重发完整到达的无差错的分组。

## 数据分片

数据从主机传送到另一个主机往往要经过路由器、网关等设备，网络设备对数据处理有一定的限制，不能处理超过额外字节的数据，所以发送数据的时候需要确定发送的数据的最大字节数。这个最大字节数被称为**最大消息长度**(Maximum Segment Size，MSS)，当要发送的数据超过该值，就需要将数据分为多个包，以此发送，该操作叫做数据分片。

那不同网络设备 和 MSS 怎么取舍，过一个设备分片多次？

MSS 是TCP数据包每次能够传输的最大数据量。通常最大值为1460 字节，(回忆一下，以太网数据桢最大传输单元MTU 通常1500字节)，

如果发送的数据包大小大于MSS值，数据包将会被分片传输。

在TCP前两次握手时，TCP首部包含**MSS**选项，互相通知对方网络接口能够适应的MSS大小，然后双方使用较小的MSS值进行传输。

## 停止和等待

发送方发送一个分组，等待一个ACK，等不到就重发可以保证可靠，但是效率不高。如果一个分组的发送都要很长时间(推迟或延迟)，发送方注入一个分组到通信链路中，然后停下来等待知道它收到ACK，这个协议也被称为 "**停止和等待**"。这种一个分组接连一个分组的同步发送，使得**吞吐量**性能(每单位时间发送在网络中的数据量) 与 **M/R** 成正比，M 是分组大小，R 是往返时间 (Round-Trip Time，RTT)。如果有分组丢失或损害，吞吐量更低。**吞吐质**(每单位时间传送的有用数据量)明显要比吞吐量低。

## 多个分组进入让网络更繁忙

不会损害和丢失太多分组的网络，低吞吐量的原因是网络经常没有处于繁忙状态。如果允许同一时间多个分组进入网络，就可以使它更繁忙，从而得到更高的吞吐量。

允许多个分组同时进入网络后，发送方不仅要决定什么时候注入一个分组到网络中，还要考虑注入多少个。并且必须要指出在等待ACK时，怎么**维持计时器**，同时还必须要保存每个还没确认的分组的一个副本以防需要重传。

接受方需要有一个更复杂的**ACK机制**：**可以区分哪些分组已经收到，哪些还没有**。接受方可能需要一个更复杂的缓存(分组保存)机制允许**维护次序杂乱的分组**(那些比预想要先到的分组更早到达的分组，因为丢包和次序重排的原因)，除非简单的抛弃这些分组(效率降低)。

另外还要有**差速控制**，如果接受方的接受速率比发送方的发送速率要慢，发送发简单地以很高速速率发送很多分组，接受方可能会因为处理或内存限制而丢到这些分组。中间的路由器也是会有这些问题。

## 滑动窗口机制

在进行数据传输时，如果传输的数据比较大，就需要拆分为多个数据包进行发送。TCP协议需要对数据进行确认后，才可以发送下一个数据包。

发送端每发送一个数据包，都需要得到接受端的确认应答以后，才可以发送下一个数据包，这样以来，就会在等待确认应答包环节浪费时间。为了避免这种情况，TCP引入了滑动窗口概念。窗口大小指的是不需要等待确认应答包而可以继续发送数据包的最大值(**已经被发送和注入单还没完全确认的分组数量**)，避免了网络的吞吐量降低。如果把通信会话中的所有分组排成长长的一行，但只能通过一个小孔观察他们，就只能看到他们的一个子集，像通过一个窗口观察一样。窗口的这种滑动，给这种类型的协议增加了一个名字，**滑动窗口**(**sliding window**)协议。

窗口大小是指可以发送数据包的最大数量，可以分为两部分，一部分是数据已经发送，但是没有得到确认应答包；第二部分表示允许发送，但未发送的数据包。

当发送了最大数量的数据包(窗口大小数据包)，有时不会同时收到这些数据包的确认应答包，而是收到部分确认应答包。这个时候窗口就通过滑动的方式，向后移动，确保下一次发送仍然可以发送窗口大小的数据包。

窗口结构在发送发和接受方都有，**在发送方，窗口记录哪些分组可以被释放、哪些分组正在等待ACK，以及哪些分组还不能被发送。在接受方，记录着哪些分组已经被接受和确认，哪些分组是下一步期望的(和已经分配多少内存来保存它们)，以及哪些分组即使被接受也将会因内存限制而被丢弃，操作系统内核位了维护这个滑动窗口，需要开辟发送缓冲区来记录当前还有哪些数据没有应答，只有确认应答的数据，才能从缓存区删掉**。

## 数据重发

在进行数据包传输时，难免会出现数据丢失情况。

- **未使用滑动窗口机制，发送的数据包没有收到确认应答包，数据都会被重发**
- **使用了滑动窗口，确认应答包丢失，也不会导致数据包重发**
- **发送的数据包丢失，将导致数据包重发。**

### 确认应答包丢失

**前面发送的数据包没有收到对应的确认应答，当收到后面数据包的确认应答包，表示前面的数据包已经成功被接受端接收了，发送端不需要重新发送前面的数据包。**

那意思是只有窗口大小里面都失败才会重发？

为什么滑动窗口确认包丢失可以不重发？

### 发送数据包丢失

**发送端的部分数据包没有到达接收端。**

**如果在接受端收到的数据包，不是本应该要接收的数据包，就会给发送端返回消息，告诉发送端自己应该接收到的数据包。如果发送端连续3次收到这样的数据包，就认为该数据包成功发送到接收端，这时就开始重发该数据包。**

**同时在收到不是希望的序号+1包时，也会确认收到数据包，并且在发送端重发是，确认返回下次应该收到的最新的序号+1数据包。**

可以调整嘛？怎么调整可以达到最优吞吐量？

## 变量窗口

### 流量控制

为了处理的那个接受方相对发送发太慢时产生的问题，在接受方跟不上时会强迫发送方慢下来，这称为**流量控制(flow control)**。一种方式是基于**速率**(rate-based)流量控制，它是给发送方指定某个速率，同时**确保数据永远不能超过这个速率发送**。这种类型的流量控制最适合流应用程序，可被用于广播和组网发现。另外一种方式就是基于窗口流量控制，是使用滑动窗口时最流行的方法，**窗口的大小不是固定的，而是允许随着时间变动的**。可以变动，就必须有一种方法让接受方通知发送方使用多大的窗口。这个一般称为**窗口通告(window advertisement)**，或简单地称为**窗口更新(window update)**。发送方(窗口通告的接受者)使用该值调整其窗口的大小，窗口更新和ACK 是由同一个分组携带的，意味着**发送方往往会在它的窗口滑动到右边的同时调整它的大小**。

在使用滑动窗口机制进行数据传输时，发送方根据实际情况发送数据包，接收端接收数据包，但是接收端处理数据的能力是不同的。

- **窗口过小**，发送端发送少量的数据包，接收端很快就处理了，并且还能处理更多的数据包，当传输比较大的数据时，需要不停地等待发送方，造成很大的延迟，也就是饥饿现象(都喜欢整出来这样的一个名词)。
- **窗口过大**，发送端发送大量的数据包，接收端处理不了那么多的数据包，就会堵塞链路，如果丢弃这些本应该接收的数据包，又会触发重发机制

为了避免两端数据速率的不对称，TCP 提供了流控制，就是使用了不同的窗口大小发送数据包，发送端第一次从窗口大小 (窗口大小是根据链路带宽的大小来决定的) 发送数据包，接收端收到这些数据包，并返回确认应答包，告诉发送端自己下次希望收到的数据包是多少 (新的窗口大小 )，发送端收到确认应答包以后，将以该窗口大小进行发送数据包。

### 带流控制的流程

- 发送端根据当前的链路带宽大小决定了发送数据包的窗口大小，如果当前窗口为3，可以发送3个数据包
- 接收端收到数据包，但是只能处理2个数据包，第三个数据包就没有被处理，确认应答包反馈希望下次窗口大小为2
- 发送端收到确认应答包，知道了希望的窗口大小为2，并且上次发送的数据包处理了2个，还有1个没处理，第三个数据包还需要重发

如果接收端返回的确认应答包中，窗口设置为0，表示当前不能接收到任何数据(缓冲区满了)，发送端将不会在发送数据包，只有等待接收端发送窗口更新通知才可以继续发送数据包。

如果这个时候更新的通知在传输中丢失了，那么就可能导致无法继续通信。为了避免这样的情况发生，发送端会时不时地发送窗口探测包(发送端等待时间超过了重发超时时间)，该包仅1字节，用来获取最新的窗口大小信息，接收端收到发送端的探测包，再次发送窗口大小更新包。

每次应答都多了一个下次窗口的大小，如果发送端很快，接收端很慢，不是会拖垮发送端的速度，视频的话，因为一方网卡，就会有声音、画面延迟，卡屏，不卡的一方是正常的视频，只是收到对方的视频是卡的，好像就是快的快，慢的慢。这个流控制没感觉到让快的快，慢的慢慢的快，不能让慢的一方在中间缓存层?中慢慢处理？

### 拥塞控制

在发送发第一次发送数据时，是不知道对接受方的影响的。在没有收到任何一个ACK 之前，发送方允许注入 W 个分组到网络中。如果发送发和接受方足够快，网络中没有丢失一个分组以及有无穷的空间的话，这意味着通信速率正比于(SW/R) b/s，W 是窗口，S 是分组大小(bit)，R 是往返时间(RTT)。当来自接受方的窗口通道夹带着发送方的值W时，那么发送方的全部速率就被限制而不能超越接受方。这种方法可以很好的保护接受方，但是对于中间网络(有限内存的路由器)，他们与低速网络链路抗争着。当这种情况出现时，发送方的速率可能超过某个路由器的能力，从而导致丢包，这由一种特殊的称为**拥塞控制(congestion control)**的流量控制形式来处理，涉及到发送方减低发送速度不至于压垮与其接受方之间的网络。

使用一个窗口通告来告之发送方为接受方减慢速度，这个称为明确发信，因为有一个协议字段专用于通告发送方正在发生什么，另一个选项可能被用于猜测(guess)它需要慢下来。这种方法设计隐私(implicit) 发信------设计根据其他某些证据来决定速度。

### 拥塞控制技术



## 重传超时

基于重传的可靠协议要面对的重要性能问题是要等待多久才能判定一个分组已丢失并将它重发。

发送方在重发一个分组之前应等待的时间量大概是下面时间的总和

- 发送分组所用的时间
- 接受方处理它和发送一个ACK所用的时间
- ACK 返回到发送方所用的时间
- 发送方处理ACK所用的时间

然而，上面的时候没有一个可以确切知道的，另外这些时间还会随着网络设备的额外负载的变化而变化。

**往返时间估计(round-trip time estimation)**：让协议去估计它们，是一个统计过程。

选择一组 RTT 样本的样本均值作为真实的RTT是最有可能的，这个平均值自然地会随着时间改变，因为通信穿过的网络的路径可能会改变。往返时间估计直接用于触发重传的时间计时值是 不合理的，会引起一些不必要的重传或者网络空闲。

## 设置RTT

todo

## TCP的可靠性

TCP 提供了一个字节流接口，TCP必须把一个发送应用程序的字节流转换成一组IP可以携带的分组，这被称为**组包(packetization)**。这些分组包含序列号，该序列号在TCP中实际代表了每个分组的第一个字节在整个数据流中的字节偏移，而不是分组号。这允许分组在传送中是可变大小的，并允许他们组合，称为**重新组包(repacketization)**。应用程序数据被打散成TCP认为的最佳大小的块来发送，一般使得每个报文段按照不会被分片的单个IP层数据报的大小来划分，由TCP传给IP的块称为**报文段(segment)**。

**TCP维持了一个强制的校验和**，该检验和涉及它的头部、任何相关应用程序数据和IP头部的所有字段。这是一个端到端的伪头部。它用于检测传送中引入的比特差错。如果一个带无效校验和的报文段到达，TCP会丢弃它，**不为被丢弃的分组发送任何确认**。TCP接受端可能会对一个以前的(已经确认的)报文段进行确认，以帮助发送方计算它的拥塞控制。TCP校验和使用的数学函数与其他互联网协议一样。对于大数据的传送，对这个校验和要注意是否强壮，必要的应用程序应该用自己的差错保护方法(更强的校验和或CRC)，或者使用一种中间层来达到同样的效果。

当TCP发送一组报文时，它通常设置一个重传计时器，等待对方的确认接收。TCP不会为每个报文段设置一个不同的重传计时器。相反，发送一个窗口的数据，它只设置一个计时器，当ACK报文到达时再更新超时，如果有一个确认没有及时接收到，这个报文段就会被重传。

当TCP接收到连接的另一端的数据时，他会发送一个确认。这个确认可能不会立即发送，而一般会延迟片刻。TCP 使用的ACK 是累积的，从某种意义来讲，一个指示字节号N的ACK暗示所有直到N的字节(但不包含N)已经成功被接收了。这对于ACK丢失来说带来了一定的鲁棒性--------如果一个ACK丢失，很有可能后续的ACK就足以确认前面的报文段了。

TCP 给应用程序提供一种**双工**服务。就是说数据可向两个方向流动，两个方向互相独立。因此，连接的每个端点必须对每个方向维持数据流的一个序列号。**一旦建立了一个连接，这个连接的一个方向上的包含数据流的每个TCP报文段也包含了相反方向上的报文段的一个ACK**。每个报文段也包含了一个窗口通告以实现相反方向上的流量控制。为此，在一个连接中，当一个TCP报文段到达时，窗口可能像前移动，窗口大小可能改变，同时新数据可能已到达。

一个完整的TCP连接是双向和对称的，数据可以在两个方向上平等地流动。

一旦建立了一个连接，这个连接的一个方向上的包含数据流的每个TCP报文段也包含了相反方向上的报文段的一个ACK ???

使用序列号，一个TCP接受端可丢弃重复的报文段和记录以杂乱次序到达的报文段。因为TCP使用IP来封装传递它的报文段，IP不提供重复消除或保证次序正确的功能。然而TCP是一个字节流协议，绝不会以杂乱的次序给接受应用程序发送数据。因此，TCP接受端可能会被迫先保持大序列号的数据不交给应用程序，直到缺失的小序列号的报文段(一个“洞“)被填满。

设计的真好啊

- [ ] Nagle
- [ ] 拥塞控制算法



