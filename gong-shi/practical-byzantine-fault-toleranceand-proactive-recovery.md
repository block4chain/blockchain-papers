# Practical Byzantine Fault Toleranceand Proactive Recovery

[论文原文](https://drive.google.com/open?id=1t0-TWaR66MPQx-2v8N-Syzgm4VSvTRlp)

## 系统模型

![](../.gitbook/assets/pbft-model.png)

### 拜占庭错误

replica可以是任意行为：下线、崩溃、拒绝服务、作恶、篡改数据等

### 网络模型

replica消息在网络上传输时可以出现丢失、延时、重复、乱序。

### 系统假设

* 出现错误的replica数量不超过 $$f$$ :

$$
f = \lfloor \frac{(n-1)}{3}\rfloor
$$

* 敌手的算力有限，无法攻破算法用到的密码学技术
* 弱同步: 消息延迟时间有上界，即消息从发出到被接收，只要没有丢失，就一定能被接收，并且延迟时间有上界。

## 算法性质

### 安全性

所有的replica按照同样的顺序执行相同操作

### 活性

最多允许 $$f = \lfloor \frac{(n-1)}{3}\rfloor$$ 个replica出现错误

## 系统参数

### Replica集合

replica集合用 $$R$$ 表示，replica集群大小记为 $$|R|$$ 

每个replica存在一个唯一标识 $$p$$ , 其中:$$p \in [0,1,2,3,...,|R|-1]$$ 

系统允许replica错误的最大数量 $$f$$ , 其中: $$|R|=3f+1$$ 

### 视图

视图用整数 $$v$$ 表示. 每个视图存在一个主replica，主replica的标识: $$p = v \  mod \  |R|$$ 

### Quorum

至少 $$2f +1$$ 个replica构成的子集称为quorum, 记为 $$Q$$ , 即: $$ |Q| \ge 2f +1,Q \subset R$$

### Quorum Certificates

至少 $$2f+1$$ 个不同replica的消息构成一个quorum certificates, 记为 $$QC$$，即 $$|QC| \ge 2f+1$$

至少 $$f+1$$ 个不同replica的消息构成一个weak certificates, 记为 $$WC$$，即 $$|WC| \ge f+1$$

## 主要技术

状态机复制\(State Machine Replication\)技术的难点是保证非错误的replica节点以相同的顺序执行相同的操作。算法采用两个技术解决这个问题:

* 主备机制\(Primary-backup\)
* Quorum

算法使用Primary-backup技术决定_请求操作的执行顺序_\(ordering for execution of operations\)，Quorum技术解决节点异常\(崩溃等非拜占庭问题\)问题。

### Primary-backup

* replica被划分成_primary_和_backup_两种角色。
* primary稳定运行的时间窗口称为_视图\(View\)_，当primary失败会发生视图切换
* 每个视图只有一个primary，其它replica都是backup
* primary负责决定客户端请求的顺序，为每个客户端请求分配一个_序列号\(sequence number\)_
* 在一个视图内请求序列号连续递增。

### Quorum

算法使用**Byzantine dissemination quorum system**，这些quorum满足2个性质:

* 任意两个quorum: $$Q_1$$和 $$Q_2$$ ，则至少有一个正确的replica节点 $$p$$ 满足:

$$
p \in Q_1 \cap Q_2
$$

* 总存在至少一个quorum，这个quorum不存在错误的replica节点。

这两个性质保证可以将这个quorum system看成一个可靠存储存放一些协议信息。

## 客户端

### 会话

客户端会和replica建立连接，并协商一个会话密钥，用于验证消息MAC

### 请求

客户端 $$c$$向replica集群组播\(multicast\)一个操作请求: $$<Request, o, t, c>$$ ，其中:

* $$o$$ 代表操作
* $$t$$ 代表操作发起的时间戳

### 响应

replica节点在操作处理完成后会返回客户端一个响应: $$<Reply, v, t, c,i,r>$$ ，其中:

* $$v$$ 是当前的视图号
* $$i$$ 是响应节点的标识
* $$r$$ 是操作结果

客户端会收集来自 $$f+1$$ 个不同replica节点的响应，并验证:

* 响应消息的MAC合法有效
* 所有响应的 $$(c,v, t, r)$$ 相等

验证通过后，则请求成功。$$f+1$$ 个响应构成一个Weak Certificates，又称为_Reply Certificate_。

### 请求重发

超过一定的时间后，如果客户端没有接收到Reply Certificate\(没有收集到足够的响应\)，就会重发请求。

replica节点会保存它向客户端响应的最后一条Reply，如果replica节点已经响应过该请求，则将保存的reply响应给客户端。

### 资源消耗

为了支持客户端，replica节点会有一些额外消耗:

* 客户端ID与会话密钥的映射
* 客户端最后一条请求的响应





















