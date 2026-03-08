# Contributing to OpenRadioFramework

Thank you for your interest in contributing to OpenRadioFramework. This project modernizes decades of amateur radio software into a unified C++23 framework, and contributions from the community are essential to that mission.

## Before You Start

**Read the TSRP.** All contributions must conform to the [Master Technical Standards & Refactor Protocol (TSRP)](context/TSRP.md). This document governs every aspect of ORF development — from naming conventions and namespace structure to memory management patterns and error handling. Pull requests that do not adhere to the TSRP will be requested to revise before review.

Key requirements from the TSRP that every contributor must understand:

- **C++23 Standard**: All new code must target C++23. Use `std::expected<T, E>` for error handling, `std::span` for buffer passing, and RAII for all resource management.
- **No raw memory management**: `new`/`delete` and raw owning pointers are prohibited. Use `std::unique_ptr` and `std::shared_ptr` exclusively.
- **No exceptions in the DSP path**: Real-time signal processing code must never throw. Use `std::expected` with monadic chaining (`and_then`, `transform`, `or_else`).
- **Namespace**: All code lives under `ORF::` with appropriate sub-namespaces (e.g., `ORF::Rig`, `ORF::Modem`, `ORF::DSP`).
- **File size limit**: No single file may exceed 1,500 lines. Target 1,000 lines or fewer.

## Development Phases

ORF is developed in strict sequential phases. Ensure your contribution targets the correct phase:

| Phase | Scope | Status |
|---|---|---|
| **Phase 1** | Core C++23 MicroAPI libraries — no networking, no daemon | Active |
| **Phase 2** | Headless daemon with gRPC/Protobuf IPC | Not started |
| **Phase 3** | Native UI shells (Kotlin, Swift, WinUI, Qt) | Not started |

Do not introduce Phase 2 concerns (gRPC, Protobuf, daemon lifecycle) into Phase 1 library code.

## The `legacy/` Directory is Read-Only

The `legacy/` submodules are **frozen reference snapshots**. They exist solely as a mathematical oracle for parity testing. **Never modify, update, or commit changes to any file under `legacy/`.**

If you discover a bug in legacy code, document it — do not fix it. The frozen baseline is the specification.

## Naming Conventions

The TSRP mandates the following (see TSRP Section 2.5 for the full table):

| Element | Convention | Example |
|---|---|---|
| Namespaces | PascalCase | `ORF::SignalProcessing` |
| Classes / Structs | PascalCase | `FastFourierTransform` |
| Methods / Functions | camelCase | `computePhaseOffset()` |
| Local variables | snake_case | `payload_buffer_size` |
| Private members | `m_` prefix | `m_active_vfo_frequency` |
| Constants / Macros | SCREAMING_SNAKE | `MAX_PAYLOAD_LENGTH` |
| Header files | snake_case + `.hpp` | `fast_fourier_transform.hpp` |
| Source files | snake_case + `.cpp` | `fast_fourier_transform.cpp` |

## Documentation Standards

### Doxygen

All public headers must be documented using **Doxygen** syntax. This is not optional — undocumented public APIs will not be merged.

### `@legacy_origin` Tags

When porting algorithms or math from a legacy project, you **must** include a `@legacy_origin` tag in the Doxygen comment that traces the new code back to its source in the frozen `legacy/` submodules. This maintains FOSS attribution and mathematical traceability.

```cpp
/**
 * @brief Compute the Goertzel filter magnitude for a single frequency bin.
 *
 * @param samples Input signal buffer.
 * @param target_freq_hz Target frequency in Hz.
 * @param sample_rate_hz Sample rate in Hz.
 * @return Magnitude of the target frequency component.
 *
 * @legacy_origin legacy/fldigi/src/misc/goertzel.cxx:42
 */
[[nodiscard]] auto computeGoertzelMagnitude(
    std::span<const float> samples,
    float target_freq_hz,
    float sample_rate_hz
) -> std::expected<float, ORF::Error>;
```

This tag creates an auditable chain from every new function back to the original open-source work it derives from.

## The Dual-Phase Parity Test

All refactored algorithms must pass the **Dual-Phase Parity Test** (TSRP Section 7.2):

1. **Phase A (Analysis)**: Write a unit test that wraps the legacy C/Fortran function and captures its exact input/output behavior.
2. **Phase B (Implementation)**: Write the new C++23 implementation. It must produce **bit-for-bit identical output** against the Phase A test before any optimization is permitted.

Pull requests that port legacy algorithms without accompanying parity tests will not be merged.

## Pull Request Workflow

1. **Fork** the repository and create a feature branch from `main`.
2. **Read** the relevant legacy source code in `legacy/` to understand the algorithm you are porting or the interface you are extending.
3. **Write the header first.** ORF uses header-driven development — the `.hpp` API contract is designed and reviewed before any `.cpp` implementation is written.
4. **Write Phase A parity tests** if you are porting legacy logic.
5. **Write the implementation** and ensure Phase B parity passes.
6. **Document** all public APIs with Doxygen, including `@legacy_origin` tags where applicable.
7. **Submit your PR** against `main`. Include a clear description of what was ported, which legacy files were referenced, and the parity test results.

## Build and Test

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build
ctest --test-dir build --output-on-failure
```

## Code of Conduct

Be respectful, constructive, and collaborative. The amateur radio community has a long tradition of mutual aid and knowledge sharing — we carry that tradition forward here.

## License

By contributing to OpenRadioFramework, you agree that your contributions will be licensed under the **GNU General Public License v3.0**.

## Questions?

Open an issue on GitHub or start a discussion. We're happy to help new contributors get oriented in the codebase and the TSRP.

73 and thanks for contributing.
