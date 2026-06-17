# Contributing to AeroPortOSS

Thank you for your interest in contributing to AeroPort. This document outlines the technical standards, repository management rules, and development workflows required to ensure consistency across our paravirtualized hypervisor stack.

---

## Codebase Architecture Review

Before submitting code, ensure you are committing to the correct repository based on our decoupled architecture:

* **aeroport-macos:** Swift/SwiftUI and AppKit window management.
* **aeroport-wddm-kernel:** C++ Windows Kernel-Mode Driver (WDDM KMD).
* **aeroport-guest-agent:** C++ Windows User-Mode Driver (UMD) and guest daemons.
* **aeroport-iso-tool:** PowerShell and DISM deployment automation.

---

## Technical and Style Standards

### C++ and Driver Code (`aeroport-wddm-kernel` / `aeroport-guest-agent`)
* **Standard:** Modern C++ (C++20 or later where supported by the WDK/SDK).
* **Memory Management:** Memory safety is paramount in the kernel layer. Use explicit lock-free atomic primitives for ring buffer communication (`std::memory_order_acquire` / `std::memory_order_release`). Avoid heap allocations within execution hot-paths; utilize the flat lookup array framework.
* **Architecture-Specific Code:** Ensure explicit compiler hardware memory barriers are used when targeting ARM64 execution consistency models (e.g., `__dmb`).

### Swift Code (`aeroport-macos`)
* **Standard:** Swift 5.x / SwiftUI matching the latest macOS SDK baseline features.
* **UI Design:** Adhere closely to native macOS design semantics. Utilize system visual effects materials and structural sidebars over custom, non-standard UI component libraries.

### PowerShell Automation (`aeroport-iso-tool`)
* **Standard:** PowerShell 5.1+ compatibility for maximum out-of-the-box system image manipulation. All scripts must be modular and heavily documented regarding the specific DISM or BCD parameters utilized.

---

## Development Workflow

### 1. Branch Strategy
We use a structured branch naming convention. Branch out from `main` using the following prefixes:
* `feature/` for new architectural blocks or enhancements (e.g., `feature/neon-diff-engine`).
* `bugfix/` for resolving existing repository issues.
* `docs/` for purely documentation or configuration changes.

### 2. Commit Message Guidelines
Commit messages must be concise, direct, and specify the component altered:
* Format: `<type>(<component>): <description>`
* Example: `feat(kmd): implement explicit hardware memory barrier in submit loop`
* Example: `fix(umd): resolve memory leak in shadow buffer registry allocation`

### 3. Pull Request Process
1. Fork the target repository and create your branch from `main`.
2. Ensure your code compiles without warnings (especially kernel-mode driver code using the WDK).
3. Open a Pull Request targeting the `main` branch of the upstream repository.
4. Provide a clear description of the problem solved, the architectural files modified, and verification steps performed.
5. All pull requests require a review and approval from a repository maintainer before merging.

---

## Privacy and Security Notice

Do not commit raw, local configuration footprints, private build schemas, or unencrypted personal identification data. Ensure your local `.gitignore` is verified before pushing to the remote repository. We recommend checking the "Keep my email addresses private" option in your personal GitHub account settings to use GitHub's anonymous commit addresses.
