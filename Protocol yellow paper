(Yellow Paper)
## Introduction
This protocol allows users to create a private pool for an arbitrary token (e.g., ERC20 tokens and native currency). The pool supports the following functionalities:
1. **Mint**: a user mints a new coin with a value equal to the amount deposited into the pool.
2. **Spend**: a user spends a set of input coins and generates a new set of output coins. The user can open an output coin in case they want to withdraw its amount from the pool.

Given the public state of the underlying blockchain, it is reasonable to assume the following:
* Mint transactions cannot hide the deposited amount and the user’s blockchain wallet.
* Withdrawing an amount in spend transactions reveal the recipient’s blockchain wallet.

Note, blockchain wallet refers to the user's account used to interact with blockchain (e.g., EoA on Ethereum)

## Analysis
We use **UTXO** model of *e-note* which represents a confidential balance redeemable by its owner
### Requirements
Design a protocol that provide the following features:
* **Transparent setup**: the protocol doesn’t rely on any trust assumptions (no zk-SNARK)
* **Confidential transactions**: transferred amounts are hidden.
* **Sender’s annoymity**: an observer cannot determine the sender's identity.
* **Recipient’s annoymity**: an observer cannot determine the recipient's identity.
* **Regulation**: a user can grant third parties visibility into incoming and/or outgoing amounts.
* **Multi-signature**: allow a group of users to cooperate before spending funds.

Transaction protocol must always ensure the following rules hold:
* **Membership**: Spent coins must already exist in the ledger.
* **Unspentness**: Only unspent coins can be input to a spend transaction.
* **Ownership**: Only the input coins' owner can spend them. 
* **Balance**: The input coins amount must equal the output coins amount plus a fee.

An ideal protocol should satisfy the following privacy objectives:
* **Recipients** 
    * **Know**: Amounts received, and when they were received. 
    * **Don’t know:** Who sent them any given amount.
* **Senders** 
    * **Know**: Amounts sent, when they were sent, and who they were sent to.
    * **Don’t know**: If an amount sent to someone else has been spent.
* **Observers**
    * **Know**: The number of inputs/outputs in each transaction, fees paid by each transaction, and when each transaction was added to the ledger.
    * **Don’t know**: The amounts involved in any transaction (other than fees), transactio's parties, the relationships between transactions, or the amounts owned by any user.

#### Anonymity Set Size
A transaction author proves that each e-note spent by their transaction exists in a small set of e-notes, and further proves that that small set is a subset of e-notes that exist in the ledger. Unfortunately, if observers see that one transaction references an e-note created by another transaction, then they know there is more likely to be a relationship between those transactions.

Increasing the anonymity set size of membership proofs naturally reduces how much information observers can glean from transactions. However, combining membership proofs with ownership and unspentness proofs in one large proving structure significantly increase the proof size, generation, and verification cost.

## Cryptographic Primitives 
### Notations 
Let $\mathbb{G}$ be a cyclic group in which the discrete logarithm problem is hard, and let $\mathbb{F}$ be the scalar field of $\mathbb{G}$. Let $F,G,H,U\in \mathbb{G}$ be random group generators. Let $\mathcal{H} : \{0, 1\}^∗ → \mathbb{F}$ be a cryptographic hash function. 

### Pedersen Commitment
The Pedersen commitment scheme is a homomorphic commitment scheme that contains an algorithm $\textsf{Com} : \mathbb{F}^2 \rightarrow \mathbb{G}$, where $\textsf{Com}(v, r) = vG+ rH$ that is homomorphic in the sense that
$$
\textsf{Com}(v_1, r_1) + \textsf{Com}(v_2, r_2) = \textsf{Com}(v_1 + v_2, r_1 + r_2)
$$

#### Double Masked Commitment
The commitment scheme contains an algorithm $\textsf{Comm} : \mathbb{F}^3 \rightarrow \mathbb{G}$, where $\textsf{Comm}(v, r, s) = vF+ rG+ sH$ that is homomorphic in the sense that

$$
\textsf{Comm}(v_1, r_1, s_1) + \textsf{Comm}(v_2, r_2, s_2) = \textsf{Comm}(v_1 + v_2, r_1 + r_2, s_1+s_2)
$$

### Proof of Discrete Log Knowledge ([1](https://https://link.springer.com/chapter/10.1007/0-387-34805-0_22))
It consists of two algorithms $(\textsf{Prove}_{DL},  \textsf{Verify}_{DL})$ for the following relation:
$$
\{G, X\in \mathbb{G}; x\in \mathbb{F}: X=xG\}
$$

#### $\textsf{Prove}_{DL}(pp, X, x) \rightarrow \pi_{DL}$
1. Parse $G$ from $pp$
2. Sample $a\in \mathbb{F}$
3. Compute $A \gets aG$
4. Compute $c \gets \mathcal{H}(G, X, A )$
5. Compute $z \gets a+cx$
6. Set $\pi_{DL} \gets (X,A,z)$

#### $\textsf{Verify}_{DL}(pp, X, \pi_{DL}) \rightarrow \pi_{DL}$
1. Compute $c \gets \mathcal{H}(G, X, A )$
2. Assert $zG = AX^c$


### Proof of Discrete Log Equality
It consists of two algorithms $(\textsf{Prove}_{EQ},  \textsf{Verify}_{EQ})$ for the following relation:
$$
\{\{S_i, T_i\}_{i=0}^{l-1} \subset \mathbb{G}^2; \{x_i, y_i, z_i\}_{i=0}^{l-1}: \forall i \in [0,l-1], S_i = x_iF+y_iG+z_iH,\, U=x_iT_i+y_iG\}
$$

### Proof of One-out-of-Many([2]())
It consists of two algorithms ($\textsf{Prove}_{RP}, \textsf{Verify}_{RP}$):


### Bulletproofs([3](https://ieeexplore.ieee.org/abstract/document/8418611?casa_token=UKvVm4IE7EcAAAAA:pmS82DV1-6C9KEu8wlKWkqkBZq04VCfUQw3z_vdq4sSi-4YskoPNoR_gw8KNj7-M3x1mOBgHu-Cf))
It consists of two algorithms ($\textsf{Prove}_{BF}, \textsf{Verify}_{BF}$):
$$
\{C\in\mathbb{G}; (v,r)\in\mathbb{F}^2: 0\leq v<v_{max}, C=\textsf{Com}(v,r) \}
$$

## Protocol Design
### Concepts and Terminology
#### Keys and Addresses
Each user generates the following tuple of keys:
1. $k$: Base key to derive public receiving addresses 
2. $k_{vr}$: Viewing key for received coins.
3. $k_{vf}$: Full viewing key for received and spent coins. 
4. $k_{sp}$: Spending key for generating spend transactions.

Using the keys tuple, a  user can generate any number of indistinguishable *diversified* public addresses. Diversified addressing allows a recipient to provide distinct public addresses to different senders.

#### Coins
A coin encodes the abstract value which is transferred through the private transactions. Each $\textsf{coin}$ is associated with:
* Private: 
    * Serial number that uniquely defines the coin.
    * Value of the coin. (It is public in Mint transactions)
* Public: 
    * Serial commitment.
    * Value commitment.
    * Range proof for the value commitment. (Proof of correct commitment in Mint transactions)
    * Ciphertext to be decrypted by recipient.
    * Recovery key to help the recipient decrypt the ciphertext.

#### Nullifiers
Nullifiers are used to prevent double-spending of coins. When generating a spend transaction, the sender produces a nullifier for each input coin. When verifying the transaction, the nullifiers must haven't been seen before.

### Algorithms

#### Setup$() \rightarrow pp$
Generates the public parameters used in the protocol
1. Sample a prime-order group $\mathbb{G}$ where DDH is hard. Let $\mathbb{F}$ be the scalar field of $\mathbb{G}$.
2. Sample $F, G, H, U \in \mathbb{G}$ uniformaly at random as group generators.
3. Sampe cryptographic hash functions:
$$
\mathcal{H}_{Q_0}, \mathcal{H}_{Q_2}, \mathcal{H}_{S}, \mathcal{H}_{V}, \mathcal{H}_{S^\prime}, \mathcal{H}_{V^\prime}, \mathcal{H}_{bind}: \{0,1\}^* \rightarrow \mathbb{F}
$$
4. Set maximum value as $v_{max}$.
5. Return all of the above as public parameters $pp$.

#### CreateKeys $(pp)\rightarrow (k_{vr}, k_{vf}, k_{sp})$
Creates keys tuple for a user
1. Sample $s_1, s_2, r \in \mathbb{F}$.
2. Compute $D \gets \textsf{Comm}(0,r,0), P_1 \gets \textsf{Comm}(s_1,0,0), P_2 \gets \textsf{Comm}(s_2, r, 0)$
3. Set $k_{base} = (P_1, P_2)$
4. Set $k_{vr} = (s_1, P_1, P_2)$
5. Set $K_{vf} = (s_1, s_2, D, P_1, P_2)$
6. Set $k_{sp} = (s_1, s_2, r)$
7. Initialize an empty hash table $\textsf{AddrTable}$.

Note $k_{base}$ is used internally to generate diversified public reciving addresses (i.e., it isn't exposed to the user)

#### CreateAddress$(pp, k_{vr})\rightarrow \textsf{addr}_{pk}$
Creates a diversified public receiving address from the viewing key
1. Parse $(s_1, P_1, P_2) \gets k_{vr}$
2. Compute: 
    * $Q_{0,i} \gets \textsf{Comm}(\mathcal{H}_{Q_0}(s_1, i),0,0)$
    * $Q_{1,i} \gets s_1Q_{0,i}$
    * $Q_{2,i} \gets \textsf{Comm}(\mathcal{H}_{Q_2}(s_1,i),0,0) + P_2$
3. Add entry $Q_{2,i}\rightarrow i$ in $\textsf{AddrTable}$
4. Set $\textsf{addr}_{pk} \gets (Q_{0,i}, Q_{1,i}, Q_{2,i})$

#### CreateCoin$(pp, \textsf{addr}_{pk}, v)\rightarrow \textsf{Coin}$
Generates a coin for a destination address $\textsf{addr}_{pk}$ for a value $v\in [0, v_{max}-1]$.
1. Parse $(Q_0, Q_1, Q_2) \gets \textsf{addr}_{pk}$
2. Sample $k \in F$
3. Compute recovery key $K \gets kQ_0$ and derived recovery key $K_{der} \gets kQ_1$.
4. Generate DL proof: $\pi_{rec}\gets \textsf{ProveDL}(pp, F, kF; k)$.
5. Compute serial commitment $S\gets \textsf{Comm}(\mathcal{H}_S(K_{der}), 0, 0) + Q_2$.
6. Compute value commitment $C \gets \textsf{Com}(v, \mathcal{H}_v({K_{der}}))$.
7. Generate range proof: $\pi_{rp}\gets \textsf{ProveRange}(pp, C; v, \mathcal{H}_v({K_{der})})$
8. Generate symmetric key $k_{sym} \gets \textsf{SymKeyGen}(K_{der})$
9. Encrypt the value $v^* \gets \textsf{SymEnc}(k_{sym}, v)$
10. If Mint, set $\textsf{Coin} \gets (S, C, \pi_{rec}, v)$ else $\textsf{Coin} \gets (S, C, \pi_{rec}, \pi_{rp}, v^*)$


