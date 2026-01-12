# Complete Go Build Flags Guide: From Basics to Advanced

## TABLE OF CONTENTS
1. **Fundamentals & Command Structure**
2. **Core Build Flags (Essential)**
3. **Output & Control Flags**
4. **Compiler Flags (gcflags)**
5. **Linker Flags (ldflags)**
6. **CGO Flags**
7. **Cross-Compilation (GOOS/GOARCH)**
8. **Build Tags & Conditional Compilation**
9. **Module Management Flags**
10. **Debug & Optimization Flags**
11. **Production Build Patterns**
12. **Complete Reference & Checklists**

---

## PART 1: FUNDAMENTALS & COMMAND STRUCTURE

### 1.1 Go Build Command Overview

```
go build [build flags] [packages]

go build flags:
├── Core flags: -a, -i, -o, -v, -x, -n, -p, -work
├── Compiler: -gcflags, -gccgoflags, -asmflags
├── Linker: -ldflags
├── Dependencies: -mod, -modcacherw, -modfile
├── Tags: -tags, -race, -msan, -cover
├── Platform: -GOOS, -GOARCH, -GOCPU, -GOMIPS
└── Other: -trimpath, -toolexec
```

### 1.2 Basic Command Structure

```bash
# Basic build
go build

# Build with flags
go build -o myapp main.go

# Build entire package
go build ./cmd/app

# Build multiple packages
go build ./...

# Build and install
go build -i ./cmd/app

# Multiple flags
go build -v -x -o app -ldflags="-s -w" main.go
```

### 1.3 Build Process Pipeline

```
Source Code (.go files)
    ↓
[Compilation] → Intermediate code
    ↓
[Linking] → Object files
    ↓
[Optimization] → Binary executable
    ↓
Binary (executable)
```

---

## PART 2: CORE BUILD FLAGS (ESSENTIAL)

### 2.1 `-a` Flag (Force Rebuild)

```go
// main.go
package main

func main() {
    println("Hello World")
}
```

```bash
# Normal build (uses cache)
go build main.go
# Output: 0.1 seconds (uses cached compiled packages)

# Force rebuild everything
go build -a main.go
# Output: 0.3 seconds (rebuilds all dependencies)
```

**When to use `-a`:**
- When you've modified Go compiler or standard library
- When cache might be corrupted
- When you want fresh compilation guarantees
- Rarely needed in normal development

**Impact:** Slower builds but guarantees everything is recompiled

---

### 2.2 `-i` Flag (Install Dependencies)

```bash
# Standard build (does NOT install)
go build main.go
# Output: Binary in current directory
# Packages not installed

# Build and install dependencies
go build -i main.go
# Output: Binary in current directory
# All imported packages installed in $GOPATH/pkg
```

**When to use `-i`:**
- When building frequently and want faster subsequent builds
- When you need packages available for other projects
- Rarely used now (modules handle this better)

**Note:** Mostly deprecated. With modules, this is handled automatically.

---

### 2.3 `-o` Flag (Output File)

```bash
# Build to current directory (default: main.go → main)
go build main.go
# Output: ./main

# Build to specific path
go build -o myapp main.go
# Output: ./myapp

# Build to nested directory
go build -o bin/release/app main.go
# Output: ./bin/release/app (creates directories if they don't exist)

# Build to absolute path
go build -o /usr/local/bin/myapp main.go
# Output: /usr/local/bin/myapp

# Cross-platform specific output
GOOS=linux go build -o app-linux main.go
GOOS=darwin go build -o app-mac main.go
GOOS=windows go build -o app-windows.exe main.go
```

**Use cases:**
- Create multiple binaries for different platforms
- Organize builds in specific directories
- Name binaries appropriately for distribution

---

### 2.4 `-v` Flag (Verbose Output)

```bash
# Silent build (default)
go build main.go
# No output unless error

# Verbose build (shows what's compiling)
go build -v main.go
# Output:
# runtime
# errors
# io
# sync
# crypto
# encoding/hex
# crypto/sha1
# crypto/md5
# main

# Very useful with packages
go build -v ./...
# Shows every package being compiled
# Helps identify compilation order and issues
```

**Typical verbose output means:**
- Package name = being compiled
- Listed in order of compilation
- Helps debug build issues
- Shows dependency resolution

---

### 2.5 `-x` Flag (Print Commands)

```bash
# Show all commands executed
go build -x main.go

# Output example:
# mkdir -p $WORK/b001/
# cat >$WORK/b001/importcfg.txt << 'EOF'
# packagefile runtime=$WORK/b001/importcfg.txt
# EOF
# cd /home/user/project
# /usr/local/go/pkg/tool/linux_amd64/compile -o $WORK/b001/_pkg_.a -trimpath "$WORK/b001=>;" importcfg.txt main.go
# /usr/local/go/pkg/tool/linux_amd64/link -o main $WORK/b001/importcfg.txt
```

**What `-x` shows:**
- Exact compiler commands executed
- Include paths and configurations
- Linker commands
- All intermediate steps

**Use cases:**
- Debug build failures
- Understand Go build internals
- Verify compiler flags being applied
- Troubleshoot linking errors

---

### 2.6 `-n` Flag (Print but Don't Execute)

```bash
# Print commands without executing
go build -n main.go
# Shows all commands but doesn't actually build

# Combine with -x to debug
go build -n -x main.go
# Shows detailed commands without building
# Safe way to preview what will happen

# Useful for scripts:
#!/bin/bash
if go build -n -x ./... 2>&1 | grep -q "error"; then
    echo "Build would fail"
else
    echo "Build looks good"
    go build ./...
fi
```

**Use cases:**
- Preview build without actually building
- Check if build would succeed
- Debug build command generation
- Dry-run for CI/CD pipelines

---

### 2.7 `-p` Flag (Number of Parallel Builds)

```bash
# Auto-detect (default: number of CPUs)
go build -p 4 main.go

# Single-threaded compilation
go build -p 1 main.go
# Output: Slow but uses minimal CPU
# Use when system is under heavy load

# Maximum parallelism
go build -p 8 main.go
# Output: Fast but uses more CPU resources

# Check default parallelism
go env GOMAXPROCS
# Output: (usually = number of CPU cores)
```

**When to use different `-p` values:**
- `-p 1`: Debugging, minimal CPU usage
- `-p 4-8`: Normal development
- `-p` in CI: Match container CPU count

**Performance impact:**
- Higher `-p` = faster builds but more CPU
- Lower `-p` = slower builds, better system responsiveness

---

### 2.8 `-work` Flag (Keep Build Artifacts)

```bash
# Normal build (temporary files deleted)
go build main.go
# Output: Binary only

# Keep work directory
go build -work main.go
# Output: Binary + keeps $WORK directory
# Example: WORK=/tmp/go-build123456789
# All intermediate object files preserved

# Inspect work directory
go build -work main.go
# $WORK directory contains:
# ├── b001/           (package 1 build)
# │   ├── _pkg_.a     (compiled package)
# │   └── importcfg.txt
# ├── b002/           (package 2 build)
# │   ├── _pkg_.a
# │   └── importcfg.txt
# └── exe/
#     └── main        (final binary)
```

**Use cases:**
- Debug compilation issues
- Inspect object files
- Understand build structure
- Troubleshoot linking errors

---

---

## PART 3: OUTPUT & CONTROL FLAGS

### 3.1 Combining Common Flags

```bash
# Development build (verbose, show commands)
go build -v -x -o myapp main.go

# Quiet production build
go build -o myapp main.go

# Debug build with all information
go build -v -x -work -o myapp main.go

# Fast parallel build
go build -p 8 -o myapp main.go

# Forced clean rebuild
go build -a -v -o myapp main.go
```

---

## PART 4: COMPILER FLAGS (gcflags)

### 4.1 `-gcflags` Overview

```bash
# General syntax:
go build -gcflags="[compiler flags]" main.go

# gcflags affects: Compilation from .go to intermediate code
# NOT affected by: Linker operations, binary size
```

### 4.2 `-gcflags="-m"` (Inlining Analysis)

```go
// main.go
package main

// Simple function - candidate for inlining
func add(a, b int) int {
    return a + b
}

// Complex function - won't be inlined
func fibonacci(n int) int {
    if n <= 1 {
        return n
    }
    return fibonacci(n-1) + fibonacci(n-2)
}

func main() {
    result := add(5, 3)          // May be inlined
    fib := fibonacci(10)          // Won't be inlined
    println(result, fib)
}
```

```bash
# Show inlining decisions
go build -gcflags="-m" main.go

# Output:
# ./main.go:5:6: can inline add
# ./main.go:10:6: cannot inline fibonacci
# ./main.go:17:18: inlining call to add
```

**What inlining means:**
- Function calls replaced with actual code
- Eliminates function call overhead
- Go compiler inlines small, simple functions automatically
- Complex functions marked "cannot inline"

**Use cases:**
- Optimize performance-critical functions
- Understand compiler decisions
- Profile and optimize hot paths

---

### 4.3 `-gcflags="-m=2"` (Verbose Inlining)

```bash
# Even more detailed inlining info
go build -gcflags="-m=2" main.go

# Output includes reasons why functions can't be inlined:
# ./main.go:5:6: can inline add
# ./main.go:10:6: cannot inline fibonacci: recursive
# ./main.go:17:18: inlining call to add
# ./main.go:18:11: inlining call to fibonacci: (cannot inline recursive)
```

**Additional details shown:**
- Reasons for not inlining (recursive, too complex, etc.)
- Cost analysis
- Call site information

---

### 4.4 `-gcflags="-N"` (Disable Optimizations)

```bash
# Normal build (optimizations enabled)
go build main.go
# Compiler applies various optimizations
# Faster binary, harder to debug

# Disable optimizations
go build -gcflags="-N" main.go
# Compiler skips optimizations
# Larger binary, easier to debug
# Breakpoints work better in debugger

# Use with debugger
go build -gcflags="-N -l" main.go
go run main.go  # Now can set breakpoints
```

**Why disable optimizations:**
- Debugging is easier (variables always available)
- Breakpoints work reliably
- Stack traces are accurate
- Line numbers correspond to source

**Trade-off:**
- Binary is larger
- Binary is slower
- Only for debugging, not production

---

### 4.5 `-gcflags="-l"` (Disable Inlining)

```bash
# Default (inlining enabled)
go build main.go

# Disable inlining only
go build -gcflags="-l" main.go
# Functions won't be inlined
# Easier to profile and debug
# Slower but more predictable

# Disable both optimizations and inlining
go build -gcflags="-N -l" main.go
# No optimizations, no inlining
# Easiest for debugging
```

**Use cases:**
- Profiling (get accurate function calls)
- Debugging (stack traces are clearer)
- Testing compiler behavior

---

### 4.6 `-gcflags="-B"` (Disable Bounds Checking)

```bash
// Array access code
arr := []int{1, 2, 3, 4, 5}
x := arr[i]  // Go checks if i < len(arr)

// With bounds checking (default):
// Slower but safe (no out-of-bounds crashes)

// Without bounds checking:
// Faster but can crash if i >= len(arr)
```

```bash
# With bounds checking (default)
go build main.go

# Disable bounds checking (unsafe!)
go build -gcflags="-B" main.go
# DO NOT USE in production
# Only for specialized performance work
```

**WARNING:** Removing bounds checking is **dangerous** and can cause:
- Memory corruption
- Undefined behavior
- Security vulnerabilities
- Random crashes

---

### 4.7 Complete gcflags Examples

```bash
# Debug build (disable all optimizations)
go build -gcflags="-N -l" -o app main.go

# Performance optimization (aggressive)
go build -gcflags="-m=2" main.go

# Analysis build (show inlining decisions)
go build -gcflags="-m" main.go

# Custom combination
go build -gcflags="-m -d=inline/stats" main.go

# Multiple files
go build -gcflags="-m" ./...
```

---

## PART 5: LINKER FLAGS (ldflags)

### 5.1 `-ldflags` Overview

```bash
# Syntax:
go build -ldflags="[linker flags]" main.go

# ldflags affects: Final linking stage
# Links object files into executable
# Can modify binary without recompiling
```

### 5.2 `-ldflags="-s"` (Strip Symbols)

```bash
// main.go
package main

func main() {
    println("Hello World")
}
```

```bash
# Normal build
go build main.go
ls -lh main
# Output: 2.1 MB main
# Includes: Debug symbols, function names, line numbers

# Strip symbols
go build -ldflags="-s" main.go
ls -lh main
# Output: 1.4 MB main (33% smaller!)
# Removes: Debug symbols, function names, line numbers

# Trade-offs:
# Pros: Smaller binary
# Cons: Can't debug, stack traces show memory addresses only
```

**What gets stripped:**
- Debug symbols (function names, local variables)
- Line number information
- Symbol table
- Stack trace readability

**Use cases:**
- Distribution builds (public release)
- Embedded systems (limited space)
- Production where debugging is not needed

---

### 5.3 `-ldflags="-w"` (Strip DWARF Debug Info)

```bash
# DWARF = Debug With Arbitrary Record Format
# Extra debug information used by debuggers

# Normal build
go build main.go
ls -lh main
# Output: 2.1 MB
# Includes full DWARF debug info

# Strip DWARF only
go build -ldflags="-w" main.go
ls -lh main
# Output: 1.8 MB (14% smaller)
# Removes DWARF tables
# Symbol table still present (for stack traces)

# Full strip (symbols + DWARF)
go build -ldflags="-s -w" main.go
ls -lh main
# Output: 1.4 MB (33% smaller)
# Best size reduction
```

**Difference from `-s`:**
- `-s`: Removes symbol table (names)
- `-w`: Removes DWARF (debug tables)
- `-s -w`: Removes both (maximum reduction)

---

### 5.4 `-ldflags="-X"` (Set Variables at Link Time)

```go
// main.go
package main

// These variables are set at link time
var (
    Version   = "unknown"
    BuildTime = "unknown"
    GitCommit = "unknown"
)

func main() {
    println("Version:", Version)
    println("Built:", BuildTime)
    println("Commit:", GitCommit)
}
```

```bash
# Build without setting variables
go build main.go
./main
# Output:
# Version: unknown
# Built: unknown
# Commit: unknown

# Build with variables set at link time
go build -ldflags="-X main.Version=1.0.0" main.go
./main
# Output:
# Version: 1.0.0
# Built: unknown
# Commit: unknown

# Set multiple variables
go build -ldflags="
  -X main.Version=1.0.0 \
  -X main.BuildTime=$(date -u +%Y-%m-%dT%H:%M:%SZ) \
  -X main.GitCommit=$(git rev-parse HEAD)
" main.go

./main
# Output:
# Version: 1.0.0
# Built: 2026-01-12T12:59:00Z
# Commit: abc123def456
```

**Important rules:**
- Variable must be in package scope (not inside functions)
- Variable must be `string` type (most common) or `bool`
- Full package path: `package.VariableName`
- For main package: `main.VariableName`

**Use cases:**
- Embed version numbers
- Store build timestamps
- Store git commit hashes
- Build configuration flags

---

### 5.5 `-ldflags="-extldflags"` (External Linker Flags)

```bash
# Pass flags to external C linker
go build -ldflags="-extldflags '-lm -lstdc++'" main.go

# Static linking (no shared libraries)
go build -ldflags="-extldflags '-static'" main.go

# Link order matters
go build -ldflags="-extldflags '-L/custom/lib -lmylib'" main.go
```

**When to use:**
- Linking C libraries (requires CGO)
- Static compilation
- Custom library paths
- System libraries

---

### 5.6 Complete ldflags Examples

```bash
# Minimal production build
go build -ldflags="-s -w" -o app main.go

# Embed version info
go build -ldflags="-X main.Version=$(git describe --tags)" main.go

# Complete metadata
go build -ldflags="
  -s -w \
  -X main.Version=1.0.0 \
  -X main.BuildTime=$(date -u +%Y-%m-%dT%H:%M:%SZ) \
  -X main.GitCommit=$(git rev-parse HEAD) \
  -X main.GoVersion=$(go version | awk '{print $3}')
" -o app main.go

# Static linking with metadata
CGO_ENABLED=0 go build -ldflags="
  -extldflags '-static' \
  -X main.Version=1.0.0
" main.go
```

---

## PART 6: CGO FLAGS

### 6.1 `CGO_ENABLED` Environment Variable

```bash
# Enable CGO (default on most systems)
CGO_ENABLED=1 go build main.go
# Allows calling C functions from Go
# Binary can link to C libraries
# Requires C compiler (gcc/clang)

# Disable CGO (pure Go)
CGO_ENABLED=0 go build main.go
# Pure Go binary
# No C dependencies
# Can be statically linked
# Easier cross-compilation
```

**When CGO_ENABLED=1 (C compiler available):**
- Can call C functions: `import "C"`
- Larger binary size
- C compiler must be available
- Slower builds
- Platform dependent

**When CGO_ENABLED=0 (Pure Go):**
- No C function calls possible
- Smaller binary size
- No C compiler needed
- Faster builds
- Easy cross-compilation

### 6.2 Pure Go vs CGO Comparison

```go
// Pure Go implementation (no CGO)
package main
import "crypto/sha256"

func hashPure() {
    h := sha256.Sum256([]byte("hello"))
    println(h[:])
}

// CGO implementation (uses C libraries)
package main
import "C"
import "crypto"

func hashCGO() {
    // Would link to OpenSSL or system library
}
```

```bash
# Pure Go (always portable)
CGO_ENABLED=0 go build main.go
# Works on any GOOS/GOARCH combination

# CGO (requires C compiler for target platform)
CGO_ENABLED=1 go build main.go
# Requires cross-compilation toolchain
```

### 6.3 Cross-Compilation with CGO

```bash
# CGO disabled (pure Go) - easy cross-compilation
GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build main.go

# CGO enabled (C libraries) - requires cross-compiler
GOOS=linux GOARCH=amd64 CGO_ENABLED=1 CC=x86_64-linux-gnu-gcc go build main.go
# Requires: gcc cross-compiler for Linux/amd64
```

**Best practice:**
- Use `CGO_ENABLED=0` when possible (pure Go)
- Only enable CGO when you need C libraries
- Always disable CGO for static binaries

---

## PART 7: CROSS-COMPILATION (GOOS/GOARCH)

### 7.1 GOOS (Target Operating System)

```bash
# macOS build
GOOS=darwin go build main.go

# Linux build
GOOS=linux go build main.go

# Windows build (creates .exe)
GOOS=windows go build main.go

# Android build
GOOS=android go build main.go

# iOS build
GOOS=ios go build main.go

# WebAssembly build
GOOS=js go build main.go
```

**Supported GOOS values:**
```
darwin    (macOS)
linux     (Linux)
windows   (Windows)
freebsd   (FreeBSD)
netbsd    (NetBSD)
openbsd   (OpenBSD)
android   (Android)
ios       (iOS)
js        (JavaScript/WebAssembly)
plan9     (Plan 9)
```

### 7.2 GOARCH (Target Architecture)

```bash
# x86-64 (most common)
GOARCH=amd64 go build main.go

# ARM 64-bit (Apple Silicon, AWS Graviton)
GOARCH=arm64 go build main.go

# ARM 32-bit
GOARCH=arm go build main.go

# Intel 386
GOARCH=386 go build main.go

# MIPS 64-bit
GOARCH=mips64 go build main.go

# PowerPC
GOARCH=ppc64 go build main.go

# WebAssembly
GOARCH=wasm go build main.go
```

**Supported GOARCH values:**
```
amd64      (Intel/AMD 64-bit)
arm64      (ARM 64-bit)
arm        (ARM 32-bit)
386        (Intel 32-bit)
mips       (MIPS)
mips64     (MIPS 64-bit)
ppc64      (PowerPC 64-bit)
riscv64    (RISC-V 64-bit)
wasm       (WebAssembly)
s390x      (IBM System z)
```

---

### 7.3 Common Cross-Compilation Combinations

```bash
# Linux AMD64 (most servers)
GOOS=linux GOARCH=amd64 go build main.go

# macOS Intel
GOOS=darwin GOARCH=amd64 go build main.go

# macOS ARM (Apple Silicon M1/M2)
GOOS=darwin GOARCH=arm64 go build main.go

# Windows AMD64
GOOS=windows GOARCH=amd64 go build main.go

# AWS Lambda (Linux ARM64)
GOOS=linux GOARCH=arm64 go build -o bootstrap main.go

# Docker container (Linux)
GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build main.go

# Raspberry Pi (Linux ARM)
GOOS=linux GOARCH=arm GOARM=7 go build main.go

# Multiple platforms in one script
for OS in linux darwin windows; do
  for ARCH in amd64 arm64; do
    GOOS=$OS GOARCH=$ARCH go build -o "app-$OS-$ARCH" main.go
  done
done
```

### 7.4 GOARM & GOMIPS (For ARM/MIPS Details)

```bash
# ARM 32-bit versions
GOOS=linux GOARCH=arm GOARM=5 go build main.go  # ARMv5
GOOS=linux GOARCH=arm GOARM=6 go build main.go  # ARMv6 (Raspberry Pi Zero/1)
GOOS=linux GOARCH=arm GOARM=7 go build main.go  # ARMv7 (Raspberry Pi 2/3/4)

# MIPS endianness
GOMIPS=softfloat go build main.go   # MIPS soft float
GOMIPS=hardfloat go build main.go   # MIPS hard float (faster)

# MIPS64 endianness
GOMIPS64=softfloat go build main.go
GOMIPS64=hardfloat go build main.go
```

### 7.5 `-GOCPU` Flag (CPU Features)

```bash
# Optimize for specific CPU features
GOCPU=v1 go build main.go      # Generic (baseline)
GOCPU=v2 go build main.go      # With SSE 4.2, POPCNT
GOCPU=v3 go build main.go      # With AVX2, LZCNT
GOCPU=v4 go build main.go      # With AVX-512

# Example: Build for older CPUs (v1)
GOOS=linux GOARCH=amd64 GOCPU=v1 go build main.go
# Works on all x86-64 CPUs (older systems)

# Example: Build for newer CPUs (v4)
GOOS=linux GOARCH=amd64 GOCPU=v4 go build main.go
# Requires CPU with AVX-512 support
# Significantly faster but less compatible
```

---

## PART 8: BUILD TAGS & CONDITIONAL COMPILATION

### 8.1 Build Tags Basics

```go
// +build linux

package main

// This file only compiled on Linux

func main() {
    println("Linux version")
}
```

```bash
# Normal build (includes all non-build-tagged files)
go build main.go

# On Linux: compiles with the +build linux file
# On macOS: skips this file (tag doesn't match)
```

### 8.2 Build Tag Syntax

```go
// Single tag
// +build linux
func linuxFunc() {}

// Multiple tags (OR relationship)
// +build linux darwin
func unixFunc() {}

// Line with AND relationship
// +build linux,amd64
func linuxAMD64Only() {}

// Negation
// +build !windows
func notWindowsFunc() {}

// Complex logic
// +build (linux && amd64) || (darwin && arm64)
func complexFunc() {}

// NEW SYNTAX (Go 1.16+)
//go:build linux
//go:build darwin,arm64
//go:build !windows
```

### 8.3 Practical Build Tag Examples

```go
// platform_unix.go
//go:build linux || darwin
package main

import "syscall"

func getPID() {
    println("Unix PID:", syscall.Getpid())
}

// platform_windows.go
//go:build windows
package main

import "syscall"

func getPID() {
    println("Windows PID:", syscall.Getpid())
}
```

```bash
# On Linux
go build ./...
# Compiles: platform_unix.go

# On Windows
go build ./...
# Compiles: platform_windows.go
```

### 8.4 Custom Build Tags

```go
// debug_enabled.go
//go:build debug
package main

func debugLog(msg string) {
    println("[DEBUG]", msg)
}

// debug_disabled.go
//go:build !debug
package main

func debugLog(msg string) {
    // No-op in production
}

// main.go
package main

func main() {
    debugLog("Starting app")
}
```

```bash
# Production build (debug disabled)
go build main.go
# debugLog does nothing

# Debug build
go build -tags=debug main.go
# debugLog prints messages

# Multiple tags
go build -tags=debug,verbose main.go
```

### 8.5 Build Tags for Feature Flags

```go
// features_full.go
//go:build !lite
package main

const (
    MaxConnections = 1000
    EnableCache = true
    EnableMetrics = true
)

// features_lite.go
//go:build lite
package main

const (
    MaxConnections = 100
    EnableCache = false
    EnableMetrics = false
)
```

```bash
# Full version (all features)
go build main.go
./app  # Full-featured

# Lite version (minimal features)
go build -tags=lite main.go
./app  # Smaller, simpler
```

---

## PART 9: MODULE MANAGEMENT FLAGS

### 9.1 `-mod` Flag (Module Mode)

```bash
# Auto mode (default): downloads missing modules
go build -mod=mod main.go

# Readonly mode: fails if modules missing (CI/CD)
go build -mod=readonly main.go

# Vendor mode: uses vendor/ directory
go build -mod=vendor main.go

# Vendor directory structure:
# vendor/
# ├── github.com/
# │   └── user/
# │       └── package/
# └── modules.txt
```

**Use cases:**
- `-mod=mod`: Development (auto-download)
- `-mod=readonly`: CI/CD (reproducible builds)
- `-mod=vendor`: Offline builds, version control dependencies

### 9.2 `-modfile` Flag (Alternative go.mod)

```bash
# Use alternative go.mod file
go build -modfile=go.prod.mod main.go

# Allows multiple module configurations:
# go.mod       (development)
# go.prod.mod  (production)
# go.test.mod  (testing)

# Build with production dependencies
go build -modfile=go.prod.mod -o app-prod main.go

# Build with test dependencies
go build -modfile=go.test.mod -o app-test main.go
```

### 9.3 `-modcacherw` Flag (Cache Write Permissions)

```bash
# Normal build (read-only module cache)
go build main.go

# Writable cache (can modify cached modules)
go build -modcacherw main.go

# Useful for:
# - Debugging issues with cached modules
# - Clearing corrupted cache
# - Development with local forks
```

---

## PART 10: DEBUG & OPTIMIZATION FLAGS

### 10.1 `-race` Flag (Race Condition Detector)

```go
// main.go
package main

var counter = 0

func main() {
    // RACE CONDITION: counter accessed from multiple goroutines
    go func() {
        for i := 0; i < 1000; i++ {
            counter++  // RACE!
        }
    }()
    
    go func() {
        for i := 0; i < 1000; i++ {
            counter++  // RACE!
        }
    }()
}
```

```bash
# Normal build (doesn't detect races)
go build main.go
./main
# May or may not have race condition

# Build with race detector
go build -race main.go
./main

# Output:
# WARNING: DATA RACE
# Write at 0x00c0001200a0 by goroutine 8:
#   main.main.func2()
#       /home/user/main.go:19 +0x54
# Previous write at 0x00c0001200a0 by goroutine 7:
#   main.main.func1()
#       /home/user/main.go:13 +0x54
```

**Impact of `-race`:**
- Binary 2-3x larger
- Binary 10-20x slower
- Only for development/testing
- NOT for production

**Best practice:**
```bash
# Always test with race detector
go test -race ./...

# Dev builds with race
go build -race -o app main.go

# Prod builds without race
CGO_ENABLED=0 go build -ldflags="-s -w" -o app main.go
```

### 10.2 `-msan` Flag (Memory Sanitizer)

```bash
# Detect memory leaks and undefined behavior
go build -msan main.go

# Only works on:
# - Linux/amd64
# - Linux/arm64
# Requires: clang/LLVM

# Similar to -race but for memory issues
go test -msan ./...
```

### 10.3 `-cover` Flag (Coverage Analysis)

```bash
# Build with coverage instrumentation
go build -cover main.go
go test -cover ./...

# Output shows coverage percentage
# PASS  coverage: 75.2% of statements

# Generate coverage profile
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
# Opens HTML coverage report in browser
```

---

## PART 11: PRODUCTION BUILD PATTERNS

### 11.1 Minimal Size Build

```bash
# Strip symbols and DWARF, disable CGO
CGO_ENABLED=0 go build -ldflags="-s -w" -o app main.go

# Check size
ls -lh app
# Output: 5.2 MB (without -s -w might be 15 MB)
```

### 11.2 Static Binary (No Dependencies)

```bash
# Pure Go, no shared libraries
CGO_ENABLED=0 go build -ldflags="-extldflags -static" -o app main.go

# Verify it's truly static
ldd app
# Output: "not a dynamic executable"

# Run anywhere (containers, minimal systems)
docker run -v $(pwd):/app alpine:3.17
# /app/app  (works even in minimal Alpine!)
```

### 11.3 Metadata-Embedded Build

```bash
#!/bin/bash

VERSION=$(git describe --tags --always)
BUILD_TIME=$(date -u +%Y-%m-%dT%H:%M:%SZ)
GIT_COMMIT=$(git rev-parse HEAD)
GO_VERSION=$(go version | awk '{print $3}')

go build \
  -ldflags=" \
    -s -w \
    -X main.Version=$VERSION \
    -X main.BuildTime=$BUILD_TIME \
    -X main.GitCommit=$GIT_COMMIT \
    -X main.GoVersion=$GO_VERSION" \
  -o app main.go

echo "Built version: $VERSION"
echo "Binary size: $(du -h app | cut -f1)"
```

### 11.4 Multi-Platform Build Script

```bash
#!/bin/bash

PLATFORMS=(
  "linux/amd64"
  "linux/arm64"
  "darwin/amd64"
  "darwin/arm64"
  "windows/amd64"
)

for PLATFORM in "${PLATFORMS[@]}"; do
  IFS='/' read -r OS ARCH <<< "$PLATFORM"
  
  output_name="app-${OS}-${ARCH}"
  if [ "$OS" = "windows" ]; then
    output_name="${output_name}.exe"
  fi
  
  echo "Building $PLATFORM..."
  GOOS=$OS GOARCH=$ARCH CGO_ENABLED=0 go build \
    -ldflags="-s -w" \
    -o "$output_name" \
    main.go
  
  ls -lh "$output_name"
done
```

### 11.5 Docker Multi-Stage Build

```dockerfile
# Dockerfile

# Stage 1: Builder
FROM golang:1.23-alpine AS builder

WORKDIR /app

# Copy source
COPY . .

# Build
RUN CGO_ENABLED=0 GOOS=linux go build \
    -ldflags="-s -w -X main.Version=$(git describe --tags)" \
    -o app main.go

# Stage 2: Runtime
FROM alpine:3.17

WORKDIR /app

# Copy binary from builder
COPY --from=builder /app/app .

# Non-root user
RUN adduser -D -u 1000 appuser
USER appuser

EXPOSE 8080

ENTRYPOINT ["./app"]
```

```bash
# Build Docker image
docker build -t myapp:latest .

# Result: ~15 MB image (alpine minimal + static binary)
```

---

## PART 12: COMPLETE REFERENCE & CHECKLISTS

### 12.1 All Flags Summary Table

| Flag | Purpose | Example | Impact |
|------|---------|---------|--------|
| `-a` | Force rebuild | `go build -a` | Slower builds |
| `-i` | Install deps | `go build -i` | (Deprecated) |
| `-o` | Output file | `go build -o app` | Changes binary name |
| `-v` | Verbose | `go build -v` | Shows what's compiling |
| `-x` | Print commands | `go build -x` | Shows compiler calls |
| `-n` | Dry-run | `go build -n` | Preview only |
| `-p` | Parallelism | `go build -p 4` | Affects build speed |
| `-work` | Keep artifacts | `go build -work` | Preserves temp files |
| `-race` | Race detection | `go build -race` | 10x slower, detects races |
| `-msan` | Memory sanitizer | `go build -msan` | Detects memory issues |
| `-cover` | Coverage | `go build -cover` | Coverage analysis |
| `-tags` | Build tags | `go build -tags=debug` | Conditional compilation |
| `-trimpath` | Strip paths | `go build -trimpath` | Better reproducibility |
| `-toolexec` | Tool wrapper | `go build -toolexec=wrapper` | Custom toolchain |

### 12.2 Environment Variables Summary

| Variable | Purpose | Example | Values |
|----------|---------|---------|--------|
| `GOOS` | Target OS | `GOOS=linux` | linux, darwin, windows, etc. |
| `GOARCH` | Target arch | `GOARCH=amd64` | amd64, arm64, arm, 386, etc. |
| `GOARM` | ARM version | `GOARM=7` | 5, 6, 7 |
| `GOMIPS` | MIPS float | `GOMIPS=hardfloat` | softfloat, hardfloat |
| `GOCPU` | CPU features | `GOCPU=v3` | v1, v2, v3, v4 |
| `CGO_ENABLED` | C linking | `CGO_ENABLED=0` | 0 (disabled), 1 (enabled) |
| `CC` | C compiler | `CC=gcc` | gcc, clang, x86_64-linux-gnu-gcc |
| `GOFLAGS` | Default flags | `GOFLAGS=-race` | Any go build flags |

### 12.3 Development Build Checklist

- [ ] **Verbose output**: `go build -v main.go`
- [ ] **Show commands**: `go build -x main.go`
- [ ] **Race detection**: `go build -race main.go`
- [ ] **Disable optimizations**: `go build -gcflags="-N -l" main.go`
- [ ] **Keep debug info**: (Default - don't use -s or -w)
- [ ] **Fast builds**: `go build -p 8 main.go`

### 12.4 Production Build Checklist

- [ ] **Strip debug**: `-ldflags="-s -w"`
- [ ] **Disable CGO**: `CGO_ENABLED=0`
- [ ] **Static linking**: `-ldflags="-extldflags -static"`
- [ ] **Embed version**: `-ldflags="-X main.Version=..."`
- [ ] **Single output file**: `-o app`
- [ ] **Cross-compile**: `GOOS=linux GOARCH=amd64`
- [ ] **Test with race**: `go test -race ./...`
- [ ] **Verify size**: `ls -lh app`

### 12.5 Performance Optimization Build

```bash
# Aggressive optimization for maximum performance
GOCPU=v4 go build \
  -gcflags="-m=2" \
  -ldflags="-s -w" \
  -o app main.go

# Requires:
# - Modern CPU with AVX-512
# - Larger binary
# - Significantly faster execution
```

### 12.6 Maximum Portability Build

```bash
# Minimal dependencies, works everywhere
GOOS=linux GOARCH=amd64 GOCPU=v1 CGO_ENABLED=0 go build \
  -ldflags="-extldflags -static -s -w" \
  -o app main.go

# Works on:
# - Any Linux x86-64 CPU
# - Any container (Alpine, Debian, etc.)
# - Offline environments
# - Air-gapped systems
```

### 12.7 Build Error Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| "undefined reference" | Missing C library | Disable CGO or install C libs |
| "unsupported GOOS/GOARCH" | Invalid combination | Check supported platforms |
| "address already in use" | Port bound | Wait or kill process |
| "too many open files" | Reduce `-p` | `go build -p 1` |
| "out of memory" | Parallel builds | `go build -p 1` |
| "symbol already defined" | Duplicate link | Check `-ldflags -X` |

---

## SUMMARY: GO BUILD FLAGS COMMAND REFERENCE

```bash
# Development builds
go build -v -x -race -o myapp main.go                    # Verbose, show commands, race detect
go build -gcflags="-N -l" -o myapp main.go               # No optimizations/inlining
go build -v ./...                                         # Build all packages verbosely

# Production builds
CGO_ENABLED=0 go build -ldflags="-s -w" -o app main.go   # Minimal size
go build -ldflags="-X main.Version=1.0.0" -o app main.go # Embed version
go build -trimpath -ldflags="-s -w" -o app main.go       # Reproducible build

# Cross-compilation
GOOS=linux GOARCH=amd64 go build main.go                 # Linux x86-64
GOOS=darwin GOARCH=arm64 go build main.go                # macOS M1/M2
GOOS=windows GOARCH=amd64 go build main.go               # Windows x86-64
GOOS=linux GOARCH=arm GOARM=7 go build main.go           # Raspberry Pi

# Static binaries
CGO_ENABLED=0 go build -ldflags="-extldflags -static" main.go

# Conditional compilation
go build -tags=debug main.go                             # Build with debug tag
go build -tags=lite main.go                              # Build lite version

# Module management
go build -mod=readonly main.go                           # CI/CD reproducible
go build -mod=vendor main.go                             # Use vendored dependencies

# Advanced builds
go build -work -x main.go                                # Keep build artifacts
go build -p 1 main.go                                    # Single-threaded compile
GOCPU=v4 go build main.go                                # Modern CPU features
```

This comprehensive guide covers every Go build flag and pattern for production-grade development!
