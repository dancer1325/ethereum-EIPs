---
eip: 7
title: DELEGATECALL
author: Vitalik Buterin (@vbuterin)
status: Final
type: Standards Track
category: Core
created: 2015-11-15
---

### Hard Fork
[Homestead](./eip-606.md)

### Parameters
- Activation:
  - Block >= 1,150,000 on Mainnet
  - Block >= 494,000 on Morden
  - Block >= 0 on future testnets

### Overview

* add `DELEGATECALL` | `0xf4`
  * == NEW opcode
  * 's idea == `CALLCODE`'s idea
    * EXCEPTION: ⚠️`DELEGATECALL` propagates the parent scope's sender & value -- to the -- child scope ⚠️
      * == call created's sender & value == original call's sender & value 

### Specification

* `DELEGATECALL` : `0xf4`,
  * ' 6 operands
    - `gas`
      - amount of gas / code may use -- in order to -- execute
    - `to`
      - destination address | code -- is to be -- executed
    - `in_offset`
      - input's offset memory 
    - `in_size`
      - input's size -- in -- bytes
    - `out_offset`
      - output's offset memory 
    - `out_size`
      - scratch pad's size -- for the -- output

#### Notes on gas
- basic stipend is NOT given 
- `gas` == total amount / callee receives
- UPFRONT gas cost == ALWAYS `schedule.callGas` + `gas`
  - if `CALLCODE` -> NO create an account
- refunded GAS
  - == unused gas

#### Notes on sender
- | callee's environment, `CALLER`& `VALUE` behaviour == | caller's environment, `CALLER`& `VALUE` behaviour 

#### Other notes
- depth limit == 1024
  - STILL preserved

### Rationale

* easier for a contract
  * store ANOTHER address -- as a -- mutable source of code
    * Reason: 🧠propagating sender & value | parent scope -- to the -- child scope🧠
  * ''pass through'' calls -- to -- it
    * Reason: 🧠parent code execution environment == child code execution environment🧠
      * EXCEPT for reduced gas & increased callstack depth

* use cases
  * use case1: split code -- to -- get around 3m gas barrier
    ```python
    ~calldatacopy(0, 0, ~calldatasize())
    if ~calldataload(0) < 2**253:
        ~delegate_call(msg.gas - 10000, $ADDR1, 0, ~calldatasize(), ~calldatasize(), 10000)
        ~return(~calldatasize(), 10000)
    elif ~calldataload(0) < 2**253 * 2:
        ~delegate_call(msg.gas - 10000, $ADDR2, 0, ~calldatasize(), ~calldatasize(), 10000)
        ~return(~calldatasize(), 10000)
    ...
    ```
  * use case 2: mutable address -- for -- storing the contract's code
    ```python
    if ~calldataload(0) / 2**224 == 0x12345678 and self.owner == msg.sender:
        self.delegate = ~calldataload(4)
    else:
        ~delegate_call(msg.gas - 10000, self.delegate, 0, ~calldatasize(), ~calldatasize(), 10000)
        ~return(~calldatasize(), 10000)
    ```

* child functions / called -- by -- these methods 
  * can NOW freely reference `msg.sender` & `msg.value`

### Possible arguments against

* way to replicate this functionality
  * stick the sender | call data's FIRST twenty bytes
    * -> code would 
      * need to be specially compiled / delegated contracts, 
      * NOT be usable | delegated & raw contexts | SAME time
