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
| __riscv_vector      | 1     | Implies that any of the vector extensions (`v` or `zve*`) is available |
| __riscv_v_min_vlen    | <N> (see [__riscv_v_min_vlen](#__riscv_v_min_vlen)) | The `V` extension or one of the `Zve*` extensions is available. |
| __riscv_v_elen     | <N> (see [__riscv_v_elen](#__riscv_v_elen)) | The `V` extension or one of the `Zve*` extensions is available. |
| __riscv_v_elen_fp  | <N> (see [__riscv_v_elen_fp](#__riscv_v_elen_fp)) | The `V` extension or one of the `Zve*` extensions is available. |
| __riscv_misaligned_fast | 1  | Misaligned accesses are fast. |
| __riscv_misaligned_slow | 1  | Misaligned accesses are supported, but may be substantially slower than aligned accesses. |
| __riscv_misaligned_avoid | 1  | Misaligned accesses are not supported and could trap. (see [ __riscv_misaligned_{fast,slow,avoid}](#__riscv_misaligned_{fast,slow,avoid}) |

### __riscv_v_min_vlen

The `__riscv_v_min_vlen` macro expands to the minimal VLEN, in bits, mandated
by the available vector extension, if any.

The value of `__riscv_v_min_vlen` is defined by the following rules:
- 128, if the `V` extension is present;
- 32, if one of the `Zve32{x,f}` extensions is present;
- 64, if one of the `Zve64{x,f,d}` extensions is present;
- `N`, if one of the `Zvl<N>b` extensions, `N` in `{32,64,128,256,512,1024}`,
is present.

If multiple rules apply, the maximum value is taken.
If none of the rules apply, `__riscv_v_min_vlen` is undefined.

Examples:

- `__riscv_v_min_vlen` is 128 for `rv64gcv`
- `__riscv_v_min_vlen` is 512 for `rv32gcv_zvl512b`
- `__riscv_v_min_vlen` is 256 for `rv32gcv_zvl32b_zvl256b`
- `__riscv_v_min_vlen` is 128 for `rv64gcv_zvl32b`

### __riscv_v_elen

The `__riscv_v_elen` macro expands to the supported element length, in bits,
of any non-floating-point vector operand of any vector instruction in the
available vector extension, if any. (Stricter upper bounds may apply to
particular operands of particular instructions.)


The value of `__riscv_v_elen` is defined by the following rules:
- 64, if the `V` extension or one of the `Zve64{x,f,d}` extensions is present; and
- 32, if one of the `Zve32{x,f}` extensions is present.
If multiple rules apply, the maximum value is taken.
If none of the rules apply, `__riscv_v_elen` is undefined.

### __riscv_v_elen_fp

The `__riscv_v_elen_fp` macro expands to the supported element length, in bits,
of any floating-point vector operand of any vector instruction in the available
vector extension, if any. (Stricter upper bounds may apply to particular
operands of particular instructions.)

The value of `__riscv_v_elen_fp` is defined by the following rules:
- 64, if one of the `V` or `Zve64d` extensions is present;
- 32, if one of the `Zve{32,64}f` extensions is present; and
- 0, if one of the `Zve{32,64}x` extensions is present.
If multiple rules apply, the maximum value is taken.
If none of the rules apply, `__riscv_v_elen_fp` is undefined.

### __riscv_misaligned_{fast,slow,avoid}

These can be used in common library code to compile time segregate code which relies
on misaligned access being fast or not.
A typical complier could (but not necessarily) map fast variant to -mno-strict-align
and avoid to -mstrict-align, if specified.
Perhaps obvious, but these are mutually exclusive, so only one is defined at a time
for a compilation unit.


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

| Name                    | Value        | When defined                  |
| ----------------------- | ------------ | ----------------------------- |
| __riscv_arch_test       | 1            | Defined if compiler support new architecture extension test macro. |
| __riscv_i               | Arch Version | `I` extension is available.   |
| __riscv_e               | Arch Version | `E` extension is available.   |
| __riscv_m               | Arch Version | `M` extension is available.   |
| __riscv_a               | Arch Version | `A` extension is available.   |
| __riscv_f               | Arch Version | `F` extension is available.   |
| __riscv_d               | Arch Version | `D` extension is available.   |
| __riscv_c               | Arch Version | `C` extension is available.   |
| __riscv_p               | Arch Version | `P` extension is available.   |
| __riscv_v               | Arch Version | `V` extension is available.   |
| __riscv_zawrs           | Arch Version | `Zawrs` extension is available. |
| __riscv_zba             | Arch Version | `Zba` extension is available. |
| __riscv_zbb             | Arch Version | `Zbb` extension is available. |
| __riscv_zbc             | Arch Version | `Zbc` extension is available. |
| __riscv_zbs             | Arch Version | `Zbs` extension is available. |
| __riscv_zfh             | Arch Version | `Zfh` extension is available. |

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

*[1] Not all compilers provide `-mno-div` and `-mno-fdiv` option.

## Function Attributes

### `__attribute__((naked))`

The compiler won't generate the prologue/epilogue for those functions with
`naked` attributes. This attribute is usually used when you want to write a
function with an inline assembly body.

This attribute is incompatible with the `interrupt` attribute.

NOTE: Be aware that compilers might have further restrictions on naked
functions. Please consult your compiler's manual for more information.

### `__attribute__((interrupt))`, `__attribute__((interrupt("user")))`, `__attribute__((interrupt("supervisor")))` `__attribute__((interrupt("machine")))`

The interrupt attribute specifies that a function is an interrupt handler.
The compiler will save/restore all used registers in the prologue/epilogue
regardless of the ABI, all used registers including floating point
register/vector register if `F` extension/vector extension is enabled.

The interrupt attribute can have an optional parameter to specify the mode.
The possible values are `user`, `supervisor`, or `machine`.
The default value `machine` is used, if the mode is not specified.

The function can specify only one mode; the compiler should raise an error if a
function declares more than one mode or an undefined mode.

This attribute is incompatible with the `naked` attribute.

## Intrinsic Functions

Intrinsic functions (or intrinsics or built-ins) are expanded into instruction sequences by compilers.
They typically provide access to functionality that is otherwise not synthesizable by compilers.
Some intrinsics expand to different code sequences depending on the available instructions from the enabled ISA extensions.

Compilers typically come with their own architecture-independent intrinsics (e.g. synchronization primitives, byte-swap, etc.).
The RISC-V compiler backend can define additional target-specific intrinsics.
Providing functionality via architecture-independent intrinsics is the preferred method, as it improves code portability.

Some intrinsics are only available if a particular header file is included.
RISC-V header files that enable intrinsics require the prefix `riscv_` (e.g. `riscv_vector.h` or `riscv_crypto.h`).

RISC-V specific intrinsics use the common prefix `__riscv_` to avoid namespace collisions.

The intrinsic name describes the functional behaviour of the function.
In case the functionality can be expressed with a single instruction, the instruction's name (any '.' replaced by '_') is the preferred choice.
Note, that intrinsics that are restricted to RISC-V vendor extensions need to include the vendor prefix (as documented in the RISC-V toolchain conventions).

If intrinsics are available for multiple data types, then function overloading is preferred over multiple type-specific functions.
In case a function is only available for one data type and this type cannot be derived from the function's name, then the type should be appended to the function name, delimited by a '_' character.
Typical type postfixes are "32" (32-bit), "i32" (signed 32-bit), "i8m4" (vector register group consisting of 4 signed 8-bit vector registers).

RISC-V intrinsics follow the following naming rule:

```
INTRINSIC ::= PREFIX NAME [ '_' TYPE ]
PREFIX ::= "__riscv_"
NAME ::= Name of the intrinsic function.
TYPE ::= Optional type postfix.
```

RISC-V intrinsics examples:

```
#include <riscv_vector.h> // make RISC-V vector intrinsics available
vint8m1_t __riscv_vadd_vv_i8m1(vint8m1_t vs2, vint8m1_t vs1, size_t vl); // vadd.vv vd, vs2, vs1
```

### NTLH Intrisics 

The RISC-V zihintntl extension provides the RISC-V specific intrinsic functions for generating non-temporal memory accesses. These intrinsic functions provide the domain parameter to specify the behavior of memory accesses.

In order to access the RISC-V NTLH intrinsics, it is necessary to
include the header file `riscv_ntlh.h`.

The functions are only available if the compiler enables the zihintntl extension.

```
type __riscv_ntl_load (type *ptr, int domain);
void __riscv_ntl_store (type *ptr, type val, int domain);
```

There are overloaded functions of `__riscv_ntl_load` and `__riscv_ntl_store`. When these intrinsic functions omit the `domain` argument, the `domain` is implied as `__RISCV_NTLH_ALL`.

```
type __riscv_ntl_load (type *ptr);
void __riscv_ntl_store (type *ptr, type val);
```

The types currently supported are:

- Integer types.
- Floating-point types.
- Fixed-length vector types.

The `domain` parameter could pass the following values. Each one is mapped to the specific zihintntl instruction.

```
enum {
  __RISCV_NTLH_INNERMOST_PRIVATE = 2,
  __RISCV_NTLH_ALL_PRIVATE,
  __RISCV_NTLH_INNERMOST_SHARED,
  __RISCV_NTLH_ALL
};
```

| Domain Value                     | Instruction | 
| ---------------------------------| ------------| 
| `__RISCV_NTLH_INNERMOST_PRIVATE` | `ntl.p1`    |
| `__RISCV_NTLH_ALL_PRIVATE`       | `ntl.pall`  |
| `__RISCV_NTLH_INNERMOST_SHARED`  | `ntl.s1`    |
| `__RISCV_NTLH_ALL`               | `ntl.all`   |

### Scalar Cryptography Extension Intrinsics

In order to access the RISC-V scalar crypto intrinsics, it is necessary to
include the header file `riscv_crypto.h`.

The functions are only only available if the compiler's `-march` string
enables the required ISA extension. (Calling functions for not enabled
ISA extensions will lead to compile-time and/or link-time errors.)

Intrinsics operating on XLEN sized value are not available as there is no type
defined. If `xlen_t` is added in the future, this can be revisited. It is
assumed that cryptographic code will be written using fixed bit widths.

Unsigned types are used as that is the most logical representation for a
collection of bits.

Only 32-bit and 64-bit types are supported. In order to increase
compatibility, where it is feasible 32-bit intrinsics will be available on RV64.
This will sometimes require additional instructions.

No type overloading is supported. This avoids complications from C integer
promotion rules and how to handle signed types.

Sign extension of 32-bit values on RV64 is not reflected in the interface.

| Prototype                                                               | Instruction          | Extension         | Notes |
| ---------                                                               | -----------          | ---------         | ----- |
| `uint32_t __riscv_ror_32(uint32_t x, uint32_t shamt);`                  | `ror[i][w]`          | Zbb, Zbkb         | |
| `uint64_t __riscv_ror_64(uint64_t x, uint32_t shamt);`                  | `ror[i]`             | Zbb, Zbkb (RV64)  | |
| `uint32_t __riscv_rol_32(uint32_t x, uint32_t shamt);`                  | `rol[w]`/`ror[i][w]` | Zbb, Zbkb         | |
| `uint64_t __riscv_rol_64(uint64_t x, uint32_t shamt);`                  | `rol`/`rori`         | Zbb, Zbkb (RV64)  | |
| `uint32_t __riscv_rev8_32(uint32_t x);`                                 | `rev8`               | Zbb, Zbkb         | Emulated with `rev8`+`srai` on RV64 |
| `uint64_t __riscv_rev8_64(uint64_t x);`                                 | `rev8`               | Zbb, Zbkb (RV64)  | |
| `uint32_t __riscv_brev8_32(uint32_t x);`                                | `brev8`              | Zbkb              | Emulated with `clmul`+`sext.w` on RV64 |
| `uint64_t __riscv_brev8_64(uint64_t x);`                                | `brev8`              | Zbkb (RV64)       | |
| `uint32_t __riscv_zip_32(uint32_t x);`                                  | `zip`                | Zbkb (RV32)       | No emulation for RV64 |
| `uint32_t __riscv_unzip_32(uint32_t x);`                                | `unzip`              | Zbkb (RV32)       | No emulation for RV64 |
| `uint32_t __riscv_clmul_32(uint32_t x);`                                | `clmul`              | Zbc, Zbkc         | Emulated with `clmul`+`sext.w` on RV64 |
| `uint64_t __riscv_clmul_64(uint64_t x);`                                | `clmul`              | Zbc, Zbkc (RV64)  |                                  |
| `uint32_t __riscv_clmulh_32(uint32_t x);`                               | `clmulh`             | Zbc, Zbkc (RV32)  | Emulation on RV64 requires 4-6 instructions |
| `uint64_t __riscv_clmulh_64(uint64_t x);`                               | `clmulh`             | Zbc, Zbkc (RV64)  | |
| `uint32_t __riscv_xperm4_32(uint32_t rs1, uint32_t rs2);`               | `xperm4`             | Zbkx (RV32)       | No emulation for RV64 |
| `uint64_t __riscv_xperm4_64(uint64_t rs1, uint64_t rs2);`               | `xperm4`             | Zbkx (RV64)       | |
| `uint32_t __riscv_xperm8_32(uint32_t rs1, uint32_t rs2);`               | `xperm8`             | Zbkx (RV32)       | No emulation for RV64 |
| `uint64_t __riscv_xperm8_64(uint64_t rs1, uint64_t rs2);`               | `xperm8`             | Zbkx (RV64)       | |
| `uint32_t __riscv_aes32dsi(uint32_t rs1, uint32_t rs2, const int bs);`  | `aes32dsi`           | Zknd (RV32)       | `bs`=[0..3] |
| `uint32_t __riscv_aes32dsmi(uint32_t rs1, uint32_t rs2, const int bs);` | `aes32dsmi`          | Zknd (RV32)       | `bs`=[0..3] |
| `uint64_t __riscv_aes64ds(uint64 rs1, uint64_t rs2);`                   | `aes64ds`            | Zknd (RV64)       | |
| `uint64_t __riscv_aes64dsm(uint64 rs1, uint64_t rs2);`                  | `aes64dsm`           | Zknd (RV64)       | |
| `uint64_t __riscv_aes64im(uint64 rs1);`                                 | `aes64im`            | Zknd (RV64)       | `rnum`=[0..10] |
| `uint64_t __riscv_aes64ks1i(uint64 rs1, const int rnum);`               | `aes64ks1i`          | Zknd, Zkne (RV64) | `rnum`=[0..10] |
| `uint64_t __riscv_aes64ks2(uint64 rs1, uint64_t rs2);`                  | `aes64ks2`           | Zknd, Zkne (RV64) | |
| `uint32_t __riscv_aes32esi(uint32_t rs1, uint32_t rs2, const int bs);`  | `aes32esi`           | Zkne (RV32)       | `bs`=[0..3] |
| `uint32_t __riscv_aes32esmi(uint32_t rs1, uint32_t rs2, const int bs);` | `aes32esmi`          | Zkne (RV32)       | `bs`=[0..3] |
| `uint64_t __riscv_aes64es(uint64 rs1, uint64_t rs2);`                   | `aes32es`            | Zkne (RV64)       | |
| `uint64_t __riscv_aes64esm(uint64 rs1, uint64_t rs2);`                  | `aes32esm`           | Zkne (RV64)       | |
| `uint32_t __riscv_sha256sig0(uint32_t rs1);`                            | `sha256sig0`         | Zknh              | |
| `uint32_t __riscv_sha256sig1(uint32_t rs1);`                            | `sha256sig1`         | Zknh              | |
| `uint32_t __riscv_sha256sum0(uint32_t rs1);`                            | `sha256sum0`         | Zknh              | |
| `uint32_t __riscv_sha256sum1(uint32_t rs1);`                            | `sha256sum1`         | Zknh              | |
| `uint32_t __riscv_sha512sig0h(uint32_t rs1, uint32_t rs2);`             | `sha512sig0h`        | Zknh (RV32)       | |
| `uint32_t __riscv_sha512sig0l(uint32_t rs1, uint32_t rs2);`             | `sha512sig0l`        | Zknh (RV32)       | |
| `uint32_t __riscv_sha512sig1h(uint32_t rs1, uint32_t rs2);`             | `sha512sig1h`        | Zknh (RV32)       | |
| `uint32_t __riscv_sha512sig1l(uint32_t rs1, uint32_t rs2);`             | `sha512sig1l`        | Zknh (RV32)       | |
| `uint32_t __riscv_sha512sum0h(uint32_t rs1, uint32_t rs2);`             | `sha512sum0h`        | Zknh (RV32)       | |
| `uint32_t __riscv_sha512sum0l(uint32_t rs1, uint32_t rs2);`             | `sha512sum0l`        | Zknh (RV32)       | |
| `uint32_t __riscv_sha512sum1h(uint32_t rs1, uint32_t rs2);`             | `sha512sum1h`        | Zknh (RV32)       | |
| `uint32_t __riscv_sha512sum1l(uint32_t rs1, uint32_t rs2);`             | `sha512sum1l`        | Zknh (RV32)       | |
| `uint64_t __riscv_sha512sig0(uint64_t rs1);`                            | `sha512sig0`         | Zknh (RV64)       | |
| `uint64_t __riscv_sha512sig1(uint64_t rs1);`                            | `sha512sig1`         | Zknh (RV64)       | |
| `uint64_t __riscv_sha512sum0(uint64_t rs1);`                            | `sha512sum0`         | Zknh (RV64)       | |
| `uint64_t __riscv_sha512sum1(uint64_t rs1);`                            | `sha512sum1`         | Zknh (RV64)       | |
| `uint32_t __riscv_sm3p0(uint32_t rs1);`                                 | `sm3p0`              | Zksh              | |
| `uint32_t __riscv_sm3p1(uint32_t rs1);`                                 | `sm3p1`              | Zksh              | |
| `uint32_t __riscv_sm4ed(uint32_t rs1, uint32_t rs2, const int bs);`     | `sm4ed`              | Zksed             | `bs`=[0..3] |
| `uint32_t __riscv_sm4ks(uint32_t rs1, uint32_t rs2, const int bs);`     | `sm4ks`              | Zksed             | `bs`=[0..3] |

### Cryptography Intrinsics Implementation Guarantees

The `riscv_crypto.h` can implement the intrinsics in many ways
(early implementations used inline assembler). Builtin mapping is a
compiler and system specific issue.

Due to the data-independent latency ("constant time") assertions of
the `Zkt` extension, the header file or the compiler can't use table
lookups, conditional branching, etc., when implementing crypto intrinsics.
In production (cryptographic implementations), the execution latency of
all cryptography intrinsics must be independent of input values.

## Constraints on Operands of Inline Assembly Statements

This section lists operand constraints that can be used with inline assembly
statements, including both RISC-V specific and common operand constraints.


| Constraint         |                                    | Note        |
| ------------------ | ---------------------------------- | ----------- |
| m                  | An address that is held in a general-purpose register with offset.      |      |
| A                  | An address that is held in a general-purpose register.      |      |
| r                  | General purpose register                      |      |
| f                  | Floating-point register                       |      |
| i                  | Immediate integer operand                     |      |
| I                  | 12-bit signed immediate integer operand      |      |
| K                  | 5-bit unsigned immediate integer operand     |      |
| J                  | Zero integer immediate operand                |      |
| vr                 | Vector register                    |            |
| vd                 | Vector register, excluding v0      |            |
| vm                 | Vector register, only v0           |            |

NOTE: Immediate value must be a compile-time constant.

### The Difference Between `m` and `A` Constraints

The difference between `m` and `A` is whether the operand can have an offset;
some instructions in RISC-V do not allow an offset for the address operand,
such as atomic or vector load/store instructions.

The following example demonstrates the difference; it is trying
to load value from `foo[10]` and using `m` and `A` to pass that address.

```c
int *foo;
void bar() {
 int x;
 __asm__ volatile ("lw %0, %1" : "=r"(x) : "m" (foo[10]));
 __asm__ volatile ("lw %0, %1" : "=r"(x) : "A" (foo[10]));
}
```

Then we compile with GCC with `-O` option:

```shell
$ riscv64-unknown-elf-gcc x.c -o - -O -S
...
bar:
       lui    a5,%hi(foo)
       ld     a5,%lo(foo)(a5)
 #APP
# 4 "x.c" 1
       lw a4, 40(a5)
# 0 "" 2
 #NO_APP
       addi   a5,a5,40
 #APP
# 5 "x.c" 1
       lw a5, 0(a5)
# 0 "" 2
 #NO_APP
       ret

```

The compiler uses an immediate offset of 40 for the `m` constraint, but for the
`A` constraint uses an extra addi instruction instead.

### Operand Modifiers

This section lists operand modifiers that can be used with inline assembly
statements, including both RISC-V specific and common operand modifiers.

| Modifiers    | Description                                                                       | Note        |
| ------------ | --------------------------------------------------------------------------------- | ----------- |
| z            | Print `zero` (`x0`) register for immediate 0, typically used with constraints `J` |             |
| i            | Print `i` if corresponding operand is immediate.                                  |             |
