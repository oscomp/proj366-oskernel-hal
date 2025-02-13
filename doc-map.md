---
title: OSKernel-HAL
markmap:
  colorFreezeLevel: 2
  initialExpandLevel: 2
---

## Intro

- x86_64
  - 在 multiboot 的场景下，需要手动调整进入 long mode
  - 使用 `swapgs` 指令在用户态和内核态之间切换 `gs` 段寄存器
  - 支持复杂的中断和异常处理机制，通过 IDT（中断描述符表）管理
  - 使用 `cpuid` 指令获取 CPU 编号

- riscv64
  - 在 S 态没有单独获取 hartid 的方法，只能在进入内核的时候存储 Boot 函数传递的信息，在 M 态读取 `mhartid` 获取 CPU 编号
  - 使用 `tp` 寄存器来存储线程局部存储（TLS）指针
  - 使用 `stvec` 寄存器来存储陷阱向量的基地址，支持直接模式和向量模式

- aarch64
  - 支持 ARMv8-A 架构，包括 AArch64 和 AArch32 两种执行状态
  - 使用 `TPIDR_EL0` 寄存器来存储线程局部存储（TLS）指针
  - 使用 `VBAR_EL1` 寄存器来存储异常向量表的基地址，支持多种异常类型
  - 通过读取 `MPIDR_EL1` 寄存器获取 CPU 编号

- loongarch64
  - 使用 Loongson 自主研发的 LoongArch 指令集架构
  - 使用 `$tp` 寄存器来存储线程局部存储（TLS）指针
  - 使用 `eentry` 寄存器来存储异常入口地址，支持 TLB 缺失异常处理
  - 支持直接映射模式，通过设置页表项的方式实现内核地址空间的直接映射
  - loongarch64 支持很多 SAVE 寄存器，可以帮助在通用寄存器不够的环境下暂存数据。
  - 通过读取 `CPUID` 寄存器获取 CPU 编号

## Boot

- x86_64
  - 进入 long mode（64 位模式）
  - 初始化全局描述符表（GDT）
  - 初始化中断描述符表（IDT）
  - 开启分页模式（required）

- riscv64
  - 进入 S 态（Supervisor 模式）
  - 设置 `stvec` 寄存器为陷阱向量基地址
  - 初始化页表并开启分页模式
  - 设置 `satp` 寄存器为页表基地址

- aarch64
  - 进入 EL1（Exception Level 1）
  - 设置 `VBAR_EL1` 寄存器为异常向量基地址
  - 初始化页表并开启分页模式
  - 设置 `TTBR0_EL1` 和 `TTBR1_EL1` 寄存器为页表基地址

- loongarch64
  - 支持直接映射模式，通过设置页表项的方式实现内核地址空间的直接映射
  - 设置 `eentry` 寄存器为异常入口地址
  - 初始化页表并开启分页模式
  - 设置 `PGDL` 和 `PGDH` 寄存器为页表基地址

## Trap

### TrapFrame

- x86_64
  - 包含寄存器状态（如 RAX, RBX, RCX 等）
  - 包含段寄存器（如 CS, DS, ES 等）
  - 包含控制寄存器（如 CR2, CR3 等）
  - 包含标志寄存器（如 RFLAGS）
  - 包含指令指针（RIP）

- riscv64
  - 包含通用寄存器（如 x1, x2, x3 等）
  - 包含特权级寄存器（如 sstatus, sepc, stval 等）
  - 包含陷阱原因寄存器（scause）
  - 包含程序计数器（PC）

- aarch64
  - 包含通用寄存器（如 X0, X1, X2 等）
  - 包含状态寄存器（如 SPSR_EL1）
  - 包含异常返回地址寄存器（ELR_EL1）
  - 包含异常级别寄存器（如 ESR_EL1）
  - 包含程序计数器（PC）

- loongarch64
  - 包含通用寄存器（如 R1, R2, R3 等）
  - 包含状态寄存器（如 CSR_CRMD, CSR_PRMD 等）
  - 包含异常返回地址寄存器（ERA）
  - 包含异常原因寄存器（EPC）
  - 包含程序计数器（PC）

### TrapHandler

- x86_64
  - 使用中断描述符表（IDT）来处理陷阱和中断
  - 每个中断或陷阱都有一个对应的中断向量，指向一个中断处理程序
  - IDT 存储在内存中，并由 `lidt` 指令加载

- riscv64
  - 使用 `stvec` 寄存器来存储陷阱向量的基地址
  - 当发生陷阱时，硬件会跳转到 `stvec` 指定的地址
  - `stvec` 可以配置为直接模式或向量模式

- aarch64
  - 使用 `VBAR_EL1` 寄存器来存储异常向量表的基地址
  - 当发生异常时，硬件会跳转到 `VBAR_EL1` 指定的地址
  - 异常向量表包含不同类型异常的处理程序地址

- loongarch64
  - 使用 `eentry` 寄存器来存储异常入口地址
  - 使用 `tlbrentry` 寄存器来存储 TLB 缺失异常的入口地址
  - 当发生异常时，硬件会跳转到 `eentry` 或 `tlbrentry` 指定的地址

## IRQ Control

- x86_64
  - 使用本地高级可编程中断控制器（LAPIC）来管理中断
  - 使用中断描述符表（IDT）来定义中断向量
  - 使用 `cli` 指令来关闭中断
  - 使用 `sti` 指令来开启中断

- riscv64
  - 使用平台级中断控制器（PLIC）来管理外部中断
  - 使用 `sstatus` 寄存器中的 `sie` 位来控制中断使能

- aarch64
  - 使用通用中断控制器（GIC）来管理中断
  - 使用 `DAIF` 寄存器中的 `DAIFSet` 指令来屏蔽中断

- loongarch64
  - 使用 `estat` 寄存器中的 `is` 位来表示中断状态
  - 使用 `crmd` 寄存器中的 `ie` 位来控制中断使能

## CPU Features

### x86_64

- 支持 SSE、SSE2、SSE3、SSSE3、SSE4.1、SSE4.2、AVX、AVX2、AVX-512 等 SIMD 指令集
- 支持 x86 虚拟化技术（Intel VT-x 和 AMD-V）
- 支持多核和超线程技术
- 支持 64 位扩展（AMD64 和 Intel 64）
- 支持硬件加密（AES-NI）

### aarch64

- 支持 ARMv8-A 架构，包括 AArch64 和 AArch32 两种执行状态
- 支持 SIMD 指令集（NEON）
- 支持硬件虚拟化（ARM Virtualization Extensions）
- 支持 TrustZone 安全扩展
- 支持硬件加密（Cryptography Extensions）

### riscv64

- 支持 RISC-V 指令集架构（RV64I 基础指令集）
- 支持压缩指令集（RVC）
- 支持原子指令集（A）
- 支持浮点指令集（F 和 D）
- 支持硬件虚拟化（H 扩展）
- 支持用户自定义扩展

### loongarch64

- 支持 Loongson 自主研发的 LoongArch 指令集架构
- 支持 SIMD 指令集（LSX 和 LASX）
- 支持硬件虚拟化
- 支持多核和超线程技术
- 支持硬件加密

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
  - 使用本地高级可编程中断控制器（LAPIC）定时器
  - 使用 `rdtsc` 指令读取时间戳计数器
  - 支持高精度事件计时器（HPET）

- riscv64
  - 使用 `mtime` 和 `mtimecmp` 寄存器进行定时
  - 设置定时器需要借助 SBI 在 M 态设置
  - 支持 CLINT（Core Local Interruptor）定时器

- aarch64
  - 使用 `cntp_ctl` 和 `cntp_tval` 寄存器进行定时
  - 支持通用定时器框架（Generic Timer Framework）
  - 支持高精度定时器（CNTVCT_EL0）

- loongarch64
  - 使用 `tcfg` 寄存器进行定时配置
  - 使用 `Time` 寄存器读取当前时间
  - 支持高精度定时器

## TLS

- x86_64
  - 使用 `fs` 段寄存器来存储线程局部存储（TLS）指针
  - 通过 `wrfsbase` 指令设置 TLS 基地址

- riscv64
  - 使用 `tp` 寄存器来存储线程局部存储（TLS）指针
  - 通过设置 `tp` 寄存器的值来设置 TLS 基地址

- aarch64
  - 使用 `TPIDR_EL0` 寄存器来存储线程局部存储（TLS）指针
  - 通过写入 `TPIDR_EL0` 寄存器来设置 TLS 基地址

- loongarch64
  - 使用 `$tp` 寄存器来存储线程局部存储（TLS）指针
  - 通过设置 `$tp` 寄存器的值来设置 TLS 基地址

## Percpu

- x86_64
  - 使用 `gs` 段寄存器来存储每 CPU 数据的基地址
  - 通过 `wrgsbase` 指令设置每 CPU 数据基地址
  - 使用 `swapgs` 指令在用户态和内核态之间切换 `gs` 段寄存器

- riscv64
  - 使用 `gp` 寄存器来存储每 CPU 数据的基地址
  - 通过设置 `gp` 寄存器的值来设置每 CPU 数据基地址

- aarch64
  - 使用 `TPIDR_EL1` 寄存器来存储每 CPU 数据的基地址
  - 通过写入 `TPIDR_EL1` 寄存器来设置每 CPU 数据基地址

- loongarch64
  - 使用 `$r21` 寄存器来存储每 CPU 数据的基地址
  - 通过设置 `$r21` 寄存器的值来设置每 CPU 数据基地址
