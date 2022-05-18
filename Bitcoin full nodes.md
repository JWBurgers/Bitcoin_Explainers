# Bitcoin full nodes

If you have been around Bitcoin for a while, you will inevitably have heard people talk about full nodes and their crucial role within the network. But what are full nodes exactly? How can you operate one? What different types exist? And why are they important?

In this blurb, I will set out answers to these questions. I will begin the discussion with some general background on peer to peer networks.  


## Peer to peer networks

Think of your home computer network or that of a small office. We often describe such **local area networks**—that is, networks which are located in a very limited geographical area and serve some fairly well-defined entity such as a household, office building, or school—as either client-server or peer to peer. These labels describe how the network operates on a functional level. 

A **client-server network** generally has two main characteristics in its operation. First, in the sharing of network resources, some computers clearly function as servers and some clearly as clients. Second, these servers are managed by a central administrator (either a person or a team). 

A typical **peer to peer network** does not operate according to such a strong hierarchy. Computers in the network tend to function both as clients and servers in the provision of network resources. 

So, for instance, a small office network might have a dedicated server which stores the files of all employees, a server which handles commonly used applications, a server for e-mailing, and so on. Employees connect to these servers from their own computers. Furthermore, the servers are managed by a local network administrator. This type of small office network would normally be called a client-server network.

Your home network, by contrast, probably does not have any servers and, by extension, anyone managing them. Instead, you likely just have a number of computers and other devices that operate relatively independently from each other. Any resource sharing between these computers—say, for instance, accessing an application located on one computer from one of the other computers—is probably incidental, and in any case, not structurally one-way. This type of home network can be called a peer to peer network. 

The distinction between client-server and peer to peer networks is not binary. Even very hierarchical networks can sometimes have computers interacting in a peer to peer fashion and vice-versa. In addition, the label most approriate to a local area network can change. A home network which is originally really a peer to peer network, for instance, might better be dubbed a client-server network once you start working with centrally managed e-mail, file, and application servers. 

When looking at the data storage and application layer of the the Internet—really just a vast collection of connected networks—it is neither purely a client-server network nor purely a peer to peer network. Though many of our interactions occur within a client-server architecture, some do take place within a more peer to peer fashion. 

Using the World Wide Web is a good example of an Internet activity normally governed by a client-server architecture. A network of servers provide web resources to end-users. All these servers are owned and managed by one or more parties (e.g., organizations, private persons). E-mail is also an example of an activity that is commonly executed within a client-server architecture (unless the sender and recipient both run their own e-mail servers).  

Our peer to peer interactions on the Internet mostly occur within **distributed peer to peer networks**. In fact, when we speak of “peer to peer networks”, we are typically referring to these geographically dispersed networks, not local area networks.

Just as within peer to peer local area networks, the computers in these distributed peer to peer networks interact within relatively flattened hierarchies. But they differ in two distinct ways:

- Distributed peer to peer networks are connected via the Internet’s vast collection of routers, switches, cables, and service providers. And a connection can be to anyone, anywhere in the world. The existence of connections between peers is, therefore, more fluid than in a typical local area network. 
- Distributed peer to peer networks are specifically designed to serve resources to network participants. Local area peer to peer networks are usually appealing for their cost-efficiency and convenience, and not necessarily used much for sharing resources.   

You can participate in a distributed peer to peer network by downloading and running an application that implements certain communication protocols and standards. The success of such an application relies on many computers running it, so we often refer to it as a **distributed application**.

For a long time, the most popular distributed peer to peer networks were **file sharing networks** such as μTorrent, Vuze, and eMule. File sharing networks normally operate according to the **BitTorrent protocol**, which was first released by Bram Cohen in 2001. 

The protocol turns all files into **torrents**, which include the data of the original file split into many separate fragments as well as a metadata file about those fragments. Any peer that wants the original file will first download the metadata file and, then, start grabbing the fragments. While downloading, the peer can allow others to upload fragments of the file it already possesses. After a peer has the entire file, she can continue to allow others to upload the file from its system. 

Peers do not operate on an exactly equal footing in these file sharing networks. Some peers may act as **seeders**—peers that offer an entire torrent for uploading to others—for a vast amount of content. Some peers will, instead, just mainly download content; these types of peers are commonly known as **leechers**.<sup>[1](#footnote1)</sup> So some peers are clearly more important than others. Yet, by design, these peer to peer file sharing networks operate with markedly little hierarchy.

Peer to peer networks can have various advantageous properties, including robustness against failures and resistance to censorship. In fact, the bittorrent protocol was created precisely because an early incarnation of a distributed file sharing network, **Napster**, was shut down. The company had lost a wave of lawsuits and took the centralized servers on which the network relied offline. 

While its absence of a strict client-server architecture had allowed Napster to vastly expand the content available online, it was not very resistant to censorship. (After being shut down, it later re-incarnated as an online music store.) The idea behind the bittorrent protocol was to enable distributed file-sharing networks, which could not be shut down so easily. 

Satoshi was inspired by this history of file sharing, and saw it as as a potential way forward for digital cash systems from earlier failures such as E-gold and Ecash.<sup>[2](#footnote2)</sup> In a discussion forum of the P2P Foundation, Satoshi offered the following reflection:

> A lot of people automatically dismiss e-currency as a lost cause of all the companies that failed since the 1990’s. I hope it’s obvious it was only the centrally controlled nature of those systems that doomed them. I think this is the first time we’re trying a decentralized, non-trust-based system.*<sup>[3](#footnote3)</sup>

Before moving on, we need to wrap up one point of frequent confusion. Just because an online network is peer to peer does not mean that it has no reliance on third parties at all. Even very decentralized peer to peer networks such as Bitcoin rely on third parties. 

Specifically, any distributed peer to peer network relies on the Internet’s infrastructure, a vast array of communication equipment owned and operated by others. So, when we talk about such networks, the label normally describes the application and data storage layers, not the infrastructure and transport layer. To explain it within the context of Bitcoin: one can use bitcoin without financial intermediaries, not without infrastructural intermediaries. 

The infrastructure layer of the Internet is somewhat decentralized both in terms of its physical architecture and commercial agreements. However, it is certainly not as decentralized as Bitcoin’s peer to peer network on the application and data storage layer. And this does pose certain challenges. For instance, Bitcoin communications are by default not encrypted. So internet service providers could probably block a substantial amount of the current traffic on the network. 

Some people would respond to the observation above that Bitcoin can also be used outside of the Internet, via radio signals, satellites, and other types of communication technologies. As Bitcoin is just a protocol, this is certainly true, strictly speaking. One could implement the Bitcoin protocol via smoke signals if sufficiently many other parties were willing to cooperate. But for all practical purposes, Bitcoin strongly relies on the Internet’s infrastructure. Without the Internet, not much of a network would remain.  


## Bitcoin nodes

The Bitcoin network is a distributed peer to peer network. A **Bitcoin node** is the generic label given to any peer in this network.

You can operate a node in the Bitcoin network by installing and running certain software applications. The exact capabilities of your node will depend on the software you use and how it is configured. 

It is important to emphasize that a bitcoin node is really just an operational point within the Bitcoin network. That means you can house multiple nodes on one machine. All that requires is having multiple instances of node applications running on it at the same time. A Bitcoin node is, therefore not just “[a]ny computer that connects to the Bitcoin [peer to peer] network,” as stated in the Bitcoin wiki.<sup>[4](#footnote4)</sup>

It makes sense why people tend to think about Bitcoin nodes as computers on the network. Typically any piece of equipment on a computer network with communication abilities is called a **node**. Practically everything other than the cables, repeaters, and amplifiers in a computer network is a node in this sense. 

But this standard understanding of a network node does not apply so well to Bitcoin. In that context, we are really talking about operational points within the peer to peer network. And one computer can house one or more of these operational points. 

A full node is not the only type of node on the Bitcoin Network, but it is the most important type. From here on out, I will use the terms full node and node interchangeably, unless otherwise specified. 


## Bitcoin full node software

A **full node** is a peer on the Bitcoin network, which (a) can validate blocks that are relayed within the Bitcoin Network against its own version of the Block Chain and (b) remain on the chain backed by the most proof of work. We will expand on that definition more in the next section. But first, it is helpful to discuss how you can actually run a full node. 

The main software package available for implementing a full node is **Bitcoin Core**. This software package is also referred to as the **Satoshi client**, because it’s original incarnation was created by Satoshi. The software package includes several main applications.  

Any standard installation of the Bitcoin Core package contains a program called **bitcoind**, or the “Bitcoin daemon.” A **daemon** is any type of program that runs in the background of a computer (sometimes also called a **service**). A user does not interact with such programs directly, but can normally make requests to it. To indicate that a program is a daemon rather than an interactive application, their names often end with the letter “d”, as in bitcoind. 

Running bitcoind is what ensures a full node is operating on a computer. It is the application that validates new blocks and adds them to the node’s version of the Block Chain.    
Bitcoin Core also comes with a program known as **bitcoin-cli**, or the “Bitcoin command line interface.” This is the program that allows you to make requests to bitcoind for information about your node. It specifically makes calls to a JSON-RPC API, standarly on port 8332. The bitcoin-cli allows you to ask of bitcoind, for example, with how many peers you are currently connected, or about the latest block in your version of the Block Chain. 

Standard installation of Bitcoin Core also includes the **bitcoin-wallet** application. The bitcoin-cli tool is also used to make requests of the bitcoin-wallet in the command line interface. The bitcoin-wallet application is what allows you to create new addresses, new transactions, monitor your balance, and so on. 

Finally, when you install Bitcoin Core, you can also choose to install **bitcoin-qt**. This is a graphical interface that connects to your wallet and bitcoind in the background. You can, then, send requests to these applications via the graphical interface instead of bitcoin-cli. The graphical interface also includes the **Bitcoin console**, which allows you to make command line requests for any functionalities not available in the graphical interface.

One key point to note here is that Bitcoin wallets should really be seen as logically distinct from Bitcoin full nodes (or nodes more generally). While you could build a single application which includes all the functions of a full node and a wallet, this is not common practice. Instead, all the network information for the wallet typically comes from connecting it to a separate full node service.

In practice, we might distinguish between three types connections between wallets and full nodes from the perspective of the user. 

1. In some cases, a user’s wallet is just an interface to the proprietary back-end servers of a third party such as Coinbase, Kraken, or Binance. You do not control any of the private keys in the wallet, so “your” bitcoin in it is really just account data. For sending or receiving transactions, the back-end servers connect to a node controlled by the third party, or sometimes even a different third party. 
2. In some cases, the user’s wallet controls the keys directly, but still relies on the full node of a third party. With some wallet software, you are required to rely on the provider’s node infrastructure. With many wallet applications, you have the option of connecting to your own node infrastructure or that of a third party. 
3. In a final type of case, the user’s wallet controls the keys directly and connects to a full node operated by the user. In some cases, the user’s full node may run on the same device. Often, however, it will run on a different system. In fact, running your node on a decidated server has various benefits, particularly with regards to reliability and convenience. Many good software and hardware products have made running your own full node easier over the years, so it’s becoming really standard practice to work with dedicated servers. 

There are various other software packages than Bitcoin Core, which allow you to run a full node, including **BCoin**, **BTCD**, **Gocoin**, **Libbitcoin**, **Parity Bitcoin**, and **Stratis**. Some of the node server applications in these packages, like for Libbitcoin, are written in C++, but structured differently than Bitcoin Core. Others are implementations that use different programming languages, such as BTCD (written in Go). 

Bitcoin Core generally has better performance than these alternatives.<sup>[5](#footnote5)</sup> It is also by far also the most popular application for running a full node, and more than 95% of the network typically seems to run some version of Bitcoin Core.<sup>[6](#footnote6)</sup>

Bitcoin Core is frequently called the **reference implementation** for the Bitcoin protocol. It, in other words, is said to define the Bitcoin protocol: it is not just an implementation of the protocol, it is the implementation of the Bitcoin procotol.

There has been a long-standing discussion about the legitimacy of calling Bitcoin Core the reference implementation, and whether alternative implementations of the Bitcoin protocol strengthen or weaken the network.<sup>[7](#footnote7)</sup> 

Many Bitcoin users nowadays run Bitcoin Core with the support of **node solutions**, a software or combined hardware/software product intended to foster a more plug and play experience when setting up a Bitcoin server. Sometimes these solutions are distributed (and sold) by companies such as Start9 or Nodl. Other solutions are just open-source projects such as Raspiblitz and Umbrel. 

Node solutions typically include a host of Bitcoin-related server applications, not just Bitcoin Core. Typically, they will run server applications needed for certain wallets, an electrum server, a lightning node implementation, and so on. In certain cases, node solutions are more broadly focused on **self-hosting**.

The availability of node solutions has made it much easier for Bitcoin users to run their own Bitcoin servers. While there are no exact figures, I would expect at least a substantial portion of Bitcoin nodes to be run in this way. Given the benefits of running a node for security and privacy, this trend should be welcomed. 

That all said, while node solutions surely offer a nice and convenient solution for many people, building your own Bitcoin server from scratch does have various advantages.

1. A homespun Bitcoin server offers you more control in software and hardware choices.
2. Software problems are much easier to troubleshoot with a homespun Bitcoin server.
3. Node solutions inject another layer of trust into your Bitcoin server. You have to trust the creators, for instance, with implementing legitimate versions of a full node application and keeping it updated. Most node solution projects have a very limited number of dedicated contributors, so using them clearly introduces additional risks.
4. Your full node application is your voting right within the Bitcoin ecosystem. If Bitcoin is facing a constitutional moment—such as it did with the Segwit Civil War in 2017—you want to determine the version of the software you run for your transactions. There is no guarantee a node solution will offer support for your convictions.
5. A homespun Bitcoin server is more cost efficient.

In the next section, I will further explain full nodes, including the different types. Given my own (limited) knowledge and the implementation’s popularity, I will conduct the discussion around Bitcoin Core. It should be understood, however, that there are other implementations for running a full node, which can differ in some of their details. 


## Full nodes

A full node can validate blocks that are relayed within the Bitcoin Network against its own version of the Block Chain and remain on the chain backed by the most proof of work. Block validation includes checks such as the following:

- Are all the unlocking scripts for the transactions in the block valid?
- Do all the transactions only spend unspent transactions outputs?
- Are all transactions formatted properly?
- Does the coinbase output value equal the block subsidy plus the transaction fees?
- Is the block hash valid?
- Is the time stamp valid?

Great care is taken by developers and engineers to make Bitcoin **backwards-compatible**: older full nodes should still be able to function on the Bitcoin Network, even if they do not support all the latest validation rules. The way this is achieved is by only releasing software updates that make the validation rules stricter. 

When a new block reaches an older node, it, thus, will not be able to assess the block against all the current validation rules. So, an older node is not “fully validating” in that it knows all the latest rules. Instead, a node is “fully validating” in the sense that it validates the entire block, including the transactions, against its own rules and stored history. 

While newer nodes might sometimes not accept a block that the old node judges valid—as newer nodes apply stricter rules—this does not matter so much, as long as the majority of the network nodes and miners enforce the stricter rules. The block that the old node judged valid will, in that case, just not be included in the chain with the most proof of work, the **main chain**. In this way, an old node remains on the main chain.

So, a full node is not necessarily one that is upgraded with the latest version of the Bitcoin protocol. Many older versions of Bitcoin Core, for example, can still validate the blocks sent to it on the network and follow the chain with the most proof of work. That said, very old versions of Bitcoin Core will have substantial challenges synchronizing with the network. So there are practical limitations to the backwards compatibility of node applications like Bitcoin Core. 

When first synchronizing Bitcoin Core with the Bitcoin Network, transactions are only validated from block 295,001 onwards (April 2014). This is because there are various hardcoded checkpoints. Any **checkpoint** is just a defined a block height and a hash which points to that block, where, for the purposes of synchronization, the client assumes that all transactions which came before it are valid. 

In principle, of course, you could customize Bitcoin Core to verify all the blocks and transactions which came before the latest checkpoint at block 295,000.<sup>[6](#footnote6)</sup> Other full node implementations may have no checkpoints, or checkpoints at alternative locations. Bitcoin Core no longer adds new checkpoints. These checkpoints were originally introduced to optimize synchronization and prevent certain denial of service attacks. But due to certain changes in the Bitcoin protocol, they are not longer necessary for these purposes. 

In order to support the validation of blocks, full nodes maintain a set of **unspent transaction outputs** (also known as the **UTXO set**). These are, as the name implies, outputs from bitcoin transactions which have not yet been spent (other than OP_RETURN outputs which can never be spent and are only used to store some data on the Block Chain). Without this UTXO set, a node would have to search the Block Chain each time to verify whether an output from a transaction is indeed a UTXO. That would be incredibly inefficient. 

For security reasons, a full node should be connected to multiple peers. In Bitcoin Core, the default intended number of connections at any point in time is 10. All of these are **outbound connections**: that is, connections which the node initiates from her side. By default, other nodes cannot initiate such connection requests in return. 

In addition to the default number of outbound connections, Bitcoin Core can also be configured to accept **inbound connections**. Specifically, you can allow up to 115 inbound connections. The maximum of 125 connections that is set by default on Bitcoin Core is motivated by the typical system limitations for personal computers. 

If you are willing and able to customize the code for Bitcoin Core, you could set the default number of nodes to less than 10. But this is not recommended. Security requires that you are connected to at least honest node in the network. The default number of 10 outbound connections ensures sufficient redundancy to protect yourself from dishonest nodes, specifically to protect yourself against eclipse attacks. 

A full node that also allows inbound connections is known as a **listening node** (as it “listens” for incoming connection requests). A node that only accepts outbound connections, the default setting, is called a **non-listening node**.

Full nodes which carry the entire Block Chain history are known as **archival nodes**, while those with a partial history are called **pruned nodes**. The latter, by default, store a minimum of 550 megabytes of block data, in order to validate re-organizations of the Block Chain. Listening, archival nodes allow new nodes to synchronize with the network. Listening, pruned nodes can only support other nodes once these are already largely synchronized.  

Another key distinction is between **block-relay-only nodes** and **full relay nodes**. The former do not receive messages about new transactions on the network. They do not maintain what is known as a **memory pool**, which stores unconfirmed transactions in cache memory (and for this reason is also known as the **unconfirmed transaction pool**). Instead, block-relay nodes only validate new blocks and relay these to their peers. 

Full relay nodes, as the name implies, validate and relay both unconfirmed transactions and new blocks. Unconfirmed transactions are also stored in the memory pool, until they are included in a valid block. By default, the memory pool has a maximum size of 300 megabytes in Bitcoin Core, but this can be changed in the configuration settings. The 10 default outbound connections in Bitcoin Core include 8 full relay nodes and 2 block-relay-only nodes.  

Sometimes one also comes across the concept of a **supernode**. Usually such a node is defined as a listening, archival node, or a listening, archival, full relay node. As the idea of a supernode is that it functions as a highly connected data distribution point, lets characterize it here as a listening, archival, full relay node. The more listening connections it supports and the better its reliability and bandwidth, the more the supernode is deserving of its name.  

A final key concept is that of mining nodes. Originally, Bitcoin Core supported mining with a CPU. This feature was, however, removed as Bitcoin mining equipment became much more specialized and CPU mining too inefficient. So most full nodes today have no real role in Bitcoin mining. 

Some full nodes, however, while they do not mine directly, send new valid blocks found by miners onto the network. These nodes are typically run by mining pools, so we may refer to them as **mining nodes**. 

Full nodes form the core to the architecture of Bitcoin’s peer to peer network. That said, there is one other type of common peer known as a **light node** or **SPV node**. SPV here stands for “special payment verification,” a technique to verify transactions, already mentioned in the Bitcoin white paper.<sup>[9](#footnote9)</sup> 

Light nodes only store and validate block headers. They do not store any transactions, nor do they validate any of the transaction history. The main contrast with a full node is that the latter also validates the body of blocks. 

As a result of its design, a light node does not know whether the block headers it receives, even though these may be valid on their own, are actually coupled to a valid block body. There is no way for a light node, for example, if any of the transaction inputs of the block body had already been spent, or whether any of the unlocking scripts are actually invalid.  

In order to ascertain a balance for a **light wallet** (i.e., any wallet that connects to a light node), a light node can request full nodes to send them any transactions related to the private keys they hold. Any such transaction can be accompanied by a merkle path, if the transaction is indeed included in a block already and not unconfirmed. Using the block header and the merkle path, a light node can ascertain that the transaction is indeed included within a certain block.  

Light nodes normally use **bloom filters** in requesting transactions of full nodes: this produces a set of transactions bigger than that which is actually of interest to the light node. Bloom filters are intended as a way to enhance privacy for light nodes. 

There are not many wallets available that operate with light nodes. A **light wallet** that has long been popular is Electrum. 


## Why run a full node?

If you want to hold modern fiat currencies such as dollars and euros in digital form, you really have only one option: you have to hold them with a bank. There is no way to privately hold onto fiat money in a digital form. Your only option to hold your fiat money privately is via physical cash. 

This technical dominion of banks and other financial institutions over our digital money creates massive power disparities between them and their customers, even in places with strong legal protections and significant competition for financial services. Physical cash has long been a tool to dampen this dominion, but governments and banks are working hard at the moment to eliminate that option for you through various “innovation” and “anti-crime” masquerades. 

A key aspect to bitcoin is that it can be owned much more flexibly within the digital realm. As with fiat currencies, you can store them with third parties who, then, can make transactions on your behalf. There are many such Bitcoin custodians. You can also, however, manage your bitcoin completely privately, or manage them through some type of shared custody arrangement.  

Bitcoin custodians come with substantial personal risks. Some of these risks are why Bitcoin was created in the first place, including the risk of confiscation by the government or a third party, or the censorship of your transactions. But I would say that the risks of Bitcoin third-party custodians are even greater than for most modern banks (some exceptions, of course, notwithstanding). Some of these risks may be ameliorated with shared custody solutions, but these are not as widely available. 

Third-party custodians also create systematic risks for Bitcoin. It puts governments, for instance, in a much better position to split Bitcoin into two, de facto separate networks: one where bitcoins only travel between regulated third parties and one only used for peer to peer exchanges. The more bitcoin are stored at third-party providers, the larger those types of systematic risks. 

For less technical users, bitcoin custodians are necessary, at least for storing some of their funds. When less technical users store their own bitcoin, they are often unclear about how to do so wisely, and leave themselves exposed to loss, theft, and privacy risks. So particularly for those users, Bitcoin custodians are can be the key to managing their personal risks. 

That said, the main benefits of Bitcoin come when you store your own bitcoin, and to some extent when you use good shared custody solutions. As long as you have some technical competence, the self-custody of some bitcoin is likely a sensible strategy for managing your money. It’s really one sense of what bitcoiners mean when they say things like “Be your own bank” and “claim your financial sovereignty.” 

There is, however, a second main sense in which these slogans are appropriate. Bitcoin enables not just new, more sovereign models of control over your digital money. It also allows you to control a part of the actual financial infrastructure. This is precisely what happens when you run a full node within your self-custody solution (and in some cases within a shared custody solution). 

Why should you care about operating a part of Bitcoin’s financial infrastructure? This does not really have an easy answer because you can run a full node and use it in many different ways. That is, the benefit a full-node offers depends on how your run it, and what you do with it. 

Generally speaking, however, we can distinguish between four main possible benefits from running a full node. These are as follows: 

- A full node can give you a voice in determining the rules of the Bitcoin protocol.
- A full node can be important to supporting the network.
- A full node can offer privacy benefits.
- A full node can verify the bitcoins you own.

Lets discuss these four potential benefits in turn. We will use some of the distinctions between full nodes already made to guide the discussion.


### Vote with your node

Running a full node can give you a voice in determining whether to adopt new rules for the Bitcoin Network. The rules of concern here are those that concern the validity of transactions and blocks.  

Suppose that only mining pool operators and solo miners ran full nodes, and that all other nodes on the network were light nodes. In that case, the mining pool operators and miners really determine whether to accept new rules for transaction and block validation. They could join together, for instance, and unilaterally decide whether upgrades such as segregated witness (2017) and taproot (2021) should be implemented. 

In a network where many full nodes are run by users—including both other Bitcoin companies and regular users—they instead determine policy changes. That is, however, as long as many of these full nodes are specifically be economic nodes. 

An **economic node** is one that actually processes bitcoin transactions and blocks with those transactions which concern the operator of that node. By processing your own transactions with a node, you are committing to the rules by which you judge those transactions. You have “skin in the game”, so it is strong signal of your preferences. 

To illustrate this further, suppose that you run a book store and accept Bitcoin payments. You run your own full node. At time T<sub>0</sub> you have to decide whether to support some new rule X for transaction validation. This rule would narrow down the set of acceptable transactions. You think it is a good rule so decide to accept transactions which leverage X. Now consider the following scenario:

- At T<sub>1</sub> you receive bitcoin from the purchase of a book which requires validity of the new rule X. You store this new UTXO in your UTXO set, and eventually is added to your copy of the Block Chain. After six confirmations, you send the book to your customer. Lets call the transaction that made this happen, transaction B. 
- Contrary to your wishes, there has been little support for X within the network. And by T<sub>2</sub> practically no other nodes support the rule any more in transaction validation. Transactions are starting to be accepted, which do not meet the strict requirements of X. 
- Much to your disappointment, you discover that you cannot spend the UTXO you received for the book with transaction B. Apparently, the inputs to this transaction had already been spent in a transaction A, which made it into the Block Chain first. This transaction A did not meet the standard of rule X, but did meet all other validation standards for Bitcoin nodes. Hence, while you rejected it and, therefore, did not see anything wrong with transaction B, the majority of the rest of the network accepted transaction A first.  

This kind of scenario is harmful for an economic node, but it does not really matter to a non-economic node. If all your node does is just relay network blocks and perhaps transactions, then none of your own UTXOs will be affected by the rules you apply with it. In that case, what matters for your UTXOs is the rules applied to the economic node which processes them.

To put it differently, if you are running a full node but have all your bitcoin stored with an exchange, all the rules applied to your bitcoin are those of the exchange. What you do with your own node doesn’t really matter so much. 

Note that policy changes are not just about network upgrades, which are the subject of extensive public deliberation. Without economic nodes from Bitcoin businesses and users, pool operators and miners could, in principle, make many policy changes. For example, they could decide to increase the block subsidy, and users would not be able to stop them as long as they don’t run an economic node.  

While mining pool operators and miners would probably be careful implementing very nefarious changes in order to protect the value of the network—on which they rely for income—a network that is practically controlled by them substantially increases the risks to businesses, organizations, and users. Bitcoin should rely on decentralization for its sound operation, not the uncertainty of pure self-interest among mining pool operators and miners.  


### Support the network

Running a full node can support the health of the network in four main ways.

First, again suppose that only mining pools and miners were to run full nodes. Without full nodes from businesses and users, mining pools and miners have an incentive not to validate the transactions in blocks at all. It is a calculation-intensive activity, so ignoring the validation of transactions within blocks would leave additional resources for finding a valid proof of work. If a significant number of miners adopt this policy for the same reasons, invalid transactions may be included within blocks. 

If users and businesses run full nodes, however, it requires mining pools and miners to validate transactions within blocks. Without validation, mining pools and miners run the risk of expending costs on finding valid proofs of work for blocks that will not be accepted by the network. Again, it is particularly economic nodes which are important in this regard. 

Second, listening nodes are necessary for the network to function. If all full nodes only accepted outbound connections, then no one could connect to the Bitcoin network at all. Non- listening nodes and SPV nodes depend on listening nodes for information about the network and making transactions. 

Luke Dash Jr. maintains statistics on the total number of Bitcoin listening and non-listening nodes in the Bitcoin network.<sup>[10](#footnote10)</sup> Though particularly non-listening nodes can be difficult to estimate and both numbers, of course, fluctuate, we can make the following generalization:

- The number of non-listening nodes is typically an order of magnitude larger than the number of listening nodes. While the number of listening nodes is usually between 5,000 to 10,000 in recent years, the number of non-listening nodes is usually between 50,000 and 100,000 in recent years. 

Third, listening, archival nodes are also crucial to allowing new full nodes to synchronize with the network. While most listening nodes are probably also archival nodes, this is not necessarily the case. 

Fourth, running a listening node means you can serve as a trusted connection for family and friends. Some people are unlikely to ever run their own node, regardless of the benefits and how easy it becomes. In that case, connecting to the node of a family member or friend can be a better option than connecting to the node of a third party or just using a custodian. 


### Privacy

Running a full node can contribute to your financial privacy while using Bitcoin. However, just running a full node is not necessarily all that effective with regards to financial privacy. For instance, running a full relay node over clearnet would allow your ISP or anyone else sniffing your traffic to easily distinguish your own transactions from those only being relayed. So you need to combine your full node with a variety of other measures to achieve a good level of privacy. 

What measures to take exactly is a fairly complex matter. Privacy is easily lost and difficult to gain. That said, here are some of the main techniques that can accompany your full node usage: 

- Run all your outbound and inbound connections via an onion router such as Tor. The main benefit here is that this makes it much more difficult for attackers to distinguish between transactions which originate with a full relay node and which are just being passed forwards. In case you run a blocks-only-relay node, none of your connected peers will know your IP address, only your onion address.
- Run multiple nodes that are not associated with each other, and distribute the transactions you initiate over these various nodes.  
- Run a local block explorer to prevent a centralized server from clustering interest in particular addresses and transactions to your IP address.
- Use a coinjoin implementation such as Joinmarket to make following your transaction chains on the Block Chain more difficult to trace.
- Avoid the re-use of your addresses, including change addresses. Typically wallets are configured not to re-use addresses, but not always.
- Source your bitcoins from more privacy-friendly sources such as peer to peer exchanges without KYC. 
- Utilize the Lightning Network for your payments with your own lightning node—which requires running a full node—as this greatly improves your privacy with regards to payments. 
- Be careful with any information about Bitcoin addresses that you own.

How does running a full node with these types of measures compare to using a light node, connecting with a third-party’s full node, or just relying on a custodian? Making a full comparison is not all that straightforward, but let me offer illustrative remarks. 

To start, custodians typically will require a lot of personal data for registration, as well as monitor all your deposits and withdrawals in light of KYC/AML regulations. And unlike with a traditional financial custodian, a bitcoin custodian has a better ability to map out your overall financial activities, even those not directly associated with the platform, due to the public, collective history available on the Block Chain. All this means that a lot of your financial data will become aggregated in one place. 

While all this is a concern in its own right, the biggest risk with a custodian is that your data is leaked or just sold outright without your consent to chain analysis companies and other third parties. This would hardly be a novelty within the Bitcoin ecosystem. Of course, by government mandate, this data is often also shared with regulatory and law-enforcement agencies. Their systems are, in turn, also prime targets for hackers. 

Running a full node with the measures above would make it fairly difficult to cluster your financial activities in any meaningful way and to associate them with your identity. While the parties that seek to break your privacy on the network are surely sophisticated, a full node with sufficient privacy measures should generally serve you better than a custodian.

Running a full node can also bring other advantages for privacy. It allows you, for instance, to run your own lightning node for payments, which is much more privacy-friendly than using the base layer of the Bitcoin network to transact. For another example, a full node also allows you to host a block explorer locally, so that you do not have to search for your transactions on a website. Again, chain analysis companies and other third parties are likely to use at least some of those websites to snoop on you. 

Connecting to a third party node or using a light node has several advantages over a custodian with regards to privacy. Runing a light node requires no registration information, while connecting to the node of a third party typically only requires little personal information. In addition, while custodians know your exact balance and the transactions you run through their infrastructure without any effort, this is not necessarily the case with a light node or with using a third-party node. At least light nodes standardly operate using bloom filters, which request full nodes for sharing many more transactions than is necessary in managing the wallet. 

That all said, full nodes are still strictly better than either using a light node or connecting to a third-party node. Bloom filters have some issues, so you still run the risk of a connected full node knowing what transactions really concern your wallet. In addition, all your outgoing transactions are seen by all the connected full nodes. Finally, you also miss out on the privacy benefits of running your own block explorer, using the lightning network, and so on. 


### Verification of your bitcoin

When you use a custodian, you do not technically own any bitcoin. You own an account, which displays a number of bitcoin that the custodian promises to send you if you ask for it. Normally, there is a also a legal framework in place which supports your claim to a certain amount of bitcoin. But there is no way of directly verifying that you indeed indirectly own that amount of bitcoin somewhere on the Block Chain. 

The situation is roughly the same if you connect to a third-party’s full node. You cannot directly verify that the bitcoin outputs associated with your wallet are valid and spendable. You do not have a view on the Block Chain and the UTXO set, so have to trust your connection that they have given you valid information. 

Of course, you can take some measures to verify that the data provided to you by a third-party node is indeed accurate by cross-checking it with, for example, one or more block explorers. But in that case, you have to venture outside of the wallet application and this, in turn, creates privacy risks.

A light node is a little better. At least you can receive a proof that a transaction which concerns you is in the Block Chain somewhere. But you still face challenges in verifying the Bitcoin that you own. 

- While you can learn that a particular transaction is in the Block Chain somewhere, you have no idea whether any of the other transactions in the block are also valid. So you cannot judge whether the block might at some point be rejected by the network. 
- You also cannot verify that the transaction was not double-spent. A proof that a transaction is included in a block is not a proof that one or more of the outputs are still spendable. 
- Full nodes can also withold transactions from you. While full nodes can also withhold transactions from other full nodes with regards to their relay phase, they can never withold transactions which are included in a block.

All this means that light nodes indeed cannot verify the bitcoin owned by a user in the way that a full node can. If your light node is connected to a number of full nodes, you can probably be fairly certain of the bitcoin you own. For all practical purposes, the light node standards of verification on your bitcoin are probably enough. That said, a full node is strictly better. And particularly for a very large amount of bitcoin, this may be important. 


## Notes

<a name="footnote1">1</a>. A “leecher” can also refer to someone who is just downloading a particular torrent. In this case, it is a fairly neutral term just synonymous with downloader.

<a name="footnote2">2</a>. See, for example, the Cryptography Mailing List, “Bitcoin P2P e-cash paper,” November 7 (2008), available at: https://www.metzdowd.com/pipermail/cryptography/2008-November/014823.html.

<a name="footnote3">3</a>. Satoshi Nakamoto, “Bitcoin open source implementation of a P2P currency,” February 15 (2009), available at: http://p2pfoundation.ning.com/forum/topics/bitcoin-open-source.

<a name="footnote4">4</a>. See “Full node,” available at https://en.bitcoin.it/wiki/Full_node. 

<a name="footnote5">5</a>. See Jameson Lopp, “2021 Bitcoin node performance tests,” December 4 (2021), available at: https://blog.lopp.net/2021-bitcoin-node-performance-tests-2/. He has conducted these performance tests yearly, since 2019. 

<a name="footnote6">6</a>. See Luke Dash Jr., “Bitcoin node software” (https://luke.dashjr.org/programs/bitcoin/files/charts/software.html).

<a name="footnote7">7</a>. For a good summary of the discussion, see Aaron van Wirdum, “The long history and disputed desirability of alternative bitcoin implementations,” Bitcoin Magazine, September 23 (2016), available at: https://bitcoinmagazine.com/articles/the-long-history-and-disputed-desirability-of-alternative-bitcoin-implementations-1474637904.

<a name="footnote8">8</a>. See https://github.com/bitcoin/bitcoin/blob/master/src/chainparams.cpp#L138. 

<a name="footnote9">9</a>. See Satoshi Nakamoto, “Bitcoin: A peer to peer electronic cash system,” Section 8.

<a name="footnote10">10</a>. See “Bitcoin node count history” (http://luke.dashjr.org/programs/bitcoin/files/charts/historical.html).