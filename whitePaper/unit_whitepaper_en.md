# Unit: peer-to-peer network dedicated to decentralized big data and bring anonymity.



[Kirill Zhukov](https://github.com/Abuzik), [George Stolyarov](https://github.com/TorgaW)

E-MAIL: unitchainorg@gmail.com

www.unitchain.org

## 1. Introduction.

I think that in 2023, it is not necessary to explain what `Bitcoin` and blockchain are in general, but it is necessary to describe certain implementations of what needs to be fixed, in my humble opinion. At the moment, there are a huge number of blockchains that can process "smart contracts" invented as far back as 1995 by [Nick Szabo](https://en.wikipedia.org/wiki/Nick_Szabo), since then platforms such as [Ethereum](https://ethereum.org/), [Solana](https://solana.com/), etc. have appeared. All of them offer great opportunities for code execution to change the state of the user's account (see Ethereum's whitepaper), but if Ethereum suffers from high commissions (in 2023), then Solana is devoid of this disadvantage. It can be said that `Solana` is a solution to all problems, but in practice it turns out that this blockchain is [weakly decentralized](https://learn.bybit.com/deep-dive/is-solana-decentralized/#2). In addition, despite all the advantages of the blockchains mentioned above, there is a huge drawback, in my opinion - the inability to store decentralized applications (dApps) within the blockchain, as well as the limited memory available to the application and users. `Unit` is being developed to solve the following problems: commission (gas price), blockchain speed, decentralization, code security, code execution in a virtual machine, and most importantly - the ability to store data and applications in a decentralized way, without compromising the performance of the entire network.

## 2. Unit

Unit offers to divide the network into two types of nodes: **Shard** and **Core**.

### Core-nodes

**Core** nodes are intended to keep the main network executing ***light*** transactions (those that do not require interaction with a large amount of data).

![core-shard-diagram](https://raw.githubusercontent.com/Unit-chain/whitepaper/896829b8314f6cc14f0ae6d65660c5fc60d37b0d/whitePaper/static/core_blockchain_structure.drawio.svg)

### Future-blocks

To accelerate the "extraction" of a new block, a structure is used that stores **future-blocks** - blocks that do not have a finalized state, but at the same time, if a **Core-node** wins the right to extract a block, it is taken from the corresponding pool, thus turning it from a **future-block** category into a regular block. This is necessary so that the time is not spent on block generation, but on node selection. An important point is as follows: if the current block index is N, then the nodes that will provide blocks up to N+3 blocks will be known in advance, this is necessary to reduce latency when writing transactions.

### Recording transactions into block.

After a **Core-node** is selected to provide a block, other nodes begin to provide transactions for recording in that block. The block is alive for a certain period of time (at least 3 seconds - it is important to avoid a large number of transactions that did not make it in), when the block is selected, it is given the ***closing*** tag, which is not included in the end in the blockchain database, but is used to finalize the block and successfully complete the recording of all transactions sent from other nodes (since the transactions recorded in the block have a ***timestamp*** field, when the transaction was sent and where it was sent from (***IP*** address of the node)). When the block is selected, its timestamp, indicating its ***closing***, is broadcasted to all nodes so that they know until what point they can send transactions to this node.

### Unit Virtual Machine

The **Unit Virtual Machine** or **Theia Virtual Machine (TVM)** is nothing more than a [Turing-complete](https://en.wikipedia.org/wiki/Turing_completeness) virtual machine. The Theia virtual machine provides a wide range of tools, including unsigned integers from 32 to 256 bits. Using numbers less than 256 bits can save computational power for, for example, calculating small sums of transfers (or gas price). Full list of native supported data types (as of 2023):

| Data type | Note                                                         |
| --------- | ------------------------------------------------------------ |
| byte      | signed type from -128 to 127, inclusive                      |
| short     | signed type from -32768 to 32767, inclusive                  |
| int       | signed type from -2147483648 to 2147483647, inclusive        |
| long      | signed type from -9223372036854775808 to 9223372036854775807, inclusive |
| uint32    | unsigned type from 0 to 4294967295                           |
| uint64    | unsigned type from 0 to 18446744073709551615                 |
| uint128   | unsigned type from 0 to 340282366920938463463374607431768211455 |
| uint256   | unsigned type from 0 to 115792089237316195423570985008687907852589419931798687112530834793049593217025 |
| double    | in range: 1.7E +/- 308                                       |
| char      | `'\u0000'` to `'\uffff'` inclusive or else from 0 to 65535   |
| address   | can't be used from Theia, but used by virtual machine. **Don't confuse with WalletAddress, it's important!** WalletAddress not native datatype. |

In contrast to the [Ethereum Virtual Machine (EVM)](https://ethereum.org/en/developers/docs/evm/), TVM supports [reflection](https://en.wikipedia.org/wiki/Reflective_programming) and ***events***. They are necessary to connect contracts and their methods (both from helper classes and directly from them) and when accessing data handlers on the **Shard-node**, ***events*** are used specifically.

![TVM interacting types](https://raw.githubusercontent.com/Unit-chain/whitepaper/896829b8314f6cc14f0ae6d65660c5fc60d37b0d/whitePaper/static/VM_interacting_types.drawio.svg)

One of the major benefits of **TVM** is the ability to use it outside the blockchain for ordinary applications (more on this in the **Theia** language section). To interact with this and **TVM**, a [VFS](https://en.wikipedia.org/wiki/Virtual_file_system) is used to ensure the security of the data of the OS on which **TVM** runs, accordingly, data management within the contract will not differ significantly from working with files in a regular OS.

### Shard-nodes

**Shard-nodes** are designed to store large amounts of data and interact with them. **Shard-nodes** cannot support the network independently, but they help speed it up and are used for data storage. When data is placed on a node of this type, a contract describing the behavior for working with it should be written. If data is placed by a regular user, their interaction with it is described by a standard contract. The data placed is automatically replicated to other **Shard-nodes** to increase fault tolerance. Memory is not allocated permanently and there are various ways to obtain decentralized storage: 1) Raise your own **Shard-node**, then a certain percentage of allocated memory will be available for free for the duration of the node's operation (not the full amount of memory, because data is stored in a decentralized manner and, for example, a stored file of 1MB in total will take up, if present, around 20MB on 100 **Shard-nodes**), 2) Replenish the balance of the contract, intended to serve as a contract-handler for your data, in an amount equivalent to, for example, (average cost of 1 byte) = $a$, (volume of data occupied) = $b$, (cost of gas for data) = $c$, then the final sum ( $\mathcal{E}$ )(as of 2023, the formula is current), which will be paid for placing data will be:

$$
\mathcal{E} = a * b * c
$$

Due to this simple math, the owners of **Shard-nodes** have reason to raise these nodes.

### Interaction with data on Shard-nodes

![shard core](https://raw.githubusercontent.com/Unit-chain/whitepaper/896829b8314f6cc14f0ae6d65660c5fc60d37b0d/whitePaper/static/shard_core.drawio.svg)

With the emergence of **Shard-nodes** in the blockchain, the concept of ***request*** is first introduced, which is a regular request to a contract, for example, a social network for sending a message from *User_1* to *User_2*. Then all expenses are transferred to the contract owner, which allows this social network to interact with its storage (as in WEB1/2). However, to send data, such as files from *User_1* to *User_2*, they will have to pay a one-time commission of $\mathcal{E}$ (the formula is provided above), as this is a fee for placing data, but this is only one way of interaction, which is more of a recommendation, the final implementation is entirely up to the specific developer.

Also, users who rent decentralized file space have the right to access their data, share access to them (in the standard contract, which provides standard data processing, the addresses of wallets that have access to the storage will be specified, they can be added or removed unconditionally), throughout the entire time that they have paid for.

### Theia 

The name of the language comes from the name of the ancient Greek goddess [Theia](https://en.wikipedia.org/wiki/Theia), who was the daughter of [Uranus](https://en.wikipedia.org/wiki/Uranus_(mythology)) and [Gaia](https://en.wikipedia.org/wiki/Gaia). This goddess was not chosen by chance. Her father, Uranus, is the son of the god of chaos, [Aether](https://en.wikipedia.org/wiki/Aether_(mythology)), in some myths. Try to find the reference :)

**Theia** is an object-oriented, high-level programming language that runs on the **Theia Virtual Machine(TVM)**.

The development of **Theia** was heavily influenced by languages such as *C++*, *Java*, and *Python*. From *C++* and *Java*, classes and strict typing were borrowed, which will make it easier and more meaningful to approach project design and implementation. From *Python*, simplicity and versatility of syntax were inherited, so even inexperienced developers can try their hand at mastering the language and supporting the blockchain infrastructure. The similarity of syntax with most other languages will make it easy to transition to **Theia** from other programming languages with minimal time spent on retraining.

**Theia** is not strictly tied to blockchain. Modified versions of **TVM** are capable of working independently from the network. This means there is the possibility to use the language for everyday tasks that are not related to blockchain. If a developer wants to change the development vector of the product to blockchain, then there will be an easy way to solve this problem by changing the version of **TVM** and making small modifications to the application code. This solution will expand the circle of interested parties in programming and development not only for blockchain infrastructure but also for ordinary applications with further transition to *Web 3.0*.

The language is created according to the open-source code policy. Anyone can participate in support, development, and improvement of **Theia**. This will quickly expand the functionality and stability of the language and also increase interest in its use among the *Web 3.0* community.
