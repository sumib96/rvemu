# rvemu: RISC-V Online Emulataor
[![Build Status](https://travis-ci.com/d0iasm/rvemu.svg?branch=master)](https://travis-ci.com/d0iasm/rvemu)
[![Actions Status](https://github.com/d0iasm/rvemu/workflows/CI/badge.svg)](https://github.com/d0iasm/rvemu/actions)
[![docs.rs](https://docs.rs/rvemu/badge.svg)](https://docs.rs/rvemu)
[![crate.io](https://img.shields.io/crates/v/rvemu.svg)](https://crates.io/crates/rvemu)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/d0iasm/rvemu/master/LICENSE)

RISC-V online emulator with WebAssembly generated by Rust.

The online emulator is available here:
- [**rvemu.app**](https://rvemu.app/): Run a RISC-V binary you uploaded.
- [**rvemu.app/xv6**](https://rvemu.app/xv6.html): Run
  [`xv6`](https://github.com/mit-pdos/xv6-riscv).

The emulator supports RV64GC ISA (RV64IMAFD, Zicsr, Zifencei, RV64C), privileged
ISA, CSRs, virtual memory system (Sv39), peripheral devices (UART, CLINT, PLIC,
Virtio), and device tree. See
[the "Feature List" section]("https://github.com/d0iasm/rvemu#feature-list") for
the details of features. These features are compliant with "The RISC-V
Instruction Set Manual Volume I: Unprivileged ISADocument Version 20191213" and
"The RISC-V Instruction Set ManualVolume II: Privileged ArchitectureDocument
Version 20190608-Priv-MSU-Ratified" in
[the RISC-V specifications](https://riscv.org/specifications/).

## Usage
You can run [`xv6`](https://github.com/mit-pdos/xv6-riscv), a simple Unix-like
operating system, in [**rvemu.app/xv6**](https://rvemu.app/xv6.html).

![Demo](https://raw.githubusercontent.com/d0iasm/rvemu/master/demo.gif)

You also be able to run an arbitrary RISC-V binary in
[**rvemu.app**](https://rvemu.app/). The online emulator supports the following
commands:
- __upload__: Upload local RISC-V binaries for the execution on the emulator.
- __ls__: List the files you uploaded.
- __run [file]__: Execute a file which you uploaded or some files are already
  embedded.
- __help__: Print all commands you can use.

See
[the "Build RISC-V binary" section](https://github.com/d0iasm/rvemu#build-risc-v-binary)
for more information to build RISC-V binary.

## Build and run on the local browser
The `wasm-pack build` command generates a `pkg` directory and makes Rust source
code into `.wasm` binary. It also generates the JavaScript API for using our
Rust-generated WebAssembly. The toolchain's supported target is
`wasm32-unknown-unknown`. You need to execute this command whenever you change
your Rust code.
```
// This is the alias of `wasm-pack build lib/rvemu-wasm --out-dir <path-to-rvemu>/public/pkg --target web --no-typescript`.
$ make rvemu-wasm
```

This command installs dependencies in the `node_modules` directory. Need
`npm install --save` in the `public` directory at the first time and whenever
you change dependencies in package.json.
```
$ npm install --save // at the public directory
```

You can see the website via http://localhost:8000. `npm start` is the alias of
`python3 -m http.server` so you need Python3 in your environment.
```
$ npm start // at the public directory
```

## Build and run as a CLI tool
The emulator can be executed as a CLI tool too.
```
// This is the alias of `cargo build --release --manifest-path lib/rvemu-cli/Cargo.toml`.
$ make rvemu-cli
```

In order to execute a RISC-V binary, `xv6` in the folloing example, you can
use the `--kernel` or `-k` option to specify the kernel image. Note that
`xv6-kernel.text` is an ELF file without headers by the command
`riscv64-unknown-elf-objcopy -O binary kernel xv6-kernel.text`
```
$ ./target/release/rvemu-cli -k examples/xv6-kernel.text -f examples/xv6-fs.img
```

You can see the details of how to use by the `help` option.
```
$ ./target/release/rvemu-cli --help
rvemu: RISC-V emulator 0.0.1
Asami Doi <@d0iasm>

USAGE:
    rvemu-cli [FLAGS] [OPTIONS] --kernel <kernel>

FLAGS:
    -d, --debug      Enables to output debug messages
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -f, --file <file>        A raw disk image
    -k, --kernel <kernel>    A kernel ELF image without headers
```

## Feature List
The emulator supports the following features:
- [x] RV64G ISA
  - [x] RV64I (v2.1): supports 52/52 instructions (`fence` does nothing for now)
  - [x] RV64M (v2.0): supports 13/13 instructions
  - [x] RV64A (v2.1): supports 22/22 instructions (No atomicity for now)
  - [x] RV64F (v2.2): supports 30/30 instructions
  - [x] RV64D (v2.2): supports 32/32 instructions
  - [x] Zifencei (v2.0): supports 1/1 instructions (`fence.i` does nothing for now)
  - [x] Zicsr (v2.0): supports 6/6 instructions (No atomicity for now)
- [x] RV64C ISA (v2.0): support 36/36 instructions
- [x] Privileged ISA: supports 7/7 instructions (`sfence.vma`, `hfence.bvma`, and `hfence.gvma` do nothing for now)
- [x] Control and status registers (CSRs)
  - [x] Machine-level CSRs
  - [x] Supervisor-level CSRs
  - [ ] User-level CSRs
- [x] Virtual memory system (Sv39)
- [x] Devices
  - [x] UART: universal asynchronous receiver-transmitter
  - [x] CLINT: core local interruptor
  - [x] PLIC: platform level interrupt controller
  - [x] Virtio: virtual I/O
- [x] Device tree

## Build RISC-V binary
The emulator doesn't support C extensions yet, so make sure your RISC-V
toolchain only uses `RV64G`.
```
// In the riscv-gnu-toolchain directory.
// https://github.com/riscv/riscv-gnu-toolchain
$ ./configure --prefix=/opt/riscv --with-arch=rv64g
$ make
$ make linux
```

### Bare-metal C program
You need to make an ELF file without headers, which starts at the address
`0x8000_0000` by the following instructions:
```
// Make an assembly file from a C file.
$ riscv64-unknown-elf-gcc -S -nostdlib foo.c
// Make a binary file from an assembly file with start position 0x8000_0000.
$l riscv64-unknown-elf-gcc -Wl,-Ttext=0x80000000 -nostdlib -o foo foo.s
// Remove headers from a binary file.
$ riscv64-unknown-elf-objcopy -O binary foo foo.text
```

### Linux
#### Build vmlinux
```
wget https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/linux-5.6.7.tar.xz
tar -xf linux-5.6.7.tar.xz
cd linux-5.6.7
make ARCH=riscv defconfig
make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- -j $(nproc)
```

#### Get pre-built binaries
```
$ wget https://fedorapeople.org/groups/risc-v/disk-images/vmlinux
$ wget https://fedorapeople.org/groups/risc-v/disk-images/bbl
$ wget https://fedorapeople.org/groups/risc-v/disk-images/stage4-disk.img.xz
// Disk images.
$ xzdec -d stage4-disk.img.xz > stage4-disk.img
```

#### Get a device tree blob (dtb) from QEMU
```
$ qemu-system-riscv64 -nographic -machine virt,dumpdtb=virt.dtb

// See the content.
$ fdtdump virt.dtb
// Decompile a device tree blog to a device tree source.
$ dtc -I dtb -O dts -o virt.dts virt.dtb
// Compile a device tree source to a device tree blob.
$ dtc -I dts -O dtb -o virt.dtb virt.dts
```

## Testing
You can see the binaries for unit testings in
[riscv/riscv-tests](https://github.com/riscv/riscv-tests).
The following command executes all `rv64ua/d/f/i/m-p-*` binaries.
TODO: not all tests are passed for now.
```
$ make test
```

## Analyzing with perf
```
$ perf record -F99 --call-graph dwarf ./target/release/rvemu-cli -k examples/xv6-kernel.text -f examples/xv6-fs.img
$ perf report
```

## Publish
[The site](https://rvemu.app/) is hosted by a firebase.
```
$ firebase deploy
```

## Install
1. rustc
2. rustup nightly
3. wasm-pack
4. dtc (apt install device-tree-compiler)

## Dependencies
- [Nightly Rust](https://doc.rust-lang.org/1.2.0/book/nightly-rust.html)
- [Python3](https://www.python.org/downloads/)
- wasm-pack
- npm
  - [xterm](https://xtermjs.org/)
  - xterm-addon-fit
- dtc: device tree compiler

## Resources
### Documents
- [RISC-V Specifications](https://riscv.org/specifications/)
- [Rust and WebAssembly](https://rustwasm.github.io/docs/book/introduction.html)
- [riscv/riscv-sbi-doc](https://github.com/riscv/riscv-sbi-doc/blob/master/riscv-sbi.adoc)
- [riscv/riscv-elf-psabi-doc](https://github.com/riscv/riscv-elf-psabi-doc/blob/master/riscv-elf.md)
- [riscv/riscv-asm-manual](https://github.com/riscv/riscv-asm-manual/blob/master/riscv-asm.md)

### Implementation of other emulators
- [qemu/qemu](https://github.com/qemu/qemu)
- [riscv/riscv-isa-sim](https://github.com/riscv/riscv-isa-sim)
- [riscv/riscv-angel](https://github.com/riscv/riscv-angel)
- [riscv/riscv-ovpsim](https://github.com/riscv/riscv-ovpsim)
- [rv8-io/rv8](https://github.com/rv8-io/rv8)
- [TinyEmu](https://bellard.org/tinyemu/)
- [stephank/rvsim](https://github.com/stephank/rvsim)

### Helpful tools
- [riscv/riscv-tests](https://github.com/riscv/riscv-tests)
- [wat2wasm demo](https://webassembly.github.io/wabt/demo/wat2wasm/)
- [RISC-V Online Simulator](https://www.kvakil.me/venus/)

## Articles about this project
- [Emulate 32-Bit And 64-Bit RISC-V In Your Browser With Asami’s Open Source rvemu | Gareth Halfacree, Hackster.io](https://riscv.org/2020/01/emulate-32-bit-and-64-bit-risc-v-in-your-browser-with-asamis-open-source-rvemu-gareth-halfacree-hackster-io/)
- [Emulate 32-Bit and 64-Bit RISC-V in Your Browser with Asami's Open Source rvemu](https://www.hackster.io/news/emulate-32-bit-and-64-bit-risc-v-in-your-browser-with-asami-s-open-source-rvemu-b783f672e463)
