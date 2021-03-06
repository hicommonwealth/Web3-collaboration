# **Zero Knowledge Identity**

## **Project Description**

The goal is to implement an anonymous identity system on substrate. At the most primitive level, this anonymous identity system should allow users to prove membership in a set of individuals, stored on chain, by using zero-knowledge proofs of membership and off-chain secrets. We plan to integrate this functionality into the Commonwealth UI to allow people to anonymously link their `web2` and `web3` identities—i.e Twitter, Github, or a token balance without revealing their identity. 

Based on user feedback, by allowing people to privately link `web2` and `Web3` identities, people will be more inclined to participate in governance, knowing that they do not have to "rebuild" their reputation again. Here are some example use cases in more detail.

Linking a token balance:

    Account A has 1000 EDG and wants to prove he has 1000 EDG
    
    Account A likely does not want to post on Commonwealth with Account A
    
    Account A creates Account B to use on Commonwealth Edgeware Discussions
    
    Account B is going to submit proof for Account A on chain saying, "this is my proxy account, and I have a real account that is in that merkle tree"
    
    **Constraints**: Both A and B need EDG to send txs
    
    1. A needs to send tx to get into Sparse Merkle Tree group
    2. B needs to send tx to prove he knows A's secret in the group
    
    Problem that can't be solve, how does B get EDG without exposing his identity? Someone else needs to do so on B's behalf

Membership of a group:

    A user who qualifies for a certain group will do the following:
    1. Upon receiving membership in the group, add a leaf node to a Sparse Merkle Tree
    	(a merkle tree as a group name)
    2. This leaf node will have the form Hash(Hash(secret||nullifier)||accountId)
    3. To prove membership, a user submits a proof proving they know the preimage (secret||nullifier) to a leaf node

Linking web2 identity

    A user who wishes to link a web2 identity will do the following:
    1. Register their identity using Edgeware's Identity Service (edgeware-watcher).
    2. Attest to their identity and include a leaf node Hash(Hash(secret||nullifier)||accountId||web3Identity)
    3. If the identity passes verification, this leaf node is appended to the Sparse Merkle Tree
    4. To prove membership, a user submits a proof proving they know the preimage (secret||nullifier) of the leaf node, exposing the nullifier
    5. The prover must link an account that will act on behalf of the Web2 identity, this can only happen once.
    6. Similarly if we want to allow it, we can let this linked account transfer ownership of the identity (risky/allows selling identities)

***Nullifiers***: nullifiers are needed to prevent re-submitting proofs for auxiliary accounts.

## **Team Members**

- Drew Stone - Lead
- Dillon Chen
- Raymond Zhong
- Thom Ivy
- Jake Naviasky

## **Team Website**

[https://commonwealth.im/](https://commonwealth.im/) and [https://edgewa.re](https://edgewa.re/)

## **Legal Structure**

incorporated in USA as a Delaware C-Corp

## **Team's experience**

Commonwealth Labs is a blockchain company which is focused on building Edgeware, a WASM-based smart contract parachain as well as supporting governance tools. For Edgeware and Polkadot ecosystem governance, among other things, we've built an identity module, a UI to simply the governance process across parachains. Drew Stone, the lead engineer, has been working as an Ethereum smart contract developer and blockchain developer for 2+ years.

Relevant projects to the goal are below:

- Code
    - Miximus-style ZK membership proofs: [https://github.com/drewstone/rust-miximus](https://github.com/drewstone/rust-miximus)
    - Sparse Merkle Tree substrate module: [https://github.com/hicommonwealth/edgeware-node/tree/drew.merkle/modules/edge-merkletree](https://github.com/hicommonwealth/edgeware-node/tree/drew.merkle/modules/edge-merkletree)
- Articles
    - [https://medium.com/commonwealth-labs/anonymous-voting-a-design-survey-12de869dc97f](https://medium.com/commonwealth-labs/anonymous-voting-a-design-survey-12de869dc97f)

## **Team Code Repos**

- Edgeware: [http://github.com/hicommonwealth/edgeware-node](http://github.com/hicommonwealth/edgeware-node)
- Edgeware-watcher: [https://github.com/hicommonwealth/edgeware-watcher](https://github.com/hicommonwealth/edgeware-watcher)
- Edgeware-identity: [https://github.com/hicommonwealth/edge-identity](https://github.com/hicommonwealth/edge-identity)
- Rust-mixmus: [https://github.com/drewstone/rust-miximus](https://github.com/drewstone/rust-miximus)

## **Team LinkedIn Profiles**

[https://www.linkedin.com/in/dillchen/](https://www.linkedin.com/in/dillchen/)

[https://www.linkedin.com/in/drewcstone/](https://www.linkedin.com/in/drewcstone/)

[https://www.linkedin.com/in/raykyri/](https://www.linkedin.com/in/raykyri/)

## General Project Practices
- All code will be delivered in a dockerized/container format
- All code will be documented with guides on how to run

## **Development Roadmap**

The milestones are spread out over a total of 3 months as following. We anticipate starting this work as soon as we are able to fund the development:

#### Month One - equivalent of $33,000 USD in DOTs
- M0: Aggregate all the cryptography tools importable in a portable rust library (1 week)
    - Rust libraries ported into no_std environment
    - [pairing](https://github.com/LayerXcom/zero-chain/tree/master/core/pairing)
    - [bellman](https://github.com/LayerXcom/zero-chain/tree/master/core/bellman-verifier)
    - [jubjub](https://github.com/LayerXcom/zero-chain/blob/master/core/jubjub)
- M1: Build sparse-merkle tree Substrate module to store Pedersen hashes (3 weeks)
    - Support for `insert`ion into the tree on-chain, where roots will be recalculated after transaction success
    - Support for non-private verification of merkle proofs on-chain
    - Support for storage and retrieval of current leaf hashes in the tree
    - Support for verification of a finite number of past merkle roots to ensure we can process multiple verifications between transactions.
    - Documentation for functions and storage values
    - Tests of functionality for insertion and verification of merkle path proofs

#### Month Two - equivalent of $33,000 USD in DOTs
- M2: Build off-chain method of creating proofs (3 weeks)
    - A CLI/binary written in Rust for generating proofs of membership, given the current list of leaves in the Sparse Merkle Tree.
    - Proofs will initially be stored on Commonwealth which we are in the process of open-sourcing. Proofs can then be stored on IPFS.
    - Given a merkle root, a leaf node, the preimage of the leaf node, and a standard membership proof, we want to generate a zero-knowledge membership proof that hides the preimage secret data.
    - The public inputs for the prover are the nullifier and the merkle root
    - The private inputs for the prover are the secret preimage data and the merkle path proof
    - Documentation for using the CLI/binary, which should interact with the Substrate module
- M3: Build ability to verify proofs using bellman-verifier on chain (2 weeks)
    - A Substrate module or addition to an existing substrate module (more functions) that allow verifying ZK-proofs of membership in the Sparse Merkle Tree designed above.
    - We first need to generate a verification key. We will likely want to run a multi-party computation/ceremony to generate a random seed, destroy toxic waste, etc.
    - Above, we recall the public parameters of the prover. They are the nullifier and merkle root. Thus, when someone wants to verify an identity attestation on chain, they will expose the nullifier to the on-chain module. This will record the nullifier so re-use is impossible.
    - Once the data is marshalled and we ensure that the submitted nullifier has not been used, we will verify the proof data using the bellman verifier.
    - Tests of functionality in the Substrate module
    - CLI functions for sending the proper extrinsics to verify proofs.

#### Month Three - equivalent of $33,000 where half is in DOTs and USD
- M4: Upgrade Edgeware identity module to store verified identity auxiliary data (2 weeks)
    - A Substrate module or additional functions that fit into existing Substrate module from above.
    - The functionality will include, creating/attesting/verifying identities, inserting identities leaf hashes into a Sparse Merkle Tree, and verifying ZK-proofs of membership of identities in a Sparse Merkle Tree.
    - We need to integrate a new main verify function into the identity module and create necessary helper functions for marshaling data into the proper format.
    - We will use a governance proposal to upgrade the Edgeware Chain so that proofs can be verified. We will make available the identity module for other parachains to use.
    - We will provide adequate documentation for use by other parachains.
- M5: Integration into the Commonwealth UI (2 weeks)
    - The functionality will include UI components for registering identities, attesting to the on-chain, inserting them into a Sparse Merkle Tree, and finally verifying them with an auxiliary account using a ZK-proof.
    - The UI components will discern if extrinsics are successful, letting the user know any and all errors.
    - UI should verify proofs to generate badges that other users can see or pull verifications directly from chain.
    - Create some forums such that only those who can prove some membership are allowed to post or validate
    - Integrate into Edgeware, ChainX, Polkadot, Kusama, and Alexander Testnet?


## **Additional Information**

**What work has been done so far?**

Implemented MVP data structures + identity structure on Substrate.

**Have you applied for other grants so far?**

No.

**Are there any other projects similar to yours?**

- In terms of zero-knowledge proofs/login: [login with Semaphore](https://ethresear.ch/t/login-with-semaphore-authenticate/6034)
- In terms of zero-knowledge on substrate: [Zerochain private transactions](https://github.com/w3f/Web3-collaboration/pull/96/files#diff-3b30b4271e70cd5811f2d76758da90e0)