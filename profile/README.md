# AeroPortOSS

AeroPort is an open-source, application-forwarding hypervisor framework engineered specifically for Apple Silicon architecture.

Instead of forcing users to manage a resource-heavy secondary desktop environment, AeroPort orchestrates an optimized, headless Windows 11 ARM64 virtual machine in the background via macOS's native Virtualization.framework. Individual application windows are intercepted inside the guest subsystem, composited locally to preserve child interface elements, and mapped directly onto the macOS desktop as native, floating, transparent container windows. This provides individual Mac Dock stubs and native window snapping while eliminating secondary desktop overhead.

---

## Architecture and Repository Structure

The AeroPort ecosystem is decoupled across three specialized repositories:

* **[aeroport-macos](https://github.com/AeroPortOSS/aeroport-macos):** The host-side management application and graphics translation server. Developed in SwiftUI and AppKit, it coordinates the hypervisor lifecycle and hosts an atomic background processing thread that translates incoming execution tokens into native Metal commands on the fly.
* **[aeroport-guest-agent](https://github.com/AeroPortOSS/aeroport-guest-agent):** The unified Windows guest environment subsystem. Contains the process launcher (`aeroport-launch`), the runtime API interceptor (`aeroport-hook.dll`) utilizing Microsoft Detours, the Vectored Exception Handler memory tracker, and the UMDF 2.x Indirect Display Driver (`aeroport-idd-driver`).
* **[aeroport-iso-tool](https://github.com/AeroPortOSS/aeroport-iso-tool):** The provisioning engine. Automated PowerShell routines that query deployment manifests, strip out operating system telemetry/bloatware, pre-inject our unified guest runtime subsystem, and configure boot test-signing configurations natively.

---

## The Paravirtualized Graphics Pipeline

AeroPort achieves low rendering latency by establishing a direct, user-space API translation pathway across a zero-copy hypervisor shared memory boundary:

1. **Intercept:** The local hook runtime captures full Pipeline State Objects (PSOs)—including DXIL shader bytecode, Root Signatures, and input layouts—spoofing a discrete hardware adapter to bypass system software rendering fallbacks.
2. **Synchronize:** The integrated Indirect Display Driver handles surface swapchains and timing loops, mapping Windows presentation fences directly to host hardware synchronization primitives to prevent concurrent memory access races.
3. **Transport:** Frame data and execution tokens are written to a physical block of RAM mapped symmetrically across the hypervisor boundary using Stage 2 MMU hardware allocations.
4. **Translate & Render:** A macOS host background thread reads the ring buffer via atomic load-acquire primitives, passes raw DXIL tokens into Apple's native MetalShaderConverter.framework, and schedules execution directly on the Apple Silicon GPU cluster.

---

## Contributing to AeroPortOSS

AeroPortOSS is a community-driven project built on engineering transparency. We welcome contributors across multiple disciplines:
* **macOS Development:** Swift, SwiftUI, AppKit window composition, and MetalShaderConverter interfacing.
* **Windows Systems Engineering:** C++, COM interface overriding, Microsoft Detours hooking mechanics, UMDF 2.x Indirect Display Drivers, and ARM NEON vector assembly optimization.
