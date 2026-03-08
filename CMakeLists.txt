cmake_minimum_required(VERSION 3.25)
project(OpenRadioFramework VERSION 0.1.0 LANGUAGES CXX C)

# ==============================================================================
# 1. C++23 ENFORCEMENT & SYSTEM POLICIES
# ==============================================================================
# Mandate C++23 and strictly forbid compiler-specific dialects
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF) 

# Export compile commands for IDEs (VS Code / clangd)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# ==============================================================================
# 2. AEROSPACE-GRADE COMPILER FLAGS
# ==============================================================================
# Enforce strict warning levels to prevent memory leaks and undefined behavior
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    add_compile_options(
        -Wall -Wextra -Wpedantic
        -Wconversion -Wshadow -Wnon-virtual-dtor
        -Wold-style-cast -Wcast-align -Wunused
        -Woverloaded-virtual -Wnull-dereference
        -Wformat=2
    )
    
    # Aggressive DSP optimizations for the Release build
    set(CMAKE_CXX_FLAGS_RELEASE "-O3 -march=native -flto")
elseif(MSVC)
    # Strict mode for MSVC (Windows builds)
    add_compile_options(/W4 /permissive- /Zc:__cplusplus)
endif()

# ==============================================================================
# 3. THE LEGACY SANDBOX (READ CAREFULLY)
# ==============================================================================
# We specifically DO NOT use add_subdirectory(legacy) here.
# If we did, CMake would try to build the 11 legacy projects using their 
# outdated, conflicting build scripts, causing a fatal crash.
# 
# Instead, the modern `modules/` will selectively reach into the `legacy/` 
# directory to compile ONLY the specific source files they need for the wrapper.

# ==============================================================================
# 4. DEPENDENCY MANAGEMENT (Protobuf / gRPC)
# ==============================================================================
# The TSRP dictates that the Headless Engine communicates with Native UIs via 
# Protobuf/gRPC. We will fetch those dependencies here later.
# set(Protobuf_USE_STATIC_LIBS ON)
# find_package(Protobuf REQUIRED)

# ==============================================================================
# 5. OPEN RADIO FRAMEWORK (ORF) ARCHITECTURE INCLUDES
# ==============================================================================

# Enable CTest for the Dual-Phase Parity Testing
enable_testing()

# 1. Build the MicroAPIs (orf-rig, orf-modem, orf-weak, etc.)
add_subdirectory(modules)

# 2. Build the Core Routing Engine (Uncomment when engine is scaffolded)
# add_subdirectory(engine)

# 3. Build the Native UIs (Uncomment when shells are scaffolded)
# add_subdirectory(shells)