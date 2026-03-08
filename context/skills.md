\# Technical Skill Requirements \& Constraints



\## 1. Modern C++23 Enforcement

\* \*\*Memory Management:\*\* Absolute prohibition on `new`/`delete` and raw pointers for ownership. Strict enforcement of `std::unique\_ptr`, `std::shared\_ptr`, and RAII principles.

\* \*\*Error Handling:\*\* Exceptions are forbidden in the DSP critical path. You must use C++23 `std::expected` for all API boundaries and error returns.

\* \*\*Hardware Abstraction:\*\* Use type-safe `std::span` buffers for zero-copy data routing instead of legacy `void\*` passing.



\## 2. Inter-Process Communication (IPC)

\* The ORF core is a headless daemon. All communication with native remote shells (Kotlin, Swift, Qt/WinUI) must be structured around Protocol Buffers (Protobuf) or gRPC. NOTE: IPC and Daemon architecture are strictly PHASE 2. During Phase 1 (MicroAPI Library construction), the AI must focus purely on standard C++ interfaces and must NOT inject networking or gRPC logic into the core library modules.



\## 3. Code Documentation Standards

\* Use Doxygen formatting for all public headers.

\* Use `@legacy\_origin \[File:Line]` tags in comments when porting math or logic to trace the refactored code back to its frozen submodule source in `legacy/`.

