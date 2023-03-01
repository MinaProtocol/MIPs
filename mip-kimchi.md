---
mip: <to be assigned>
title: Kimchi, a new proof system
description: Kimchi is an update to the proof system currently used by Mina
author: David Wong (@mimoo)
discussions-to: <URL>
status: Draft
type: Standards Track
category (*only required for Standards Track): Cryptography
created: 2023-03-01
requires (*optional): <MIP number(s)>
---

## Abstract

Currently, Mina makes use of a variant of the [Plonk](https://eprint.iacr.org/2019/953) protocol as its proof system. This MIP proposes an upgrade to Mina’s proof system: Kimchi.

The improvements include more accelerated operations, at the cost of a small increase in proof size. In brief, Kimchi introduces a multitude of optimizations including more registers, lookup tables via the [Plookup protocol](https://eprint.iacr.org/2020/315), custom gates for accelerating range-checks and facilitating the implementation of non-ZKP friendly cryptographic primitives like [ECDSA](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm) on [secp256k1](https://en.bitcoin.it/wiki/Secp256k1), [SHA-2](https://en.wikipedia.org/wiki/SHA-2), [SHA-3](https://en.wikipedia.org/wiki/SHA-3), and in particular Ethereum’s  [keccak256](https://github.com/ethereum/eth-hash).

These improvements are all intended to benefit and make Mina's zkApp upgrade possible.

## Motivation

The motivation behind the Kimchi proof system is to improve on Mina’s current zero-knowledge proof system on all dimensions, at the cost of a modest increase in proof size (which is already acceptable in size, at less than 5KB for its current configuration). For example, if the Kimchi upgrade goes through, Mina’s circuits will automatically benefit from the improvements and decrease in size from [2<sup>18</sup>](https://github.com/MinaProtocol/mina/blob/master/src/lib/zexe_backend/pasta/basic.ml#L17) gates (currently) to [2<sup>16</sup>](https://github.com/MinaProtocol/mina/blob/a40d965ae6b39ca93d9eed17efcbf77e0778de0a/src/lib/crypto/kimchi_backend/pasta/basic/kimchi_pasta_basic.ml#L16) gates. The bottom line is that these improvements enable larger and more complex applications for similar performances, and higher performances for the same applications.

## Rationale

In order to enable more complex applications, Mina must support programs of different sizes and complexity. These programs are written as arithmetic circuits in order for their execution to be proven (using zero-knowledge proofs). Due to the way Mina is designed, those circuits are upper bounded in size, specifically in the number of arithmetic gates (addition and multiplication) they can have.

While Mina can dictate the actual upper bound enforced by the system (which is out of scope for this document), it is good to keep in mind that the choice of circuit size limit influences the performance of the proof system. Concretely, Mina is currently (pre-kimchi) set to allow circuits of maximum 2<sup>18</sup> gates.

The discussed parameters can be quite limiting in practice. Common procedures like elliptic curve operations and hashing require a large number of arithmetic gates. Kimchi introduces three mechanisms to alleviate this limitation:

**More custom gates**. The current proof system of Mina already supports a number of custom gates. These are gates that perform more complex operations than just addition and multiplication, allowing circuits to compress or pack expensive operations in a single or a few gates (thus, reducing the number of gates they would normally have used). Kimchi adds a number of new custom gates that we describe in more detail below. The new custom gates were specifically chosen to accelerate common cryptographic operations that will most likely be needed in applications written on top of Mina (e.g. verification of ECDSA signatures).
**Fixed table lookups**. A number of commonplace cryptographic constructions that we would like to support in Mina are not optimized for arithmetic circuits. In order to speed up such calculations, the [Plookup protocol](https://eprint.iacr.org/2020/315.pdf) was proposed in 2020 by Gabizon and Williamson. Plookup allows zero-knowledge proof systems, like Kimchi, to perform lookups in arbitrary tables. For example, if you want to check that a user-provided circuit value is within a specific range, you can provide a table of all the numbers between 0 and some upper bound number, and perform a lookup to enforce that the circuit value is included in that table. This would be a table of 1 column. An XOR table is another example of a table of 3 columns where the first column is the first input, the second column is the second input, and the third column is the result of XORing the first two columns.
**Runtime table lookups**. Since circuits are fixed in time, and must support dynamic executions, a number of statements must be encoded statically. For example, loops must be unrolled to a specific number of iterations (no matter the number of iterations actually performed by a specific run); conditional branches must all encode their logic in a circuit (no matter which branch is taken in practice); and array indexing cannot be dynamic (i.e. in `array[i]` the index `i` must be known at compile time). The latter example can be circumvented by enabling lookups on user-provided tables, which Kimchi implements.

Note that these advances were enabled by a slight increase in the proof size (around 1 KB larger).

## Backwards Compatibility

Note that Kimchi is not backward compatible with the previous proof system of Mina: proofs created with the current proof system used by Mina cannot be verified by Kimchi, and vice versa. For this reason, if this MIP gets approved, a [hard fork](https://www.investopedia.com/terms/h/hard-fork.asp) will have to be considered in order to integrate Kimchi in the Mina protocol. Discussions around a hard fork are out of scope for this document.

## Specification

The exact specification of Kimchi is out scope for this document. It currently lives in the [Mina book](https://o1-labs.github.io/proof-systems/specs/kimchi.html).

The additional custom gates brought by Kimchi are:

* **Lookup**: enables lookups within arbitrary tables of a single column (including runtime tables).
* **RangeCheck**: enables more efficient range checks and comparison of large numbers. Specs can be found [here](https://o1-labs.github.io/proof-systems/specs/kimchi.html#range-check).
* **ForeignFieldAdd and ForeignFieldMul**: enable field arithmetic for fields foreign to the Kimchi fields. This is particularly interesting to support different elliptic curves (like secp256k1 which is used in many other cryptocurrencies and protocols). Note that these have their own specs ([here](https://o1-labs.github.io/proof-systems/specs/kimchi.html#foreign-field-addition) and [here](https://o1-labs.github.io/proof-systems/specs/kimchi.html#foreign-field-multiplication)) and RFCs as well ([here](https://o1-labs.github.io/proof-systems/rfcs/foreign_field_add.html) and [here](https://o1-labs.github.io/proof-systems/rfcs/foreign_field_mul.html)).
* **Xor16**: enables efficient XORing of 16-bit values (easily chainable to XOR longer values. Specs can be found [here](https://o1-labs.github.io/proof-systems/specs/kimchi.html#xor-1).
* **Rot64**: enables efficient rotations of 64-bit strings. Specs can be found [here](https://o1-labs.github.io/proof-systems/specs/kimchi.html#rotation).

Note that all of these new tables make use of the new lookup feature internally.

## Reference Implementation

The current proof system of Mina is implemented in https://github.com/o1-labs/proof-systems/tree/bacef43ea34122286745578258066c29091dc36a

(Note: specifically, the library being used in Mina are listed here https://github.com/MinaProtocol/mina/blob/master/src/lib/marlin_plonk_bindings/stubs/Cargo.toml#L29)

The updated proof system, Kimchi, is implemented in the same repo in the HEAD of the master branch: https://github.com/o1-labs/proof-systems/ (TODO: add specific commit when we finalize the MIP)

## Security Considerations

Kimchi’s design brings a number of security considerations:

**New custom gates**. Each custom gate is a set of mathematical constraints that must be complete and sound relative to the original computation it is associated with. In this context, the terminology “complete” means that the gate should always work when given valid values (e.g. `a XOR b = c` should hold for all valid tuples `(a, b, c)`), and the term “sound” means that the gate should never work for invalid values (e.g. the XOR gate should not work for the tuple `(1, 0, 0)`). The introduction of new custom gates means new opportunities for unsafe or incomplete constraints, which would lead to vulnerabilities in user applications. Careful peer review, as well as extensive testing, were performed in order to increase confidence in the new protocol.

**Specific Lookup implementation**. As with most cryptographic constructions, the plookup protocol can be vague and omit implementation details. As such, there are opportunities for bugs if wrong assumptions were made, or vague steps were not instantiated correctly. As with the previous bullet point, extensive testing and peer review went through the design and implementation of the new feature.

**Safe API**. The introduction of lookup tables, especially user-provided runtime tables, introduce a number of edge cases which users must be aware of. While Mina’s proof system aims at providing a safe user experience through higher-level abstractions (for example, via [snarky](https://github.com/o1-labs/snarky) or [snarkyJS](https://github.com/o1-labs/snarkyjs/)), and will continue to do so with Kimchi if this MIP were to be accepted, considerations must be taken for applications that may want to directly use Kimchi before stabilization of the APIs.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
