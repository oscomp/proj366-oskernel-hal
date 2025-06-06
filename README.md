# proj366-oskernel-hal

操作系统内核硬件抽象层

## 项目描述

独立的硬件抽象层（Hardware Abstraction Layer, HAL）是操作系统中至关重要的中间层，其核心意义在于通过抽象和封装底层硬件细节，为各种操作系统内核及上层软件提供统一的硬件接口。HAL将CPU架构、外设控制器、中断机制等硬件差异隐藏在标准化的接口之下，使得操作系统无需直接处理多样化的硬件特性，从而大幅提升系统的可移植性和兼容性。不同操作系统可通过适配灵活配置的HAL实现在x86-64、ARM aarch64、RISC-V 64、Loongarch64等架构上运行，而无需重写内核代码。此外，HAL简化了驱动开发与维护，硬件驱动开展只需遵循接口规范开发驱动，而开发者则基于稳定接口编写应用，避免因硬件变动导致代码重构。这种解耦设计不仅增强了系统稳定性，还加速了软硬件生态创新，成为新一代操作系统实现“一次开发，多平台部署”的关键技术基石。

## 源码

- ArceOS: <https://github.com/oscomp/arceos>
- Starry-next: <https://github.com/oscomp/starry-next>
- PolyHAL: <https://github.com/Byte-OS/polyhal>

## 所属赛道

2025全国大学生操作系统比赛的“OS功能设计”赛道

## 参赛要求

- 以小组为单位参赛，最多三人一个小组，且小组成员是来自同一所高校的学生
- 如学生参加了多个项目，参赛学生选择一个自己参加的项目参与评奖
- 请遵循“2025全国大学生操作系统比赛”的章程和技术方案要求

## 项目导师

- 杨金博
- email <321353225@qq.com>

## 赛题分类

- 内核完善
- 新型模块化组件

## 难度

中等

## 题目要求

- 需要了解 hal 层的概念和设计
- 需要了解 Rust 编程语言

## 文档

- <https://oscomp.github.io/proj366-oskernel-hal/>

## License

Apache 2.0(<https://www.apache.org/licenses/LICENSE-2.0.html>)

## 预期目标

1. 支持 NOMMU 环境
3. 支持 32 模式 (x86, aarch32, riscv32, loongarch32)
4. 支持 Armv7、MIPS 架构
5. 为 hal 编写一个更加通用自然的文档，从零开始构建一个基础运行环境的教程。
   - 包含基础环境，编译脚本
   - 详细的接口描述
   - 内核需要提供的内容
   - 所有的配置（features，config file）和详细描述
6. 支持以 libos 方式运行
7. 支持 webassembly，运行在浏览器之上
8. 改善 HAL 架构设计，构建更自由灵活的架构设计
