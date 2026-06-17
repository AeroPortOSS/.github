# AeroPortOSS

AeroPort is an open-source, application-forwarding hypervisor framework engineered specifically for Apple Silicon architecture.

Unlike traditional virtualization platforms that force users to interact with a resource-heavy secondary desktop environment, AeroPort orchestrates an optimized, headless Windows 11 ARM64 virtual machine in the background. It intercepts individual application window layers and maps their pixel streams directly onto the macOS desktop as native, floating, transparent container windows. This approach delivers individual Mac Dock integration, native window snapping, and reduces runtime resource overhead.

---

## Architecture and Repository Structure

To maximize code maintainability and allow specialized developers to collaborate efficiently, AeroPort is decoupled across four dedicated repositories:

* **[aeroport-macos](https://github.com/AeroPortOSS/aeroport-macos):** The front-end management application. Developed in SwiftUI utilizing native macOS visual materials, it manages the virtual machine lifecycle via Apple’s Virtualization.framework and coordinates AppKit window layout routing.
* **[aeroport-wddm-kernel](https://github.com/AeroPortOSS/aeroport-wddm-kernel):** The core graphics transport layer. A C++ Kernel-Mode Driver (KMD) that interfaces directly with the Windows DirectX Graphics Kernel (Dxgkrnl.sys). It maps memory allocations using flat lookup arrays and implements explicit ARM64 data barriers to pass execution tokens across a zero-copy shared memory ring buffer.
* **[aeroport-guest-agent](https://github.com/AeroPortOSS/aeroport-guest-agent):** The guest runtime subsystem. Houses our custom User-Mode Driver (UMD) for shader interception, tracks persistent memory maps via ARM NEON vector diffing, and manages application shortcut/icon extraction over hypervisor sockets (VSOCK).
* **[aeroport-iso-tool](https://github.com/AeroPortOSS/aeroport-iso-tool):** The provisioning engine. Automated PowerShell and DISM deployment routines that query Microsoft UUP servers, strip telemetry/system tracking components, and toggle kernel test-signing configurations natively.

---

## The Core Graphics Pipeline

AeroPort avoids user-space API proxying and standard kernel command trapping to achieve near-zero rendering latency:

1. **Intercept:** The UMD intercepts DirectX 12 calls inside the guest and extracts raw DXIL shader bytecode tokens before compilation into hardware-specific binaries.
2. **Escape:** Tokens utilize a `D3DKMTEscape` bypass lane straight down to our custom KMD, avoiding standard OS graphics scheduling queues.
3. **Zero-Copy:** The KMD writes tokens directly to a physical RAM block mapped across the hypervisor boundary via Stage 2 MMU hardware allocation.
4. **Translate:** A macOS host background thread reads the tokens atomically, passing them directly to Apple's native MetalShaderConverter.framework.
5. **Render:** The compiled pipeline state object executes natively on the Apple Silicon GPU cluster, rendering frames into floating macOS NSWindow containers.

---

## Contributing to AeroPortOSS

AeroPortOSS is a completely community-driven project built on systems engineering transparency. We welcome contributors across multiple disciplines:

* **macOS Development:** Refining the SwiftUI configuration layout, window placement logic, and AppKit window servers.
* **Driver Engineering:** Optimizing the WDDM KMD execution, shared memory synchronization fences, and lock-free ring buffers.
* **Windows Development:** Enhancing the C++ UMD hooks, background guest daemons, and desktop shortcut parsers.

Please refer to the individual repository issue trackers to get involved.
