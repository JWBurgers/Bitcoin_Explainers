# Bitcoin Script and Transactions

Normal Bitcoin transactions unlock one or more **unspent transaction outputs** (**UTXOs**). After deducting a transaction fee (which must be greater than or equal to zero), the transaction distributes the remaining value among one or more outputs. Each of these outputs, in turn, have their own locks that must be unlocked for any subsequent transactions. As long as an output from a transaction is not spent, it remains a UTXO. 

There is one exception to the flow described above: the **coinbase transaction**. This is first transaction of every mined block. While a coinbase transaction indeed locks value to an output, it does not have any UTXOs as its input(s). Instead the input field is used for various other purposes, which we will not delve into here.  

One key function of the locks on Bitcoin outputs is to assure that each output can only be unlocked by the intended recipient(s) with a subsequent transaction. Often these locks involve making a signature for a public key and showing that you own the pre-image of a hash digest. The locks in Bitcoin transactions are also referred to as **encumbrances**. 

The locking and unlocking code for Bitcoin UTXOs is managed by a special programming language called **Script** or **Bitcoin Script**. Within Bitcoin Core, Script is implemented using C++. For other Bitcoin software applications that support Script, it will be implemented using those programming languages. Note that only the locking and unlocking code for Bitcoin UTXOs is handled by Script: all the other work on the validation of transactions is just done in a general purpose programming language (C++ in Bitcoin Core). 

The check on the validity of a Bitcoin transaction occurs in two stages: (1) It occurs when the transaction is originally relayed by nodes for inclusion in the memory pool; and (2) it occurs when newly mined blocks that include the transaction are validated by nodes. Any transaction with a valid unlocking and locking mechanism is accepted into a block. During the original relay phase, however, nodes check both for validity and **standardness**: only certain types of unlocking and locking scripts are allowed.

This blurb provides an overview of Script, the serialization and parsing of Bitcoin transactions, and the standard types of locking scripts relayed by nodes.


## Script

Let’s first understand Script a little better by delving into the name. 

The term **script**, with a lowercase "s", typically refers to a small program that is interpreted at run-time. An **interpreter** converts the source code into machine code for execution by the central processing unit, statement by statement. People are referring to this type of program when they speak of, for example, Python scripts, Bash scripts, or shell scripts.<sup>[1](#footnote1)</sup> 

Languages that generally work with interpreters such as Perl, Python, and JavaScript, are frequently also called **scripting languages** or **interpreted languages**. These are contrasted with **compiled languages** such as C, C++ or Rust, which typically work with compilers. A **compiler** first converts all of the source code into machine code before execution. In principle, any higher-level programming language can work with a compiler, interpreter, or both in its conversion to machine code. Nevertheless, it is typical to convert each high-level language in certain ways.

So, to be more exact, any Bitcoin UTXO is locked via a script. To use a UTXO as an input to a transaction, you must provide an unlocking script. These unlocking and locking scripts are run together. The transaction is legitimate if the combined script validates and the transaction meets certain other conditions. If the combined script does not validate, the entire transaction is invalid.   

Somewhat confusingly, the scripting language used for creating locking and unlocking scripts in Bitcoin is called Script. It is similar to an old programming language called **Forth**. 

Most importantly, Script is a **stack-based programming language**. A **stack** is a data structure, which one can manipulate from the top downwards. Adding an element to the top is typically called a **push** operation, while removing an element from the top is typically called a **pop** operation. A stack-based programming language operates explicitly on stacks. In a general purpose language such as C++, a stack can be implemented using an array or list structure.  

The core component of the Script language is its set of **operators** or **opcodes**. Serialized transaction data—that is, the transaction data in a format suitable for sending across the Bitcoin Network—is standardly represented in hexadecimal format. Transaction data in this format is also referred to as the **raw transaction data**. All the Script operators are required to have 1 byte hexademical encodings in these byte streams, so Script allows for a maximum of 256 operations. 

Out of the 256 possible operators, some have never been defined. Others were defined at some point but have now been removed from the accepted Bitcoin protocol. Current versions of full node clients, most importantly Bitcoin Core, will not support these operations either during transaction validation or block validation.  

Removed opcodes are often referred to as “disabled opcodes”. But this is a bit of a misnomer. Such opcodes simply do not exist anymore in Bitcoin Core and other implementations of the Bitcoin protocol. 

Removing and enabling opcodes is a complicated affair. When script encounters an opcode which it does not recognize, execution is aborted. First, this means that, in case of a removed opcode, old nodes will still accept the opcode, while upgraded nodes do not. The reverse is true for an enabled opcode. Such a discrepancy can cause a split in the Bitcoin Network. Second, removing an opcode means that any UTXOs which requires that upcode for spending is frozen for an upgraded node. Instead of removing and enabling opcodes, it is more common to change or add to the interpretation of opcodes or patterns in opcodes. 

Script consists of two types of operators: those that push elements onto the stack (of between 1 and 520 bytes) and those that interact with the stack. 

The operators with symbols 1 through 75 each push a subsequent element with an equivalent number of bytes onto the stack. The elements can, thus, range from 1 to 75 bytes. For instance, the operator 42 indicates that 42 bytes of subsequent data have to be pushed onto the stack. Additional operators allow for pushing data with more than 75 bytes onto the stack. OP_1 through OP_16 push integer values that range from 1 through 16 onto the stack. The operators OP_0 and OP_FALSE push an empty array onto the stack. 

Besides operators that push elements onto the stack, there are operators designed to interact with the stack. These operators might, for example, make a copy of the top element and push the copy onto the top of the stack (known as **OP_DUP**); or pop the top two elements off the top of the stack, see if they form a valid ECDSA public key and signature pair, and push either an empty array (if invalid) or a 1 (if valid) back onto the stack (known as **OP_CHECKSIG**).

When a node receives a transaction (either during initial relay or as part of a block with a valid proof of work), it will obtain the unlocking script(s) for the input(s) from the raw transaction. It will, then, find the corresponding locking script(s) from its UTXO set (or if necessary, the Block Chain). The corresponding locking and unlocking scripts for the inputs are, then, combined into a single script, which then executes. 

During execution of the combined script, the stack is constantly transformed. After the script has executed, the stack will be invalid if there is a 0 or empty array as the top element. It will be valid if there is 1 or some non-zero element. Execution of a combined script can also abort, in which case the combined script is also invalid. 

Transaction validity depends on various other conditions besides script validation: the transaction needs to be properly formatted, the value captured by the outputs should be less than or equal to that of the inputs, and so on. Suppose for the moment that a transaction has met all these other conditions. 

In that case, if the combined validation script for any UTXO fails to validate, the *entire transaction* fails to validate and the transaction is rejected. If the transaction is part of a newly mined block, the *entire block* is rejected.  

Script is a programming language of limited complexity. Most importantly, one cannot construct while- or for-loops in Script. It is an example of a **Turing-incomplete programming language**, that is, one that cannot simulate a Turing machine.

The simplicity of Script is a feature, not a bug. It minimizes the risk of errors entering into scripts as well as the risk of attacks on the network being executed via malicious scripts. It also ensures that Bitcoin scripts have a predictable and efficient execution time. This is at least one key design feature that allows even ordinary users to run full nodes. 

Script does have drawbacks. First, it is difficult to ensure the correctness and security of any non-trivial contract: that is, to ensure that the script really does what you want and is safe to use. Second, each time you have created some new script construction for Bitcoin transactions, it requires a lot of work to implement properly in wallet applications. All this means that Script is not used to its full potential. With Script it is difficult, for example, to support users creating their own types of non-standard transactions. 

At least one effort to ameliorate this situation is **miniscript**, a programming language that converts into Script, but abstracts away much of its complexity. While in theory it is more limiting than Script, in practice miniscript does everything people need from Script.<sup>[2](#footnote2)</sup> 


## Smart contracts

As already emphasized, a UTXO can only be the legitimate input into a new Bitcoin transaction if the right unlocking script is offered. The script, therefore, specifies the conditions necessary for obtaining ownership of the value associated with the UTXOs. It is typical to call these types of "contracts" where the conditions of the agreement are programmed, a **smart contract**. <sup>[3](#footnote3)</sup> Script is, in other words, Bitcoin’s **smart contracting language**. 

You will commonly hear from promoters for alternative cryptocurrencies that Bitcoin does not have smart contracts, while their particular projects do. This is complete nonsense. While Bitcoin has a less complex smart contracting language than many alternative cryptocurrencies such as Ethereum—for legitimate security reasons typically ignored by these alternatives—it has a smart contracting language nonetheless. And this smart contracting language is still very powerful, allowing, for example, for the **Lightning Network**, a payments system.  

In fact, there is another substantial difference between the smart contracting languages of Bitcoin and many of its supposed competitors. Because of the checks and balances in the Bitcoin system, the locks on Bitcoin UTXOs offer a very strong guarantee about how all the value is transferred within the system. This state of affairs cannot practically be overridden by some powerful party, such as a government. 

For example, suppose the US government wants to confiscate a 100 bitcoin UTXO from Bob. They could create a transaction in which the value of that UTXO is transferred to a wallet the US government owns, but without a valid unlocking script. The US government may demand from Bitcoin miners and nodes, even using threats, to accept this transaction, but that is extremely unlikely to succeed. The reason for this difficulty is the decentralized nature of the Bitcoin Network.

Of course, the Bitcoin system does not operate in a fictional, self-contained universe, but in the real-world with institutions, laws, and power-relationships. While the US government may not be able to force an invalid transaction onto the Bitcoin Network, it can attempt to confiscate Bob's private keys or demand of him to send his 100 BTC to a wallet the US government owns. While those actions may also not succeed if Bob properly manages his bitcoin, they are certainly much more likely to succeed than forcing the Bitcoin Network to accept an invalid transaction.

Importantly, most “blockchain” networks are not decentralized, or in any case not nearly as decentralized as Bitcoin. This makes these networks much more prone to manipulation, including the changing of their history or pushing through invalid transactions. As smart contracts are often deemed to be self-enforcing in some way, it may make the term much more befitting for the locks on Bitcoin UTXOs than for the contractual programs offered within alternative cryptocurrency systems.   


## Legacy transactions

Let's now start tackling the serialization and parsing of Bitcoin transactions. A key distinction in this discussion is between the **legacy transaction format** and the **segregated witness transaction format** (or segwit transaction format), as these are serialized and parsed in different ways. Specifically, the segwit transaction format includes additional data components. We will start by exploring legacy transactions in this section, and leave segwit transactions for later. 

Before the segwit upgrade in 2017, legacy transactions were the format in which *all transactions* would have been relayed in the Bitcoin Network by *all full nodes*. That changed in 2017, as nodes started upgrading to incorporate the segwit changes. As the segwit upgrade is backwards-compatible, however, there are still **legacy nodes** in the network today that use the legacy transaction format for relaying transactions and blocks. In addition, their copy of the Block Chain will only include legacy data for the transactions.  

The serialized legacy transaction format consists of the following six main components: 

1. **Version** (4 bytes). Typically the version number is 1, except for a certain type of transaction defined by BIP-112.<sup>[4](#footnote4)</sup>
2. **Input count** (variable bytes). Typically 1 byte representing between 1 and 252 inputs. For transactions with more inputs, there is a system for increasing the byte size of the input count by specifying 253 (followed by 2 bytes of space to list the number of transaction inputs), 254 (followed by 4 bytes), and 255 (followed by 8 bytes). Given this system, the input count field can be between 1 and 9 bytes in length. 
3. **Inputs** (variable bytes). Each input is listed in series. The order does not matter. Each input has the following data:
    - **Transaction ID** (32 bytes). This ID is a double SHA256 hash of the raw transaction data in which the UTXO is located.
    - **Index** (4 bytes). This indicates which UTXO of the referenced transaction you wish to use in the current transaction. This index is also known as the vector output or “vout” in shorthand. 
    - **Length of the ScriptSig** (variable bytes). This length subcomponent maintains the same system as the input and output count components, except here the data represents the number of bytes of the ScriptSig subcomponent. 
    - **ScriptSig** (variable bytes). The data necessary to *unlock* the referenced input.  
    - **Sequence number** (4 bytes).
4. **Output count** (variable bytes). This variable works the same way as for the input count field.
5. **Outputs**. Each output is listed in series. The order does not matter. Each output has the following data:
    - **Value** (8 bytes). The value of the output specified in Satoshis. 
    - **Length of the ScriptPubKey** (variable bytes). Maintains the same system as for the length of the ScriptSig subcomponent.  
    - **ScriptPubKey** (variable bytes). The data for *locking* the output. 
6. **Locktime** (4 bytes).

The sequence and locktime (sub)components can be used to create time-based smart contracts. I will not discuss these here. The other information in the transaction should be fairly clear from the descriptions above. Most importantly, the ScriptSig and ScriptPubKey subcomponents contain the data for unlocking and locking value. The data in these subcomponents will be leveraged to create a combined script, which will be evaluated with the Script programming language. All the other data for the transaction is just interpreted using a general programming language directly (such as C++ in Bitcoin Core). 

The ScriptPubKey and ScriptSig naming conventions are somewhat misleading for the unlocking and locking scripts. They exist because original Bitcoin transactions mostly involved UTXOs locked to a public key, which required a signature to unlock. But modern locking and unlocking scripts can actually encompass a wider variety of conditions. 

You can obtain raw transactions from the Bitcoin Core command line interface using the following command: <getrawtransaction “[transaction id]">. The following byte stream is from the transaction in block 300,000 with the transaction ID of "85e72c0814597ec52d2d178b7125af0e3cfa07821912ca81bf4b1fbe4b4b70f2". 

- 0100000002acf17f885a83c7a221ab64fda59bce530b95a131a16eff3470a6cccac6b2d312000000006b483045022100a71b9fe6d94918b436e7b949f6c49407f25e4e39fc7fe20cf22e787def43cb5602200b52e999c0e75eaef28bd97609465ff41d7dad99e06b219997c3df452251e903012102e7d08484e6c4c26bd2a3aabab09c65bbdcb4a6bba0ee5cf7008ef19b9540f818ffffffff71d4a7c7fe372cb80d7170b96a8a2b8c5a0b0015f7877f50e6709fc78f1766ae010000006b483045022100c82137d106505ab32febf6ba3a607fe62cd4a4ab96fef67bf4e379405c40836302202cf6d85f4a0e811728870d649ceb47b24986599f0f09c252e9b62c94df6a2bb5012102e7d08484e6c4c26bd2a3aabab09c65bbdcb4a6bba0ee5cf7008ef19b9540f818ffffffff0200743ba40b0000001976a91429a158767437cd82ccf4bd3e34ecd16c267fc36388ace093a7ca000000001976a9140b31340661bb7a4165736ca2fc6509164b1dc96488ac00000000

This transaction has two inputs and two outputs. Using the schema of the legacy transaction format above, the bytes from the raw transaction can be allocated to the right components and subcomponenents:

1. **Version**: 01000000
2. **Input count**: 02
3. **Inputs**: 
    - **Transaction ID**: acf17f885a83c7a221ab64fda59bce530b95a131a16eff3470a6cccac6b2d312
    - **Index**: 00000000
    - **Length of the ScriptSig**: 6b
    - **ScriptSig**: 483045022100a71b9fe6d94918b436e7b949f6c49407f25e4e39fc7fe20cf22e787def43cb5602200b52e999c0e75eaef28bd97609465ff41d7dad99e06b219997c3df452251e903012102e7d08484e6c4c26bd2a3aabab09c65bbdcb4a6bba0ee5cf7008ef19b9540f818 
    - **Sequence number**: ffffffff
    - **Transaction ID**: 71d4a7c7fe372cb80d7170b96a8a2b8c5a0b0015f7877f50e6709fc78f1766ae
    - **Index**: 01000000
    - **Length of the ScriptSig**: 6b
    - **ScriptSig**: 483045022100c82137d106505ab32febf6ba3a607fe62cd4a4ab96fef67bf4e379405c40836302202cf6d85f4a0e811728870d649ceb47b24986599f0f09c252e9b62c94df6a2bb5012102e7d08484e6c4c26bd2a3aabab09c65bbdcb4a6bba0ee5cf7008ef19b9540f818
    - **Sequence number**: ffffffff
4. **Output count**: 02
5. **Outputs**: 
    - **Value**: 00743ba40b000000 
    - **Length of the ScriptPubKey**: 19 
    - **ScriptPubKey**: 76a91429a158767437cd82ccf4bd3e34ecd16c267fc36388ac
    - **Value**: e093a7ca00000000 
    - **Length of the ScriptPubKey**: 19 
    - **ScriptPubKey**: 76a9140b31340661bb7a4165736ca2fc6509164b1dc96488ac
6. **Locktime**: 00000000

Computer memory locations are organized by bytes, and **endianness** describes the order in which bytes are stored for data items. In a **big-endian system**, the byte order is from the highest value to the lowest value for the data item. In a **little-endian** system, the byte order is from the lowest value to the highest value for the data item. For example, take the number 2,404,501. In big-endian notation using six bytes (the minimum number of bytes required to represent 2,404,501), this number is 0x24B095. In little-endian notation, this number is written as 0x95B024.

A key challenge with parsing legacy transactions (as well as segwit transactions) is that the raw transaction data contains both components with big-endian and little-endian notation. Specifically, while the ScriptSig and ScriptPubKey subcomponents use big-endian notation, all other main components and subcomponents use little-endian notation. And you need be aware of the endianness during interpretation.   

Take, for instance, the 8-byte version component, which has a value of "01000000" in the transaction above. With big-endian notation, this would be the equivalent of "64". That, however, is not the correct value for the version component. Instead, this component is in little-endian notation, so the lowest byte comes first and "1" is the correct interpretation of the decimal value. 


## Standard and non-standard transactions

As already mentioned, any transaction with outputs that have valid unlocking and locking script are normally accepted by full nodes if they are included in block. Nevertheless, when it comes to the initial relay of transactions, full nodes normally only accept certain standardized unlocking and locking scripts. 

More specifically, by default Bitcoin Core conducts two tests of a transaction during relay, among a number of other tests: that all the *outputs* in the transaction follow a standard template and that all the *inputs* follow a standard template. If either of the tests fail, a full node will normally not relay the transaction. These tests are also standardly conducted in other implementations of the Bitcoin protocol. 

As Bitcoin is open source, anyone can change the code of their Bitcoin node implementation as they please. Anyone can, therefore, eliminate one or both of these tests and relay non-standard transactions. As most other nodes do not relay non-standard transactions, however, they are unlikely to reach the memory pool of a pool operator or a Bitcoin miner. Pushing such non-standard transactions in the Bitcoin Network would require connecting with a miner directly. 

This standardization was introduced in order to prevent people casting, intentionally or unintentionally, harmful transactions, particularly those that could clog the memory pool of full nodes. In addition, it makes adding new transaction features easier. 

So why still accept non-standard transactions in blocks at all? Why not eliminate them entirely? First, some early UTXOs might still require non-standard unlocking scripts and one does not want those funds to be permanently frozen. Second, historical transactions should always remain valid, and not be made invalid by an upgrade. While technically one could probably set different validation rules for past transactions based upon where they occur in the history of the Block Chain, this is prone to error. Third, this approach still allows for flexibility with locking scripts, but in an indirect manner. We will explain this further below (in the section on pay to script hash transactions).  

While non-standard transactions do occur, they are rare. Bistarelli, Mercanti, and Santini, found that only 0,02% of the transaction outputs until block 550,000 (November 2018) were non-standard.<sup>[5](#footnote5)</sup>

The following standardized locking and unlocking scripts existed before the segwit upgrade in 2017:

* Pay to public key
* Bare multisignature
* Pay to public key hash
* Pay to script hash
* OP_RETURN


## SEC Public Keys and DER Signatures

The locking and unlocking conditions for the standard legacy output types, as well as segwit output types, heavily rely on public keys and signatures. The digital signatures are always found in the ScriptSig subcomponent, while public keys are typically seen in the ScriptSig and occassionally in the ScriptPubKey (as with pay to public key and bare multisignature outputs).

Until 2021, the only digital signature system for Bitcoin transactions was the **elliptic curve digital signature algorithm**. An ECDSA public key is a point on an elliptic curve (specifically y2 = x3 + 7 modulated by a large prime field of roughly 2<sup>256</sup>), while the private key is a multiplier needed to move from a generator point to the public key. 

ECDSA public keys are formatted according to the **SEC standard**. This standard is as follows:

* An uncompressed public key starts with 0x04 and is then followed by the x and y coordinate of the ECDSA public key.
* A compressed public key starts with a 0x02 (even y-value) or 0x3 (odd y-value) followed by the x coordinate of the public key.

An ECDSA signature is formatted according to the DER standard. The format is as follows:

* Initial byte (0x30)
* The length of the rest of the DER signature (1 byte)
* A marker byte for the r value (0x02)
* Length of the r value (1 byte)
* The value of r (variable bytes)
* A marker byte for the s value (0x02)
* The length of the s value (1 byte)
* The value of s (variable bytes)

One reason for having digital signatures in Bitcoin transactions is to unlock previous outputs. Another reason is to ensure that the transaction data is not subsequently changed while in transit across the open and public Bitcoin network. You would not, for example, want someone to be able to change the outputs on your transaction. 

But obviously, you cannot just hash the entire transaction and make digital signatures for the ScriptSig subcomponents as needed, given that these subcomponents are actually included in the transaction. So over what data exactly are the digital signatures made in Bitcoin transactions?

Creating the ScriptSig subcomponents is a bit complex. In essence, you start with a dummy transaction and then manipulate it in order to end up with a valid Bitcoin transaction. To start, set all the data in the transaction, but set the length of the ScriptSig subcomponent for each UTXO to 0x00 and leave all the ScriptSig subcomponents empty. Then proceed as follows for making signatures on any UTXO:

1. For the particular UTXO for which you need one or more signatures (the latter may be the case for a multi-signature transaction), (a) insert the ScriptPubKey into the corresponding ScriptSig subcomponent, and (b) place the length of the ScriptPubKey subcomponent into the length of the ScriptSig.  
2. Choose what data to hash from the dummy transaction by appending a **sighash code**. Typically this sighash code indicates that all the data in the dummy transaction is hashed, though the signer does have some other options.
3. Create a double SHA256 hash value over data in the dummy transaction as specified by the sighash code. 
4. Create one or more DER ECDSA signatures over this hash value with one or more private keys as is required by the UTXO. Include a sighash at the end of each DER signature (typically 0x01).

Technically speaking, the sighash byte is not a part of the DER standard. Whenever we refer to DER signatures in Bitcoin, however, we generally mean it to include the sighash as well. 

To be clear, ensure that the other ScriptSig subcomponents are always left empty when working on any particular UTXO. Once you have generated all the DER signatures needed for your transaction, you can set all the data in your transaction, this time including for every input a ScriptSig subcomponent as appropriate. Many of these subcomponents will include one or more DER signatures. The length of the ScriptSig subcomponents should have the length of the ScriptSig for every input.

Let's now turn our discussion to the legacy standard transaction types. 


## P2PK outputs

A bitcoin output which is locked to a public key directly in the ScriptPubKey is known as a **pay-to-public-key output** (P2PK output). 

P2PK outputs are generally supported by nodes as a standard transaction type on the Bitcoin Network. If you send a transaction with a P2PK output, most nodes will validate, store, and relay the transaction. (Of course, they will also accept any such transaction in a block with a valid proof of work.)

That said, I do not know many wallets that support receiving and sending bitcoins via the P2PK standard. You would have to handle this manually with full node software. All this means that a transaction with a P2PK output is nowadays certainly a rare occurrence. 

The locking and unlocking scripts for a P2PK output are respectively placed in the ScriptPubKey and ScriptSig subcomponents of the transaction. These scripts are described below. Please note that in Script the integers 1 through 75 are the symbols for operations to subsequently push the equivalent amount of data onto the stack. I will from here on out adopt the notation of OP_PUSHBYTES_X, where X can range from 1 through 75, for convenience of exposition. In addition, both the public keys and digital signatures are from here on out to be understood as ECDSA-based and in the SEC and DER formats, respectively, unless otherwise specified.

*ScriptPubKey*

1. OP_PUSHBYTES_N
2. \<public key> of N bytes. 
3. OP_CHECKSIG 

*ScriptSig*

1. OP_PUSHBYTES_M
2. \<signature> of M bytes. 
    - Initial byte (0x30)
    - The length of the rest of the DER signature (1 byte)
    - A marker byte for the r value (0x02)
    - Length of the r value (1 byte)
    - The value of r (variable bytes)
    - A marker byte for the s value (0x02)
    - The length of the s value (1 byte)
    - The value of s (variable bytes)
    - A sighash byte (1 byte)
    
Again, a sighash byte is technically not part of a DER signature, but it is typically understood to be included for Bitcoin transactions.  

The combined script must validate for a P2PK input to be accepted by nodes during relay and in a block with a valid proof of work (as well as a number of other transaction conditions). The unlocking script is always executed first in the combined script. This yields the following combined script for P2PK inputs to a transaction: 

1. An M-byte signature is pushed onto the stack (ScriptSig)
2. An N-byte public key is pushed onto the stack (ScriptPubKey)
3. The OP_CHECKSIG operation pops the top two elements from the stack. It understands the first as a public key and the second as a signature. It then verifies that the signature is valid over a hash of the transaction data (according to the standard defined in the signature). It pushes a 1 onto the stack if the signature is valid. Otherwise, it pushes an empty array so that the script becomes invalid.   

The first ever non-coinbase transaction on the Bitcoin Network occurred in block 170. It has one input and two outputs, all of which are of the P2PK variety. From the raw transaction data, we can obtain the following ScriptSig subcomponent:

- ScriptSig: 47304402204e45e16932b8af514961a1d3a1a25fdf3f4f7732e9d624c6c61548ab5fb8cd410220181522ec8eca07de4860a4acdd12909d831cc56cbbac4622082221a8768d1d0901

This ScriptSig unlocks the single output from the coinbase transaction in block 9. This ScriptPubKey is as follows: 

- ScriptPubKey: 410411db93e1dcdb8a016b49840f8c53bc1eb68a382e97b1482ecad7b148a6909a5cb2e0eaddfb84ccf9744464f82e160bfa9b8b64f9d4c03f999b8643f656b412a3ac

The combined script looks as follows:

* OP_PUSHBYTES_71 (0x47)
* <304402204e45e16932b8af514961a1d3a1a25fdf3f4f7732e9d624c6c61548ab5fb8cd410220181522ec8eca07de4860a4acdd12909d831cc56cbbac4622082221a8768d1d0901>
* OP_PUSHBYTES_65 (0x41)
* <0411db93e1dcdb8a016b49840f8c53bc1eb68a382e97b1482ecad7b148a6909a5cb2e0eaddfb84ccf9744464f82e160bfa9b8b64f9d4c03f999b8643f656b412a3>
* OP_CHECKSIG (0xac)

This combined script evaluates to 1, hence the input was successfully spent. 


### Bare Multisignature outputs

Multisignature locks in Bitcoin are typically made via P2SH outputs, or P2WSH outputs. Increasingly, they will likely be made to P2TR outputs. These types of outputs will all be discussed later. 

In this section, I will look at the earliest implementation of the multisignature functionality in Bitcoin. The outputs are sometimes called **bare multisignature outputs**, and I will adopt that terminology here. These outputs were already supported in the original version of Bitcoin Core and standardized by **BIP 11**.<sup>[6](#footnote6)</sup>

Creating bare multisignature outputs is not typically possible with standard wallet software. So you would have to create these types of outputs manually and use node software to send it as a raw transaction. 
 
The bare multisignature functionality within Bitcoin only supports up to a 3 out of 3 multisignature output. For illustrative purposes, I will show a ScriptPubKey and a ScriptSig associated with a 2 out of 3 multisignature output. 

*ScriptPubKey*

* OP_2
* OP_PUSHBYTES_N
* <public key 1> of N bytes
* OP_PUSHBYTES_M
* <public key 2> of M bytes
* OP_PUSHBYTES_L
* <public key 3> of L bytes
* OP_3
* OP_CHECKMULTISIG 

*ScriptSig*

* OP_0
* OP_PUSHBYTES_X
* <signature 1> of X bytes
* OP_PUSHBYTES_Y
* <signature 2> of Y bytes

Any node verifies a 2 out of 3 bare multisignature input in the following manner:

* A 0 is first pushed onto the stack (ScriptSig). This is necessary due to an error in the CHECKMULTISIG operator which consumes an additional element over the number of signatures specified in the contract. Without the extra 0, the script would invalidate. 
* The signature 1 is pushed onto the stack (ScriptSig)
* The signature 2 is pushed onto the stack (ScriptSig)
* The value 2 is pushed onto the stack (ScriptPubKey)
* The public key 1 is pushed onto the stack (ScriptPubKey)
* The public key 2 is pushed onto the stack (ScriptPubKey)
* The public key 3 is pushed onto the stack (ScriptPubKey)
* A value of 3 is pushed onto the stack (ScriptPubKey)
* The OP_CHECKMULTISIG operation checks whether the two signatures each belong to one of the public keys (ScriptPubKey). If so, the script validates.  


## P2PKH outputs

A pay to public key hash output locks some bitcoin to, as the name implies, the hash of a public key. Unlocking such UTXOs requires the public key that was used to make the hash value as well as a valid signature over the transaction. 

P2PKH outputs are still supported by many wallets, though its native and nested segwit counterparts are really becoming the standard as these are more secure and efficient. The scripts to use a P2PKH UTXO in a transaction are as follows:

*ScriptPubKey*

* OP_DUP
* OP_HASH160
* OP_PUSHBYTES_20
* \<public key hash> of 20 bytes
* OP_EQUALVERIFY
* OP_CHECKSIG 

*ScriptSig*

* OP_PUSHBYTES_N
* \<signature> of N bytes
* OP_PUSHBYTES_M
* \<public key> of M bytes

The combined script evaluates as follows: 

* The signature is pushed onto the stack (ScriptSig).
* The unlocking public key is pushed onto the stack (ScriptSig).
* A copy is made of the unlocking public key at the top of the stack, and this copy is pushed to the top of the stack (ScriptPubKey).
* The copy of the unlocking public key at the top of the stack is popped, a **HASH160** is made of it—that is, SHA256\[RIPEMD160\[public key]]—and the result is pushed back onto the stack.
* The HASH160 of the locking public key as specified in the ScriptPubKey is pushed onto the stack (ScriptPubKey).
* The OP_EQUALVERIFY function pops the two hashes off the stack and sees whether the HASH160 of the unlocking and locking public key are equal. If so, the stack continues execution. If not, the stack stops processing immediately. 
* The OP_CHECKSIG operation pops the unlocking public key and signature from the stack and verifies that the signature is valid. 

There are a few advantages to P2PKH scripts over P2PK scripts. First, the ScriptPubKey is only 25 bytes. Even a ScriptPubKey for a compressed public key is 34 bytes. This saves Block Chain space as well as space in the mempool and the UTXO set. Second, the ScriptPubKey is much more secure. Someone wanting to steal the coins would have to find a pre-image to HASH160, as well as create a valid digital signature. So even if the ECDSA signature scheme becomes broken in some way (e.g., due to advances in quantum computing), the HASH160 ensures an additional layer of protection. 


## P2PKH addresses

If you want to receive bitcoin from another party using a P2PKH output, you could just share the 20-byte hexadecimal hash of your public key. Unlocking the value would just require the public key and making a digital signature. However, the 20-byte hashes from a P2PKH output are typically not shared in this direct manner, but via a **base58 encoding standard**. This standard includes manipulation of the SHA160 public key hash as follows:

1. Prepend the 20-byte hash with a 0x00 version number (other types of data encoded by the standard such as private keys start with a different version number)
2. DoubleSHA256 the result and take the resulting first four bytes as the checksum
3. Add the checksum to the string from step 1
4. Convert the characters from hex into base58

The resulting Bitcoin address will start with a 1. Although people sometimes use the term **Bitcoin address** in different ways, it typically refers to certain public information about UTXO locks represented via a Base58 or Bech32 scheme (a newer encoding scheme I will cover later). 

Sharing information about P2PKH outputs via a base58 address rather than as a 20-byte hexadecimal hash has various advantages.  

* It is a somewhat shorter string, so easier to share. 
* The version byte makes it easier to identify the type of information that is encoded. 
* The checksum ensures that mistakes are less easily made.  


## P2SH Outputs

If a user wants to send a transaction with a non-standard but valid output script, she would have to convince a miner to accept it into a block. Standard full nodes will not relay it. This state of affairs is clearly not amenable to users making non-standard output scripts. 

**Pay to script hash** (P2SH) outputs offer a way around this. These are a standard type of output, where the value is locked to a 20-byte hash of a script. Specifically, the output script is as follows: OP_HASH160 OP_PUSHBYTES_20 \<redeem script hash> OP_EQUAL.

In principle, you can hash any valid script into such an output, and it will be seen as standard by nodes on the network. The script contained within the hash is known as the *redeem script*. 

Unlocking the value of the output requires revealing the script, as well as offering the unlocking conditions for the redeem script. The standardized unlocking script for a P2SH UTXO is as follows: |unlocking conditions for the redeem script| OP_PUSH_X \<redeem script>. 

As P2SH outputs are most commonly used to create multisignature outputs, I will illustrate how they work via an example of a 2 out of 3 multisignature output. 

*ScriptPubKey*

- OP_HASH160
- OP_PUSHBYTES_20
- \<redeem script hash> of 20 bytes
- OP_EQUAL

*ScriptSig*

- OP_0
- OP_PUSHBYTES_N
- <signature 1> of N bytes
- OP_PUSHBYTES_M
- <signature 2> of M bytes
- OP_PUSHBYTES_X
- \<Redeem script> as a data item with X bytes: 
    - OP_2 
    - OP_PUSHBYTES_N 
    - <public key 1> of N bytes
    - OP_PUSHBYTES_M 
    - <public key 2> of M bytes
    - OP_PUSHBYTES_L 
    - <public key 3> of L bytes
    - OP_3 
    - OP_CHECKMULTISIG 

Given the specifications above, this script will evaluate in the following manner:

* A 0 value is pushed onto the stack (ScriptSig)
* A signature 1 is pushed onto the stack (ScriptSig)
* A signature 2 is pushed onto the stack (ScriptSig)
* The redeem script from the ScriptSig is pushed onto the stack as a data item (ScriptSig)
* A HASH160 is made of the redeem script and this hash value is pushed onto the top of the stack (ScriptPubKey)
* The redeem script hash from the ScriptPubKey is pushed onto the stack (ScriptPubKey)
* OP_EQUAL pops the top two values of the stack, that is, the HASH160 of the redeem script offered in the ScriptSig and the redeem script hash from the ScriptPubKey. If the operation verifies that they are equal, it pushes a 1 onto the stack. Otherwise, it pushes an empty array.  

The P2SH upgrade is described in **BIP-16**.<sup>[7](#footnote7)</sup> At this point in the validation process, if the stack concludes with a value of 1, what a node does depends on whether they are BIP-16 compliant. 

A pre BIP-16 node would just see a 1 on the top of the stack and verify the combined script at this point. BIP-16 compliant nodes, however, would know instead to also execute a separate stack for the ScriptSig data with the redeem script deserialized and the OP_PUSHBYTES_X operation removed. They know to do this on the basis of the ScriptPubKey pattern associated with the UTXO (namely, OP_HASH160 OP_PUSHBYTES_20 \<redeem script hash>). The second combined script to be executed from the ScriptSig data is then as follows:

- OP_0
- OP_PUSHBYTES_N
- <signature 1> of N bytes
- OP_PUSHBYTES_M
- <signature 2> of M bytes
- OP_2 
- OP_PUSHBYTES_N 
- <public key 1> of N bytes
- OP_PUSHBYTES_M 
- <public key 2> of M bytes
- OP_PUSHBYTES_L 
- <public key 3> of L bytes
- OP_3 
- OP_CHECKMULTISIG 

The second stack will execute as a bare multisignature output. If everything checks out, then a 1 is pushed onto the stack by the OP_CHECKMULTISIG operation and the script is valid. In principle, however, this second combined script could have also contained non-standard conditions. 

BIP-16 nodes have stricter validation standards than pre BIP-16 nodes. This may lead the latter group of nodes to accept certain transactions which are rejected by BIP-16 nodes. This type of upgrade is known as a **soft fork**: Upgraded nodes have a stricter set of validation standards. 

Given that most of the Bitcoin Network is BIP-16 compliant, only BIP-16 valid transactions will enter into the Block Chain. That pre BIP-16 nodes might still accept certain transactions which are not valid for BIP-16 nodes, therefore, has no consequences for ultimate network consensus over the Block Chain.  

P2SH outputs are also standardly represented to the user with the Base58 address format, and are recognizable by the fact that they always start with a “3”. While most wallets will not support the creation of non-standard redeem scripts, a user can create these manually and send a raw transaction with full node software. In the future, advances might make it easier for users to create their own non-standard redeem scripts more easily.   


## OP_RETURN outputs

A ScriptPubKey that begins with OP_RETURN can never be unlocked. No matter what condition you put into the ScriptSig, the script will invalidate as soon as a node sees the OP_RETURN operator. These types of transactions are, therefore, not stored in the UTXO set, but are only registered in the Block Chain. 

The point of this kind of transaction is to lock up to 80 bytes of data onto the Block Chain forever. Various types of higher-level protocols interpret and use the data in these OP_RETURN transactions. OP_RETURN outputs in the coinbase transaction are also used to store the witness commitments for segwit blocks, as we will discuss shortly. 

The ScriptPubKey for an OP_RETURN transaction looks as follows:

* OP_RETURN
* An OP_PUSHBYTES or an OP_PUSHDATA operation for N bytes
* \<data> of N bytes


## Segregated Witness Transactions

With all the standard legacy outputs just described as well as the legacy transaction format, you understand how transactions worked before the segregated witness upgrade in 2017 (Bitcoin Core 0.16.0). From that point onwards, however, a lot has changed. Lets start with the segwit transaction format. 

Any transaction that only unlocks legacy UTXOs, even if it has segwit outputs, still has the legacy transaction format for any node on the network. However, as soon as a transaction references any segwit UTXOs for inputs, it is serialized and parsed according to the **segwit transaction format**, at least by segwit nodes. This type of transaction has three additional components: the **segwit marker**, the **segwit flag**, and the **witness**. Below you can find the format of a segwit transaction.

1. **Version number** (4 bytes). 
2. **Segwit marker** (1 byte). Set at 0x00. Any legacy node that receives a transaction with the segwit marker will fail validation (as it understands the transaction as having zero inputs). 
3. **Segwit flag** (1 byte). Set at 0x01. The segwit marker and segwit flag clearly indicate to segwit nodes that they are receiving a segwit transaction.  
4. **Input count** (variable bytes). 
5. **Inputs** (variable bytes). Each input is listed in series. The order does not matter. Each input has the following data:
    - Transaction ID (32 bytes). 
    - Index (4 bytes).  
    - Length of the ScriptSig (variable bytes).
    - ScriptSig (variable bytes).   
    - A sequence number (4 bytes).
6. **Output count** (variable bytes). 
7. **Outputs**. Each output is listed in series with the following data.
    - Value (8 bytes). This value is specified in Satoshis. 
    - Length of the ScriptPubKey (variable bytes). 
    - ScriptPubKey (variable bytes)
8. **Witness data** (variable bytes) 
9. **Locktime** (4 bytes)

Each input in a segwit transaction has a witness field in the witness data component. The order of these witness fields for the UTXOs must follow their order in the inputs component. The witness field for each UTXO is as follows: 
 
1. **Number of witness elements** (variable bytes). This indicates the number of elements relevant to the validation script. 
2. Then for each element of the witness field, there are two pieces of data:
    - **The length of the element** (variable bytes).
    - **The element** (variable bytes). This is some operation or data element for the unlocking script. 

For any legacy UTXOs in a segwit transaction, the unlocking script data is just placed entirely into the ScriptSig component of the transaction. This is no different than with a legacy transaction. Additionally, a 0x00 is placed into number of witness elements subcomponent of the UTXO’s witness field to indicate that it has no witness elements. 

For segwit UTXOs, some or all of the unlocking data is placed into a witness field. Initially, there were four types of segwit outputs. This includes two native segwit outputs—pay to witness public key hash (P2WPKH) and pay to witness script hash (P2WSH)—and two nested segwit outputs—P2SH-P2WPKH and P2SH-P2WSH.   

To use a **native segwit output** in a transaction, the ScriptSig length for that particular UTXO must be set to 0x00. All the unlocking data is instead moved to the witness component of the transaction. 

To understand nested segwit outputs, we need to look at the addressing standard for native segwit outputs. Importantly, no Base58 encoding standard exists for native segwit outputs. Instead, they can only be encoded with a **Bech32 addressing standard**.<sup>[8](#footnote8)</sup>

Bech32 addresses start with “bc” to indicate Bitcoin mainnet (“tb” for the testnet), followed by a separator of “1”, and then the data encoded by all alphanumeric characters except “0”, “b”, “i", and “o” (so each character can encode 32 bits of information). 

The data here pertains to the 20-byte or 32-byte **witness program**, which is the main component of a ScriptPubKey that belongs to a native segwit output. The 20-byte program belongs to a P2WPKH output, and the 32-byte program to a P2WSH output. 

The Bech32 address format has several advantages over the Base58 format, including a better error checking mechanism and improved readability. As it is the intention to slowly migrate to Bech32 addresses, the Bitcoin community decided not to make any encoding scheme for native segwit addresses using the Base58 standard. 

Nevertheless, when segwit was first introduced to the network, many wallets did not yet support the sending of bitcoin to Bech32 addresses. In order to still allow these wallets to send money to segwit addresses, developers created a roundabout way of nesting the two native segwit transactions within a P2SH transaction. These are called **nested segwit addresses** and start with a “3”, just as any other P2SH address. 

To use any nested segwit UTXO in a transaction, the redeem script must be placed in the ScriptSig field, while the rest of the unlocking conditions are placed in the witness field. A BIP-16 pre-segwit node in principle just ensures that the hash of the redeem script from the UTXO is equivalent to the hash of the redeem script in the ScriptSig. Only segwit nodes fully validate the unlocking conditions for the redeem script.  

The transaction ID of any segwit transaction is calculated in the same way as before: it includes all and only the data that would be included in a legacy transaction. The segwit marker, segwit flag, and witness fields are ignored in the calculation of the transaction ID. For each transaction, including legacy transactions, a **witness transaction ID** can also be calculated. This is equal to the double SHA256 over **all the data** in the transaction.  

When a miner creates a block, it will calculate a merkle root of all the witness transaction IDs, under the assumption that the coinbase transaction has an ID of 0 represented by 32 bytes. This merkle root is placed in an OP_RETURN output in the coinbase transaction with a commitment header. Together these two pieces of data are known as the **witness commitment**.

A witness transaction ID is not a data component of segregated witness transactions in the way that a transaction ID is. Nevertheless, via the witness commitment, this data is indirectly locked into the Block Chain. 

Any node that is compliant with the segwit upgrade, can recognize a segwit transaction and fully validate it. They will relay valid segwit transactions to other segwit nodes. 

A legacy node instead cannot parse a segwit transaction. It can, however, be sent a watered-down version of the segwit transaction data according to the standards of a legacy transaction (i.e., one that excludes the segwit marker, segwit flag, and witness data). 

The segwit standard was created in such a way that even legacy nodes would consider a stripped down version of a valid segwit transaction as valid in a block. As it is a non-standard transaction, however, legacy nodes will refuse to process it if the transaction was simply being relayed. 

The general flow of segwit transactions through the network is as follows:

* Segwit transactions are relayed between segwit nodes for inclusion into the memory pool. Legacy nodes generally do not see these transactions, not even in legacy format, during this phase. If they do see such a transaction, they will just reject it, as they interpet the transaction as having 0 inputs. 
* Eventually these segwit transactions will be included in a valid block by a miner. This block “must” contain all the data for segwit transactions. It will also include a witness commitment in the coinbase transaction. A valid proof of work is still over the traditional block-header.   
* The valid block gets sent around the network. Any segwit nodes will send a watered-down version of the block to legacy nodes, that is, one which only includes the legacy data for segwit transactions. If a legacy node, in turn, relays the trimmed block to another segwit node, the latter will reject it as the segwit transactions do not include all the unlocking data. 
* In this manner, all segwit nodes store a valid segwit block, while all the legacy nodes will store a valid legacy block. Note that “segwit Block Chains” contain transactions with exactly the same transaction IDs as their legacy counterparts.  

As stated above, a miner “must” send a block with all the data for segwit transactions. I mean here that a miner which does not support segregated witness will be unsuccessful in propagating any blocks, even if these contain a valid proof of work. Segwit nodes will namely not accept stripped down blocks. As most nodes in the Bitcoin Network are segwit nodes, the most proof of work will always return to a chain with only segwit blocks.  

A key insight here is that, while legacy nodes cannot fully validate segwit transactions, they can track how segwit outputs are moved. 


## An example segwit transaction

In block 735,750, you can find a transaction with the following transaction ID: 67b94f347eed483b4ccad276b775561b97e0ac371624efd883f7c54e39c1e65d. 

This transaction spends two inputs, a native segwit UTXO and a legacy UTXO, and has one segwit UTXO as an output. Because it has at least one segwit UTXO, a segwit node will store and relay it in the segwit transaction format. 

The raw transaction data for this segwit transaction is as follows:

* 0200000000010278feb269f70e935fcb1b3372d67192eed8e52f4c4abcc5597f00a1439da9e8340100000000ffffffff057a1613f2d014b71644b7cd87c790ab645d33e1ecf237c7d40e447659b46160000000006a473044022034802476eefa0c954c3ff7e907032faeb7bcd245abda223a1a47057642b7f4930220185fef340b9f305299a503036683e317345b95c3f62897a4c318007a7f1d70210121034fc35c4651642f52c175b69e3f0021e48d6b07f7cee2e1d709a21d1df388acecffffffff01978800000000000016001459d171f541cc79f4220ee9012bbc99ec32286edf02473044022020dbcbed6125c3cdca0d99a1765ae7308675662b93a5faadddf99c16f7b43b95022016cf9d1525d194411995615ee914a5db981e1f69b25b93aaac335078748e95190121026b77118b2044cd77f8c7913dde71ccbe58438537a068a5079eac3b6888a74bc50000000000

Using the schema from the previous section, we can split this raw transaction into all its components and subcomponents. This is done below.

1. **Version number**: 02000000 
2. **Segwit marker**: 00 
3. **Segwit flag**: 01  
4. **Input count**: 02 
5. **Inputs**
    - Transaction ID: 78feb269f70e935fcb1b3372d67192eed8e52f4c4abcc5597f00a1439da9e834 
    - Index: 01000000  
    - Length of the ScriptSig: 00
    - ScriptSig: (empty)   
    - A sequence number: ffffffff
    - Transaction ID: 057a1613f2d014b71644b7cd87c790ab645d33e1ecf237c7d40e447659b46160 
    - Index: 00000000  
    - Length of the ScriptSig: 6a
    - ScriptSig: 473044022034802476eefa0c954c3ff7e907032faeb7bcd245abda223a1a47057642b7f4930220185fef340b9f305299a503036683e317345b95c3f62897a4c318007a7f1d70210121034fc35c4651642f52c175b69e3f0021e48d6b07f7cee2e1d709a21d1df388acec   
    - A sequence number: ffffffff
6. **Output count**: 01 
7. **Outputs**
    - Value: 9788000000000000 
    - Length of the ScriptPubKey: 16 
    - ScriptPubKey: 001459d171f541cc79f4220ee9012bbc99ec32286edf
8. **Witness data**
    - Number of witness elements (Input 1): 02
    - Length of the element: 47
    - Element: 3044022020dbcbed6125c3cdca0d99a1765ae7308675662b93a5faadddf99c16f7b43b95022016cf9d1525d194411995615ee914a5db981e1f69b25b93aaac335078748e951901
    - Length of the element: 33
    - Element: 026b77118b2044cd77f8c7913dde71ccbe58438537a068a5079eac3b6888a74bc5
    - Number of witness elements (Input 2): 00
9. **Locktime**: 00000000


## Why segwit?

The segwit upgrade is, many would argue, the biggest upgrade that has occurred in Bitcoin's history. It involves the upgrade to the transaction format discussed above, but it includes a number of additional changes.<sup>[9](#footnote9)</sup> Before turning to segwit outputs, it is helpful here to first explore the benefits of segwit. I will cover four main benefits that motivated the proposal here, though there are more.<sup>[10](#footnote10)</sup> 

Lets start with transaction malleability. 

The transaction ID of a legacy transaction is a double SHA256 hash over all the data. This includes the ScriptSig data. As explained in the section on "SEC public keys and DER signatures," the ScriptSig subcomponent is populated by the ScriptPubKey data during the signing process. Throughout Bitcoin's history ways have been discovered to change the ScriptSig of a transaction and subsequently changing the transaction ID; that is, ways were discovered in which the ScriptSig was **malleable**. 

For example, Bitcoin originally used the OpenSSL standards for interpreting DER signatures in network data. However, those encoding standards were fairly relaxed, as they were not created for consensus-critical protocols. One could, for instance, add some padding and the signature would still be valid. This malleability issue was addressed by **BIP-66**, and all blocks from 363,724 onwards enforce a strict standard on DER signatures.<sup>[11](#footnote11)</sup>

Another famous malleability issue was the fact that for any valid ECDSA signature pair (r,s), the signature pair (r, -s mod n), where n is the order of the elliptic curve, is also valid. Hence, anyone could just calculate (r, -s mod n) and create a new transaction ID. While the lower s value was standardized for transaction relay in 2015,<sup>[12](#footnote12)</sup> miners could still manipulate signatures via this method by the time of the segwit upgrade.

Another way that signatures were malleable was simply due to the nature of ECDSA. The algorithm works on the basis of a random nonce, so the digital signatures are not deterministic. So anyone with the private key can make multiple valid signatures on the same transaction without changing any of the data. 

These malleability issues had a significant impact on Bitcoin at the time. First of all, malleability attacks had almost certainly been carried out. As Mt. Gox's withdrawal delays started in early 2014, it released a statement claiming that its funds were stolen via malleability attacks.<sup>[13](#footnote13)</sup> While malleability attacks were originally often cited as a reason for the exchange's collapse shortly after this statement,<sup>[14](#footnote14)</sup> subsequent research has thrown doubt on the size and impact of malleability attacks.<sup>[15](#footnote15)</sup> Second, even a transaction ID that unintentionally changes before confirmation poses substantial security and useability risks. Finally, the Lighting Network would have been much more difficult to implement reliably if transaction IDs were malleable, as its architecture utilizes transactions that are dependent on unconfirmed transactions. 

While some of the malleability issues in Bitcoin could be addressed by stricter validation rules, this could not address the problem entirely. First, some malleability issues were impossible to fix using stricter validation rules, such as the fact that anyone with a private key can make multiple valid signatures over the same data. Second, addressing malleability issues by constant upgrades is risky and cumbersome. 

Segregated witness addresses malleability by moving the ScriptSig data for a segwit UTXO to a new data structure in the transaction, the witness. The witness is not used in calculating the transaction ID. Hence, the name “segregated witness.” 

If you have a segwit transaction that includes only inputs from native segwit UTXOs, the transaction ID is no longer malleable, as the ScriptSig subcomponent is empty. If some UTXOs in a segwit transaction are nested segwit inputs and the rest native segwit inputs, then all the data for the ScriptSig is non-malleable (it’s the hash of a redeem script). All the sensitive unlocking data is instead in the witness data component. Finally, if your segwit transaction includes one or more legacy UTXOs, it is still malleable. Reducing the risks of malleability is, therefore, a matter of switching to segwit outputs in transactions. 

A second main benefit of segregated witness was that it led to an increase in the block size. 

Segwit nodes store blocks which include all the data for segwit transactions (that is, they store the segwit marker, segwit flag, and notably the segwit witness data). These nodes no longer maintain a maximum block size of 1 MB, as do legacy nodes. Instead, they accept a maximum **block weight** of 4,000,000 bytes. This block weight is calculated as follows: *block weight* = 3 * *base size* + *total size*. 

The **base size** in this case is the block size for non-segwit nodes using the legacy data format for all transactions. The **total size** is the block size as seen by segwit nodes using the segwit data scheme for all transactions that use at least one segwit UTXO. 

Given that the total size of any segwit block must be greater or equal to its base size, the block weight formula ensures that the base size can at a maximum be 1,000,000 bytes. This occurs in the unlikely event that all the transactions in a block only utilize legacy UTXOs—so base size equals total size—and the total data is exactly equal to 1,000,000 bytes. The formula, thus, ensures that legacy nodes will always accept any legacy block version of a segwit block.  

This all constitutes an increase in the block size of sorts. Segwit nodes just have a differently calculated data limit. Legacy nodes can accept more transactions in their blocks because the size of the ScriptSig data is decreased substantially for segwit inputs. It becomes 0x00 for a native segwit input and only the redeem script data with a pushbyte operation for nested segwit inputs. As the ScriptSig component constitutes a major portion of the size of legacy transactions, this savings in data is significant. 

Note that I have presented the block size increase here as a benefit. Some would argue that this is not actually a benefit, but a downside. Luke Dash Jr., for example, has been a notable proponent of smaller blocks.<sup>[16](#footnote16)</sup> 

A third benefit from segregated witness is related to the hashing procedure for generating signatures. This procedure was changed for native segwit outputs and has two benefits:

- For legacy outputs, the hashing procedure before making a signature increases quadratically in complexity as the size of the transaction grows. This is known as the **quadratic hashing problem**. The hash procedure for native segwit outputs is scales more efficiently with the data, so that even large transactions can be handled effectively.
- The hashed information in a legacy transaction does not include data about the value of the UTXOs unlocked. This makes makes it more difficult for an offline wallet to verify the total value of the inputs to a transaction. The total value of UTXOs is included in the hashing procedure for a segwit UTXO. This means that, as long as all inputs to a transaction are native segwit, an offline wallet can verify the total value of the UTXOs much more easily from an external source: an incorrect amount means that the signatures on the transaction will not verify. 

A fourth benefit relates to the upgrading mechanism for Script. Before segregated witness, upgrades could generally be brought about by replacing the meaning of one of the OP_NOP opcodes (which do nothing, but do not invalidate the script) through a soft fork. But this was slightly hacky and also prevented certain upgrades. Segregated witness outputs have a versioning system, to be implemented by using the OP_1 through OP_16 opcodes. This makes advanced scripting much easier. 

As legacy nodes see a segwit output as an output that anyone can spend, they are not thrown off by these version numbers for upgrading. That legacy nodes see them as outputs that anyone can spend would have been a problem if most of the network had not implemented the segwit upgrade. As this is not the case, it does not matter for consensus.  

So far I have only discussed the main benefits of the upgrade to segregated witness that were intentional. There was, however, another benefit that I would argue was even more important, though not entirely by design: the upgrade to segregated witness showed that miners were not in control of Bitcoin. Many miners had been strongly opposed to the upgrade for various reasons. At the end of the day, it was the full nodes that forced the miners to accept the upgrade.<sup>[17](#footnote17)</sup>  

Now that we have formed a solid understanding of what the segwit upgrade entails and its benefits, lets turn to the segwit outputs that were contained within the upgrade. As stated above, there are two types of native segwit outputs. These are the **pay to witness public key hash** (P2WPKH) and **pay to witness script hash** (P2WSH) outputs. In addition, both of these can also be nested in P2SH transactions. Lets discuss these four outputs in turn.


## P2WPKH scripts

The segwit upgrade defined two types of native segwit outputs. Any serialized ScriptPubKey for a transaction contains the string 0x0014 followed by 20 bytes of further hexadecimal output is known as **pay to witness public key hash output** (P2WPKH output). The 0 here is a version number, while the 20 bytes of data, the witness program, is the OP_HASH160 hash of a public key. 

The ScriptSig length for unlocking this type of output has to be set to 0x00. Instead, all the relevant data should be included in a witness field. The serialized witness field should look as follows, where the byte needed to express X and Y depends on the size of the signature and public key respectively. 

* The number of witness elements: 0x02
* Length of element 1: X
* Element 1: The signature
* Length of element 2: Y
* The compressed public key

Note that a P2WPKH output, as well as a P2WSH output, only allows compressed SEC public keys. 

A P2WPKH output locks some bitcoin to, as the name implies, the hash of a public key. This works just like a pay to public key hash output. The big difference is that the unlocking data for a P2WPKH UTXO is placed in the witness component of a transaction. So unlike a P2PKH UTXO which allows for a malleable transaction ID, a P2WPKH does not allow for a malleable transaction ID. 

For legacy UTXOs, the ScriptPubKey and ScriptSig translate directly into Script. This is not the case for the ScriptPubKey and the witness field of a P2WPKH UTXO. Instead, on seeing the ScriptPubKey pattern 0x0014 followed by 20-bytes, a segwit node will know how to create a locking and unlocking script with the data in the witness field for the UTXO. More specifically, a segwit node will know to execute the combined script below.  

* OP_PUSHBYTES_X
* \<signature> of X bytes
* OP_PUSHBYTES_Y
* \<public key> of Y bytes
* OP_DUP
* OP_HASH160
* OP_PUSHBYTES_20
* <20-byte public key hash>
* OP_EQUALVERIFY
* OP_CHECKSIG

This is just the same combined script as for a P2PKH UTXO. The execution of this script proceeds in this same way. 

* The signature is pushed onto the stack (witness).
* The compressed public key is pushed onto the stack (witness). 
* A copy is made of the public key and pushed onto the stack.
* The copy of the public key is put through the HASH160 function. The result is pushed back onto the stack. 
* The 20-byte hash from the witness program is pushed onto the stack (ScriptPubKey).
* The EQUALVERIFY operation pops the top two elements of the stack and checks whether they are equal. If so, execution continues. Otherwise execution halts immediately. 
* The CHECKSIG operation verifies that the signature is valid for the public key. It pushes a value of 1 if valid. Otherwise, it pushes an empty array. 

In contrast to a regular P2PKH transaction, much of the script language is not included in the transaction directly. This was done to save space on the Block Chain. 
 
In addition, also notice that the zero which is pushed by the ScriptPubKey is not an element of the validation script. It is only an indicator with regards to the structure of the validation script. Given that you can also push the values 1 through 16 with Script, pattern interpretation in line with native segwit transactions offers an easy path to upgrades. 

A legacy node will not see any of the witness data, and only executes the script according to legacy rules. This is as follows:

* The ScriptSig is empty, so nothing is pushed to the stack
* A 0 is pushed to the stack (ScriptPubKey)
* The 20-byte public key hash is pushed to the stack (ScriptPubKey)
* As the top element of the stack is not zero, the script will validate

This is a valid combined script for a legacy node, as the top of the stack is a non-zero element; so legacy nodes see a P2WPKH output as an anyone can spend output (same goes for all other native segwit outputs)


## P2WSH outputs

A **pay to witness script hash output** (P2WSH output) locks some bitcoin to, as the name implies, the hash of a script. This is just like a pay to script hash output. The big difference is that the unlocking data for a P2WSH UTXO are placed in the witness component of a transaction. So unlike a P2SH UTXO which allows for a malleable transaction ID, a P2WSH does not enable a malleable transaction ID. 

Any SciptPubKey for a P2WSH output serialized has the following pattern: a string 0x0020 followed by 32 bytes. The 32 bytes here is the witness program and it indicates to a segwit node that this is a P2WSH output. The witness program is the SHA-256 hash of a **witness script** (not the HASH160). The latter is the equivalent of a redeem script for a P2SH output. 

The ScriptSig length for unlocking a P2WSH output has to be set to 0x00. Instead, all the relevant data should be included in a witness field (number of witness elements, length of each witness element, and the witness element).

Unlike for a P2WPKH output, where a segwit node knows the witness program is the 20-byte hash of a public key, a segwit node cannot infer anything about the witness script from the 32-byte witness program; so all the relevant data must be included in the witness field for the UTXO. 

When a segwit node sees a P2WSH UTXO in a transaction, it will first check that the SHA-256 of the witness script contained in the witness field is equivalent to the 32-byte witness program; it will, then, evaluate the script that can be constructed from the witness field.

Lets work through an example with a 2 out of 3 multisignature address. In that case, the ScriptPubKey will look as follows:

* OP_0 
* OP_PUSHBYTES_32
* <32-byte redeem script hash> 

The ScriptSig for unlocking this type of output also has to be set to 0x00. Instead, all the data should be included in a witness field as follows, where the byte(s) needed to express the size of M, N, L, X, Y, and Z can vary: 

- Number of witness elements: 0x04
- Length of element 1: 0x00 (indicates that the first element is to be interpreted as OP_0)
- Length of element 2: M 
- Signature 1
- Length of element 3: N
- Signature 2
- Length of element 4: L
- The witness script. This should include all the following data:
    - 0x52 (represents OP_2 in Script)
    - Length of X bytes
    - Public key 1 of X bytes 
    - Length of Y bytes
    - Public key 2 of Y bytes 
    - Length of Z bytes
    - Public key 3 of Z bytes 
    - 0x53 (represents OP_3 in Script) 
    - 0xae (represents OP_CHECKMULTISIG in Script)

At this point, any legacy node will just execute the following script, as it does not see the witness data:

* Push a 0 onto the stack
* Push a 32-byte value onto the stack

As the top of the stack is not zero or an empty array, the script will validate for aNY legacy node. Segwit nodes on the other hand know to first check whether the SHA256 of the witness script equals the 32-byte witness program provided in the ScriptPubKey. If so, they will next execute the following script for validation:

* OP_0
* OP_PUSHBYTES_M
* \<signature> of M bytes
* OP_PUSHBYTES_N
* \<signature> of N bytes
* OP_2
* OP_PUSHBYTES_X
* <public key 1> of X bytes 
* OP_PUSHBYTES_Y
* <public key 2> of Y bytes 
* OP_PUSHBYTES_Z
* <public key 3> of Z bytes 
* OP_3
* OP_CHECKMULTISIG

The execution of this script proceeds as with a regular 2 out of 3 multisignature output:

* The 0 is pushed onto the stack
* DER Signature 1 is pushed onto the stack
* DER Signature 2 is pushed onto the stack
* The 2 is pushed onto the stack
* The compressed SEC public key 1 is pushed onto the stack
* The compressed SEC public key 2 is pushed onto the stack
* The compressed public key 3 is pushed onto the stack
* The 3 is pushed onto the stack
* The OP_CHECKSIG operation verifies that the two signatures are valid for different public keys. 

Note that by contrast to P2WPKH outputs, all the necessary data for the script is included in the witness field. Given that this output type allows any admissible script, it cannot execute according to any pre-stored template. 


## P2SH-P2WPKH outputs

In a P2SH-P2WPKH output, the redeem script of a P2SH transaction is the ScriptPubKey of a P2WPKH output understood as data. The ScriptSig field contains the redeem script, while the witness field contains the digital signature and public key.

*ScriptPubKey*

- OP_HASH160
- OP_PUSHBYTES_20
- \<redeem script hash>
- OP_EQUAL

*ScriptSig*

- OP_PUSHBYTES_22
- \<Redeem script> pushed as data with the following information:
    - OP_0
    - OP_PUSHBYTES_20
    - \<public key hash> of 20 bytes

*Witness Field*

- 0x02: Indicates the number of witness elements
- X: The length of the signature
- The DER signature
- Y: The length of the public key
- The compressed SEC public key

Any node that is not BIP-16 compliant will just verify that the redeem script in the ScriptSig has the same HASH160 value as specified in the ScriptPubKey. A BIP-16 compliant node will know to deserialize the ScriptSig data and run the contained script. A segwit node will know to create a script that executes as follows on the basis of the pattern in the redeem script:

* A DER signature is pushed onto the stack (witness)
* A compressed SEC public key is pushed onto the stack (witness)
* A duplication is made of the SEC public key and added to the top of the stack
* This copy is inserted into the HASH160 function. The result is added to the top of the stack.
* The HASH160 value, or witness program, from the redeem script is added to the top of the stack (ScriptPubKey)
* The EQUALVERIFY operation pops the top two elements of the stack and checks whether they are equal. If so, execution continues. Otherwise execution halts immediately. 
* The CHECKSIG operation verifies that the signature is valid for the public key. It pushes a value of 1 if valid. Otherwise, it pushes an empty array. 

Notice that the ScriptSig field only contains data that is not malleable, namely two opcodes and a 20-byte HASH160 of the public key. In this way, P2SH-P2WPHK inputs are not malleable, therefore, preventing transaction ID malleability. 


## P2SH-P2WSH outputs

In a P2SH-P2WSH output, the redeem script of a P2SH transaction is a P2WSH output. The ScriptSig field contains the redeem script, while the witness field contains the further unlocking conditions for the redeem script. 

Lets take a 2 out of 3 multisignature P2WSH output as the example. The information needed to validate the scripts for this type of output if nested in a P2SH output is as follows: 

*ScriptPubKey*

- OP_HASH160
- OP_PUSHBYTES_20
- \<Redeem script hash>
- OP_EQUAL

*ScriptSig*

- OP_PUSHBYTES_Z
- A redeem script of Z bytes as data with the following elements:
    - OP_0
    - OP_PUSHBYTES_32
    - <32-byte hash>

*Witness*

- 0x04: Indicates the number of witness elements
- 0x00: Indicates that the first element is OP_0
- X: The length of signature 1
- DER signature 1
- Y: The length of signature 2
- DER signature 2
- The witness script as data with the following elements:
    - OP_2
    - OP_PUSHBYTES_N
    - <public key 1> of N bytes 
    - OP_PUSHBYTES_M
    - <public key 2> of M bytes 
    - OP_PUSHBYTES_L
    - <public key 3> of L bytes 
    - OP_3
    - OP_CHECKMULTISIG

Any node that is not BIP-16 compliant will just verify that the redeem script in the ScriptSig has the same HASH160 value as specified in the ScriptPubKey. A BIP-16 compliant node will know to deserialize the redeem script and run it as a separate script. If the node does not support segwit standards, it will just end up with a 32-byte hash on the stack and validate. 

A segwit node will recognize the OP_0 OP_PUSHBYTES_32 <32-bytes hash> pattern. It will first check whether the SHA256 hash of the witness script equals the 32-byte witness program (that is, the 32-byte hash in the redeem script). If so, it will create a script that executes as follows:

* A 0 is pushed onto the stack (witness)
* A DER signature is pushed onto the stack (witness)
* A compressed SEC public key is pushed onto the stack (witness)
* A value of 2 is pushed onto the stack (witness)
* Three public keys are pushed onto the stack (witness)
* A value of 3 is pushed onto the stack
* OP_CHECKMULTISIG verifies that the two signatures are valid for different public keys. It pushes a value of 1 if valid. Otherwise, it pushes an empty array.

Notice that as with a P2SH-P2WPKH output, the data in the ScriptSig field is not malleable. It only contains an operation that pushes a 0 and a 32-byte hash of the witness script. None of the data components of the witness script can be altered without changing the meaning. Hence, the transaction ID is not malleable as a consequence of the data in the ScriptSig.


## Taproot

A **pay to taproot** (P2TR) output is marked by the OP_1 OP_PUSHBYTES_32 <32-byte witness program> pattern in the ScriptPubKey. This 32-byte witness program is a public key. So in total, a taproot ScriptPubKey is 34 bytes, just as a P2WSH output.  

Bitcoin public keys are frequently indicated by 33 bytes. This includes 32 bytes for the public key’s x-value and one byte to signal whether the y-value is even (0x02) or odd (0x03). For a taproot output, the y-value is implicitly understood to always be even. This supports more efficient verification without really impacting security. 

As seen from ScriptPubKey pattern, Taproot outputs are version 1 segwit transactions. As with version 0 segwit transactions, the unlocking conditions for UTXOs are placed into the witness component of a transaction, rather than in the ScriptSig. Furthermore, taproot outputs also employ the bech32 address standard. Each taproot address starts with “bc1p”, rather than “bc1q”, reflecting the new version number. 

Just as the segregated witness soft fork, the taproot soft fork includes a number of upgrades, so it takes some effort to unpack. Lets start with the signatures for taproot outputs: the digital signature algorithm (DSA) scheme has been replaced by the Schnorr signature scheme.  

The **Schnorr signature scheme** has various advantages over DSA. A key advantage is that it offers much better methods of **signature aggregation**. The idea of signature aggregation is as follows: 

* You can add two or more component public keys to form a composite public key.
* You can add the signatures over a message from the component public keys to form a composite signature.
* The composite signature will be valid for the composite public key.

While you could perform straightforward addition of public keys and signatures to get the result, this is insecure and subject to what is known as a **key cancellation attack**. A more secure, though also more complicated method of addition is known as **MuSig**.<sup>[18](#footnote18)</sup> Other addition schemes can also be used. While aggregation is possible with DSA signatures, it is much more complicated. 

Importantly, the aggregation property ensures that multiple Schnorr signatures over a message can be validated through one composite signature. 

Schnorr signatures also have various other advantages. These include provable security under less strict assumptions than ECDSA, non-malleability, and batch verification capabilities (that is, verification of multiple Schnorr signatures is relatively more efficient than individual verification). 

A Schnorr signature is 64 bytes and no longer appears with DER encoding in the witness data. Without an additional SIGHASH flag, the SIGHASH_ALL flag is implied for hashing the data over which the signature is created. For any other SIGHASH flag an additional byte is added. So Schnorr signatures are generally 64 and sometimes 65 bytes. 

This makes Schnorr signatures somewhat shorter than DER encoded DSA signatures. These are historically between 71 and 73 bytes, though they should always be 71 bytes in modern wallet implementations. 

Further details of Schnorr signatures are provided in **BIP-340**.<sup>[19](#footnote19)</sup>  

Next, the script organization logic of taproot outputs offers more flexibility. Specifically, any taproot output can consist of one or more locking scripts. If you want to spend a taproot UTXO, you have to select one of these locking scripts and provide a valid unlocking script. Any one of them will allow the UTXO to be spent. 

To see the difference with the traditional organizational logic of Bitcoin locking scripts, consider a 2 out of 3 multisignature output using bare multisignature, P2SH, or P2WSH. In such outputs, the 2 out of 3 logic is included in a single script. With a taproot output, however, you can organize the logic into three 2 out of 2 multisignature scripts. You can choose any one of these 2 out of 2 multisignature scripts to unlock the UTXO. 

The taproot upgrade also includes some substantial changes to Bitcoin’s Script language. The CHECKSIG and CHECKSIGVERIFY operations, for instance, now verify Schnorr signatures. In addition,  the CHECKMULTISG and CHECMULTISIGVERIFY operations have been disabled, and multisignature scripts use CHECKSIGADD instead. 

These and other Script changes are captured in **BIP-342**.<sup>[20](#footnote20)</sup> The upgraded version of Script is often called **Tapscript**. 

Any taproot output will have a default locking script. In principle, you can make a taproot output that just has this default locking script. Frequently, however, a taproot output will also have one or more alternative scripts. In fact, it may have quite a few. A taproot output might easily include, for example, fifty alternative scripts with a variety of complex locking conditions. 

The default locking script encumbers the bitcoin to a public key for which a single Schnorr signature has to be provided. This public key is known as the **taproot output key** and is what is represented by the 32-byte witness program.

In some cases, the taproot public key is just a regular public key corresponding to a single private key created by the owner. Frequently, however, it is a composite key, which also includes a commitment to alternative scripts. 

Lets unpack how this is possible. To start, BIP-340 introduced a new hashing scheme for Bitcoin transactions called **tagged hashes**. These include identification tags in creating certain hash values which prevents reinterpretation of the hash in another context. Specifically, one creates a tagged hash has as follows:

* TaggedHash (data) = SHA256 \[ SHA256(“Tag”) | SHA256(“Tag”) | data ]

A composite taproot output key usually starts by creating two or more public keys. The public keys are then added to produce the **taproot internal key**. 

In case of no alternative scripts, one can just use the taproot internal key as the taproot output key. Typically, however, information about one or more alternative scripts are used to alter the taproot internal key. This works as follows. 

First, each alternative script in a taproot output is accompanied by a version and size indicator for the script, and tagged with the “TapLeaf” tag. Specifically, each **taproot leaf** is calculated in the following manner:

* TapLeaf = TaggedHash (“TapLeaf”, version, size, script)

Suppose that the taproot transaction only had one alternative script. In that case, the taproot leaf would be hashed together with the taproot internal key and multiplied with Bitcoin’s generator point to produce the **tweaked public key**. Specifically, the tweaked public key would be created as follows: 

* Tweaked public key = TaggedHash (“TapTweak”, Taproot Internal Key, TapLeaf) * G

The value produced by TaggedHash (“TapTweak”, Taproot Internal Key, TapLeaf) is known as the **taptweak**.

Combining the tweaked public key with the taproot internal key produces the **taproot output key**. In other words, a commitment to the alternative script is included via public key aggregation to the taproot output key.  

Now lets turn to the case in which there are two or more scripts. Each taproot leaf is hashed as before. All the leaves are, then, combined to form a tree-like structure to produce a merkle root. Every higher-level node is called a **tap branch**. Each tap branch to two nodes A and B is calculated as follows:

TapBranch = TaggedHash (“TapBranch”, A, B)

Eventually, the tree will converge to a single tap branch, the merkle root. The tree is also called the **tap tree**. The tweaked public key can now be calculated as follows (where the TapBranch in this case is that of the merkle root):

* Tweaked public key = (“TapTweak”, P, TapBranch)

The taproot output key can now be calculated in the same way as before by combining the taproot internal key with the tweaked public key. 

The tree structure for the alternative scripts is sometimes called the **merkelized alternative script tree**. Originally, the general idea for these types of trees went by the name of **merkelized abstract syntax trees**. 

It is important to emphasize that the taproot output key can commit to a great number of alternative scripts. As spending any of these scripts only requires revealing certain hash values along its merkle path, the tap tree scales really well.  

Spending from a taproot output key that has made a commitment to alternative scripts can now happen in two ways. 

First, in the default case, one participant in the construction can create a signature for the combination of their public key with the tweaked public key. Its private key is, namely, just tweaked in the same way as the public key. In addition, each other participant that helped compose the taproot internal key can create a signature. 

Adding all these signatures—by a mechanism such as MuSig—gives a valid signature for the taproot output key. Anyone who sees this transaction has no idea of the complexity underlying taproot. As far as they are concerned, it might just have been a regular public key output without any alternative scripts. 

Spending an alternative script path is a little more data intensive. It requires revealing the script, the relevant hashes in the tree, the internal taproot key, and proving the commitment of the script to the taproot output key. In addition, it requires an unlocking script.

All this has substantial privacy and efficiency benefits for Bitcoin transactions, particularly when those involve complex scripts. 

Suppose that you were making a multisignature output on your own. In that case, you could select the most likely signature pattern to create the taproot internal key. Signing the taproot output key directly now provides substantial privacy. No one can tell whether you spent from a regular single public key or a composite public key. In addition, no alternative script options have been revealed. 

If you have to spend from one of the other scripts, perhaps due to losing one of your private keys, then you will lose some privacy. But you still only have to reveal only one of the leaves of the taptree. No one will be able to see what other valid scripts were in it. This is unlike a P2SH or P2WSH UTXO where all the relevant script has to be revealed upon spending.  

Note also that taproot outputs make multisignature much more efficient, at least if you use the taproot output key directly. Instead of providing multiple signatures, multiple public keys, and a script as is needed for P2SH and P2WSH multisignature addresses, all you need is a single signature. As the public key is offered directly by the output, you also do not need to provide it in the witness field.  

The same benefits can, of course, be extended to multisignature outputs with multiple parties. But taproot goes much further than just multisignature constructions. Typically, complex contracts involving multiple parties can be closed cooperatively by a signature from all the parties, regardless of the nature and complexity of the contract. This is precisely what the default spending option achieves. The alternative script options can, then, serve as fallback options for redundancy and/or keeping other parties honest. 

Hence, as long as no one is attempting dishonesty, complex bitcoin contracts can always be closed by a single signature. This again offers substantial privacy and efficiency benefits. In case of dishonesty, one of the other scripts can be executed by some subset of the parties to the contract. This still offers some privacy and efficiency benefits over creating complex contracts with P2SH and P2WSH. 

In sum, taproot is essentially the idea that a default multisignature key branch can be combined with alternative spending scripts in a single public key, and that spending occurs via a single signature in the default case. Presumably, the name “taproot” draws a comparison with the dominant root from which other roots sprout in plants.<sup>[21](#footnote21)</sup>

Lets now look at the script validation a little more closely. 

Any taproot UTXO that spends from the **key path** instead of the **script path** will have two elements in the witness field. The last element is known as an **annex**. It begins with the value 0x50. This annex is included for upgrades, but is ignored during the validation process. Having only one element left after the annex, a taproot compatible client will know to run the following script:

* OP_PUSHBYTES_64
* \<Schnorr signature> of 64 bytes
* OP_PUSHBYTES_32
* \<taproot output key> of Y bytes
* OP_CHECKSIG

Note that any client which does not support segwit will just end up with a public key on top of the stack (from the ScriptPubKey), which is seen as a valid transaction. Clients that only support segwit version 0 will also just end up with a public key on top of the stack and validate  the script.  

Instead of spending from the key path, one can also spend from a script path. A taproot supporting client recognizes a script path spend if there are two or more items in the witness field after eliminating the annex. In this case, you will need to include a number of unlocking conditions in the witness field, the script, and the merkle path with the relevant hash values. 

**TO DO: Work out example for script path spending**

A notable difference between typical Bitcoin output types and taproot is that the latter directly contains a public key, not a hash value. Only the original P2PK and bare multisignature outputs directly included public keys into the ScriptPubKey.

Putting a public key directly in the output saves on transaction size. It means that the public key does not have to be revealed within the witness field. Only a signature is sufficient. In addition, directly having the public key can make certain advanced protocols easier. 

Some have criticized this architecture against not providing sufficient security against quantum-based computing attacks.<sup>[22](#footnote22)</sup> Overall, however, the sentiment on this seems to be that hashing provides weak protection and/or the quantum computing threat is far from materializing. 


## Notes

<a name="footnote1">1</a>. Sometimes the term **script** has a more specific meaning, as any small program interpreted at run-time that is also a top-level program file or as any small program interpreted at run-time that is intended to be executed directly by a user. But I will use the term in its more general sense. 

<a name="footnote2">2</a>. See Pieter Wuille and Andrew Poelstra, "Miniscript: Streamlined bitcoin scripting," September 7 (2019), available at: https://medium.com/blockstream/miniscript-bitcoin-scripting-3aeff3853620; and Aaron van Wirdum, "Miniscript: How blockstream engineers are making Bitcoin programming easy(er)," August 20 (2019), available at: https://bitcoinmagazine.com/technical/miniscript-how-blockstream-engineers-are-making-bitcoin-programming-easyer.  

<a name="footnote3">3</a> Nick Szabo, “Smart contracts: Building blocks for digital markets”, 1996. 

<a name="footnote4">4</a>. BIP-112, BtcDrak, Mark Friedenbach, and Eric Lombrozo, “CHECKSEQUENCEVERIFY”, August 10 (2015), available at https://en.bitcoin.it/wiki/BIP_0112.

<a name="footnote5">5</a>. Stefano Bistarelli, Ivan Mercanti, and Francesco Santini, "An analysis of non-standard transactions," *Frontiers in Blockchain* (August 13, 2019), available at https://www.frontiersin.org/articles/10.3389/fbloc.2019.00007/full. 

<a name="footnote6">6</a>. BIP-11, Gavin Andresen, “M-of-N standard transactions”, October 18 (2011), available at https://en.bitcoin.it/wiki/BIP_0011.

<a name="footnote7">7</a>. BIP-16, Gavin Andresen, “Pay to script hash”, January 3 (2012), available at https://en.bitcoin.it/wiki/BIP_0016.

<a name="footnote8">8</a>. The Bech32 address standard is described in Pieter Wuille and Greg Maxwell, BIP-173, “Base32 address format for native v0-16 witness outputs”, available at https://en.bitcoin.it/wiki/BIP_0173.

<a name="footnote9">9</a>. Details about segwit transaction serialization and network messaging are offered in **BIP-144**. Changes to the hashing algorithm for signatures are discussed in **BIP-143**. All the other main details of segwit transactions are found in **BIP-141**. BIP-141, “Segregated witness (consensus layer)”, December 21 (2015), available at https://en.bitcoin.it/wiki/BIP_0141. BIP-143, Johnson Lau and Pieter Wuille, "Transaction signature verification for version 0 witness program," January 3 (2016), available at https://en.bitcoin.it/wiki/BIP_0143. BIP-144, “Segregated witness (peer services)”, January 8 (2016), available at https://en.bitcoin.it/wiki/BIP_0144. 

<a name="footnote10">10</a>. Bitcoin Core, "Segregated witness benefits," January 26 (2016), available at https://bitcoincore.org/en/2016/01/26/segwit-benefits/. 

<a name="footnote11">11</a>. See BIP-66, Pieter Wuille, "Strict DER Signatures," January 10 (2015), available at https://en.bitcoin.it/wiki/BIP_0066. 

<a name="footnote12">12</a>. See Bitcoin pull request no. 6769, Gregory Maxwell, "Test lowS in standardness, removes nuisance malleability vector," merged on October 7, 2015, available at https://github.com/bitcoin/bitcoin/pull/6769. 

<a name="footnote13">13</a>. Nermin Hajdarbegovic, "Price drops as Mt. Gox blames Bitcoin flaw for withdrawal delays," *Coindesk*, February 14 (2014), available at https://www.coindesk.com/markets/2014/02/10/price-drops-as-mt-gox-blames-bitcoin-flaw-for-withdrawal-delays/.  

<a name="footnote14">14</a>. See, for example, Alex Hern, "How a bug in Bitcoin led to Mt GOX's collapse," February 27 (2014), available at https://www.theguardian.com/technology/2014/feb/27/how-does-a-bug-in-bitcoin-lead-to-mtgoxs-collapse.

<a name="footnote15">15</a>. Christian Decker and Roger Wattenhofer, "Bitcoin transaction malleability and Mt Gox," *European Symposium on Research in Computer Security* (2014), pp. 313-326.  

<a name="footnote16">16</a>. See, e.g., Luke Dash Jr., "What Bitcoin Did," May 23 (2019), available at https://www.whatbitcoindid.com/podcast/luke-dashjr-on-300k-blocks-and-full-nodes. 

<a name="footnote17">17</a>. A great overview of this Bitcoin civil war is given by Jonathan Bier, *The Blocksize War: The battle for control over Bitcoin's protocol rules*, Amazon Fulfillment (Wroclaw, Poland: 2021). 

<a name="footnote18">18</a>. Gregory Maxwell, Andrew Poelstra, Yannick Seurin, and Pieter Wuille, “Simple Schnorr multi-signatures with application to Bitcoin”, May 20 (2018), available at https://eprint.iacr.org/2018/068.pdf. 

<a name="footnote19">19</a>. Pieter Wuille, Jonas Nick, and Tim Ruffing, "Schnorr signatures for secp256k1", January 19 (2020), available at https://en.bitcoin.it/wiki/BIP_0340.

<a name="footnote20">21</a>. Pieter Wuille, Jonas Nick, and Anthony Towns, "Validation of taproot scripts", January 19 (2020), available at https://en.bitcoin.it/wiki/BIP_0342.

<a name="footnote22">22</a>. The term was baptized by Gregory Maxwell, “Taproot: Privacy preserving switchable scripting”, January 23 (2018), Bitcoin-dev mailing list, available at https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-January/015614.html. I was unable to find a post in which he clarifies the name further. 

<a name="footnote23">23</a>. See Marc Friedenbach, “Why I’m against taproot”, March 15 (2021), available at https://freicoin.substack.com/p/why-im-against-taproot.
