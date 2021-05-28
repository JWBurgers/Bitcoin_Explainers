# Bitcoin Script and Transaction Parsing

This blurb provides an overview of Bitcoin’s scripting language as well as the creation and parsing of transactions. 

For further references, I would recommend you consult Jimmy Sung’s *Programming Bitcoin* or Andreas Antonopoulos’ *Mastering Bitcoin*. In addition, you should look into the **Bitcoin Improvement Proposals**, or BIPs, which I cite throughout the discussion. 

Lets start with an overview of Bitcoin’s scripting language. 


## Script

A Bitcoin transaction unlocks one or more unspent outputs from previous transactions. After deducting a transaction fee, the transaction distributes the remaining value among one or more outputs. Each of these outputs, in turn, also have their own locks that must be unlocked for any subsequent transactions. 

The unspent outputs that are used as inputs in Bitcoin transactions are more specifically known as **unspent transaction outputs** (UTXOs). The locking and unlocking code for Bitcoin UTXOs is managed by a special programming language called **Script**. 

Let’s first understand this language a little better by delving into the name. 

The term **script** typically refers to a small program that is interpreted at run-time (as opposed to a program that is compiled into binaries before execution). This is, for example, what people mean when they speak of Python scripts, Bash scripts, or shell scripts. Furthermore, languages that rely on interpreters such as Perl, Python or JavaScript, are frequently called **scripting languages**.

So to be more exact, any Bitcoin UTXO is locked via a script. To use it, you must provide the right unlocking script in the transaction. These unlocking and locking scripts are run together. The transaction is legitimate according to the Bitcoin protocol if the combined script validates and the transaction meets certain other conditions. If the combined script does not validate, the transaction is invalid.   

Somewhat confusingly, the scripting language used for creating locking and unlocking scripts in Bitcoin is called **Script**. It is similar to an old programming language called **Forth**.

Script is a **stack-based programming language**. This means that any Script program—or “Script script” if you will—pushes data items onto a stack, which can only be operated on from the top of the stack downwards.   

The core component of the Script language are its **operators** (or “opcodes”). All these operators have 1 byte encodings, so the language allows for a maximum of 256 operations. Script operators are represented by hexadecimal bytes in raw transaction data.

Some bytes have never been defined. Others were defined at some point but have now been removed from the accepted Bitcoin protocol. Current versions of full node clients, most importantly Bitcoin Core, will not support these operations either during transaction validation or block validation.  

Removed opcodes are often referred to as “disabled opcodes”. But this is a bit of a misnomer. Such opcodes simply do not exist anymore in Bitcoin Core and other implementations of the Bitcoin protocol. 

Script generally consists of two types of operators: those that push elements onto the stack and those that interact with the stack. 

Expressed in hexadecimal format, the operators from 0x01 through 0x4b each push a certain number of equivalent bytes onto the stack. This can, thus, range from 1 to 75 bytes. For instance, the hex value 0x2a indicates to the software that 42 bytes of subsequent data have to be pushed onto the stack. 

Additional operators allow for pushing data with more than 75 bytes onto the stack. You can also push integer values that range from 1 through 16 onto the stack (with operations 0x51 through 0x60). The operator 0x00 pushes an empty array onto the stack. 

When a node process a transaction (either during initial relay or as part of a block with a valid proof of work), the corresponding locking and unlocking scripts for unspent transaction outputs are combined into a single script. 

During execution of the script, each byte is interpreted by Script as an operation until it hits an operation that requires it to push some subsequent data to the stack (typically an operator from 0x01 through 0x4b). In those cases, the subsequent bytes are simply pushed onto the stack, and Script will not interpret any of them as operators. The first byte following the data pushing operation will be interpreted as an operator again by Script.  

Besides operators that push elements onto the stack, most of the other operators in Script are designed to interact with the stack. These operators might, for example, make a copy of the top element and push the copy onto the top of the stack (known as OP_DUP); or pop the top two elements off the top of the stack, see if they form a valid ECDSA public key and signature pair, and push either an empty array (if invalid) or a 1 (if valid) back onto the stack (known as OP_CHECKSIG). 

Transaction validity depends on various other conditions besides script validation: the transaction needs to be properly formatted, equal or less value captured by the outputs than the inputs, and so on. Suppose for the moment that a transaction has met all these other conditions. 

In that case, if the script of any UTXO input fails to validate, the entire transaction fails to validate. If that transaction is in the initial relay phase, a node will just ignore it. If that transaction was included in a new block, the block will be rejected by the node. 

Instead, suppose that all the scripts of the transaction UTXOs indeed validate. Such a transaction would always be considered valid during the processing of a new block. This is not the case, however, when it comes to the initial relay phase. Not only do all input scripts have to be validated, all inputs and outputs also need to have a certain standard format. 

We can make a broad distinction between two types of outputs in Bitcoin: legacy and segwit outputs. The details of the difference will be discussed later. 

For now, you should know that the data for scripts that lock and unlock legacy outputs are found in transaction data components labeled as **ScriptPubKey** and **ScriptSig** respectively. With native segwit outputs, the unlocking script is at least partially provided in the **witness** component of the transaction.   

The ScriptPubKey and ScriptSig naming conventions are somewhat misleading. They exist because original Bitcoin transactions mostly involved UTXOs locked to a public key that required a signature to unlock. You should be aware that locking and unlocking scripts for both legacy and segwit outputs can encompass a fairly wide variety of conditions. 

As already emphasized, a UTXO can only be the legitimate input into a new Bitcoin transaction if the right unlocking script is offered. This state of affairs cannot be overridden by a central authority and is independent of the personal identities involved in the transaction. It is typical to call the locks on Bitcoin outputs **smart contracts**.<sup>[1](#footnote1)</sup> Script is, in other words, Bitcoin’s **smart contracting language**. 

Bitcoin transaction outputs are only smart contracts because of the decentralized character of the network. In many “blockchain” networks history can more easily be rewritten by a central authority or manipulated by rogue miners, making the term “smart contracts” really an inappropriate way to describe the locks on the outputs of transactions in those networks. 

Bitcoin’s smart contracting language is designed to be simple. It minimizes the risk of errors entering into scripts as well as the risk of attacks on the network being executed via malicious scripts. It also ensures that Bitcoin scripts have a predictable and efficient execution time. This is at least one key design feature that allows even ordinary users to run full nodes. 

Script does have drawbacks. It is difficult to ensure the security and efficiency of any non-standard contract. Putting support for particular types of transactions into software is also a lot of work. All of this makes it difficult for software to support users creating their own types of non-standard transactions. Various projects are currently under way to resolve some of these issues, including miniscript.<sup>[2](#footnote2)</sup>


## Legacy transactions

Before segregated witness, the main output types in Bitcoin included pay to public key, bare multisignature, pay to public key hash, pay to script hash, and OP_RETURN outputs. Any transactions limited to such UTXOs as inputs—other than OP_RETURN outputs which cannot be used as inputs—has the **legacy transaction** format. It does not matter if the transaction has segwit outputs. 

The legacy transaction format consists of the following six main elements: 

1. **Version number** (4 bytes). Typically the version number is 1, except for a certain type of transaction defined by BIP-112.<sup>[3](#footnote3)</sup>
2. **Input count** (variable bytes). Typically 1 byte representing between 1 and 252 inputs. For transactions with more inputs, there is a system for increasing the byte size of the input count by specifying 253 (followed by 2 bytes of space to list the number of transaction inputs), 254 (4 bytes), and 255 (8 bytes). 
3. **Inputs** (variable bytes). Each input is listed in series. The order does not matter. Each input has the following data:
    - **Transaction ID** (32 bytes). This ID is a double SHA256 hash of the raw transaction data in which the UTXO is located.
    - **Index** (4 bytes). This indicates which UTXO of the referenced transaction you wish to use in the current transaction. This index is also known as the vector output or “vout” in shorthand. 
    - **Length of the ScriptSig** (variable bytes).
    - **ScriptSig** (variable bytes). The data necessary to unlock the referenced input.  
    - **Sequence number** (4 bytes).
4. **Output count** (variable bytes). This variable works the same way as for the input count field.
5. **Outputs**. Each output is listed in series. The order does not matter. Each output has the following data:
    - **Value** (8 bytes). The value of the output specified in Satoshis. 
    - **Length of the ScriptPubKey** (variable bytes). 
    - **ScriptPubKey** (variable bytes). The data for locking the output. 
6. **Locktime** (4 bytes). 

The sequence and locktime fields can be used to create time-based smart contracts. I will not discuss these here. The other information in the transaction should be clear from the descriptions above. 

This gives you an overview of how locking and unlocking scripts are organized in a legacy transaction. This all works a little different for segwit transactions—again, any transaction with at least one segwit input—but I will turn to that format at a later point. 

I will now first delve into the main locking and unlocking scripts for non-segwit outputs. These are as follows:

* Pay to public key
* Bare multisignature
* Pay to public key hash
* Pay to script hash
* OP_RETURN


## P2PK outputs

A bitcoin output which is locked to a public key directly in the ScriptPubKey is known as a **Pay-to-public-key output** (P2PK output). 

P2PK outputs are generally supported by nodes as a standard transaction type on the Bitcoin Network. If you send a transaction with a P2PK output, most nodes will validate, store, and relay the transaction. They will also accept any such transaction in a block with a valid proof of work.

That said, I do not know many wallets that support receiving and sending bitcoins via the P2PK standard. You would have to handle this manually via the Bitcoin Core software (GUI console or the command line interface), or via some other manual method. All this means that a transaction with a P2PK output is nowadays certainly a rare occurrence. 

The locking and unlocking scripts for a P2PK output are respectively placed in the ScriptPubKey and ScriptSig data components of the transaction. These data components are explained below. 

*ScriptPubKey*

1. OP_PUSHBYTES_N
2. \<SEC public key> of N bytes. The ECDSA public key is formatted according to the **SEC standard**. An uncompressed public key starts with 0x04 and is then followed by the x and y coordinate of the ECDSA public key. A compressed public key starts with a 0x02 (even y-value) or 0x3 (odd y-value) followed by the x coordinate of the public key.
3. OP_CHECKSIG 

*ScriptSig*

1. OP_PUSHBYTES_M
2. \<DER signature> of M bytes. The ECDSA signature is formatted according to the **DER standard**. The format is as follows:
    - Initial byte (0x30)
    - The length of the rest of the DER signature (1 byte)
    - A marker byte for the r value (0x02)
    - Length of the r value (1 byte)
    - The value of r (variable bytes)
    - A marker byte for the s value (0x02)
    - The length of the s value (1 byte)
    - The value of s (variable bytes)
    - A sighash byte. This indicates which part of the transaction data is hashed in order to create the signature. 


### Script execution

The combined script must validate for the P2PK transaction to be accepted by nodes during relay and in a block with a valid proof of work (as well as a number of other transaction conditions).  

In order to understand how the combined script for a P2PK UTXO executes in the validation process, we first need to understand how the ScriptSig data component is created. 

Creating the ScriptSig is a bit complex. In essence, you start with a dummy transaction where the ScriptSig is empty. You, then, manipulate the dummy transaction as described below in order to end up with a valid Bitcoin transaction. 

1. The signer inserts the ScriptPubKey of the P2PK UTXO into the ScriptSig. In addition, the signer places in the length of the ScriptSig data component, the length of the UTXO’s ScriptPubKey. In case there were multiple inputs to a transaction, the signer would perform the same operation for every nonsegwit input before moving on to step (2). Segwit UTXOs are instead just identified by a 0x00 in the length of the ScriptSig.  
2. The signer then chooses what data to hash from the dummy transaction by appending a **sighash code**. Typically this sighash code indicates all the data in the dummy transaction is hashed, though the signer does have some other options.
3. The signer then creates a hash over certain data in the dummy transaction—typically all—as specified by the sighash code. 
4. The signer then creates a DER ECDSA signature over this hash value. 
5. The signer then removes the ScriptPubKey information from the dummy transaction’s ScriptSig field and populates it with a pushbyte operation and the DER signature just created. The length of ScriptSig field replaces the length of the UTXO’s ScriptPubKey.
6. As the sighash code is put into the DER signature (in the last field), it is stripped from the end of the dummy transaction.
7. The dummy transaction has now been converted into a legitimate transaction acceptable to the Bitcoin network. 

A node will perform many operations on transactions to assure their validity before placing them into their mempool and relaying them to peers before script validation. That includes verifying that all the inputs referenced in the transaction are unspent, and that the total satoshi value of the outputs is equal or less than that of the inputs. 

After executing these and some other checks, the node will turn to script validation. The node will first search for the UTXOs referenced by each of the inputs via the transaction ID and output index. It will fetch the ScriptPubKey belonging to each of these unspent outputs.
 
For each input, the node will then use Script to execute the ScriptSig specified in the current transaction followed by the ScriptPubKey of the UTXO. Above these data components have been listed in the order in which they occur in a transaction from the top down. Executing the ScriptSig and ScriptPubKey with the order specifications provided above, has the following result: 

1. An M-byte signature is pushed onto the stack (ScriptSig)
2. An N-byte public key is pushed onto the stack (ScriptPubKey)
3. The OP_CHECKSIG operation pops the top two elements from the stack. It understands the first as an SEC public key and the second as a DER signature. It then verifies that the signature is valid over a hash of the transaction data (according to the standard defined in the DER signature). It pushes a 1 onto the stack if the signature is valid. Otherwise, it pushes an empty array so that the script becomes invalid.   


### Signature malleability

The transaction ID of a Bitcoin transaction is a double SHA256 hash over all the data. This includes the ScriptSig data. It turns out that any P2PK UTXO input can have its ScriptSig constructed in multiple ways with the same meaning—that is, the ScriptSig is **malleable**. This, in turn, means that the transaction ID is also malleable. 

The same goes for any transaction with bare multisignature, P2PKH, and P2SH inputs. (As an OP_RETURN output can never be spent, it can never serve as a valid UTXO to a transaction. Hence, ScriptSig and transaction ID malleability cannot be a concern with an OP_RETURN UTXO.) 

Transaction ID malleability has a number of downsides. It is particularly detrimental to creating payment channels for the Lightning Network, as these involve referencing transactions that are not yet registered on the Block Chain. 

While some malleability patches were introduced earlier, the major reason for the upgrade to the segregated witness transaction type in the Bitcoin protocol was to fix any remaining malleability issues. This includes the fact that any valid ECDSA signature pair (r, s) also has (r, -s) as a valid counterpart. 

Segregated witness addresses malleability by moving ScriptSig data to a new data structure in the transaction, called the **witness**, which is not used in calculating the transaction ID. Hence, the name “segregated witness”. 

We will introduce the segwit transaction format after finishing our discussion of non-segwit outputs. 


### Bare Multisignature outputs

Multisignature locks in Bitcoin are typically made via P2SH outputs, or P2WSH outputs. These will be discussed later. 

In this section, I will look at the earliest implementation of the multisignature functionality in Bitcoin. The outputs are sometimes called **bare multisignature outputs**. These outputs were supported already in the original version of Bitcoin Core and standardized by **BIP 11**.<sup>[4](#footnote4)</sup>

Sending or receiving bitcoin via bare multisignature outputs is not typically possible with standard wallet software. But you can still create and send to such outputs via Bitcoin Core’s console or command line interface.
 
The bare multisignature functionality within Bitcoin only supports up to a 3 out of 3 multisignature output. For illustrative purposes, I will show a ScriptPubKey and a ScriptSig associated with a 2 out of 3 bare multisignature output. 

*ScriptPubKey*

* OP_2
* OP_PUSHBYTES_N
* <SEC public key 1> of N bytes
* OP_PUSHBYTES_M
* <SEC public key 2> of M bytes
* OP_PUSHBYTES_L
* <SEC public key 3> of L bytes
* OP_3
* OP_CHECKMULTISIG 

*ScriptSig*

* OP_0
* OP_PUSHBYTES_X
* <DER signature 1> of X bytes
* OP_PUSHBYTES_Y
* <DER signature 2> of Y bytes


### Script execution

The ScriptSig field for a 2 out of 3 multisignature input is constructed in a similar manner as for a P2PK output. It follows these steps: 

* Create a dummy transaction with an empty ScriptSig field. 
* Enter the ScriptPubKey of the previous transaction output into the ScriptSig field. Indicate the length of the ScriptPubKey in the ScriptSig length field.    
* Choose what data to hash by appending a sighash code to the dummy transaction. 
* Create two ECDSA signatures over the hash belonging to two different public keys identified in the ScriptPubKey. 
* Place the appropriate data in the ScriptSig field and the ScriptSig length field. This includes the pushing of the <0> in the ScriptSig fields due to a bug in the multisignature implementation (to be explained shortly). Also strip the sighash at the end of the dummy transaction which enters into the ScriptSig instead. 
* The dummy transaction has now been transformed into a valid Bitcoin transaction. 

Any node verifies a 2 out of 3 bare multisignature input in the following manner:

* A 0 is first pushed onto the stack (ScriptSig). This is necessary due to an error in the CHECKMULTISIG operator which consumes an additional element over the number of signatures specified in the contract. Without the extra 0, the script would invalidate. 
* The DER Signature 1 is pushed onto the stack (ScriptSig)
* The DER Signature 2 is pushed onto the stack (ScriptSig)
* The value 2 is pushed onto the stack (ScriptPubKey)
* The SEC public key 1 is pushed onto the stack (ScriptPubKey)
* The SEC public key 2 is pushed onto the stack (ScriptPubKey)
* The SEC public key 3 is pushed onto the stack (ScriptPubKey)
* A value of 3 is pushed onto the stack (ScriptPubKey)
* The OP_CHECKMULTISIG operation checks whether the two signatures each belong to one of the public keys (ScriptPubKey). If so, the script validates.  


## P2PKH outputs

A pay to public key hash output locks some bitcoin to, as the name implies, the hash of a public key. Unlocking such UTXOs requires providing the public key that was used to make the hash value as well as a valid signature. 

P2PKH outputs are still supported by many wallets, though its native and nested segwit counterparts are really becoming the standard as these are more secure and efficient. The scripts in transactions involving P2PKH outputs are as follows:

*ScriptPubKey*

* OP_DUP
* OP_HASH160
* OP_PUSHBYTES_20
* \<SEC Public key hash> of 20 bytes
* OP_EQUALVERIFY
* OP_CHECKSIG 

*ScriptSig*

* OP_PUSHBYTES_N
* \<DER Signature> of N bytes
* OP_PUSHBYTES_M
* \<SEC Public key> of M bytes


### Script execution

The ScriptSig field is created in a similar manner as with P2PK and bare multisignature transactions. The combined script evaluates as follows: 

* The DER signature is pushed onto the stack (ScriptSig).
* The SEC public key is pushed onto the stack (ScriptSig).
* A copy is made of the SEC public key at the top of the stack, and this copy is pushed to the top of the stack (ScriptPubKey).
* The copy of the SEC public key at the top of the stack is popped, a **HASH160** is made of it—that is, SHA256\[RIPEMD160\[public key]]—and the result is pushed back onto the stack.
* The HASH160 of the public key as specified in the ScriptPubKey is pushed onto the stack (ScriptPubKey).
* The OP_EQUALVERIFY function pops the two SHA160 hashes off the stack and sees whether they are equal. If so, the stack continues execution. If not, the stack stops processing immediately. 
* The OP_CHECKSIG operation pops the public key and signature from the stack and verifies that the signature is valid. 


## P2PKH addresses

For any payment to a P2PK lock, you would just have to hand someone the ScriptPubKey (or at least the public key). In addition to making the public key shorter via a SHA160 hash with a P2PKH output, the way this hash is typically shared is via a Base58 encoding standard. This standard includes manipulation of the SHA160 public key hash as follows:

** Prepend the 160-byte hash with a 0x00 version number indicating a P2PKH address; other types of data encoded by the standard (e.g. private keys) start with a different version number.
** DoubleSHA256 the result and take the resulting first four bytes as the checksum
** Add the checksum to the string from step 1
** Convert the characters from hex into base58

The resulting Bitcoin address will start with a 1. Although people sometimes use the term **Bitcoin address** in different ways, it typically refers to certain public information about UTXO locks represented via a Base58 or Bech32 scheme (a newer encoding scheme I will cover later). 

A P2PKH address has a number of advantages over just sharing the ScriptPubKey or the public key of a P2PK output. 

* It is much shorter, so easier to share. 
* The version byte makes it easier to identify the type of information that is encoded. 
* The checksum ensures that mistakes are less easily made when using the address. Particularly when using compressed public keys with P2PK outputs, you might unintentionally lock the output with the wrong public key. This is because the x and y coordinates cannot be cross-checked by software. 
* Finally, the format adds to security. Even if the ECDSA signature scheme becomes broken in some way (e.g., due to advances in quantum computing), the HASH160 ensures an additional layer of protection. 


## P2SH Outputs

**Pay to script hash** (P2SH) outputs are still commonly implemented by wallets. 

As already noted, nodes in the Bitcoin network have different policies for deciding which transactions to include in their mempool and relay, and which transactions to accept in blocks. 

In terms of inclusion in the mempool and relay, nodes only accept certain standard transactions, such as those involving P2PK, P2PKH, and bare multisignature inputs and outputs. When it comes to transactions within valid blocks, nodes will accept a wider variety of transactions, as long as the script is all valid. The reason for this policy is that non-standard transaction types can clog the memory pool.

A user, however, does not have to be a miner or convince a miner to process a transaction with non-standard script. That is because nodes will also process and relay P2SH outputs. In principle, you can insert any admissible locking script into such an output. 

While most wallets will not support the creation of non-standard locking scripts, a user can create them through the Bitcoin Core console or command line interface. In the future, advances might make it easier for users to create their own non-standard locking scripts more easily.   

As P2SH outputs are most commonly multisignature outputs, I will illustrate how they work via an example 2 out of 3 signature output. 

*ScriptPubKey*

- OP_HASH160
- OP_PUSHBYTES_20
- \<Redeem script hash> of 20 bytes
- OP_EQUAL

*ScriptSig*

- OP_0
- OP_PUSHBYTES_N
- <DER Signature 1> of N bytes
- OP_PUSHBYTES_M
- <DER Signature 2> of M bytes
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

P2SH addresses are recognizable by the fact that they always start with a “3”. The **redeem script** contains the core locking conditions on the UTXO. In general, the ScriptSig of a P2SH output contains the solution to the redeem script and the redeem script (as a data item). 


Script execution

The ScriptSig field for a P2SH output is created somewhat differently than in the transactions that we have seen so far. Below are the steps for a 2 out of 3 multisignature output. 

* Enter the HASH160 of the redeem script from the UTXO into the ScriptSig and not the entire ScriptPubKey. Insert data into the ScriptSig length field accordingly. If there are any other inputs, handle them accordingly before moving to step (2). 
* Choose what data to hash from the transaction by appending the sighash. 
* Create an ECDSA signature with two different private keys belonging to valid public keys over the hash of the transaction data. Place the appropriate data in the ScriptSig removing the previous redeem script hash. Also strip the sighash at the end of the transaction which enters into the ScriptSig instead. Update the ScriptSig length field. 

Given the specifications above, this script will evaluate in the following manner:

* A 0 value is pushed onto the stack (ScriptSig)
* A DER signature 1 is pushed onto the stack (ScriptSig)
* A DER signature 2 is pushed onto the stack (ScriptSig)
* The redeem script is pushed onto the stack as a data element (ScriptSig)
* A HASH160 is made of the redeem script and this is pushed onto the top of the stack (ScriptPubKey)
* The HASH160 of the redeem script in the ScriptPubKey is pushed onto the stack (ScriptPubKey)
* OP_EQUAL pops the top two values of the stack, that is, the HASH160 of the redeem script offered in the ScriptSig and the HASH160 in the ScriptPubKey. If the operation verifies that they are equal, it pushes a 1 onto the stack. Otherwise, it pushes an empty array.  

The P2SH upgrade is described in **BIP-16**.<sup>[5](#footnote5)</sup> At this point, if the stack concludes with a value of 1, what a node does depends on whether they are BIP-16 compliant. 

A pre BIP-16 node would just verify the transaction at this point. BIP-16 compliant nodes, however, would have instead also made a separate stack of the ScriptSig data, with the redeem script deserialized. They know to do this on the basis of the ScriptPubKey pattern associated with the UTXO (namely, OP_HASH160 OP_PUSHBYTES_20 \<redeem script hash>). 

At this point, a BIP-16 node will just start executing the second stack in the same way as a bare multisignature output. If everything checks out, then a 1 is pushed onto the stack by the OP_CHECKMULTISIG operation and the script is valid. 

BIP-16 nodes have stricter validation standards than Pre BIP-16 nodes. This may lead the latter group of nodes to accept certain transactions which are rejected by BIP-16 nodes. This type of upgrade is known as a **soft fork**.

Given that most of the network is BIP-16 compliant, only BIP-16 valid transactions will enter into the Block Chain. That Pre BIP-16 nodes might still accept certain transactions which are not valid for BIP-16 nodes in terms of storage and relay, therefore, has no consequences for ultimate network consensus over the Block Chain.  


## OP_RETURN outputs

An OP_RETURN ScriptPubKey can never be unlocked. No matter what condition you put into the ScriptSig, the script will invalidate as soon as a node sees the RETURN operator. These types of transactions are, therefore, not stored in the UTXO set, but only in the Block Chain. 

The point of this kind of transaction is to lock up to 80 bytes of data onto the Block Chain forever. Various types of higher-level protocols interpret and use the data in these OP_RETURN transactions. OP_RETURN outputs in the coinbase transaction are also used to store the witness commitments for segwit blocks. 

The ScriptPubKey for an OP_RETURN transaction looks as follows:

* OP_RETURN
* An OP_PUSHBYTES or an OP_PUSHDATA operation for N bytes
* \<data> of N bytes


## Segregated Witness Transactions

With all the standard legacy outputs just described as well as the legacy transaction format, you understand how practically all transactions worked before the segregated witness upgrade in 2017. From that point onwards, however, a lot changed. Lets start with the segwit transaction format. 

A legacy transaction can have outputs locked by segwit ScriptPubKeys. All of its inputs, however, must refer to legacy UTXOs. 

As soon as a transaction references any segwit UTXOs for inputs, it becomes a **segwit transaction**. This type of transaction has three additional components over a legacy transaction: the segwit marker, the segwit flag, and the witness. Below you can find the overall format of a segwit transaction.

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
    - A ScriptPubKey for locking the output.
8. **Witness data** (variable bytes). 
9. **Locktime** (4 bytes)

For any segwit transaction, some or all of their locking data is placed into the witness data component. Each segwit input then has a witness field that consists of the following components: 

1. **Number of witness elements** (variable bytes). This indicates the number of elements that will be pushed to the stack for the validation script. 
2. Then for each element of the witness field, there are two pieces of data:
    - **The length of the element** (variable bytes).
    - **The element** (variable bytes). 

For any segwit transaction which also includes legacy UTXOs as inputs, their script data is just placed entirely into the ScriptSig component of the transaction, just as with a legacy transaction. Additionally, a 0x00 is placed in the legacy UTXO’s witness field to indicate it has no witness components. Note that the order of the inputs must reflect the order of the witness fields in a segwit transaction. 

By contrast, for segwit UTXOs some or all of the unlocking data is placed into a witness field. Currently, there are four types of segwit outputs. This includes two native segwit outputs—P2WPKH and P2WSH—and two nested segwit outputs—P2SH-P2WPKH and P2SH-P2WSH. Let’s delve into that distinction a little.  

To use a **native segwit output** in a transaction, the ScriptSig length for that particular UTXO must be set to 0x00. All the unlocking data is instead moved to the witness data component of the transaction. 

To understand nested segwit outputs, we need to look at the addressing standard for native segwit outputs. Importantly, no Base58 encoding standard exists for native segwit outputs. Instead, they can only be encoded with a **Bech32 addressing standard**.<sup>[6](#footnote6)</sup>

Bech32 addresses start with “bc” to indicate Bitcoin mainnet (“tb” for the testnet), followed by a separator of “1”, and then the data encoded by all alphanumeric characters except “0”, “b”, “i", and “o” (so each character can encode 32 bits of information). 

The data here pertains to the 20-byte or 32-byte **witness program**, which is the main component of a ScriptPubKey that belongs to a native segwit output.

The Bech32 address format has several advantages over the Base58 format, including a better error checking mechanism and improved readability. As it is the intention to slowly migrate to Bech32 addresses, the Bitcoin community decided not to make any encoding scheme for native segwit addresses using the Base58 standard. 

Nevertheless, when segwit was first introduced to the network, many wallets did not yet support the sending of bitcoin to Bech32 addresses. In order to still allow these wallets to send money to segwit addresses, developers created a roundabout way of nesting the two native segwit transactions within a P2SH transaction. These are called **nested segwit addresses** and start with a “3”, just as any other P2SH address. 

To use any nested segwit UTXO in a transaction, the redeem script must be placed in the ScriptSig field, while the rest of the unlocking conditions are placed in the witness field. 

The transaction ID of any segwit transaction is calculated in the same way as before: it includes all and only the data that would be included in a legacy transaction. The segwit marker, segwit flag, and witness fields are ignored in the calculation of the transaction ID. 

Hence, if you have a transaction that includes only inputs from segregated witness UTXOs, the transaction ID is no longer malleable, as no malleable ScriptSig data is included in its calculation. 

For each segwit transaction a **witness transaction ID** can also be calculated. This is equal to the double SHA256 over all the data in the segwit transaction. 

When a miner creates a block, it will calculate a merkle root of all the witness transaction IDs, under the assumption that the coinbase transaction has an ID of 0 represented by 32 bytes. This merkle root is placed in an OP_RETURN output in the coinbase transaction with a commitment header. Together these two pieces of data are known as the **witness commitment**.

A witness transaction ID is not a data component of segregated witness transactions in the way that a transaction ID is. Nevertheless, via the witness commitment, this data is indirectly locked into the Block Chain. 

Any node that is compliant with the segwit upgrade, can recognize a segwit transaction and fully validate it. They will relay valid segwit transactions to other segwit nodes. 

A legacy node instead cannot parse a segwit transaction. It can, however, be sent a watered-down version of the segwit transaction data according to the standards of a legacy transaction (i.e., one that excludes the segwit marker, segwit flag, and witness data). 

The segwit standard was created in such a way that even legacy nodes would consider a stripped down version of a valid segwit transaction as valid in a block. As it is a non-standard transaction, however, legacy nodes will refuse to process it if the transaction was simply being relayed. 

The general flow of segwit transactions through the network is as follows:

* Segwit transactions are relayed between segwit nodes for inclusion into the memory pool. Legacy nodes generally do not see these transactions, not even in legacy format, during this phase. 
* Eventually these segwit transactions will be included in a valid block by a miner. This block “must” contain all the data for segwit transactions. It will also include a witness commitment in the coinbase transaction.   
* The valid block gets sent around the network. Any segwit nodes will send a watered-down version of the block to legacy nodes, that is, one which only includes the legacy data for segwit transactions. If a legacy node, in turn, relays the trimmed block to another segwit node, the latter will reject it as the segwit transactions do not include all the unlocking data. 
* In this manner, all segwit nodes store a valid segwit block, while all the legacy nodes will store a valid legacy block. Note that “segwit Block Chains” contain transactions with exactly the same transaction IDs as their legacy counterparts.  

As stated above, a miner “must” send a block with all the data for segwit transactions. I mean here that a miner which does not support segregated witness will be unsuccessful in propagating any blocks, even if these contain a valid proof of work. Segwit nodes will namely not accepted stripped down blocks. As most nodes in the Bitcoin Network are segwit nodes, the most proof of work will always return to a chain with only segwit blocks.  

A key insight here is that, while legacy nodes cannot fully validate segwit transactions, they can track how segwit outputs are moved.    

A key upgrade for segwit nodes was with regards to the size of the blocks accepted by them. Segwit nodes no longer maintain a maximum block size. Instead, they accept a maximum **block weight** of 4,000,000 bytes. This block weight is calculated as follows: *block weight* = 3 * *base size* + *total size*. 

The **base size** in this case is the block size as received by non-segwit nodes using the traditional serialization scheme for transactions. The **total size** is the block size as seen by segwit nodes using the segwit serialization scheme for transactions.

Given that the total size of any segwit block must be greater or equal to its base size, the block weight formula ensures that the base size can at a maximum be 1,000,000 bytes. This occurs in the unlikely event that all the transactions in a block are legacy—so base size equals total size—and the total data is exactly equal to 1,000,000 bytes. The formula, thus, ensures that legacy nodes will always accept the legacy version of valid segwit blocks. 

This all constitutes an increase in the block size of sorts. Segwit nodes just have a higher tolerance for data. Legacy nodes can accept more transactions in their blocks because the size of the ScriptSig data is decreased substantially for segwit inputs. It becomes 0x00 for a native segwit input and only the redeem script data with a pushbyte operation for nested segwit inputs. As the ScriptSig component constitutes a major portion of the size of legacy transactions, this savings in data is significant. 

Details about segwit transaction serialization and network messaging are offered in **BIP-144**.<sup>[7](#footnote7)</sup> All the other main details of segwit transactions are found in **BIP-141**.<sup>[8](#footnote8)</sup>

As stated above, there are two types of native segwit outputs. These are the **pay to witness public key hash** (P2WPKH) and **pay to witness script hash** (P2WSH) outputs. In addition, both of these can also be nested in P2SH transactions. Lets discuss these four formats in turn.


## P2WPKH scripts

Any ScriptPubKey for a transaction output that consists of a 1-byte push operation of a 0 followed by a data push of 2 to 40 bytes is a native segwit transaction. The 0 here is a version number. Currently, it only has meaning if followed by 20 or 32 bytes of data. 

When 20 bytes are pushed, the ScriptPubKey is interpreted as a P2WPKH output. That is, in other words, when the ScriptPubKey looks as follows: 

* OP_0 
* OP_PUSHBYTES_20
* <20-byte hash> 

The 20-byte hash in this format is the SHA160 of the compressed SEC public key, and, as already mentioned above, is called the witness program. 

The ScriptSig length for unlocking this type of output has to be set to 0x00. Instead, all the data should be included in a witness field. The witness field should look as follows:

* 0x02: Indicates the number of witness elements
* X: The length of the signature
* The DER signature
* Y: The length of the public key
* The compressed SEC public key

Note that native segwit transactions only allow compressed SEC public keys. 

On seeing the ScriptPubKey pattern, a segwit node will know how to create a script that must validate for the transaction to be legitimate. More specifically, a segwit node will know to execute the following script given the ScriptPubKey and the witness field:

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

The execution of this script proceeds as with a regular P2PKH output:

* The DER signature is pushed onto the stack (witness).
* The compressed SEC public key is pushed onto the stack (witness). 
* A copy is made of the public key and pushed onto the stack.
* The copy of the public key is put through the HASH160 function. The result is pushed back onto the stack. 
* The 20-byte hash from the witness program is pushed onto the stack (ScriptPubKey).
* The EQUALVERIFY operation pops the top two elements of the stack and checks whether they are equal. If so, execution continues. Otherwise execution halts immediately. 
* The CHECKSIG operation verifies that the signature is valid for the public key. It pushes a value of 1 if valid. Otherwise, it pushes an empty array. 

Note that in contrast to a regular P2PKH transaction, much of the script language is not included in the transaction. Instead, it is understood by segwit nodes to create a script in a certain way on the basis of the pattern in the ScriptPubKey. 

In addition, note that the zero which is pushed by the ScriptPubKey is not an element of the validation script. It is only an indicator with regards to the structure of the validation script. Given that you can also push the values 1 through 16 with Script, pattern interpretation in line with native segwit transactions offers an easy path to upgrades. 

A legacy node will not see any of the witness data, and only executes the script according to legacy rules. This is as follows:

* The ScriptSig is empty, so nothing is pushed to the stack
* A 0 is pushed to the stack (ScriptPubKey)
* The 20-byte public key hash is pushed to the stack (ScriptPubKey)
* As the top element of the stack is not zero, the script will validate


## P2WSH outputs

If a ScriptPubKey for a transaction output consists of a 1-byte push operation of a 0 followed by a data push of 32 bytes, it is interpreted as a pay to witness script hash output. This native segwit output type has its legacy counterpart in P2SH outputs. 

Lets work through an example with a 2 out of 3 multisignature address. In that case, the ScriptPubKey will look as follows:

* OP_0 
* OP_PUSHBYTES_32
* <32-byte redeem script hash> 

Again, the 32-byte hash is called the witness program. The ScriptSig for unlocking this type of output also has to be set to 0x00. Instead, all the data should be included in a witness field as follows: 

- 0x04: Indicates the number of witness elements
- 0x00: Indicates that the first element is OP_0
- X: The length of signature 1
- DER signature 1
- Y: The length of signature 2
- DER signature 2
- L: The length of the witness script
- The witness script. This should include all the following as data:
    - OP_2
    - OP_PUSHBYTES_N
    - <public key 1> of N bytes 
    - OP_PUSHBYTES_M
    - <public key 2> of M bytes 
    - OP_PUSHBYTES_L
    - <public key 3> of L bytes 
    - OP_3
    - OP_CHECKSIG

Note that the witness script is analogous to the redeem script of a P2SH transaction. 

At this point, any legacy node will just execute the following script, as it does not see the witness data:

* Push a 0 onto the stack
* Push a 32-byte value onto the stack

As the top of the stack is not zero or an empty array, the script will validate for a legacy node. Segwit nodes on the other hand know to first check whether the SHA256 of the witness script equals the 32-byte hash provided in the ScriptPubKey. If so, they will next execute the following script for validation:

* OP_0
* OP_PUSHBYTES_X
* \<signature> of X bytes
* OP_PUSHBYTES_Y
* \<signature> of Y bytes
* OP_2
* OP_PUSHBYTES_N
* <public key 1> of N bytes 
* OP_PUSHBYTES_M
* <public key 2> of M bytes 
* OP_PUSHBYTES_L
* <public key 3> of L bytes 
* OP_3
* OP_CHECKSIG

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

Note that by contrast to P2WPKH outputs, all the data for the script is included in the script witness. Given that this output type allows any admissible script, it cannot execute according to any pre-stored template. 


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


## Notes

<a name="footnote1">1</a>. Nick Szabo, “Smart contracts: Building blocks for digital markets”, 1996. 

<a name="footnote2">2</a>. See Blockstream, “Miniscript”, available at: http://bitcoin.sipa.be/miniscript/, and Pieter Wuille and Andrew Poelstra, “Miniscript: Streamlined bitcoin scripting”, September 7 (2019), available at: https://medium.com/blockstream/miniscript-bitcoin-scripting-3aeff3853620. 

<a name="footnote3">3</a>. BIP-112, BtcDrak, Mark Friedenbach, and Eric Lombrozo, “CHECKSEQUENCEVERIFY”, August 10 (2015), available at https://en.bitcoin.it/wiki/BIP_0112.

<a name="footnote4">4</a>. BIP-11, Gavin Andresen, “M-of-N standard transactions”, October 18 (2011), available at https://en.bitcoin.it/wiki/BIP_0011.

<a name="footnote5">5</a>. BIP-16, Gavin Andresen, “Pay to script hash”, January 3 (2012), available at https://en.bitcoin.it/wiki/BIP_0016.

<a name="footnote6">6</a>. The Bech32 address standard is described in Pieter Wuille and Greg Maxwell, BIP-173, “Base32 address format for native v0-16 witness outputs”, available at https://en.bitcoin.it/wiki/BIP_0173.

<a name="footnote7">7</a>. BIP-144, “Segregated witness (peer services)”, January 8 (2016), available at https://en.bitcoin.it/wiki/BIP_0144. 

<a name="footnote8">8</a>. BIP-141, “Segregated witness (consensus layer)”, December 21 (2015), available at https://en.bitcoin.it/wiki/BIP_0141.