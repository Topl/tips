<pre>
TIP: 4
Title: Wallet
Author: Edmundo López Bóbeda <e.lopez@topl.me>
Status: Draft
Type: Standard
Created: 2023-08-08
License: CC-BY-4.0
</pre>
## Abstract

This TIP proposes a specification for a wallet implementation the Topl blockchain.

## Motivation

Blockchain wallets serve to store and index private information such as keys to faciliate the creation and signing of new transactions. In some early implementations, wallets stored only a single keypair corresponding to the user's account, which was then used to sign all transactions. However, reusing the same keypair is not recognized as a good security practice as it leaves the account vulnerable to attack, and therefore this approach has fallen out of favor.

To address this reuse problem, some proposals such as [BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) introduced the concept of a Hierarchical Deterministic (HD) wallet. In these approaches, a single main key is generated, and then used to derive new public/private key pairs as needed in a deterministic process. This allows different keypairs to be used for each transaction, while the user need only store the original main key in order to re-derive all of the child keypairs. The public key hashes, which are also called addresses, may then be used to initiate transactions.

However, this approach introduces limitations in how keys may be used or composed to maintain the deterministic derivation criteria. For example, a user may wish to re-use a keypair in a subset of nonsensitive transactions for convenience, add propositional constraints to other addresses, or maintain several distinct identities that they share with different participants. We can think of these more complex use cases as transaction contracts, since there may be several criteria that must be satisfied before they are valid.

A compatible wallet must therefore store:

- the propositions that were used to create the contract,
- the public keys of the participants in that contract,
- and the private key used to sign (a portion of) the contract, or the steps necessary to derive that private key from a main key.

Given these added constraints, we now describe a novel approach to store this information in a principled way so that users can reference information quickly without needing to re-derive the information each time it's needed.

## Specification

This specification is organized as follows. First we present the scope of the specification. Next, we formally define a wallet. Finally, we present a canonical implementation of a wallet.

### Scope

We limit this specification to the wallet only. We assume that the user is familiar with, or aware of, propositional contracts and how to prove them. For more information on propositional contracts, see [TIP-5[(https://github.com/Topl/tips/blob/main/TIP-0005).

### Formal Definition

#### Preamble and Notation

A blockchain _network_ is a technical infrastructure that scopes ledger and smart contract services to applications [^1]. Let $N$ be the set of identifiers of blockchain networks within the Topl protocol. Typically, the following networks minimally populate the set $N$: `private`, `testnet` and `mainnet`. 

Let $n \in N$ be a network. We say that $A_n$ is the set of _addresses_ of the network $n$. Within this address space, consider a subset $W_n \subset A_n$; we refer to this subset as the set of _wallet addresses_.

We say that a wallet _owns_ the assets at addresses in $W_n$ if, for each $a \in W_n$ the wallet can provide:

- the propositional contract used to create $a$, and,
- at least one signature needed to unlock the contract to access assets from $a$.

Similarly, if the wallet can provide the contract used to create $a$ but not the signature that unlocks the assets from $a$, we say that the wallet _references_ $a$. The set $W_n \subset A_n$, where all addresses $a \in W_n$ are either owned or referenced by the wallet is called _set of wallet addresses_.

[BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) describes a case in which funds are fully spent from one address, and the outputs are split between multiple addresses. If funds are transferred to a new address also owned by the wallet, this new address is called the _change address_. We assume that there is a function:
```math
\begin{align}
c & : & a \in W_n & \rightarrow & a_c \in W_n
\end{align}
```
which takes an address $a$ and returns the corresponding change address $a_c$. 

Let also $m$ be a master extended key, as defined in [BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki). We use the same notation as BIP-32 to denote the derived keys, _e.g._ $m / i' / j$ denotes the $i$-th _hardened_ derivation of $m$ followed by the $j$-th _softened_ derivation. We denote $K$ the set of master extended keys and all their derivations. A derived key that is used to further derive child keys is called a _root key_.

Each propositional contract has an indexed set of verification keys $V$, which may be the empty set. A contract in which each verification key is replaced by a placeholder, is called a _contract template_. We can instantiate a contract template by replacing each placeholder with a corresponding verification key. The owners of the verification keys that appear in a contract are called _participants_; a set of participants is called a _party_. 

#### Wallet Definition

We define a wallet as:

- A main key pair $m = \langle v, s \rangle$, where $v$ is a verification key, and $s$ is a signing key. 

- An ordered list of parties $p_1$, $p_2$, $\cdots$, $p_n \in P$

- An ordered list of contract templates $t_1$, $t_2$, $\cdots$, $t_n \in T$

- A set of party and contract template indexed maps:
```math
\begin{align}
V_{p,t}  : & \mathbb{N} & \rightarrow & K \\
& l & \mapsto & v & \text{s.t. $v$ is the root key that populates placeholder $l$ in $t$}   
\end{align}
```
Let $p_i$ be a party, $t_j$ a contract template, $L$ the set of indexes of the verification keys for $t_j$, and $k \in \mathbb{N}$. We first generate the root verification keys $r_l = V_{p_i,t_j}(l)$, where $l \in L$. We then define each $v_l$ to instantiate the contract template as $v_l = r_l / k$, _i.e._ adding one soft derivation to the root verification key . Finally, we define the proposition for a given address as the instantiation of $t_j$ using the $v_l$. The address derived from this proposition is noted $a_{i, j, k}$. 

Any address in the set of wallet addresses may be indexed using a triplet $\langle i, j, k \rangle$. Using this indexing, we define the change function as follows:
```math
\begin{align}
c_{i,j,k} & : & a_{i,j,k} \in W_n & \mapsto & a_{i,j,k+1} \in W_n
\end{align}
```
We say that a wallet controls the assets at an address $a_{i,j,k}$, for one of the placeholders, if the root verification key $V_{p_i,t_j}$ is the root verification key from the derived key $m / i' / j$. Hence, to partially (or completely) prove an owned address $a_{i,j,k}$, we can derive the signing key for the address using the following derivation of the main key $m / i' / j / k$.

[^1]: Definition taken from https://hyperledger-fabric.readthedocs.io/en/release-2.2/network/network.html

### Implementation Details

We provide a reference implementation of the wallet as part of the Topl ServiceKit. In this implementaiton, we separate the main key pair from the rest of the wallet by storing it as an encrypted file, while the remaining information is stored in a relational sqlite database. The database stores the parties and contract templates in their respective tables. Finally, we store $V_{p,t}$ in its own table. To simplify the design of the database, we store the list of verification keys as a JSON list.

In this section, we first present the main key derivation. Then we present the database schema that was used to implement the wallet.

#### Topl Main Key Derivation

The Main Key derivation is done by adhering to the [BIP-44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) structure, along with the modification outlined in [CIP-2852](https://github.com/cardano-foundation/CIPs/blob/master/CIP-1852/README.md). We generate the Topl main key by deriving  the main key $m$ twice:

```math
ToplMainKey = m / purpose' / coinType'
```

We use a hardened key when we see an apostrophe. We set the value set for $purpose$ to 1852, as per [CIP-2852](https://cips.cardano.org/cips/cip1852/). The value of $coinType$ is set to 7091, as registered in [SLIP-0044](https://github.com/satoshilabs/slips/blob/master/slip-0044.md).

#### Database Schema

The following figure describes the database schema used to store wallet information.

![](imgs/erd.png)

##### `cartesian`

This table represents the Cartesian indexing.

| Field          | Type    | Description                                                  |
| -------------- | ------- | ------------------------------------------------------------ |
| id             | Integer | Identifier of the row.                                       |
| x_party        | Integer | The x-index of the cartesian indexing. Represents the index of the party. |
| y_contract     | Integer | The y-index of the cartesian indexing. Represents the index of the contract. |
| z_state        | Integer | The z-index of the cartesian indexing. Represents the current state of the protocol described by both the party and the contract. |
| lock_predicate | text    | The Lock Predicate serialized using protobuf and then encoded in Base58. |
| address        | text    | The address encoded as described in TIP-0002.                |
| routine        | text    | The routine used for signing. Currently, the only possible value is ExtendedEd25519. |
| vk             | text    | The Verification Key of the user in this coordinates. It can be obtained by deriving the main key using the coordinates in this row. |

##### `parties`

The table contains the list of parties. The wallet reserves the index 0 for the `noparty` party. The `noparty` party is used for contracts that have no party associated with them, for example, a simple height lock. The wallet also reserves the index 1 for the `self` party, which is the default owner of the wallet.

| Field   | Type    | Description                                                  |
| ------- | ------- | ------------------------------------------------------------ |
| x_party | Integer | The x-index of the cartesian indexing. Represents the index of the party. This is the index of this table. |
| party   | Text    | The identifier that identifiers the party.                   |

##### `contracts`

The table contains the list of contracts. The wallet reserves the index 1 for the "default" contract, which represents the following lock: `threshold(1, sign(0))`.  This is the standard contract used when storing assets.

| Field      | Type    | Description                                                  |
| ---------- | ------- | ------------------------------------------------------------ |
| y_contract | Integer | The y-index of the cartesian indexing. Represents the index of the contract. This is the index of this table. |
| contract   | Text    | The identifier that identifies a contract.                   |
| lock       | Text    | The lock template of this contract, serialized in JSON format. |

##### `verification_keys`

The table contains the list of verification keys of parties. It maps a pair of party and contract to a list of verification keys.

|   Field    | Type    | Description                                                  |
| :--------: | ------- | ------------------------------------------------------------ |
|  x_party   | Integer | The x-index of the cartesian indexing. Represents the index of the party. |
| y_contract | Text    | The y-index of the cartesian indexing. Represents the index of the contract. |
|    vks     | Text    | A list of verification keys in JSON format. Each verification key is serialized using protobuf spec and then encoded in Base58. |

## Backwards Compatibility

The described wallet is incompatible with previous implementations.

## Copyright

We license this work under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

