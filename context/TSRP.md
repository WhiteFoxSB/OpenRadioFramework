Project OpenRadioFramework (ORF): Master Technical Standards \& Refactor Protocol (TSRP v0.0.1)

Index

Chapter 1: Architectural Philosophy \& Overview



Chapter 2: Code Structure \& Build System



2.1 Namespace Encapsulation



2.2 Directory Hierarchy (Pitchfork Layout)



2.3 Monorepo \& Git Subtree Workflow



2.4 Build System \& Linkage



2.5 Semantic \& File Naming Conventions



2.6 File Size Constraints



Chapter 3: The MicroAPI Specification \& IPC



3.1 ORF-Calling-Convention



3.2 Zero-Copy Serialization



Chapter 4: Hardware Abstraction \& Memory Management



4.1 Zero-Copy HAL via std::span



4.2 RAII Resource Management



4.3 Deterministic Error Handling



Chapter 5: Concurrency \& Telemetry



5.1 Priority Threading Model



5.2 Observable State Pattern



5.3 Safe RF Buffer Handling



Chapter 6: Documentation \& Metadata



6.1 In-Code Documentation



6.2 Automated Specification Generation



Chapter 7: Autonomous Agent Engineering Protocols



7.1 The Persistence Rule



7.2 The Dual-Phase Parity Test



7.3 The Plan Rotation Protocol



Chapter 1: Architectural Philosophy \& Overview

Project OpenRadioFramework (ORF) shall consolidate fragmented digital radio signal processing, hardware abstraction, and protocol decoding components into a unified, location-agnostic suite of high-performance, C++23-native 'MicroAPI' modules.



The architecture dictates a strict, headless, Linux-first design. The core digital signal processing (DSP) and radio control logic shall be decoupled from platform-specific User Interface (UI) frameworks. All modules shall operate as headless daemons capable of executing on resource-constrained embedded platforms (e.g., Raspberry Pi), providing remote control capabilities to native UI clients (Kotlin, Swift, WinUI, Qt) strictly via structured Inter-Process Communication (IPC).



Chapter 2: Code Structure \& Build System

2.1 Namespace Encapsulation

To guarantee isolation and prevent symbol collisions, all codebase components must reside strictly within the ORF:: namespace. Sub-namespaces shall be utilized to logically demarcate specific micro-modules (e.g., ORF::Rig, ORF::Modem, ORF::DSP).



2.2 Directory Hierarchy (Pitchfork Layout)

Modules shall implement a strict Pitchfork Layout to separate public interfaces from private implementations.



Directory Path	Functional Purpose

build/	Ephemeral directory for all CMake build artifacts. Ignored by version control.

src/	Contains all private implementation files (.cpp) and module-private headers (.hpp).

include/ORF/	Houses the public API headers, mirroring the namespace hierarchy.

tests/	Contains unit and integration test suites.

external/	Repository for third-party dependencies.

docs/	Contains Doxygen configuration files, markdown manuals, and SPECS.md.

schemas/	Houses Protocol Buffer (.proto) definition files dictating IPC contracts.

2.3 Monorepo \& Git Subtree Workflow

The system shall utilize a monorepo architecture to ensure atomic commits across DSP engines, hardware abstractions, and IPC definitions. For external dependencies and the standalone release of individual modules to the open-source community, Git Subtrees shall be mandated over Git Submodules to maintain a linear, unified commit history and facilitate seamless transition workflows.   



2.4 Build System \& Linkage

The build system shall be driven by CMake (v3.20+).



Modules shall be defined as strongly-typed targets using add\_library().



Dependencies shall be managed exclusively via target\_link\_libraries() with explicit PUBLIC, PRIVATE, and INTERFACE scoping.



The architecture defaults to static linkage (BUILD\_SHARED\_LIBS=OFF) to eliminate dependency resolution issues in deployed environments, creating self-contained executable binaries.



2.5 Semantic \& File Naming Conventions

The codebase shall enforce the following deterministic naming conventions for code elements and files:



Code Element / File	Naming Convention	Example

Namespaces	PascalCase	ORF::SignalProcessing

Classes / Structs	PascalCase	FastFourierTransform

Methods / Functions	camelCase	computePhaseOffset()

Local Variables	snake\_case	payload\_buffer\_size

Private Members	m\_ prefix	m\_active\_vfo\_frequency

Constants / Macros	SCREAMING\_SNAKE	MAX\_PAYLOAD\_LENGTH

Header Files	snake\_case + .hpp	fast\_fourier\_transform.hpp

Source Files	snake\_case + .cpp	fast\_fourier\_transform.cpp

2.6 File Size Constraints

To guarantee maximum readability, maintainability, and compatibility with AI context windows, all individual files shall be strictly capped at a maximum of 1,500 lines of code. The ideal target length for any single file is 1,000 lines or fewer. While this constraint generates a higher total file count, it is architecturally mandated to enforce strict modularity and prevent logic entanglement.



Chapter 3: The MicroAPI Specification \& IPC

3.1 ORF-Calling-Convention

The core DSP and hardware modules shall execute entirely as a headless background daemon. This daemon shall communicate with external, dynamically typed remote shells (Swift, Kotlin) exclusively via an API defined by Google's Protocol Buffers (Protobuf). Local IPC shall utilize gRPC layered over Unix Domain Sockets (UDS) to bypass TCP/IP networking overhead.



3.2 Zero-Copy Serialization

Internal memory handling shall leverage Protobuf's ZeroCopyInputStream and ZeroCopyOutputStream interfaces. The stream shall return a pointer directly to the backing array (e.g., a memory-mapped file or hardware DMA buffer), completely bypassing intermediate copy operations during the ingestion of massive baseband data streams.   



Chapter 4: Hardware Abstraction \& Memory Management

4.1 Zero-Copy HAL via std::span

The Hardware Abstraction Layer (ORF\_HAL) shall utilize C++20 std::span to pass contiguous memory blocks from hardware drivers to DSP processing modules. When hardware writes baseband I/Q data via DMA into a pinned kernel buffer, ORF\_HAL shall map that memory and pass a std::span<std::complex<float>> directly down the pipeline. This strictly forbids intermediate std::vector allocations and deep copies within the critical audio path.



4.2 RAII Resource Management

Manual memory and resource lifecycle management is strictly prohibited. All interactions with opaque C handles, hardware resources, or file descriptors must be comprehensively encapsulated within Resource Acquisition Is Initialization (RAII) wrappers. Custom deleters paired with std::unique\_ptr shall be used to guarantee deterministic cleanup the exact moment a wrapper object falls out of scope.



4.3 Deterministic Error Handling

The usage of exceptions or integer return codes for flow control within real-time loops is forbidden. All MicroAPIs shall return C++23 std::expected<T, E> for operations possessing a failure state. Developers shall utilize monadic chaining operations (and\_then, transform, or\_else) to write linear pipelines that elegantly short-circuit upon failure without the overhead of try-catch blocks.   



Chapter 5: Concurrency \& Telemetry

5.1 Priority Threading Model

Computational workloads shall be partitioned into strict priority tiers. Core audio acquisition and DSP loops shall execute on real-time scheduled threads operating at maximum system priority. These threads are strictly forbidden from performing blocking system calls, including dynamic memory allocation (new/malloc), file I/O, or acquiring kernel-level mutexes.



5.2 Observable State Pattern

Telemetry broadcasts (e.g., signal-to-noise ratio, spectrum waterfalls) from the high-priority DSP thread to the low-priority UI thread shall utilize a wait-free Observable State pattern.



For single-value metrics, native machine-instruction std::atomic variables shall be used.



For dynamically sized data, the system shall employ a Sequence Lock (SeqLock) or a wait-free atomic pointer exchange paired with a "zombie list". The UI thread shall assume responsibility for safely freeing old zombie nodes, ensuring the DSP thread never invokes the system memory allocator.   



5.3 Safe RF Buffer Handling

All memory required for RF buffers shall be strictly pre-allocated at application startup. Data structures accessed across thread boundaries shall be padded with dead zones matching the CPU's cache line size to entirely eliminate false sharing.



Chapter 6: Documentation \& Metadata

6.1 In-Code Documentation

All source code shall be exhaustively documented using Doxygen syntax. To maintain mathematical traceability for algorithms adapted from legacy systems, the @legacy\_origin custom tag must be used to provide an immutable cryptographic hash or URL link to the original historical source file and commit.



6.2 Automated Specification Generation

The CI/CD pipeline shall automatically parse Doxygen output, .proto schemas, and CMake target dependencies to generate a living SPECS.md document for each module. This guarantees the architectural blueprint remains continuously synchronized with the compiled codebase.



Chapter 7: Autonomous Agent Engineering Protocols

The utilization of AI coding agents requires strict context engineering protocols to prevent hallucinated APIs and architectural drift.   



7.1 The Persistence Rule

The repository root (\\OpenRadioFrameWork) shall contain a dedicated \\context directory housing master machine-readable constraint files: agents.md, claude.md, and skills.md. AI agents are strictly mandated to read these files every time a new conversation is started and after every conversation compaction. This ensures the agent remains perpetually grounded in the project's core philosophy (e.g., file size limits, mandatory std::span usage, prohibition of std::mutex in DSP) prior to generating any code modifications.   



7.2 The Dual-Phase Parity Test

Agents must employ a strict Test-Driven Development (TDD) methodology when refactoring legacy logic:



Phase A (Analysis): The agent generates a comprehensive unit test wrapping the legacy C/C++ logic to capture exact input/output behaviors.



Phase B (Implementation): The agent writes the new C++23 module, which must achieve absolute bit-for-bit parity against the Phase A legacy test before any optimization is permitted.



7.3 The Plan Rotation Protocol

To actively combat attention dilution and context rot, no single context plan shall exceed 400 lines of code. Before writing code, the agent must generate a sequential plan located at \\OpenRadioFrameWork\\context\\plan\\plan.md. Once the 400-line complexity threshold is reached, the agent must summarize its progress. It must then archive the old plan into the \\OpenRadioFrameWork\\context\\plan\\archive\\ directory for reference purposes, wipe the ephemeral chat history to clear deprecated code iterations, ingest the new summary, and begin a fresh plan.md in the \\context\\plan\\ folder.   

Chapter 8: Project Phasing & Execution Strategy
To systematically manage the architectural reboot, development shall proceed in three strict, sequential phases. Work on a subsequent phase shall not commence until the prior phase achieves full feature parity and test validation.

8.1 Phase 1: ORF Core Library (MicroAPIs)
The initial development phase shall exclusively focus on building the unified C++23 ORF:: namespace libraries.

This phase shall strictly exclude all daemonization, networking, and inter-process communication logic.

Development shall concentrate solely on refactoring hardware abstractions, digital signal processing, memory management, and protocol decoding into modular, statically linkable C++23 libraries.

8.2 Phase 2: ORF-Engine (Headless Daemon)
Upon completion of the core library, development shall transition to building the ORF-Engine.

The system shall implement a headless background daemon that consumes the Phase 1 MicroAPIs.

This phase shall establish the networking and IPC layers, strictly implementing the ORF-Calling-Convention utilizing Protobuf and gRPC over Unix Domain Sockets (UDS).

8.3 Phase 3: Native UI Shells
The final phase shall consist of constructing platform-specific, native user interfaces.

Development shall target native frameworks tailored to each OS (e.g., Kotlin for Android, Swift for macOS/iOS, WinUI for Windows, Qt for Linux desktop).

The UI shells shall contain absolutely zero signal processing or hardware logic.

These shells shall act strictly as remote clients, communicating with the headless ORF-Engine exclusively via the Protobuf contracts established in Phase 2.





