# Embedded Systems and RTOS: Post-POSIX Computing at the Edge

> Processed: 2026-03-13
> Source: ~/dev/clade/links/other-links.txt

---

## Tock Embedded Operating System

**URL:** https://www.tockos.org/
**Type:** project

### Summary

Tock is a safe, multitasking operating system designed specifically for low-power, low-memory microcontrollers. It represents a fundamental rethinking of embedded OS architecture, built from the ground up in Rust to provide memory safety at the compile-time level rather than relying solely on hardware protection mechanisms.

The system is designed for running multiple concurrent, mutually distrustful applications on Cortex-M based embedded platforms. Tock's architecture centers around protection - both from potentially malicious applications and from device drivers. This is achieved through two primary mechanisms: the kernel and device drivers written in Rust (providing compile-time memory safety, type safety, and strict aliasing), and hardware Memory Protection Units (MPUs) to isolate applications from each other and the kernel.

### Key Points

- Written entirely in Rust, leveraging its compile-time safety guarantees for kernel and driver code
- Uses MPU hardware to isolate untrusted applications without the overhead of virtual memory
- Designed for sensor networks, security-critical devices (TPMs, USB authentication fobs), and wearables
- Supports third-party applications running safely on resource-constrained devices
- Enables true multiprogramming on microcontrollers that traditionally run single applications

### Relevance to CLADE

Tock exemplifies post-POSIX computing by demonstrating that safe, multi-process computing is possible without the heavy abstraction layers of traditional operating systems. It replaces POSIX's process isolation model with a Rust-based capability system combined with hardware MPU enforcement - achieving similar security guarantees with orders of magnitude less overhead. This is exactly the kind of rethinking required for edge computing where POSIX is too heavy.

---

## FreeRTOS

**URL:** https://www.freertos.org/
**Type:** project

### Summary

FreeRTOS is the market-leading real-time operating system for microcontrollers and small microprocessors, supporting over 40 processor architectures. Distributed under the MIT license, it has become the de facto standard for embedded RTOS, trusted by world-leading companies across all industry sectors.

The system provides a tiny footprint kernel with proven robustness, fast execution times, and a growing set of libraries including Symmetric Multiprocessing (SMP) support, a thread-safe TCP stack with IPv6 support, and seamless cloud service integration. FreeRTOS is maintained by AWS and offers Long Term Support (LTS) releases with security updates for two years.

### Key Points

- Supports 40+ MCU architectures including RISC-V and ARMv8-M (Cortex-M33)
- Recently added ARMv8.1-M PACBTI (Pointer Authentication and Branch Target Identification) support for enhanced security
- Includes FreeRTOS-Plus-TCP, a popular TCP/IP stack designed specifically for embedded systems
- Offers pre-configured demos and IoT reference integrations for rapid development
- Open source for 20+ years with professional support ecosystem

### Relevance to CLADE

While FreeRTOS predates the post-POSIX movement, its minimalist approach to system services demonstrates that full POSIX compliance is unnecessary for most embedded workloads. Its recent security enhancements (PACBTI, TrustZone integration) show how traditional RTOS architectures are evolving to meet modern security requirements. FreeRTOS represents the "working baseline" against which post-POSIX systems must compete.

---

## Ariel OS

**URL:** https://ariel-os.github.io/ariel-os/dev/docs/book/
**Type:** project

### Summary

Ariel OS is an operating system for secure, memory-safe, networked applications running on low-power microcontrollers. Built on Rust from the ground up, it supports hardware based on 32-bit microcontroller architectures including Cortex-M, RISC-V, and Xtensa.

Ariel OS builds on existing projects from the Embedded Rust ecosystem including Embassy, esp-hal, defmt, probe-rs, sequential-storage, and embedded-test. It provides a preemptive multicore scheduler, portable peripheral APIs, and additional network security facilities. The design philosophy allows low-power IoT developers to focus on business logic sitting on top of APIs that remain close to the hardware but stay consistent across all supported hardware.

### Key Points

- Combines curated ecosystem of libraries with missing OS functionalities
- Preemptive multicore scheduler designed for modern multi-core microcontrollers
- Portable peripheral APIs inspired by RIOT OS's approach to hardware abstraction
- Focus on reducing application development time, increasing code portability, and minimizing core vulnerabilities
- Actively rewriting IoT system software foundations on safer grounds than traditional C-based building blocks

### Relevance to CLADE

Ariel OS directly addresses the post-POSIX challenge by creating a cohesive operating system from safe building blocks. Rather than porting POSIX to microcontrollers, it defines new APIs appropriate for the constraints of embedded systems while maintaining security through Rust's type system. This represents the "build new" approach to post-POSIX computing.

---

## Embedded Rust Documentation

**URL:** https://docs.rust-embedded.org/
**Type:** docs

### Summary

The Embedded Rust Working Group provides comprehensive documentation for using Rust in embedded development, collectively known as "The Embedded Rust Bookshelf." These resources cover everything from introductory microcontroller programming to advanced topics like writing freestanding binaries.

Key resources include "The Discovery Book" (teaching microcontrollers, peripherals, sensors and bare metal programming through hands-on projects), "The Embedded Rust Book" (patterns for building correct embedded software), OS development tutorials for Raspberry Pi's ARMv8-A architecture, and "The Embedonomicon" (diving into linker scripts, symbols, and ABIs for creating no_std programs from scratch).

### Key Points

- Assumes Rust knowledge but no embedded experience for introductory materials
- Covers ARM Cortex-M architecture extensively
- Includes tutorials for hobby OS developers targeting ARMv8-A
- Provides guidance on linker scripts and low-level ABI control
- Curated "Awesome embedded Rust" list of no_std crates

### Relevance to CLADE

The embedded Rust ecosystem demonstrates that safe systems programming is possible without POSIX. The `no_std` paradigm eliminates dependencies on operating system abstractions entirely, proving that modern software can run directly on hardware with only language-level safety guarantees. This is foundational infrastructure for post-POSIX computing.

---

## Freestanding Rust Binary (Philipp Oppermann)

**URL:** https://os.phil-opp.com/freestanding-rust-binary/
**Type:** blog/tutorial

### Summary

This comprehensive tutorial explains how to create a Rust executable that does not link the standard library, making it possible to run Rust code on bare metal without an underlying operating system. It's part of a larger series on writing an OS kernel in Rust.

The guide covers disabling the standard library with `#![no_std]`, implementing panic handlers, disabling stack unwinding, overwriting the default entry point chain, and handling linker errors across Linux, Windows, and macOS. It demonstrates compiling for bare metal targets like `thumbv7em-none-eabihf` to avoid C runtime dependencies entirely.

### Key Points

- Shows how to create minimal freestanding binaries with `#![no_std]` and `#![no_main]`
- Explains the `start` attribute and how runtime initialization works
- Covers linker configuration for bare metal targets
- Demonstrates cross-platform build configurations in `.cargo/config.toml`
- Foundation for building OS kernels and bare metal applications

### Relevance to CLADE

This tutorial is essential reading for understanding what's required to escape POSIX entirely. It demonstrates that Rust's type system and ownership model provide sufficient safety guarantees that we don't need operating system-enforced process isolation at the language level. The techniques here enable entirely new post-POSIX operating system designs.

---

## Introduction to Embedded Systems with Rust (ESP32)

**URL:** https://rust-dd.com/post/introduction-to-embedded-systems-with-rust-a-beginner-s-guide-using-esp32
**Type:** blog/tutorial

### Summary

This beginner-friendly guide walks through setting up embedded development with Rust on the ESP32 microcontroller, covering both hardware basics (Arduino, ESP32, Raspberry Pi) and software fundamentals (std vs no_std approaches).

The tutorial covers installing the `espup` toolchain manager for Espressif's SoCs (supporting both Xtensa and RISC-V architectures), using `cargo-generate` with the `esp-template`, and understanding HAL (Hardware Abstraction Layer) concepts. It demonstrates practical examples including the classic "Hello World" via serial output and flashing an LED using GPIO.

### Key Points

- ESP32 supports both Xtensa and RISC-V architectures
- `no_std` approach strips away features requiring system resources for embedded use
- Hardware Abstraction Layer (HAL) provides portable APIs across different microcontrollers
- Peripherals access pattern: `Peripherals::take()` provides singleton access to hardware
- ClockControl configuration affects timing-sensitive peripherals

### Relevance to CLADE

This practical guide shows how embedded development with Rust enables safe, direct hardware access without POSIX intermediaries. The HAL abstraction pattern demonstrates that hardware portability can be achieved through language-level abstractions rather than operating system system calls - a key insight for post-POSIX system design.

---

## WebAssembly on Zephyr

**URL:** https://blog.golioth.io/webassembly-on-zephyr/
**Type:** blog

### Summary

This article explores running WebAssembly (Wasm) on microcontrollers using Zephyr RTOS, demonstrating how the WebAssembly Micro Runtime (WAMR) enables safe, portable code execution on resource-constrained devices.

The approach allows building the Wasm runtime into firmware, then supplying new Wasm binaries to change application behavior without full firmware updates or device reboots. The demonstration implements a temperature threshold mechanism where native firmware passes values to the runtime, and Wasm code calls native functions. The Wasm binary can be passed as a base64-encoded string and deployed fleet-wide via services like Golioth Settings.

### Key Points

- Wasm provides sandboxing and security for running untrusted code on microcontrollers
- Same Wasm binary can run on embedded devices, cloud servers, and browsers
- Enables "brain surgery" on firmware - changing algorithms without OTA firmware updates
- Trade-off: uses more resources and slower than native code, but offers portability and security
- WAMR is optimized for embedded systems with minimal overhead

### Relevance to CLADE

WebAssembly represents a compelling post-POSIX execution model. Rather than POSIX processes, Wasm modules provide isolated, sandboxed execution with well-defined host interfaces. The ability to run the same binary across edge devices, servers, and browsers demonstrates the "write once, run anywhere" promise that Java never fully delivered - but without requiring a full operating system runtime.

---

## Wasm for Embedded and IoT

**URL:** https://medium.com/wasm-radar/wasm-for-embedded-and-iot-why-portability-matters-at-the-edge-55f64801e7bc
**Type:** blog

### Summary

This article argues that WebAssembly is an excellent fit for embedded and IoT development, offering a lightweight, secure, and runtime-agnostic foundation for next-generation embedded systems. The edge is characterized by constraints: low-power chips, limited memory, no standard OS, and diverse hardware.

While Wasm wasn't originally designed for embedded systems, it addresses core IoT challenges: platform-specific toolchains, difficult upgrades and patching, and security risks when running dynamic code. The article highlights real-world adopters including WASI proposals for modular host-agnostic APIs, WasmEdge for lightweight edge workloads, and successful deployments on ESP32 and ARM Cortex-M platforms using runtimes like wasm3, Wasmer, and WAMR.

### Key Points

- Wasm decouples "what your app does" from "where it runs"
- Ideal for pluggable logic modules, multi-language plugins, ML inference at edge, and sensor preprocessing
- Toolchains like `cargo-component`, `wit-bindgen`, and `wasi-sdk` enable cross-language bindings
- Turns devices into hosts for portable functionality with language-agnostic deployment
- ESP32 and ARM Cortex-M can run Wasm modules today

### Relevance to CLADE

Wasm represents the most mature post-POSIX execution model currently available. Its component model (via WIT interfaces) provides a cleaner abstraction than POSIX system calls, and its sandboxing eliminates the need for OS-enforced process isolation. For edge computing where POSIX is too heavy but bare metal is too inflexible, Wasm offers the right trade-offs.

---

## Wasefire (Google)

**URL:** https://google.github.io/wasefire/dev/tooling/index.html
**Type:** project/docs

### Summary

Wasefire is Google's framework for secure-by-design firmware development using WebAssembly. It provides a complete toolchain for building "platforms" (the firmware that runs on hardware) and "applets" (WebAssembly modules that run on the platform).

The system uses `cargo xtask` for most operations, with commands for compiling platforms and applets. Wasefire treats firmware as a host runtime for Wasm applets, enabling secure, updatable logic deployment without firmware reflashing. The framework provides APIs for LEDs, buttons, timers, USB, UART, RPC, and storage - covering common embedded use cases.

### Key Points

- Separates "platform" (firmware) from "applets" (Wasm modules)
- Provides standardized APIs for common embedded peripherals
- Google-backed project indicating enterprise interest in secure firmware development
- Open source with active development
- Demonstrates production-ready Wasm-on-embedded architecture

### Relevance to CLADE

Wasefire represents Google's bet on Wasm as the post-POSIX execution model for embedded systems. The platform/applet separation mirrors cloud-native patterns (platform vs application) while remaining appropriate for resource-constrained devices. This is evidence that major technology companies see Wasm as the future of embedded software development.

---

## Enarx

**URL:** https://enarx.dev/
**Type:** project

### Summary

Enarx is a leading open source framework for running applications in Trusted Execution Environments (TEEs), part of the Confidential Computing Consortium from the Linux Foundation. It provides a runtime TEE based on WebAssembly, allowing deployment of applications written in Rust, C/C++, C#, Go, Java, Python, Haskell, and many more languages.

The framework is CPU-architecture independent, letting developers deploy the same application code transparently across multiple hardware targets. It provides a single runtime and attestation framework that is hardware vendor and cloud service provider neutral - addressing the fragmentation problem in confidential computing.

### Key Points

- 100% open source, part of Linux Foundation's Confidential Computing Consortium
- Language-agnostic: supports Rust, C/C++, Go, Python, Java, and many more
- Hardware-neutral: abstracts over different TEE implementations (Intel SGX, AMD SEV, ARM CCA)
- Provides unified attestation framework across vendors
- WebAssembly as the portable, sandboxed execution format

### Relevance to CLADE

Enarx demonstrates that WebAssembly can serve as a secure execution substrate for confidential computing - one of the most security-sensitive deployment scenarios. This validates Wasm as a serious post-POSIX runtime for enterprise workloads. The hardware abstraction over different TEE implementations shows how post-POSIX systems can provide portability without sacrificing security.

---

## Synthesis: The Embedded Post-POSIX Landscape

### Common Themes

The embedded systems ecosystem is leading the post-POSIX revolution out of necessity. With memory constraints measured in kilobytes rather than gigabytes, and power budgets measured in milliwatts, traditional operating system abstractions are simply too heavy. The sources reviewed reveal several converging trends:

**1. Rust as the New C**
The Embedded Rust ecosystem (docs.rust-embedded.org, Phil Opp's tutorials, ESP32 guides) demonstrates that memory-safe systems programming is achievable without operating system support. Rust's ownership model replaces POSIX's process isolation at the language level, enabling safe multi-tasking on hardware too constrained for MMU-based virtual memory.

**2. WebAssembly as Universal Edge Runtime**
Wasm emerges as the most promising post-POSIX execution model. Projects like Wasefire (Google), WAMR on Zephyr, and industry adoption patterns show Wasm providing:
- Sandbox isolation without OS support
- Language-agnostic deployment
- Hot-swappable logic modules
- Portable binaries across edge, cloud, and browser

**3. Capability-Based Security**
Tock OS, Ariel OS, and Wasefire all use capability-based security models rather than POSIX's user/group/permission model. This shift enables fine-grained, composable security policies appropriate for mutually distrustful components sharing a microcontroller.

**4. Hardware Abstraction at the Language Level**
HAL (Hardware Abstraction Layer) patterns in embedded Rust show that hardware portability can be achieved through trait-based abstractions rather than operating system drivers. This eliminates an entire layer of indirection while maintaining portability.

### The Post-POSIX Pattern

What emerges is a pattern for post-POSIX systems:

```
Traditional POSIX:  [Hardware] -> [Kernel] -> [System Calls] -> [Processes] -> [Application]
Post-POSIX:         [Hardware] -> [HAL/Drivers] -> [Runtime/Wasm] -> [Components/Applets]
```

The embedded world has already made this transition. The question for CLADE is whether these patterns scale up to server and desktop workloads, or whether different abstractions are needed for different computing tiers.

### Implications for CLADE

1. **Wasm Component Model** appears to be the leading candidate for a post-POSIX application binary interface (ABI). It's production-ready, cross-platform, and supported by major industry players.

2. **Rust's no_std ecosystem** provides the foundation for building post-POSIX systems from scratch without depending on legacy operating system code.

3. **Capability-based security** models from embedded systems (Tock, seL4) offer alternatives to POSIX permissions that scale from microcontrollers to distributed systems.

4. **The edge is the proving ground** - embedded systems demonstrate daily that full POSIX compliance is unnecessary for real-world workloads. The patterns developed here will inform post-POSIX designs for larger systems.

---

*Next in series: [05-capability-systems.md] - Exploring capability-based security models from seL4, CHERI, and Capsicum*
