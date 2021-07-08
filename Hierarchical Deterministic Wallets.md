## Hierarchical Deterministic Wallets

In this explainer, I want to lay out the basics of how standard hardware wallets work. Specifically, I will discuss the following topics:

* How mnemonic words and the master seed are created via the BIP-39 standard
* How a single master seed can produce many different Bitcoin private and public keys via the BIP-32 standard.
* How legacy, P2SH, native segwit, and multisig addresses are created in hardware wal-lets using industry standards.  

Hardware wallets are not rocket science. Knowing the details of how they work will help you keep your funds save, and also provide a basis for exploring more advanced techniques (includ-ing their pitfals) such as multisignature wallets.


## A note on terminology

There is a lot of terminology that you should know when working with Bitcoin, particularly when it comes to hardware wallets. I’ll introduce that terminology as we move along. Here, I just want to start with a short note on the term “wallet”. 

The term “wallet” can have at least two distinct meanings in the Bitcoin ecosystem. In a broad sense, it can refer to a hardware wallet such as a Trezor or a Cold Card, or a software applica-tion such as Specter, Sparrow, or Electrum. In a more narrow sense, these hardware devices and software applications actually have the ability to support the use of multiple bitcoin “wal-lets”. 

A wallet in this more narrow sense refers to certain collections of data, the main functions of which are to help you create receive addresses, track your funds, and potentially sign transac-tions (depending on the type of wallet). 

In this explainer, I will use the term wallet in both the broad and the narrow sense. The meaning intended should usually be clear from the context. I will clarify when needed. 



## BIP-39 mnemonic words

If you have ever used a common hardware wallet such as a Trezor, Cold Card, or Cobo Vault, you will know that making a proper backup requires storing a series of 12 to 24 **mnemonic words**. This is a good place to start for weaving through the background information necessary to understand hardware wallets. 

After the initial set up of a hardware wallet, the mnemonic words you were forced to write down will also be stored on the device (typically in a **secure element**). Each time you start up the hardware wallet, you can, then, load a wallet (in the narrow sense) by entering a password (including a blank password). Note that your password is a 25th mnemonic word not stored on your device, and does not refer to your pin code. 

In practice, each unique password loads a unique and separate wallet onto the device.<sup>[1](#footnote1)</sup> Hence, hardware wallets can really support a massive number of wallets (in the narrow sense).

In case the hardware wallet is ever lost or destroyed, you can easily recover the funds on a different device as long as you have the mnemonic words and any passwords you used. 

The most common way of creating a set of mnemonic words is described by **BIP-39**.<sup>[2](#footnote2)</sup> The steps are relatively straightforward.

1. Select a random number **R**. The size of **R** can be 128, 160, 192, 224, or 256 bits. Note that the acceptable sizes come in 32-bit intervals.  
2. Take the length of the random number **R** and divide it by 32. This is the checksum length **N**.
3. Calculate SHA256 (**R**). The first **N** bits of the hash value constitute the checksum **C**. 
4. Take the original random number **R** and append the checksum **C**.  
5. Divide the result into equally-sized strings of 11 bits. Convert each string into its decimal value. The decimal values can range from 0 to 2047.  
6. Each decimal value corresponds to one of 2048 words in the **BIP-39** dictionary. The dictionary comes in a few languages such as English, Chinese, Japanese, French, and Italian. Convert all the decimal values into words using the dictionary. 
7. The result is a list of mnemonic words. Depending on the size of R, the result is 12 words (128 bits), 15 words (160 bits), 18 words (192 bits), 21 words (224 bits), or 24 words (256 bits).    

The steps above are fully automated in most hardware wallets. Typically, if you want to create a new set of mnemonic words, the hardware device will first utilize its own random number generator to produce **R**. It then proceeds through all the necessary steps above to give you a new list of mnemonic words. 

Though tedious, you do not need the hardware wallet to create a list of mnemonic words. You can also create them manually by dice rolls, coin flips, or some other suitably random process. The words can, then, be imported into the hardware wallet. If executed properly, the manual process offers much stronger assurance of entropy. 

Cold Card offers a more user-friendly solution to these two alternatives. Instead of just creating a list of mnemonic words for you, it can also create them on the basis of dice roll data that you enter. As long as you enter a sufficient number of dice rolls and performed the rolls correctly, this offers an improvement over purely relying on the random number generator on the device.  

Most modern wallet software (e.g., Electrum, Specter, and Sparrow) allows you to manage the content of multiple hardware wallets from their interface. For each of these wallets, you will need a different combination of mnemonic words and one or more passwords as part of a back up.   

The list of mnemonic words is used to produce the **master seed** (also known as the “BIP-39 seed” or simply the “seed”) as follows: 

1. Concatenate all the mnemonic words into one long string.
2. Create a SALT. This salt—that is, a string of random bits—is the string “mnemonic + \[passphrase]\”. If you do not enter a passphrase, the SALT will simply be “mnemonic”. 
3. Combine the long string of mnemonic words and the SALT. 
4. Use the Password Based Key Derrivation Function 2 (PBKDF2) on the mnemonic string + SALT to produce a 512 bit master seed. 

The iteration count in PBKDF2 is set at 2048. This feature ensures that brute forcing a password on the basis of a captured set of mnemonic words is practically impossible, at least if your password is sufficiently random. A short, but unusual sentence should be sufficient in this regard. 

Creating a master private key from a master seed is a relatively easy process which we will discuss in the next section. Given that this latter process is not calculation-intensive, it should be clear why the master seed is not stored directly on hardware wallets. By only storing the mnemonic words on the device, the use of a sufficiently random password ensures that brute forcing a master seed—and, ipso facto, the master private key—is practically impossible. So even if someone can log into the device or extract the words in some way, your funds should still be safe with a secure password. 

BIP-39 was first implemented in the Trezor line of hardware wallets and is now the generally accepted standard for creating mnemonic words. As far as I know, each major consumer hardware wallet employs this standard. 

That said, the BIP-39 standard is not ubiquitous. Notably, Electrum allows you to create wallets directly within the software based on mnemonic words that are derrived in a different manner.<sup>[3](#footnote3)</sup>


## Hierarchical Deterministic Wallets

A master seed, typically created via the BIP-39 standard, forms the basis of a hierarchical deterministic wallet (HD wallet). 

A **deterministic wallet** is simply any wallet where all the private keys, public keys, and addresses belonging to the wallet are created on the basis of a single master seed. 

While almost all modern software and hardware wallets use deterministic wallets (at least by default), this was not always the case. Traditionally, wallets were just a bunch of random private and public key pairs. The possession of one key pair did not give any (mathematical) insight into the other keypairs. 

These **just-a-bunch-of-keys wallets** are generally not supported anymore. Bitcoin Core does still support them, but the default wallet has been deterministic for a few years already.  

Deterministic wallets are the industry standard nowadays for a number of reasons. At least one important reason is that they make backing up a wallet much more secure for the average user. If you are using a hardware wallet, all you really need are your mnemonic words and, if you use it, a password to recover a wallet. 

By contrast, for the traditional just-a-bunch-of-keys-wallets, you needed every single relevant private key to restore your funds. As private keys were created in batches when needed, you would frequently have to ensure that all your keys were properly backed up.    

Practically any modern deterministic wallet is more specifically an **HD wallet**. This means that all the keys and addresses associated with your wallet are hierarchically organized.  

All major software and hardware wallets, by and large, follow a common set of industry standards for HD wallets. The general framework for these standards is defined by **BIP-32**.<sup>[4](#footnote4)</sup> The further details of HD wallets are defined primarily by BIP-43<sup>[5](#footnote5)</sup> and BIP-44.<sup>[6](#footnote6)</sup> 


## BIP-32

To create a BIP-32 HD wallet, the master seed needs to be transformed into a master node. This done as follows: 

* You take the (BIP-39) master seed and insert it into the HMAC-SHA512 function. 
* The resulting hash is 512 bits in size. The first 256 bits are the **master private key**. The last 256 bits are the **master chain code**. 
* From the master private key you can derrive the **master public key** in the same way you would derrive any Bitcoin public key from a private key.  
* These three items—the master private key, the master public key, and the master chain code—are called the **master node**. 

An **extended private key** is just any Bitcoin private key that is accompanied by chain code. An **extended public key** is just any Bitcoin public key that is accompanied by chain code. So the master private key and the master chain code together are called the **master extended private key**. The master public key and the master chain code together are called the **master extended public key**. 

The master node forms the root of the BIP-32 HD wallet. From it, you can derrive two types of child nodes and further descendants to produce a tree-like structure. For producing a normal child node from the master node, the process is as follows with the BIP-32 standard:  

* You determine an index number. For normal child nodes, valid index numbers run from 0 to 2<sup>31 – 1</sup>. Any wallet typically starts with 0 and then works its way upwards.
* Take the index number, the master chain code, and the *master public key*, and plug them into the HMAC-SHA512 function. This results in a hash value of 512 bits. 
* The last 256 bits of the hash value is the **child chain code**. 
* The first 256 bits is added to the *master private key* modulo 2<sup>256</sup>. The result is the **child private key**. 
* You can now produce the **child public key** in the standard manner from the child private key. Alternatively, you can also produce the child public key by walking through the above steps, but adding the first 256 bits of the hash value to the *master public key* modulo 2<sup>256</sup>. Due to the way Bitcoin’s elliptic curve cryptography works, the private and public key are shifted by the same amount in the transformation process. 
* The resulting child private key, child public key, and child chain code together form a **normal child node**. 

For hardened child nodes, the derrivation process works as follows in BIP-32:

* You determine an index number. For hardened child nodes, these run from 2<sup>31 – 1</sup> to 2<sup>32 - 1</sup>. You typically start with 0 and then work upwards.
* Take the index number, the master chain code, and the **master private key**, and plug them into the HMAC-SHA512 function. This results in a hash value of 512 bits. 
* The right 256 bits of this HASH are the child chain code. 
* The left 256 bits are added to the **master private key** modulo 2<sup>256</sup>. The result is the child private key. 
* From the child private key, you can calculate the child public key in a standard manner. Alternatively, you can add the left 256 bits of the hash value to the *master public key* modulo 2<sup>256</sup> to produce the same result.  
* The result is a child private key, a child public key, and a child chain code. Together these form a **hardened child node**.

If you are working further down the tree in a hierarchical deterministic wallet—so have a different **parent node** than the master node—these derrivation principles stay the same. For instance, to produce a normal child node from a parent, you would proceed as follows: 

* Determine an appropriate index number 
* Insert the index number, the parent public key, and the parent chain code into the HMAC-SHA256 function.
* The last 256 bits of the hash are the child chain code.
* The first 256 bits of the hash are added to the parent private key modulo 2<sup>256</sup> in order to obtain the child private key.
* You can, then, calculate the child public key in either of the two ways discussed above. 

It is crucial to note that a normal child node and a hardened child node at the same index are completely different with regards to the private key, public key, and chain code. Why have these two derrivation procedures?

Consider what you can do with just an extended public key. You know all the inputs to the HMAC-SHA512 function (index, parent public key, and parent chain code) for derriving normal child nodes. Hence, you can derrive any normal child public key just from the parent extended public key. While you cannot create any child private keys with just the parent public key and chain code, you can create receive addresses and monitor them on the Block Chain.

By contrast, you cannot create hardened child public keys by just having the parent extended public key. That is because you need the parent private key for the HMAC-SHA512 function, not the parent public key. 
 
Extended public keys keys are great for importing into wallet software such as Electrum, Specter, or Sparrow. As long as you are only working with normal descendants, you can create a **watch-only wallet**. The latter will allow you to make receiving addresses and track balances. 

But as you will not have the private keys belonging to these addresses, you cannot sign for any transactions. Any transactions you create with the software will require obtaining signatures via private key information elsewhere (typically on a hardware wallet). 

Specter is an example of an application that only supports watch-only wallets coupled to hardware devices. An application like Electrum also supports the creation of **standard wallets**—which contain private key information—in the software directly. 

Unless you need to use an extended public key of a parent node for a watch-only wallet, any children should always be derrived in hardened fashion. This has a number of privacy and security benefits. 

Any leakage of the parent extended public key would not provide the ability to produce any hardened child public keys. This is not the case with normal children, where the extended public key of the parent is enough to produce all child public keys and those of further descendants. While this does not mean you are at risk of losing your bitcoins, it does expose your privacy. 

Additionally, a leakage of a hardened child private key does not expose the private keys of any of the other children or the parent. This is not the case with normal children. If you have the extended public key of the parent and a private key of one of the children, you know exactly the first 256 bits of the hash value needed to produce the child public key. On that basis, you can calculate what the parent private key should be when generating the exposed normal child private key. From there, you can calculate the private keys of all the children.  

The chain code in the BIP-32 standard offers additional protections in the derrivation process. For instance, just having a public key does not allow you to derrive any normal or hardened child public keys with the BIP-32 standard. Any constructions without the chain code, however, run the risk that anyone could create child public keys from the public key alone. 


## BIP-43 and BIP-44

BIP-32 just shows how to create child nodes and further descendants from a master seed. It does not attribute any structural meaning to these descendants. 

The basis of a structural framework that has widespread adoption among Bitcoin hardware and software wallets is provided by **BIP-43** and **BIP-44**. According to these two BIPs, the key path of a single signature receiving address should have the following structure:

* m/purpose’/cointype’/account’/change/index

A ‘ sign after any of these fields indicates that it is a hardened child node. Without a ‘ indicates a normal child node. 

Each of the fields in the standard have the following meaning:

* The purpose field specifies the kind of address (legacy, nested segwit, or native segwit). 
* The cointype specifies for which coin the address is intended. Bitcoin is 0, Bitcoin testnet is 1, and Litecoin is 2. There are many other coins supported by the standard. 
* Accounts are numbered from 0 onwards. Each account has two types of addresses: regular and change addresses.
* In the change field, a 0 indicates a regular address, while a 1 indicates a change address.
* Finally, the index field can be used to sequentially create unique addresses.   

In BIP-44, a purpose field of “44” was defined as a legacy address. In later BIPs, different purposes were specified. A 49 indicates nested segwit addresses (defined in **BIP-49**),<sup>[7](#footnote7)</sup> and 84 indicates native segwit addresses (defined in **BIP-84**).<sup>[8](#footnote8)</sup>

From the discussion in the previous section, it should be clear why all the child nodes in the derrivation path for single signature addresses are hardened except for the change address and index elements. The extended public key at the account level is typically exported from hardware wallets to software wallets such as Specter, Sparrow, and Electrum. As only normal nodes are derrived at the change and index levels, these wallet applications can create create and monitor addresses on the basis of the extended public key at the account level alone.

It also important to note why these standard derrivation paths exist. Suppose that I randomly selected a 256 bit number for the purpose element. On the basis of a master seed that you own, I then deposited funds to an address that was otherwise created according to the standards. 

Even though you possess the master seed, you would have to brute force the purpose path in order to retrieve your funds. This is practically infeasible. Hence, the standardization exists so that you can easily retrieve funds with on the basis of a master seed.


## Multisignature address paths

While single-signature address paths are well-defined within the Bitcoin ecosystem, there are no BIPs that specify the BIP-43/44 paths for multisignature addresses. That said, typically the derrivation paths in hardware and software wallets are the same as those used in the Electrum wallet. These paths are as follows:

* m’/48’/0’/0’/1’/… = Nested segwit addresses for multisignature wallets
* m’/48’/0’/0’/2’/… = Native segwit addresses for multisignature wallets

There are no specific change addresses for multisignature addresses. Generally the next address in the derrivation path should be employed by the coordinating wallet software. 

In principle, you can use an address from one of the other derrivation paths (as long as it is the same type of address) for creating multisignature wallets. For instance, you could use the public address at m’/84’/0’/0’/0/0. But this kind of approach will run into a couple of issues:

* If the address is on a common path such as at m’/84’/0’/0’/0/0, you run the risk of using the address for receipt of single funds as well as multisig funds. That is bad for privacy and security, as you might still be using the address even after exposing the public key to the network in a transaction. If you are using some uncommon path, you might have problems retrieving funds if you ever forget the path. 
* Software wallets might not be compatible with custom derrivation paths. In some wallets like Specter you can indeed include custom paths, but in other wallets you might run into problems.  
* Hardware wallets that support multisig might also potentially experience problems with non-standard derrivation paths. 


## Which Xpub?

You can have very many extended private keys and/or extended public keys in any particular hardware wallet. Typically, however, Bitcoiners talk about the following extended public keys in practice:

* For single signature addresses, the extended public keys at m’/44’/0’/0’, m’/49’/0’/0’, or m’/84’/0’/0’. These extended public keys can be incorporated into software wallets to generate public addresses for receiving funds and making change. The native segwit standard is growing for single signature transactions, so most typically the relevant xpub is at m’/84’/0’/0’ for single signature addresses. 
* For multisignature addresses, the extended public keys at m’/48’/0’/0’/1’ (nested segwit) and m’/48’/0’/0’/2’ (native segwit). Again, with the growing use of native segwit addresses, it is the latter that is most important. 

In Bitcoin, extended public keys are often formatted so as to be easily recognizable. They frequently start in one of three ways:

* **xpub**: An extended public key that is used to create legacy addresses (“1”)
* **ypub**: An extended public key that is used to generate nested segwit addresses (“3”)
* **zpub**: An extended public key that is used to generate native segwit addresses (“bc1”) 

The extended private keys that accompany these public keys are formatted in a similar manner, namely as starting with “xprv”, “yprv”, and “zprv” respectively.

Confusion can arise because some wallets implement a standard where every extended public key starts with xpub, regardless of the type of address for which it used. The same goes for the accompanying private extended key. The standard which differentiates them as explained above is, however, becoming more typical. The standard is captured by **SLIP-132**.


## Notes

<a name="footnote1">1</a>. In theory, two different passwords can produce identical wallets. But the odds of such an event are astronomically small.  

<a name="footnote2">2</a>. Marek Palatinus, Pavol Rusnak, Aaron Voisine, and Sean Bowe, “Mnemonic code generating deterministic keys”, *BIP-39*, September 10 (2013), available at https://en.bitcoin.it/wiki/BIP_0039.  

<a name="footnote3">3</a>. See Thomas Voegtlin, “Electrum seed version system”, available at https://electrum.readthedocs.io/en/latest/seedphrase.html. 

<a name="footnote4">4</a>. Pieter Wuille, “Hierarchical deterministic wallets”, *BIP-32*, February 11 (2012), available at https://en.bitcoin.it/wiki/BIP_0032. 

<a name="footnote5">5</a>. Marek Palatinus and Pavol Rusnak, “Purpose field for deterministic wallets”, *BIP-43*, April 24 (2014), available at https://en.bitcoin.it/wiki/BIP_0043. 

<a name="footnote6">6</a>. Marek Palatinus and Pavol Rusnak, “Multi-account hierarchy for deterministic wallets”, *BIP-44*, April 24 (2014), available at https://en.bitcoin.it/wiki/BIP_0044. 

<a name="footnote7">7</a>. Daniel Weigl, “Derrivation scheme for P2WPKH-nested-in-P2SH based accounts”, **BIP-49**, May 19 (2016), available at https://en.bitcoin.it/wiki/BIP_0049.  

<a name="footnote8">8</a>. Pavol Rusnak, “Derrivation scheme for P2WPKH based accounts”, *BIP-84*, December 28 (2017), available at https://en.bitcoin.it/wiki/BIP_0084. 