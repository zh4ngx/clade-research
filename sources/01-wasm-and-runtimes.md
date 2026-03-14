# WebAssembly and Runtimes - Research Summary

A collection of research on WebAssembly runtimes, the Component Model, and their relevance to post-POSIX computing paradigms.

---

## The WebAssembly Component Model

**URL:** https://component-model.bytecodealliance.org/
**Type:** Documentation / Specification

### Summary
The WebAssembly Component Model is a broad-reaching architecture for building interoperable WebAssembly libraries, applications, and environments. It represents a fundamental shift from monolithic WebAssembly modules to composable, language-agnostic components that can communicate through well-defined interfaces.

The model introduces several key concepts: Components are units of distribution and composition that run on platforms (browsers, standalone runtimes, or operating systems). Interfaces define the contracts between components. Worlds describe the complete set of imports and exports available to a component. WASI (WebAssembly System Interface) provides standard APIs that components can depend on for system interactions.

The specification is actively maintained by the Bytecode Alliance, with WASI 0.2.0 released as a stable target in January 2024. This stability enables developers to build production components that can run across any compliant runtime.

### Key Points
- Components are the fundamental unit of composition, not raw WebAssembly modules
- WIT (WebAssembly Interface Types) defines cross-language interface contracts
- WASI provides standardized system APIs, enabling portable system-level code
- The model supports polyglot composition - components written in different languages can interoperate seamlessly
- Stable releases allow pinning to specific versions for production use

### Relevance to CLADE
The Component Model represents a fundamental break from the POSIX process model. Instead of processes communicating through OS-defined primitives (pipes, sockets, files), components communicate through typed interfaces defined at the application level. This is exactly the kind of abstraction shift that post-POSIX computing requires - defining interfaces semantically rather than through byte streams. The Component Model's approach to capability-based security and interface-first design could serve as a blueprint for how CLADE components interact.

---

## WASM Micro Runtime with Rust

**URL:** https://anoopelias.github.io/posts/wasm-micro-runtime-with-rust/
**Type:** Blog / Tutorial

### Summary
This article documents the practical experience of running WebAssembly on ESP32 microcontrollers using the WebAssembly Micro Runtime (WAMR) from the Bytecode Alliance. The author walks through compiling both C and Rust code to WASM, then running it on resource-constrained hardware.

The key challenge is binary size. A standard Rust "Hello World" compiled to wasm32-wasi produces a 2MB binary - far too large for microcontrollers. The solution involves using `no_std` to opt out of the standard library, providing custom panic handlers, and carefully managing linker flags to control stack size and eliminate unnecessary runtime code. The final working binary is just 142 bytes for a minimal program.

The article demonstrates that WASM on embedded is practical today, but requires understanding the tradeoffs between convenience and size. The sandbox security model remains intact even on constrained devices, and the same binary can potentially run on microcontrollers, servers, and browsers.

### Key Points
- WAMR enables WASM execution on ESP32 and similar microcontrollers
- Binary size is the primary constraint - `no_std` Rust is essential for embedded WASM
- Stack size must be explicitly configured to match target hardware constraints
- The same WASM binary can run across dramatically different platforms
- Ahead-of-Time (AOT) compilation can significantly improve performance on embedded targets

### Relevance to CLADE
This demonstrates that WASM is viable even in the most resource-constrained environments - exactly where post-POSIX systems need to operate. The ability to run sandboxed, portable code on microcontrollers without a full OS opens possibilities for "compute everywhere" architectures. The techniques for minimizing binary size while maintaining security are directly applicable to building CLADE's minimal, capability-secure components.

---

## WasmEdge

**URL:** https://wasmedge.org/
**Type:** Project / Runtime

### Summary
WasmEdge is a lightweight, high-performance, and extensible WebAssembly runtime designed specifically for cloud-native, edge, and decentralized applications. It is a CNCF sandbox project that positions itself as a replacement for Linux containers in many use cases.

The runtime offers impressive performance characteristics: 100x faster startup than Linux containers and 20% faster runtime execution, while using 1/100 of the size of comparable container images. WasmEdge extends beyond basic WASI with support for network sockets, async processing, TensorFlow inference, key-value stores, and database connectors.

A key differentiator is JavaScript support - WasmEdge can run ES6 modules with Node and NPM API compatibility, enabling React SSR streaming and implementation of JS APIs in Rust. This makes it significantly lighter than containerized V8 while providing similar capabilities. The runtime integrates with Kubernetes, Docker (via Docker+Wasm), Dapr, and various service mesh technologies.

### Key Points
- 100x faster startup and 1/100 the size compared to Linux containers
- Native support for async networking, ML inference, and database operations
- JavaScript runtime capability without full V8 overhead
- Kubernetes and Docker integration for cloud-native deployments
- Cross-platform: Linux, macOS, Windows, RTOS, ARM, and x86

### Relevance to CLADE
WasmEdge demonstrates that WASM can replace containers for many workloads while providing better resource efficiency and faster startup. This is the "execution layer" that post-POSIX systems need - a runtime that isn't bound to process isolation and filesystem-based interfaces. The extension model (adding capabilities like ML inference) shows how CLADE could expose hardware acceleration and specialized functionality to sandboxed components.

---

## wasmCloud

**URL:** https://wasmcloud.com/
**Type:** Platform / CNCF Incubating Project

### Summary
wasmCloud is a CNCF incubating project that provides orchestration for WebAssembly applications across clouds, Kubernetes, datacenters, and edge environments. It focuses on building polyglot applications from reusable Wasm components that can run resiliently and efficiently anywhere.

The platform addresses three key problems: idle infrastructure (through sub-millisecond cold starts and scale-to-zero), maintenance burden (through reusable, versioned components), and distributed deployments (through portable, platform-agnostic binaries). It uses a contract-driven approach where application dependencies are defined at runtime, allowing different implementations for dev, QA, and production without code changes.

wasmCloud implements capability-based security where components can only access resources through explicitly granted capabilities. The platform integrates with the broader cloud-native ecosystem including Kubernetes, AWS, Azure, GCP, ArgoCD, Backstage, and various databases and messaging systems.

### Key Points
- Sub-millisecond startup with scale-to-zero capability
- Contract-driven interfaces enable runtime dependency injection
- Capability-based security model for fine-grained access control
- Polyglot components can be written in Rust, Go, JavaScript, Python, etc.
- First-class Kubernetes integration with a dedicated operator

### Relevance to CLADE
wasmCloud represents a concrete implementation of post-POSIX orchestration. Instead of managing processes and containers, it manages components with explicit capability contracts. The ability to swap implementations at runtime based on contracts (not just configuration) is a key insight for CLADE. The "distributed capabilities" model - where capabilities can span edges and clouds - shows how post-POSIX systems can handle the distributed nature of modern computing.

---

## Making Really Tiny WebAssembly Graphics Demos

**URL:** http://cliffle.com/blog/bare-metal-wasm/
**Type:** Blog / Tutorial

### Summary
This deep technical article explores creating minimal WebAssembly binaries by stripping away all abstractions. The author demonstrates building animated graphics demos in under 300 bytes - no wasm-bindgen, no webpack, no npm, just hand-written Rust and JavaScript.

The key insight is understanding what adds size: the Rust standard library, panic handling code, and unnecessary linker output. By using `#![no_std]`, custom panic handlers, careful linker flags (`-C link-self-contained=no`, `--no-entry`, `-zstack-size`), and tools like `wasm-strip` and `wasm-opt`, binaries can be reduced from 800KB to under 300 bytes.

The article also covers importing functions from JavaScript (like `Math.sin`) to avoid bundling implementations, using atomics for frame counting, and diagnosing binary bloat through careful inspection with `wasm-objdump` and `rustfilt`. The result is a methodology for creating truly minimal WASM modules.

### Key Points
- `#![no_std]` eliminates standard library overhead but requires custom panic handling
- Tools: `wasm-strip`, `wasm-opt -Oz`, `wasm-snip` for binary minimization
- Import functions from host environment to avoid bundling implementations
- Bounds checks and panics add 300 bytes to 2KB of code
- Iterators over fixed-size arrays can avoid bounds checking

### Relevance to CLADE
This article provides essential techniques for building minimal CLADE components. Post-POSIX systems need to be lean, and understanding how to eliminate unnecessary code while maintaining correctness is crucial. The bare-metal approach - treating WASM as a direct compilation target without framework overhead - aligns with CLADE's goal of minimal, understandable systems. The techniques for importing host functions rather than bundling implementations directly support CLADE's capability-based security model.

---

## WebAssembly on Zephyr

**URL:** https://blog.golioth.io/webassembly-on-zephyr/
**Type:** Blog / Case Study

### Summary
This article from Golioth documents using WebAssembly with Zephyr RTOS on microcontrollers, presented by Dan Mangum at the Embedded Open Source Summit. It demonstrates running WASM binaries on Nordic nRF52840 hardware and explores the tradeoffs of this approach.

The key advantage is dynamic code execution: WASM binaries can be updated without full firmware updates or device reboots. The demonstration shows loading new WASM modules via base64-encoded strings delivered through the Golioth Settings service. This enables fleet-wide algorithm updates targeting specific devices or groups.

The portability story is compelling: the same WASM binary runs on microcontrollers, cloud servers, and browsers. This enables "sliding compute" - moving processing between edge and cloud based on requirements. The WebAssembly Micro Runtime (WAMR) provides the embedded runtime with sandboxing security.

### Key Points
- WAMR provides a Zephyr port optimized for embedded systems
- Dynamic code updates without firmware reflashing or reboots
- Same binary runs on microcontroller, cloud, and browser
- Tradeoff: more resources and slower execution than native code
- Ideal for pluggable logic modules and algorithm experimentation

### Relevance to CLADE
This demonstrates a practical path to post-POSIX computing on embedded devices. Instead of treating microcontrollers as dumb peripherals, they become hosts for portable, sandboxed components. The ability to update logic without firmware updates is exactly the kind of agility CLADE needs. The "sliding compute" concept - moving the same binary between edge and cloud - illustrates how post-POSIX systems can handle the distributed, heterogeneous nature of modern computing infrastructure.

---

## Wasm for Embedded and IoT: Why Portability Matters at the Edge

**URL:** https://medium.com/wasm-radar/wasm-for-embedded-and-iot-why-portability-matters-at-the-edge-55f64801e7bc
**Type:** Article / Analysis

### Summary
This article from the WASM Radar series examines why WebAssembly is emerging as a compelling solution for embedded and IoT development, despite not being originally designed for these use cases.

The edge presents unique constraints: low-power chips, limited memory, no standard OS, and diverse hardware. Traditional solutions like native binaries or Linux containers require platform-specific toolchains, difficult upgrades, and introduce security risks. WASM offers an alternative: a portable, secure, and lightweight runtime perfectly suited for constrained environments.

The article highlights key use cases: pluggable logic modules that update without reflashing, multi-language plugins on shared runtimes, ML inference at the edge, and sensor preprocessing. Real-world implementations exist on ESP32 and ARM Cortex-M platforms using runtimes like wasm3, Wasmer, and WAMR. The tooling ecosystem (cargo-component, wit-bindgen, wasi-sdk) is maturing rapidly.

### Key Points
- Edge computing has inherent fragmentation - WASM decouples logic from hardware
- Compile once, ship across environments, maintain safety and agility
- WASI provides modular, host-agnostic APIs for embedded runtimes
- Real implementations exist on ESP32 and ARM Cortex-M platforms
- Key use cases: pluggable logic, multi-language plugins, edge ML, sensor preprocessing

### Relevance to CLADE
This article makes the strongest case for why CLADE should embrace WASM: portability matters because IoT is inherently fragmented. The principle of decoupling "what your app does" from "where it runs" is fundamental to post-POSIX thinking. WASM's ability to run the same binary across dramatically different platforms - from microcontrollers to cloud servers - enables the kind of ubiquitous computing that CLADE aims to support. The emphasis on security through sandboxing aligns with CLADE's capability-based approach to system design.

---

## Synthesis: Implications for Post-POSIX Computing

These resources collectively paint a picture of WebAssembly as a foundation for post-POSIX systems:

1. **Components Replace Processes**: The Component Model provides a more fine-grained, secure, and composable unit of execution than the POSIX process.

2. **Interfaces Replace Byte Streams**: WIT-defined interfaces enable semantic, type-safe communication between components, replacing the impoverished byte-stream model of pipes and sockets.

3. **Capabilities Replace Permissions**: WASM's sandbox model and wasmCloud's capability-based security provide fine-grained access control without relying on OS-level users and permissions.

4. **Portability Enables Distribution**: The ability to run the same binary from microcontroller to cloud enables new architectural patterns where compute slides to where it's needed.

5. **Minimal is Possible**: The techniques for building sub-KB WASM binaries demonstrate that lean, understandable systems are achievable - a key requirement for CLADE.

6. **The Ecosystem is Ready**: Production-ready runtimes (WasmEdge, wasmCloud, WAMR), stable specifications (WASI 0.2.0), and mature tooling exist today.
