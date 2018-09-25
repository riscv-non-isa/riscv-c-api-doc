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
* `-mstrict-align` '-mno-strict-align`
* `-mfdiv` `-mno-fdiv`
* `-mdiv` `-mno-div`
* `-mpreferred-stack-boundary=N`
* `-msmall-data-limit=N`
* `-mexplicit-relocs` '-mno-explicit-relocs`
* `-mrelax` `-mno-relax`
* `-msave-restore` `-mno-save-restore`
* `-mbranch-cost=N`

## Preprocessor Definitions

* `__riscv`
* `__riscv_32e`
* `__riscv_xlen`
* `__riscv_flen`
* `__riscv_atomic`
* `__riscv_compressed`
* `__riscv_mul`
* `__riscv_div`
* `__riscv_muldiv`
* `__riscv_fdiv`
* `__riscv_fsqrt`
* `__riscv_float_abi_soft`
* `__riscv_float_abi_single`
* `__riscv_float_abi_double`
* `__riscv_float_abi_quad`
* `__riscv_cmodel_medlow`
* `__riscv_cmodel_medany`
* `__riscv_cmodel_pic`

## Function Attributes

* `__attribute__((naked))`
* `__attribute__((interrupt))`
* `__attribute__((interrupt("user")))`
* `__attribute__((interrupt("supervisor")))`
* `__attribute__((interrupt("machine")))`

## Intrinsic Functions

Do we really have none of these?  I can't figure out
`gcc/gcc/config/riscv/riscv-builtins.c`...
