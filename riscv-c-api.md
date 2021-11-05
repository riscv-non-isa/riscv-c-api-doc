# RISC-V C API Specification

This file contains the RISC-V additions to the standard C APIs.  It
merges together the compiler command-line API, the preprocessor API, and
the C language extensions (function attributes and intrinsic functions).

## Copyright and license information

 &copy; 2018 Palmer Dabbelt <palmer@sifive.com>

 &copy; 2018 SiFive, Inc

It is licensed under the Creative Commons Attribution 4.0 International
License (CC-BY 4.0). The full license text is available at
https://creativecommons.org/licenses/by/4.0/.

## Compiler Command-Line Arguments

* `-mabi=ABI`
* `-march=ISA`
* `-mtune=MICRO_ARCHITECTURE`
* `-mplt` `-mno-plt`
* `-mcmodel=CODE_MODEL`
* `-mstrict-align` `-mno-strict-align`
* `-mfdiv` `-mno-fdiv`
* `-mdiv` `-mno-div`
* `-mpreferred-stack-boundary=N`
* `-msmall-data-limit=N`
* `-mexplicit-relocs` `-mno-explicit-relocs`
* `-mrelax` `-mno-relax`
* `-msave-restore` `-mno-save-restore`
* `-mbranch-cost=N`

## Preprocessor Definitions

| Name                | Value | When defined                  |
| ------------------- | ----- | ----------------------------- |
| __riscv             | 1     | Always defined.               |
| __riscv_xlen        | <ul><li>32 for rv32</li><li>64 for rv64</li><li>128 for rv128</ul> | Always defined.             |
| __riscv_flen        | <ul><li>32 if the F extension is available **or**</li><li>64 if `D` extension available **or**</li><li>128 if `Q` extension available</li></ul> | `F` extension is available. |
| __riscv_32e         | 1     | `E` extension is available.   |

### Architecture Extension Test Macro

Architecture extension test macro is a new set of test macro to checking the
availability and version for certain extension, however not all compilers are
supported, so you should check `__riscv_arch_test` to make sure this compiler
is supporting those preprocessor definitions.

The value of architecture extension test macro are defined as its version,
which is compute by following formula:

```
<MAJOR_VERSION> * 1,000,000 + <MINOR_VERSION> * 1,000 + <REVISION_VERSION>
```

For example:
- F-extension v2.2 will define `__riscv_f` as `2002000`.
- B-extension v0.92 will define `__riscv_b` as `92000`.


| Name                    | Value        | When defined                                                 |
| ----------------------- | ------------ | ------------------------------------------------------------ |
| __riscv_arch_test       | 1            | Compiler supports architecture extension test macros.        |
| __riscv_i               | Arch Version | `I` extension is available.                                  |
| __riscv_e               | Arch Version | `E` extension is available.                                  |
| __riscv_m               | Arch Version | `M` extension is available.                                  |
| __riscv_a               | Arch Version | `A` extension is available.                                  |
| __riscv_f               | Arch Version | `F` extension is available.                                  |
| __riscv_d               | Arch Version | `D` extension is available.                                  |
| __riscv_c               | Arch Version | `C` extension is available.                                  |
| __riscv_b               | Arch Version | `B` extension is available.                                  |
| __riscv_v               | Arch Version | `V` extension is available.                                  |
| __riscv_zfh             | Arch Version | `Zfh` extension is available.                                |
| __riscv_zk              | Arch Version | Extensions of `Zkn` are available and `Zkt` is asserted.     |
| __riscv_zkn             | Arch Version | Zkn: `Zbkb` `Zbkc` `Zbkx` `Zkne` `Zknd` `Zknh` are all available. |
| __riscv_zks             | Arch Version | Zks: `Zbkb` `Zbkc` `Zbkx` `Zksed` `Zksh` are all available.       |
| __riscv_zbkb            | Arch Version | `Zbkb` extension is available.                               |
| __riscv_zbkc            | Arch Version | `Zbkc` extension is available.                               |
| __riscv_zbkx            | Arch Version | `Zbkx` extension is available.                               |
| __riscv_zknd            | Arch Version | `Zknd` extension is available.                               |
| __riscv_zkne            | Arch Version | `Zkne` extension is available.                               |
| __riscv_zknh            | Arch Version | `Zknh` extension is available.                               |
| __riscv_zksed           | Arch Version | `Zksed` extension is available.                              |
| __riscv_zksh            | Arch Version | `Zksh` extension is available.                               |
| __riscv_zkt             | Arch Version | Target asserts `Zkt` (data-independent latency extension).   |
| __riscv_zkr             | Arch Version | `Zkr`  extension is available.                               |


### ABI Related Preprocessor Definitions

| Name                     | Value | When defined                  |
| ------------------------ | ----- | ----------------------------- |
| __riscv_abi_rve          | 1     | Defined if using `ilp32e` ABI |
| __riscv_float_abi_soft   | 1     | Defined if using `ilp32`, `ilp32e` or `lp64` ABI. |
| __riscv_float_abi_single | 1     | Defined if using `ilp32f` or `lp64f` ABI. |
| __riscv_float_abi_double | 1     | Defined if using `ilp32d` or `lp64d` ABI. |
| __riscv_float_abi_quad   | 1     | Defined if using `ilp32q` or `lp64q` ABI. |

### Code Model Related Preprocessor Definitions

| Name                  | Value    | When defined                          |
| --------------------- | -------- | ------------------------------------- |
| __riscv_cmodel_medlow | 1        | Defined if using `medlow` code model. |
| __riscv_cmodel_medany | 1        | Defined if using `medany` code model. |

### Deprecated Preprocessor Definitions

| Name                  | Value    | When defined                          | Alternative |
| --------------------- | -------- | ------------------------------------- | ----------- |
| __riscv_cmodel_pic    | 1        | GCC defines this when compiling with `-fPIC`, `-fpic`, `-fPIE` or `-fpie`. | `__PIC__` or `__PIE__` |
| __riscv_mul           | 1        | `M` extension is available.   | `__riscv_m` |
| __riscv_div           | 1        | `M` extension is available and `-mno-div` is not given.*[1]    | `__riscv_m` |
| __riscv_muldiv        | 1        | `M` extension is available and `-mno-div` is not given.*[1]    | `__riscv_m` |
| __riscv_atomic        | 1        | `A` extension is available.   | `__riscv_a` |
| __riscv_fdiv          | 1        | `F` extension is available and `-mno-fdiv` is not given.*[1]   | `__riscv_f` or `__riscv_d` |
| __riscv_fsqrt         | 1        | `F` extension is available and `-mno-fdiv` is not given.*[1]   | `__riscv_f` or `__riscv_d` |
| __riscv_compressed    | 1        | `C` extension is available.   | `__riscv_c` |
| __riscv_vector        | 1        | `V` extension is available.   | `__riscv_v` |
| __riscv_bitmanip      | 1        | `B` extension is available.   | `__riscv_b` |

*[1] Not all compilers provide `-mno-div` and `-mno-fdiv` option.

## Function Attributes

* `__attribute__((naked))`
* `__attribute__((interrupt))`
* `__attribute__((interrupt("user")))`
* `__attribute__((interrupt("supervisor")))`
* `__attribute__((interrupt("machine")))`


## Scalar Cryptography Extension Intrinsics

In order to access the RISC-V scalar crypto intrinsics, it is necessary to
include the header file `riscv_crypto.h`.

The functions are only only available if the compiler's `-march` string
enables the required ISA extension. (Calling functions for not enabled
ISA extensions will lead to compile-time and/or link-time errors.)

The prefixes and data types are:

* `_rv_*(...)`: intrinsics that operate on the `long` data type.
* `_rv32_*(...)`: intrinsics that operate on the `int32_t` data type.
* `_rv64_*(...)`: RV64-only intrinsics that operate on the `int64_t` data type.

The extensions `Zkt` or `Zkr` do not currently introduce intrinsics.
Data-independent latency is a property of the implementation, while
`Zkr` (the entropy source interface) is implemented as a CSR only
(`seed` at CSR address `0x015`).

Many of the `Zbkb` instructions can be inferred compilers so they do not
have direct intrinsics equivalents. Rotations are included mainly as RISC-V
sign extension behavior, and clamping of rotation amount may generate extra
instructions unless the programmer is very careful.

The `_rv64_sha512*` intrinsics are marked with optional [RV32] support.
This will require the compiler to decompose them into pairs of the
corresponding `_rv32_sha512*` instructions.

### Scalar Cryptography Extension short form intrinsics (alphabetically)

| Prototype                                                     | Mnemonic      | Short Description                         | Supported in                  |
| ------------------------------------------------------------- | ------------- | ----------------------------------------- | ----------------------------- |
| `int32_t _rv32_aes32dsi(int32_t rs1, int32_t rs2, int bs);`   | `aes32dsi`    | AES final round decryption / RV32.        | Zknd, Zkn, Zk (RV32)          |
| `int32_t _rv32_aes32dsmi(int32_t rs1, int32_t rs2, int bs);`  | `aes32dsmi`   | AES middle round decryption / RV32.       | Zknd, Zkn, Zk (RV32)          |
| `int32_t _rv32_aes32esi(int32_t rs1, int32_t rs2, int bs);`   | `aes32esi`    | AES final round encryption / RV32.        | Zkne, Zkn, Zk (RV32)          |
| `int32_t _rv32_aes32esmi(int32_t rs1, int32_t rs2, int bs);`  | `aes32esmi`   | AES middle round encryption / RV32.       | Zkne, Zkn, Zk (RV32)          |
| `int64_t _rv64_aes64ds(int64_t rs1, int64_t rs2);`            | `aes64ds`     | AES final round decryption / RV64.        | Zknd, Zkn, Zk (RV64)          |
| `int64_t _rv64_aes64dsm(int64_t rs1, int64_t rs2);`           | `aes64dsm`    | AES middle round decryption / RV64        | Zknd, Zkn, Zk (RV64)          |
| `int64_t _rv64_aes64es(int64_t rs1, int64_t rs2);`            | `aes64es`     | AES final round encryption / RV64.        | Zkne, Zkn, Zk (RV64)          |
| `int64_t _rv64_aes64esm(int64_t rs1, int64_t rs2);`           | `aes64esm`    | AES middle round encryption / RV64.       | Zkne, Zkn, Zk (RV64)          |
| `int64_t _rv64_aes64im(int64_t rs1);`                         | `aes64im`     | AES Inverse MixColumns, key schedule.     | Zknd, Zkn, Zk (RV64)          |
| `int64_t _rv64_aes64ks1i(int64_t rs1, int rnum);`             | `aes64ks1i`   | AES key schedule, round number.           | Zkne, Zknd, Zkn, Zk (RV64)    |
| `int64_t _rv64_aes64ks2(int64_t rs1, int64_t rs2);`           | `aes64ks2`    | AES key schedule, word mixing.            | Zkne, Zknd, Zkn, Zk (RV64)    |
| `int32_t _rv32_brev8(int32_t rs1);`                           | `brev8`       | Reverse order of bits within each byte.   | Zbkb (RV32)                   |
| `int64_t _rv64_brev8(int64_t rs1);`                           | `brev8`       | Reverse order of bits within each byte.   | Zbkb (RV64)                   |
| `int32_t _rv32_clmul(int32_t rs1, int32_t rs2);`              | `clmul`       | Carry-less multiply (low 32 bits).        | Zbc, Zbkc (RV32)              |
| `int64_t _rv64_clmul(int64_t rs1, int64_t rs2);`              | `clmul`       | Carry-less multiply (low 64 bits).        | Zbc, Zbkc (RV64)              |
| `int32_t _rv32_clmulh(int32_t rs1, int32_t rs2);`             | `clmulh`      | Carry-less multiply (high 32 bits).       | Zbc, Zbkc (RV32)              |
| `int64_t _rv64_clmulh(int64_t rs1, int64_t rs2);`             | `clmulh`      | Carry-less multiply (high 64 bits).       | Zbc, Zbkc (RV64)              |
| `int32_t _rv32_rol(int32_t rs1, int32_t rs2);`                | `rol[i][w]`   | Circular left rotate of 32 bits.          | Zbb, Zbkb (RV32,RV64)         |
| `int64_t _rv64_rol(int64_t rs1, int64_t rs2);`                | `rol`/`rori`  | Circular left rotate of 64 bits.          | Zbb, Zbkb (RV64)              |
| `int32_t _rv32_ror(int32_t rs1, int32_t rs2);`                | `ror[i][w]`   | Circular right rotate of 32 bits.         | Zbb, Zbkb (RV32,RV64)         |
| `int64_t _rv64_ror(int64_t rs1, int64_t rs2);`                | `ror[i]`      | Circular right rotate of 64 bits.         | Zbb, Zbkb (RV64)              |
| `long _rv_sha256sig0(long rs1);`                              | `sha256sig0`  | Sigma0 function for SHA2-256.             | Zknh, Zkn, Zk (RV32,RV64)     |
| `long _rv_sha256sig1(long rs1);`                              | `sha256sig1`  | Sigma1 function for SHA2-256.             | Zknh, Zkn, Zk (RV32,RV64)     |
| `long _rv_sha256sum0(long rs1);`                              | `sha256sum0`  | Sum0 function for SHA2-256.               | Zknh, Zkn, Zk (RV32,RV64)     |
| `long _rv_sha256sum1(long rs1);`                              | `sha256sum1`  | Sum1 function for SHA2-256.               | Zknh, Zkn, Zk (RV32,RV64)     |
| `int32_t _rv32_sha512sig0h(int32_t rs1, int32_t rs2);`        | `sha512sig0h` | Sigma0 high half for SHA2-512.            | Zknh, Zkn, Zk (RV32)          |
| `int32_t _rv32_sha512sig0l(int32_t rs1, int32_t rs2);`        | `sha512sig0l` | Sigma0 low half for SHA2-512.             | Zknh, Zkn, Zk (RV32)          |
| `int32_t _rv32_sha512sig1h(int32_t rs1, int32_t rs2);`        | `sha512sig1h` | Sigma1 high half for SHA2-512.            | Zknh, Zkn, Zk (RV32)          |
| `int32_t _rv32_sha512sig1l(int32_t rs1, int32_t rs2);`        | `sha512sig1l` | Sigma1 low half for SHA2-512.             | Zknh, Zkn, Zk (RV32)          |
| `int32_t _rv32_sha512sum0r(int32_t rs1, int32_t rs2);`        | `sha512sum0r` | Sum0 function for SHA2-512.               | Zknh, Zkn, Zk (RV32)          |
| `int32_t _rv32_sha512sum1r(int32_t rs1, int32_t rs2);`        | `sha512sum1r` | Sum1 function for SHA2-512.               | Zknh, Zkn, Zk (RV32)          |
| `int64_t _rv64_sha512sig0(int64_t rs1);`                      | `sha512sig0`  | Sigma0 function for SHA2-512.             | Zknh, Zkn, Zk (RV64 [RV32])   |
| `int64_t _rv64_sha512sig1(int64_t rs1);`                      | `sha512sig1`  | Sigma1 function for SHA2-512.             | Zknh, Zkn, Zk (RV64 [RV32])   |
| `int64_t _rv64_sha512sum0(int64_t rs1);`                      | `sha512sum0`  | Sum0 function for SHA2-512.               | Zknh, Zkn, Zk (RV64 [RV32])   |
| `int64_t _rv64_sha512sum1(int64_t rs1);`                      | `sha512sum1`  | Sum1 function for SHA2-512.               | Zknh, Zkn, Zk (RV64 [RV32])   |
| `long _rv_sm3p0(long rs1);`                                   | `sm3p0`       | P0 function for SM3 hash.                 | Zksh, Zks (RV32,RV64)         |
| `long _rv_sm3p1(long rs1);`                                   | `sm3p1`       | P1 function for SM3 hash.                 | Zksh, Zks (RV32,RV64)         |
| `long _rv_sm4ed(int32_t rs1, int32_t rs2, int bs);`           | `sm4ed`       | Accelerate SM4 cipher encrypt/decrypt.    | Zksed, Zks (RV32,RV64)        |
| `long _rv_sm4ks(int32_t rs1, int32_t rs2, int bs);`           | `sm4ks`       | Accelerate SM4 cipher key schedule.       | Zksed, Zks (RV32,RV64)        |
| `int32_t _rv32_unzip(int32_t rs1);`                           | `unzip`       | Odd/even bits into upper/lower halves.    | Zbkb (RV32)                   |
| `int32_t _rv32_xperm4(int32_t rs1, int32_t rs2);`             | `xperm4`      | Nibble-wise lookup of indices.            | Zbkx (RV32)                   |
| `int64_t _rv64_xperm4(int64_t rs1, int64_t rs2);`             | `xperm4`      | Nibble-wise lookup of indices.            | Zbkx (RV64)                   |
| `int32_t _rv32_xperm8(int32_t rs1, int32_t rs2);`             | `xperm8`      | Byte-wise lookup of indices.              | Zbkx (RV32)                   |
| `int64_t _rv64_xperm8(int64_t rs1, int64_t rs2);`             | `xperm8`      | Byte-wise lookup of indices.              | Zbkx (RV64)                   |
| `int32_t _rv32_zip(int32_t rs1);`                             | `zip`         | Upper/lower halves into odd/even bits.    | Zbkb (RV32)                   |

### Cryptography Intrinsics Implementation Guarantees

The `riscv_crypto.h` can implement the intrinsics in many ways
(early implementations used inline assembler). Builtin mapping is a
compiler and system specific issue.

Due to the data-independent latency ("constant time") assertions of
the `Zkt` extension, the header file or the compiler can't use table
lookups, conditional branching, etc., when implementing crypto intrinsics.
However, this approach has been used for functional validation.
In production (cryptographic implementations), the execution latency of
all cryptography intrinsics must be independent of input values.

