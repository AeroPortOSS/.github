# AeroPort

AeroPort is an open-source, high-performance virtualization and application translation ecosystem designed to run Windows applications headlessly on Apple Silicon macOS. By bypassing traditional full-desktop rendering paths and directly intercepting graphics APIs, AeroPort seamlessly projects Windows application windows into native macOS AppKit containers with near-zero overhead.

## Architecture Overview

The ecosystem is divided into three core subsystems to optimize compilation toolchains and maintain clean separation of concerns:

1. **Host Core (`aeroport-core`)**: Managed in Swift, this subsystem utilizes macOS `Virtualization.framework` to run the ARM64 Windows guest, handles native window compositions, and provides a built-in curated storefront.
2. **Guest Agent (`aeroport-guest-agent`)**: Developed in C++ and C, this subsystem compiles inside the Windows guest environment. It contains the binary process injector, the D3D12-to-Metal translation hooking layer, and the UMDF 2.x Indirect Display Driver.
3. **Provisioning Engine (`aeroport-iso-tool`)**: A PowerShell-based utility suite designed to streamline official Windows 11 ARM64 installation media, stripping telemetry and injecting required VirtIO and runtime dependencies.

## Key Features

* **Headless Window Forwarding**: Windows apps render without the underlying Windows desktop shell, appearing as native, floating, transparent containers on macOS.
* **Direct Graphics Pipeline**: Intercepts Direct3D 12 and DXGI calls within user-space, translating instructions directly into native Apple Metal pipelines.
* **Local Shader Caching**: Eliminates real-time translation stutter by compiling and caching Metal binary archives directly on the host file system.
* **Integrated Storefront**: Features a curated registry for common productivity tools, engineering software, and performance titles, funded entirely by non-intrusive B2B developer placements.

## Contributing

We welcome contributions across all layers of the stack. Please review the contribution guidelines in each specific repository before submitting a Pull Request.

## License

AeroPort is open-source software licensed under the AGPL-3.0 License. Commercial enterprise licensing and support options are available for corporate environments requiring custom deployments.
