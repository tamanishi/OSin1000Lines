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

### https://operating-system-in-1000-lines.vercel.app/ja/05-hello-world

- `kernel.c`に`boot`関数を残しておかなければならなかった。  
まぁエントリポイントなので当然。  
なのでリストにも載っていなかったのかな。。
  
  ``` c
  #include "kernel.h"

  extern char __bss[], __bss_end[], __stack_top[];

  struct sbiret sbi_call(long arg0, long arg1, long arg2, long arg3, long arg4,
                        long arg5, long fid, long eid) {
      register long a0 __asm__("a0") = arg0;
      register long a1 __asm__("a1") = arg1;
      register long a2 __asm__("a2") = arg2;
      register long a3 __asm__("a3") = arg3;
      register long a4 __asm__("a4") = arg4;
      register long a5 __asm__("a5") = arg5;
      register long a6 __asm__("a6") = fid;
      register long a7 __asm__("a7") = eid;

      __asm__ __volatile__("ecall"
                          : "=r"(a0), "=r"(a1)
                          : "r"(a0), "r"(a1), "r"(a2), "r"(a3), "r"(a4), "r"(a5),
                            "r"(a6), "r"(a7)
                          : "memory");
      return (struct sbiret){.error = a0, .value = a1};
  }

  void putchar(char ch) {
      sbi_call(ch, 0, 0, 0, 0, 0, 0, 1 /* Console Putchar */);
  }

  void kernel_main(void) {
      const char *s = "\n\nHello World!\n";
      for (int i = 0; s[i] != '\0'; i++) {
          putchar(s[i]);
      }

      for (;;) {
          __asm__ __volatile__("wfi");
      }
  }

  __attribute__((section(".text.boot")))
  __attribute__((naked))
  void boot(void) {
      __asm__ __volatile__(
          "mv sp, %[stack_top]\n"
          "j kernel_main\n"
          :
          : [stack_top] "r" (__stack_top)
      );
  }
  ```

