# AeroPortOSS

AeroPort is an open-source, application-forwarding hypervisor framework engineered specifically for Apple Silicon architecture.

Instead of forcing users to manage a resource-heavy secondary desktop environment, AeroPort orchestrates an optimized, headless Windows 11 ARM64 virtual machine in the background via macOS's native Virtualization.framework. Individual application windows are intercepted inside the guest subsystem and mapped directly onto the macOS desktop as native, floating, transparent container windows. This provides individual Mac Dock stubs and native window snapping while eliminating secondary desktop overhead.

---

## Architecture and Repository Structure

AeroPort is decoupled across four dedicated repositories to separate concerns across the hypervisor boundary:

* **[aeroport-macos](https://github.com/AeroPortOSS/aeroport-macos):** The front-end management application and graphics translation server. Developed in SwiftUI and AppKit, it manages the VM lifecycle and implements a background thread that translates incoming execution tokens into native Metal commands on the fly.
* **[aeroport-idd-driver](https://github.com/AeroPortOSS/aeroport-idd-driver):** The core display subsystem. A C++ User-Mode Indirect Display Driver utilizing Microsoft's UMDF 2.x framework. It interfaces with the OS graphics runtime to handle surface swapchains, capture presentation fences, and stream token streams securely across the hypervisor memory map.
* **[aeroport-guest-agent](https://github.com/AeroPortOSS/aeroport-guest-agent):** The guest integration engine. Intercepts pipeline state objects (PSOs), matches internal handles to host resource identifiers, executes vectorized ARM NEON memory diffing to stream sparse delta updates, and handles window focus and shortcut indexing over VSOCK.
* **[aeroport-iso-tool](https://github.com/AeroPortOSS/aeroport-iso-tool):** The provisioning engine. Automated PowerShell routines that query deployment manifests, strip out operating system telemetry/bloatware, inject our user-mode driver stack, and pre-configure kernel test-signing.

---

## The Paravirtualized Graphics Pipeline

AeroPort achieves near-zero rendering latency by establishing a direct, zero-copy shared memory pathway across the hypervisor:

1. **Intercept:** The Guest Agent user-mode hooks capture DXIL shader bytecode, Root Signatures, and input layouts during Pipeline State Object (PSO) initialization.
2. **Synchronize:** The Indirect Display Driver handles frame presentation swapchains, pinning memory footprints and synchronizing guest application fences with native host hardware synchronization primitives.
3. **Transport:** Frame data and execution tokens are written to a physical block of RAM mapped symmetrically across the hypervisor boundary using Stage 2 MMU hardware allocations.
4. **Translate & Render:** A macOS host background thread reads the ring buffer via atomic load-acquire primitives, passes raw DXIL tokens into Apple's native MetalShaderConverter.framework, and schedules execution directly on the Apple Silicon GPU cluster.

---

## Contributing to AeroPortOSS

AeroPortOSS is a community-driven project built on engineering transparency. We welcome contributors across multiple disciplines:
* **macOS Development:** Swift, SwiftUI, AppKit window composition, and MetalShaderConverter interfacing.
* **Windows Driver Engineering:** C++, UMDF 2.x Indirect Display Driver architectures, and swapchain manipulation.
* **Systems Engineering:** Low-level lock-free shared memory structures, atomic sync protocols, and ARM NEON performance optimization.
