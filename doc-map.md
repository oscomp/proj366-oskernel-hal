---
title: OSKernel-HAL
markmap:
  colorFreezeLevel: 2
  initialExpandLevel: 2
---

## Intro

- x86_64
  - 在 multiboot 的场景下，需要手动调整进入 long mode
- riscv64
  - riscv 在 S 态没有单独获取 hartid 的方法，只能在进入内核的时候存储 Boot 函数传递的信息，
- aarch64
- loongarch64


## Boot

- x86_64
  - enter lond mode(64 bit)
  - init gdt
  - init idt
  - open paging mode (required)
- riscv64
- aarch64
- loongarch64

## Trap
### TrapFrame

- x86_64
- riscv64
- aarch64
- loongarch64

### TrapHandler

- x86_64
  - idt
- riscv64
  - stvec
- aarch64
  - VBAR_EL1
- loongarch64
  - eentry
  - tlbrentry


## IRQ Control

- x86_64
  - lapic
  - idt
  - cli(关中断)
  - sti(开中断)
- riscv64
  - plic
  - sstatus sie bit
- aarch64
  - gic
  - daifset
- loongarch64
  - estat is bits
  - crmd ie bit

## CPU Features

## PageTable

### x86_64

- 4 级页表
- 硬件页翻译
- cr4

### aarch64

- 4 级页表
- 可配置为三级页表
- 硬件页翻译
- TTBR0 (低地址空间)
- TTBR1 (高地址空间)

### riscv64

- 3 级页表（sv39）
- 4 级页表（sv48）
- 硬件页翻译
- satp

### Loongarch64

- 4 级页表
- 页级数可以自由控制
- 暂时只支持软件页翻译
  - tlb缺失 异常入口 tlbrentry
  - 需要根据配置的页级数信息填充
- PGDH （高地址空间）
- PGDL （低地址空间）

## Time

- x86_64
  - lapic
  - rdtsc
- riscv64
  - 设置定时器需要借助 sbi 在 M 态设置
  - time
- aarch64
  - cntp_ctl
  - cntp_val
- loongarch64
  - tcfg
  - Time

## TLS

- x86_64
  - fs
- riscv64
  - tp
- aarch64
  - TPIDR_EL0
- loongarch64
  - $tp

## Percpu

- x86_64
  - gs
- riscv64
  - gp
- aarch64
  - TPIDR_EL1
- loongarch64
  - $r21
