---
Class: "[[Security]]"
Sem: WiSe 26/27
---
---
## Index
- [[#Introduction|Introduction]]
- [[#Wallets|Wallets]]
- [[#Transaction|Transaction]]
- [[#Blocks|Blocks]]
	- [[#Blocks#Why is block generation difficult?|Why is block generation difficult?]]
- [[#Blockchain|Blockchain]]
- [[#Rewards for mining|Rewards for mining]]
- [[#Transaction acceptance|Transaction acceptance]]
- [[#Attacks on Bitcoin|Attacks on Bitcoin]]
- [[#Privacy/anonymity vs. Pseudoanonymity|Privacy/anonymity vs. Pseudoanonymity]]
	- [[#Privacy/anonymity vs. Pseudoanonymity#Bitcoin mixing schemes|Bitcoin mixing schemes]]
- [[#Legal and economic aspects|Legal and economic aspects]]
- [[#Altcoins and Meme-Coins|Altcoins and Meme-Coins]]
- [[#Alternative to PoW|Alternative to PoW]]
---
## Introduction
One of the main motivation for creating digital currencies was in the beginning to have a decentralized way to manage money (i.e., to not rely on banks).

Bitcoin is a distributed currency/payment system which was proposed in 2008 and a client was published in 2009 by an anonymous person (supposedly Satoshi Nakamoto, but it’s probably a pseudonym). It relies on public-ket cryptography and crypto-hashes, and the money is transferred between *wallets* with public/private key(s).

![[Pasted image 20261105162637.png]]

Each transaction is:
- signed using spending wallet’s private key
- addressed to the receiver’s public key (pseudonymous, identity is hidden)
- broadcasted to the bitcoin network (peer-to-peer network)

Given this we are able to achieve a **global consensus** on all valid transaction through *blockchain*. In particular when a transaction is performed, it is sent to a so called “miner” who will add the transaction to the block he is currently mining; once the block is complete, it is added to che blockchain achieving global consensus.

---
## Wallets
Wallets have $1^{*}$ addresses (each 26-35 alphanumeric characters), but actually each address is a 160-bit hash of the public key (encoded in base58, plus checksum and version number; base58 has fewer characters compared base64 in order to remove the characters that look similar in ASCII to avoid human error).

The hash is done using SHA256, and then it’s digested into 160 bits using RIPEMD-160 (the design choice of this was just to use another layer of hashing, using an hash that generated a shorter message seemed unsafe in 2008). This made the address easily sharable with third parties.

$$
address = Base 58(version \mid\mid RIPEMD-160(SHA256(pk))\mid\mid checksum)
$$
where the $checksum$ are the first 4 bytes of $SHA 256(SHA 256(version \mid \mid hash))$.

Modern address mostly use Bech32 instead of base58 encoding (they start with `bc`, from SegWit/Taproot).

---
## Transaction
Transactions are using digital signatures, implemented with ECDSA (elliptic curve digital signature algorithm) but since Taproot upgrade in 2021, Schnorr signatures are also supported.

A single transaction consist of:
- input transaction(s) hash(es) → previous transactions to prove that I own those money (e.g. if have to send 10$ but the previous transaction is of 15\$, I attach that transaction and then I send the remaining 5\$ back to myself)
- one (or more) target addresses, and respective amounts
- signature of spending wallet

>[!warning]
>Each transaction may be only used once as input transaction in the entire network.

Transactions might have a small fee.

>[!example]
>![[Pasted image 20261105165204.png|550]]
>
>This is a simplification the blockchain is missing

---
## Blocks
A block is a record of some (or all) of the most recent Bitcoin transactions (that have not yet been recorded in any prior blocks). Each block is strictly tied with the previous one by including the previous block’s hash in the header.

![[Pasted image 20261105170133.png|487]]

Here’s some of the header fields:
- $prevHash$ → hash value of previous block
- $transHash$ → hash value of all transaction in the block (root of Merkle tree)
- $nonce$ → an incrementing nonce

The nonce is the crucial number, it can be arbitrary chosen by the miner in order to find a valid hash.

### Why is block generation difficult?
Most of the content of the new block is fixed and the goal is to find a nonce such that: “the hash value of the block header is smaller than the difficulty value”.

The current difficulty (Oct 2026) is ~140 trilion; equivalently. ~79 of the 256 hash bits must be 0 ($\approx 2^{79}$ hash attempts per block).

This is probabilistic, so only trying all possible nonces will work, and this process is called **mining** (many miners may be computing the same block, but only the one that manage to solve it submit the block to the blockchain and get rewarded). Once a working nonce is found, the finder broadcasts the block to the whole network.

The reason why we would need such a complexity is to make it costly to falsify. If it would have been free, anyone can create any block and rewrite the history.

>[!question] But having such a high complexity wouldn’t slow down the computation of a single block, increasing the possibility for an attacker to falsify the transaction?
>No, the difficulty doesn’t make the blocks more difficult to compute. It is based on the availability of computation power (it will always take about 10 minutes to compute an hash)

>[!info]
>![[bangdesk.gif]]
>
>La difficulty restringe lo spazio degli hash validi, quindi la probabilità di collision aumenta. Però restano circa $2^{176}$ valori possibili contro meno di un milione di blocchi, quindi la probabilità è di circa $2^{-137}$

---
## Blockchain
The blockchain is the global state of the bitcoin network: it consists of valid blocks, chained together by their hashes. When a valid block is found, everyone start mining for the next one

>[!warning]
>The *longest* valid chain is the accepted here. If two miners, mine the same block at the same time, global consensus is not anymore achievable so the miners will attach new blocks to the chain they trust the most, and in the end the longest wins.
>
>To check for such cases (i.e., if a block is valid or not), other miners are involved.

![[Pasted image 20261105172707.png]]

Due to the difficulty of finding a valid hash, there are mining pools where people collaborate trying to find an hash and then split the reward when solved it.

>[!tip] Losing wallets…
>If, for any reason, you happen to lose your private key your money is lost, as theoretically there is no way to recover money or key (unless you convince the global network to reject incoming transactions)
>
>However users are often not directly are using bitcoins tools, but they instead use online services to store wallets (becomes almost like traditional banking).

---
## Rewards for mining
Every new block contains a *coinbase transaction* that creates new bitcoins (not to be confused with the *genesis block* which is the first block of the chain). Once the miner get this transaction he can put its own address as destination and if the block gets accepted, the miner can spend those coinds.

The current reward is $3.125$ BTC per block (since the April 2024 halving; rewards halves every 210.000 block, about 4 years).

In addition for each transaction, you may include a "tip" for the miner (transaction fee). An higher fee would mean an higher chance for a transaction to be included in the next mined block and consequently in the blockchain.

---
## Transaction acceptance
Transaction become valid after:
- they are first listed in a valid block
- that block has been followed by further block → common convention is 6 confirmations (the block itself plus 5 more)

This will prevent potential double-spending with two parallel block chains. Miners will also check signatures in transaction before including, and an invalid transaction will lead to invalid blocks

>[!tldr]
>De-facto, ir is a majority vote on which current chain is the accepted one.

---
## Attacks on Bitcoin
To fake transactions, an attacker must either generate victim’s signature or introduce manipulated blocks into the blockchain.

An attacker can try to *double-spend* which consists in creating two different transactions using the same input transaction. In this case the second transaction will be rejected in block generation, but in the worst case (if two miners find a block at the exact same time for the two transactions) the blockchain will fork but the error is going to be detected soon.

The only feasible attack if an attacker can somehow generate blocks faster then the rest of the network (attacker determines which transactions are accepted and which are rejected, and can create it’s own chain).

---
## Privacy/anonymity vs. Pseudoanonymity
Bitcoin relies on persistent wallet IDs, which are not tied to personal data (the IDs are pseudonyms, but not anonymous). However transactions of wallet are visible (while if transactions where anonymous they couldn’t be linked).

Once items are bought with a wallet, they can be associated to user and as a result, all prior transactions can be mapped.

### Bitcoin mixing schemes
*Bitcoin Mixers* aggregate a number of incoming transactions from different wallets, and then generate a new set of outgoing transactions with different amounts (to targets specified by users). In this way you can hide association between in and out transaction.

![[Pasted image 20261105201957.png|363]]

>[!warning] Dangers
>- mixer will “own” incoming transaction and might betray you
>- legal issues if transaction is mixed with illegal ones

---
## Legal and economic aspects
Bitcoins are not directly regulated (no banks/governments are involved). But all transactions are public, if wallet owner is somehow identified, all transactions can be reconstructed.

---
## Altcoins and Meme-Coins
Success of Bitcoin spawned countless other cryptocurrencies. Some are essentially the same as bitcoin), while others cryptographically broken by design

Many are essentially pyramid schemes, relying on low cost at introduction (early adopters hope to benefit from rising demand) and some coins have even a fixed marketing budget

Meme-Coins rely on social media to get attention (Dogecoin as best example)

---
## Alternative to PoW
Hash-based mining is an example of *Proof-of-Work* where:
- the cost incurred is operations/hardware/energy
- the reward is influenced by block coining (without cost, anyone could create new blocks)
- some cost is required for consensus finding

*Proof-of-Stake* is a valid alternative to PoW. Ethereum switched from PoW to PoS in 2022, The Merge), cutting its energy use by ~99.9%. In this case the mining power is relative to amount of coins held (e.g. in ETH you are chosen randomly to create block). Creation involves transaction validation and reward and attesting malicious blocks is punished by losing stake, but the chance depends on number of ETH held.