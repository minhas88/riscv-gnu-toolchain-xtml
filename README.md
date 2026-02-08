# RISC-V GNU Toolchain with TML Extensions

This directory contains the modified RISC-V GNU toolchain with support for TML (Tile-based Matrix Extension) custom instructions.

## Structure

```
toolchain/
└── riscv-gnu-toolchain/
    ├── binutils/           # Assembler/disassembler (TML modifications)
    ├── gcc/                # Compiler (TML modifications)
    ├── newlib/             # C library for bare-metal
    ├── linux-headers/      # Linux kernel headers
    ├── contrib/            # Contributions
    ├── regression/         # Regression tests
    ├── scripts/            # Build and test scripts
    ├── test/               # Test suite and allowlists
    ├── configure           # Top-level configure
    ├── Makefile.in         # Top-level Makefile
    └── build-*/            # Build directories (created by make, regenerable)
        ├── build-binutils-newlib/
        ├── build-gcc-newlib-stage1/
        ├── build-gcc-newlib-stage2/
        ├── build-newlib/
        └── build-newlib-nano/
```

## TML Modifications

### Binutils (4 files modified)

| File | Purpose |
|------|---------|
| `binutils/include/opcode/riscv-opc.h` | Instruction match/mask patterns |
| `binutils/include/opcode/riscv.h` | Instruction class enum |
| `binutils/opcodes/riscv-opc.c` | Opcode table entries |
| `binutils/bfd/elfxx-riscv.c` | Extension support and parsing |

### GCC (8 files modified)

| File | Purpose |
|------|---------|
| `gcc/gcc/config/riscv/riscv.opt` | CLI option definition |
| `gcc/gcc/config/riscv/riscv-opts.h` | Target flag macro |
| `gcc/gcc/config/riscv/riscv.cc` | Storage and handling |
| `gcc/gcc/config/riscv/riscv-c.cc` | Preprocessor macros |
| `gcc/gcc/config/riscv/riscv-builtins.cc` | Builtin definitions |
| `gcc/gcc/config/riscv/riscv-ftypes.def` | Function type signatures |
| `gcc/gcc/config/riscv/riscv.md` | Machine description patterns |
| `gcc/gcc/common/config/riscv/riscv-common.cc` | Extension registration |

## Building the Toolchain

From the `riscv-gnu-toolchain` directory:

```bash
cd riscv-gnu-toolchain
./configure --prefix=/opt/riscv --with-arch=rv32im --with-abi=ilp32
make -j$(nproc)
```

Install (optional):

```bash
make install
```

Ensure the install prefix (e.g. `/opt/riscv`) is on your PATH so that `riscv32-unknown-elf-gcc`, `riscv32-unknown-elf-as`, and `riscv32-unknown-elf-objcopy` are available.

## Using TML Instructions

Assembler (assembly source):

```bash
riscv32-unknown-elf-as -march=rv32im_zicsr_zifencei_xtml1p0 -mabi=ilp32 test.S -o test.o
```

Compiler (C source, with TML builtins):

```bash
riscv32-unknown-elf-gcc -march=rv32im_zicsr_zifencei_xtml1p0 -mabi=ilp32 -mxtml -O2 test.c -o test.elf
```

For VexRiscv simulation (bare-metal, no stdlib):

```bash
riscv32-unknown-elf-gcc -march=rv32im_zicsr_zifencei_xtml1p0 -mabi=ilp32 -mxtml -O2 -nostdlib -nostartfiles -Ttext=0x80000000 -o test.elf test.c
```

The hardware simulation Makefile in `hardware/VexRiscv/sim/` uses the same `-march` and `-mxtml` flags.

## Size

- **Total size:** about 15 GB (with build directories).
- **Source only:** about 1.7 GB (binutils, gcc, newlib).
- **Build directories:** about 8.4 GB (can be regenerated with `make`).

## Build Directories

The `build-*` directories hold compiled toolchain components. They can be:

- **Kept:** avoids full rebuild (GCC rebuild is on the order of 30–60 minutes).
- **Removed:** frees about 8.4 GB; run `make` again to regenerate.

## Documentation

- **TML toolchain extension (methodology and usage):** `docs/methodology/TOOLCHAIN_EXTENSION_METHODOLOGY.md`
- **Broader toolchain integration (GMX/TML):** `docs/guides/GMX_TOOLCHAIN_INTEGRATION_GUIDE.md`
