# OpenRadioFramework

**A modern, unified C++23 framework for digital radio signal processing, hardware abstraction, and protocol decoding.**

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![C++23](https://img.shields.io/badge/C%2B%2B-23-blue.svg)](https://en.cppreference.com/w/cpp/23)
[![CMake](https://img.shields.io/badge/CMake-3.20%2B-blue.svg)](https://cmake.org/)

---

## Mission

For over 40 years, the amateur radio community has built extraordinary software: Hamlib for rig control, fldigi for digital modes, WSJT-X for weak-signal decoding, Codec2 for voice compression, Liquid-DSP for signal processing, and many more. Each project is a masterwork of radio engineering — and each is a standalone, monolithic application with its own UI toolkit, build system, and memory model.

**OpenRadioFramework (ORF)** consolidates these fragmented capabilities into a single, high-performance C++23 library suite. The goal is not to replace these projects, but to honor their math, algorithms, and decades of domain expertise by re-expressing them as clean, composable, stateless MicroAPIs that any application — from a Raspberry Pi daemon to a native mobile app — can consume.

## Architecture

ORF follows a strict three-phase architecture designed to permanently separate signal processing logic from platform-specific concerns.

### Phase 1 — ORF Core Library (MicroAPIs)

Pure C++23 static libraries under the `ORF::` namespace. No daemon, no networking, no UI. Each module is independently testable with a simple `main()`:

| Module | Purpose |
|---|---|
| `orf-hal` | Hardware abstraction via `std::span` zero-copy buffers |
| `orf-rig` | Rig control (frequency, mode, PTT) |
| `orf-modem` | Digital mode modulation/demodulation |
| `orf-dsp` | Core DSP primitives (FFT, filters, resampling) |

**Key constraints:**
- Memory safety enforced via RAII, `std::unique_ptr`, and `std::shared_ptr` — raw `new`/`delete` is prohibited
- Error handling via `std::expected<T, E>` with monadic chaining — exceptions are forbidden in the DSP critical path
- Zero-copy data routing via `std::span<std::complex<float>>` — no intermediate `std::vector` allocations in the audio pipeline

### Phase 2 — ORF-Engine (Headless Daemon)

A headless background daemon that consumes the Phase 1 MicroAPIs and exposes them via **gRPC over Unix Domain Sockets** using Protocol Buffer contracts. Designed to run on resource-constrained embedded platforms (Raspberry Pi, single-board computers) while serving native UI clients remotely.

### Phase 3 — Native UI Shells

Thin, platform-native clients (Kotlin/Android, Swift/iOS, WinUI/Windows, Qt/Linux) that contain **zero** signal processing logic. They are strictly remote terminals that communicate with the ORF-Engine via Protobuf.

## The `legacy/` Directory

The `legacy/` directory contains **frozen, read-only** Git submodule snapshots of the upstream projects whose algorithms ORF is re-implementing:

- [Hamlib](https://github.com/Hamlib/Hamlib) — Rig control
- [fldigi](http://www.w1hkj.com/) — Digital modes
- [WSJT-X](https://wsjt.sourceforge.io/) — Weak-signal communication (FT8, JT65, etc.)
- [Codec2](https://www.rowetel.com/codec2.html) — Low-bitrate voice codec
- [Liquid-DSP](https://liquidsdr.org/) — DSP primitives
- [Direwolf](https://github.com/wb2osz/direwolf) — AX.25 / APRS
- [SoapySDR](https://github.com/pothosware/SoapySDR) — SDR abstraction
- [ARDOP](https://ardop.groups.io/) — HF data modem
- [gpredict](http://gpredict.oz9aec.net/) — Satellite tracking
- [pat](https://getpat.io/) — Winlink client
- [libusb](https://libusb.info/) — USB device access

These submodules are pinned to `*-ref03_26` baseline branches and are **never modified**. They serve as a **mathematical oracle** for the Dual-Phase Parity Test: new C++23 code must produce bit-for-bit identical output against the legacy baselines before any optimization is permitted.

**We do not modify legacy code. We do not link it into production builds. We only read it, test against it, and cite it.**

## Building

```bash
# Clone with submodules
git clone --recurse-submodules https://github.com/WhiteFoxSB/OpenRadioFramework.git
cd OpenRadioFramework

# Configure and build
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build
```

**Requirements:**
- C++23-compliant compiler (GCC 13+, Clang 17+, MSVC 19.37+)
- CMake 3.20+

## Project Structure

```
OpenRadioFramework/
├── context/          # Engineering constraints, plans, and TSRP
│   ├── agents.md     # AI agent persona definitions
│   ├── claude.md     # Core project directives
│   ├── skills.md     # Technical skill requirements
│   ├── TSRP.md       # Master Technical Standards & Refactor Protocol
│   └── plan/         # Living roadmap and archived plans
├── modules/          # Phase 1: C++23 MicroAPI libraries
├── engine/           # Phase 2: Headless daemon (future)
├── shells/           # Phase 3: Native UI clients (future)
│   ├── android/
│   ├── ios/
│   └── desktop/
├── legacy/           # Frozen upstream submodules (read-only)
├── docs/             # Documentation and Doxygen config
├── scripts/          # Build and utility scripts
└── CMakeLists.txt    # Root build configuration
```

## Contributing

We welcome contributions from radio operators, DSP engineers, and C++ developers. Please read **[CONTRIBUTING.md](CONTRIBUTING.md)** before submitting a pull request. All contributions must adhere to the [Master Technical Standards & Refactor Protocol (TSRP)](context/TSRP.md).

## License

OpenRadioFramework is licensed under the **GNU General Public License v3.0** — see [LICENSE](LICENSE) for details.

This project creates derivative works from GPLv3-licensed upstream projects. In accordance with the GPL, the ORF core and all modules are released under the same license to ensure the amateur radio community's work remains free and open.

## Acknowledgments

OpenRadioFramework stands on the shoulders of decades of work by the amateur radio open-source community. We gratefully acknowledge the authors and maintainers of Hamlib, fldigi, WSJT-X, Codec2, Liquid-DSP, Direwolf, SoapySDR, ARDOP, gpredict, pat, and libusb. Their engineering is the foundation this project builds upon.

73 de the ORF team.
