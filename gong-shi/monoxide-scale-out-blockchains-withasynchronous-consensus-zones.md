# Monoxide: Scale Out Blockchains with Asynchronous Consensus Zones

## 系统设计

![](../.gitbook/assets/monoxide_arch.png)

### 帐户

Monoxide的帐户采用Account/Balance模型，帐户包含一些状态，公钥哈希值用作帐户地址\(Address\)，可以唯一标识帐户。所有帐户按照地址被分割到$$2^k$$ 个分区。

### 共识组

节点拥有一个持久化的标识符\(可在第一次启动时随机初始化\)，所有的节点根据标识符也被分割 为 $$2^k$$个分区。每个分区用 $$(s,k)$$ 唯一表示，其中 $$s$$ 是分区号， $$k$$ 是分区系数， $$2^k$$ 是分区总数。

属于同一个分区的全节点组成一个共识组\(Consensus Zone\)。

###  网络

系统网络被划分为一些逻辑分区，称为Swarm。节点加入Swarm后可以向本Swarm广播交易或者接受本Swarm内其它节点广播的块或交易。

每个共识组都有一个自己的Swarm，同时系统中存在一个全局的Swarm，所有的节点都会加入该Swarm，用来完成跨共识组通讯。

网络通信采用DHT算法，用来实现Swarm寻址和节点发现。

