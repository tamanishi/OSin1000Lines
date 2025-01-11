# OSin1000Lines

- [OS in 1,000 Lines](https://operating-system-in-1000-lines.vercel.app/ja/) を試してみた。

## 備忘

### https://operating-system-in-1000-lines.vercel.app/ja/04-boot

- `llvm-objdump`が使えなかった。
  - 「代替に`objdump`を使えばよい」とあったので試してみたが、それはXcodeに付属のものだったのでrisc-vをサポートしていなった。

  ``` sh
  ❯ objdump --version
  Apple LLVM version 16.0.0
  (clang-1600.0.26.6)Optimized build.


  Registered Targets:
  aarch64    - AArch64 (little endian)
  aarch64_32 - AArch64 (little endian ILP32)
  aarch64_be - AArch64 (big endian)
  arm        - ARM
  arm64      - ARM64 (little endian)
  arm64_32   - ARM64 (little endian ILP32)
  armeb      - ARM (big endian)
  thumb      - Thumb
  thumbeb    - Thumb (big endian)
  x86        - 32-bit X86: Pentium-Pro and above
  x86-64     - 64-bit X86: EM64T and AMD64
  ```

  - `/opt/homebrew/opt/llvm/bin`にパスが通っていなかっただけだった。。

  ``` sh
  ❯ which llvm-objdump
  /opt/homebrew/opt/llvm/bin/llvm-objdump
  OSin1000Lines on  main [?]
  ❯ llvm-objdump -d kernel.elf

  kernel.elf:     file format elf32-littleriscv

  Disassembly of section .text:

  80200000 <boot>:
  80200000: 80220537      lui     a0, 0x80220
  80200004: 04c50513      addi    a0, a0, 0x4c
  80200008: 812a          mv      sp, a0
  8020000a: 01a0006f      j       0x80200024 <kernel_main>

  ...

  ```
