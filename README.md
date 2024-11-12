# bprm_loader

ELF binary program loader implementation

This module provides functionality for loading and executing ELF binaries, including:

+ Program execution (execve)
+ Dynamic linking suppor
+ Stack setup and argument passing
+ Memory mapping of program segments

Features

+ ELF binary parsing and loading
+ Dynamic interpreter (ld.so) support
+ Proper stack setup for user programs
+ Security checks and validations

Core Components

+ Program execution interface
+ ELF loader implementation
+ Stack and argument setup
+ Memory management integration

## Examples

```rust
#![no_std]
#![no_main]

#[macro_use]
extern crate axlog2;
extern crate alloc;

use core::panic::PanicInfo;
use alloc::vec;

/// The main entry point for monolithic kernel startup.
#[cfg_attr(not(test), no_mangle)]
pub extern "Rust" fn runtime_main(cpu_id: usize, dtb: usize) {
    init(cpu_id, dtb);
    start(cpu_id, dtb);
    panic!("Never reach here!");
}

pub fn init(cpu_id: usize, dtb: usize) {
    axlog2::init("info");
    bprm_loader::init(cpu_id, dtb);
    axtrap::init(cpu_id, dtb);
    task::alloc_mm();
}

pub fn start(_cpu_id: usize, _dtb: usize) {
    let filename = "/sbin/init";
    let args = vec![filename.into()];
    let (entry, sp) = bprm_loader::execve(filename, 0, args, vec![]).unwrap();

    // Todo: check entry and sp for ld.so
    panic!("Reach here! entry: {:#X}; sp: {:#X}", entry, sp);
}

#[panic_handler]
pub fn panic(info: &PanicInfo) -> ! {
    error!("{}", info);
    axhal::misc::terminate();
    #[allow(unreachable_code)]
    arch_boot::panic(info)
}
```

## Functions

### `execve`

```rust
pub fn execve(
    filename: &str,
    flags: usize,
    argv: Vec<String>,
    envp: Vec<String>
) -> LinuxResult<(usize, usize)>
```

executes a new program.

### `init`

```rust
pub fn init(cpu_id: usize, dtb_pa: usize)
```

Initialize the binary loader
