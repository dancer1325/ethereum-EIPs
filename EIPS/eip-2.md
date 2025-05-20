---
eip: 2
title: Homestead Hard-fork Changes
author: Vitalik Buterin (@vbuterin)
status: Final
type: Standards Track
category: Core
created: 2015-11-15
---

### Meta reference

[Homestead](./eip-606.md).

### Parameters

|   FORK_BLKNUM   | CHAIN_NAME      |
|-----------------|-----------------|
|    1,150,000    | Main net        |
|   494,000       | Morden          |
|    0            | Future testnets |

# Specification

* if `block.number >= HOMESTEAD_FORK_BLKNUM` -> do
  1. | create contracts -- via a -- transaction -> gas cost is increased from 21,000 -- to -- 53,000
     1. != create a contract -- via -- `CREATE` opcode
     2. if you send a transaction / to address == empty string -> gas subtracted = 53,000 + gas cost of the tx data > 21,000
  2. ALL transaction signatures / s-value > `secp256k1n/2` -> FROM now, invalid
     1. TODO: ECDSA recover precompiled contract remains unchanged and will keep accepting high s-values; this is useful e.g. if a contract recovers old Bitcoin signatures.
  3. if contract creation go "out-of-gas" -> contract creation fails
     1. "out-of-gas" := required gas -- for -- creating the contract (== add the contract code | state) < final gas
     2. contract creation fails != leaving an EMPTY contract (TODO: ❓)
  4. ⭐️change the difficulty adjustment algorithm ⭐️
        ```
        // CURRENT one
        block_diff = parent_diff + parent_diff // 2048 * (1 if block_timestamp - parent_timestamp < 13 else -1) + int(2**((block.number // 100000) - 2))
        // int(2**((block.number // 100000) - 2))   := exponential difficulty adjustment component 
        //          "//"        == integer division operator, _Example:_   6 // 2 = 3        7 // 2 = 3      8 // 2 = 4 
        
        
        // PROPOSED one
        block_diff = parent_diff + parent_diff // 2048 * max(1 - (block_timestamp - parent_timestamp) // 10, -99) + int(2**((block.number // 100000) - 2))
        ```
     1. `minDifficulty` := MINIMUM ALLOWED difficulty
        1. == ALL difficulty adjustmentS > `minDifficulty`
     2. goal
        1. targeting the mean; one can prove that with the formula in use,
     3. | CURRENT one,
        1. average block time > 24 seconds, mathematically impossible | long term
        2. `block_timestamp - parent_timestamp`
           1. MAIN input variable
     4. | PROPOSAL one,
        1. `(block_timestamp - parent_timestamp) // 10`
           1. 👀MAIN input variable👀
           2. maintains the coarse-grained nature of the algorithm
           3. prevent an EXCESSIVE incentive -- to set -- blocks timestamp difference == 1
        2. `-99`
           1. enable, if 2 blocks are mined far in time -> ensure that the difficulty does NOT fall EXTREMELY

# Rationale

* Problem / NOT serious & arguably a bug
  * Problem1:
    * contract creation incentives -- via -- transactions / cost 21,000, EXIST >> those / cost 32,000
    * ether value transfer can be -- , via suicide refunds, -- done by ONLY 11,664 gas
      ```python
      from ethereum import tester as t
    >   from ethereum import utils
    >   s = t.state()
    >   c = s.abi_contract('def init():\n suicide(0x47e25df8822538a8596b28c637896b4d143c351e)', endowment=10**15)
    >   s.block.get_receipts()[-1].gas_used
      11664
    >   s.block.get_balance(utils.normalize_address(0x47e25df8822538a8596b28c637896b4d143c351e))
      1000000000000000
      ``` 
  * Problem2:
    * ⚠️we allow transactions / s  [0, secp256k1n] -> POSSIBLE transaction malleability concern⚠️
      * Reason: 🧠 | any transaction, if you flip s value from `s` -- to -- `secp256k1n - s` -> flip the v value (`27 -> 28`, `28 -> 27`) & resulting signature STILL valid 🧠
      * ❌NOT serious security flaw ❌
        * Reason of "NOT serious": 🧠| transfer ether value OR other transaction,
          * 's input
            * == addresses
            * NOT transaction hashes 🧠
        * _Example of inconveniences:_ | UI / track transactions -- via -- hashes
          * | transaction / gets confirmed | block,
            * if attacker can change transaction's hash / != transaction's hash / user sends -> UI / track transactions -- via -- hashes, inconsistency 
    * Proposal
      * 💡prevent high s values💡
  * Problem3: MANY miners were mining blocks / timestamp == `parent_timestamp + 1` -> skew block time distribution / median increasing of 13" & mean increasing infinitely

* contract creation go out-of-gas's benefits
  - result of a contract creation process is MORE intuitive
    - THIS proposal, ALLOWED values
      - "success"
      - "fail"
    - CURRENTLY, 
      - "success"
      - "fail"
      - "empty contract"
  - easier to detect failures
    - Reason: 🧠
      - if & ONLY if contract creation FULLY succeeds -> contract account created
      - otherwise -> contract account NOT created🧠
  - if there is an endowment -> contract creation safer  
    - Reason: 🧠guarantee that
      - entire initiation process happens or 
      - transaction fails & endowment is refunded🧠

# Implementation

This is implemented in Python here:

1. https://github.com/ethereum/pyethereum/blob/d117c8f3fd93359fc641fd850fa799436f7c43b5/ethereum/processblock.py#L130
2. https://github.com/ethereum/pyethereum/blob/d117c8f3fd93359fc641fd850fa799436f7c43b5/ethereum/processblock.py#L129
3. https://github.com/ethereum/pyethereum/blob/d117c8f3fd93359fc641fd850fa799436f7c43b5/ethereum/processblock.py#L304
4. https://github.com/ethereum/pyethereum/blob/d117c8f3fd93359fc641fd850fa799436f7c43b5/ethereum/blocks.py#L42
