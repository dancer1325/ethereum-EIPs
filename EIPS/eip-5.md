---
eip: 5
title: Gas Usage for `RETURN` and `CALL*`
author: Christian Reitwiessner <c@ethdev.com>
status: Final
type: Standards Track
category: Core
created: 2015-11-22
---

### Abstract

* goal
  * enable call functions / return strings & other dynamically-sized arrays

* CURRENTLY, 
  * 👀| EVM, if you call ANOTHER contract / function -> output's size needs to be specified in advance👀
    * Reason: 🧠
      * you can give larger size
      * gas has to be paid -- for -- memory / NOT written to -> returning DYNAMICALLY-sized data to extent is costly & inflexible🧠 

* proposed solution
  * charge gas ONLY -- for -- memory / actually written | time the `CALL` returns

### Specification

* `CALL*`
  * == `CALL`, `CALLCODE` & `DELEGATECALL`
  * == gas & memory semantics /
    * `CREATE` does NOT charge
      * Reason: 🧠NOT write -- to -- memory 🧠
    * way to be charged
      * if `CALL*`'s arguments are `gas, address, value, input_start, input_size, output_start, output_size` -> | beginning of the opcode, gas for growing memory is charged -- 
        * for --`input_start + input_size`
        * ❌NOT for -- `output_start + output_size` ❌
      * if the called contract returns data / size `n` ->
        * calling contract's memory -- is grown to -- `output_start + min(output_size, n)`
          * -> calling contract is charged -- for -- that gas
        * output is written | `[output_start, output_start + min(n, output_size))`
      * calling contract can run out of gas | 
        * beginning of the opcode 
        * end of the opcode
    * AFTER the call, 
      * `MSIZE` opcode == memory's size / was actually grown to

### Motivation

* recommendations
  * reserve certain memory area -- for the -- call's output
    * Reason: 🧠letting a subroutine write | arbitrary areas in memory -- might be -- dangerous🧠

* | in advance (== BEFORE performing the call), 
  * know the call's output size is DIFFICULT
    * Reason: 🧠data could be | ANOTHER contract's storage / generally INACCESSIBLE 
      * if you want to determine its size -> you would require ANOTHER call | that contract 🧠
  * charging gas | areas of memory / NOT actually written
    * unnecessary

* proposal
  * | end of memory area, caller can provide -- a -- gigantic area of memory 
  * callee can -- , by returning, -- "write" | it
  * caller is ONLY charged -- for the -- memory area / ACTUALLY written

* -> makes possible
  * return flexibly dynamic data -- like -- strings & dynamically-sized arrays
  * determine the returned data's size
    * if the caller uses `output_start = MSIZE` & `output_size = 2**256-1` -> area of memory / written == `(output_start, MSIZE)`
      * `MSIZE` is evaluated | AFTER the call
  * "proxy" contracts
    * == contracts / call OTHER contracts /
      * interface they do NOT know
      * ONLY return their output,

### Rationale

* proposal's implications
  * change the Ethereum Virtual Machine

* ALTERNATIVES
  * would have changed the
    * opcodes themselves OR
    * number of opcodes' arguments
  * if `output_size` == `2**256-1` -> ONLY change the gas mechanics
    * MORE complex
      * Reason: 🧠| implementation, MAIN difficulty : enlarge memory has | 2 parts in the code -- around -- `CALL`🧠
  * | stack, add the returned data's size
    * Reason of rejection: 🧠vs `MSIZE` mechanism
      * NOT sufficient
      * worst -- for -- backwards compatibility🧠

* see https://github.com/ethereum/EIPs/issues/8

### Backwards Compatibility

* this proposal -- changes the -- semantics of contracts
  * Reason: 🧠contracts -- can access the -- gas counter & memory's size 🧠

* 👀| EXISTING contracts, unlike to suffer this change 👀
  * Reason: 🧠
    * Gas
      * VM will NOT charge MORE gas
      * if contracts use up LESS gas -- , due to how usually contracts are written, -> contracts' semantics do NOT change 
      * if MORE gas were used & perform a tight estimation for gas needed by sub-calls -> contracts might go out-of-gas
      * contracts might -- , via this approach, -- ONLY return MORE gas | their callers
    * Memory size
      * `MSIZE` opcode
        * uses
          * | PREVIOUSLY unused spot, allocate memory 
      *  if semantics change -> | EXISTING contracts, affect
        1. overlaps | allocated memory
           * contract might have wanted -- , via `CALL`, -- to allocate a certain slice of memory
             * even if that is NOT written to -- by the -- called contract
             * subsequent uses of `MSIZE` to allocate memory -- might overlap with -- this slice /
               * NOW -- smaller than -- BEFORE the change
           * unlikely / contracts exist (TODO: ❓)
        2. Memory addresses change
           * if memory is allocated via `MSIZE` -> | CURRENTLY, addresses of objects in memory != | AFTER the change, addresses of objects in memory
           * 👀contract should be written / objects in memory are _relocatable_👀
             * == NOT care, their
               * absolute position | memory
               * relative position -- to -- other objects
           * NOT affect arrays
             * Reason: 🧠they are allocated 
               * | 1! allocation
               * NOT -- via -- intermediate `CALL`🧠

### Implementation

* recommendations
  * | VM implementers, NOT to grow the memory 
    * UNTIL end of the call
    * AFTER checking sufficient gas is STILL AVAILABLE

* uses
  * "reserving" `2**256-1` bytes of memory -- for the -- output

* Python implementation -- TODO: Where ❓--
  * old: http://vitalik.ca/files/old.py
  * new: http://vitalik.ca/files/new.py
