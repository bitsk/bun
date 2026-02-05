# RISC-V Support for Bun

## Overview

This document describes the RISC-V 64-bit architecture support for Bun. **Note: This is currently experimental and blocked by JavaScriptCore build issues.**

## Status

| Component | Status | Notes |
|-----------|--------|-------|
| Architecture Detection | ✅ Complete | RISC-V 64-bit detection added |
| Build System | ✅ Complete | CMake and Zig configuration updated |
| FFI Support | ✅ Complete | RISC-V calling convention supported |
| JSC Integration | ❌ Blocked | WebKit build fails on RISC-V |
| WebAssembly | ⚠️ Not Supported | Requires JSC fix |
| Testing | ⏸️ Pending | Waiting for JSC build |

## Known Issues

### JavaScriptCore Build Failure

**Problem**: WebKit JSC cannot be built for RISC-V due to WebAssembly header conflicts.

**Error**:
```
WasmValueLocation.h:58:39: error: expected ')' before 'regs'
```

**Root Cause**:
- WebKit's build system copies WASM headers unconditionally
- LLInt (interpreter) includes WASM headers even with ENABLE_WEBASSEMBLY=OFF
- WASM headers depend on JIT types not available in C_LOOP mode

**Attempted Solutions**:
1. ✅ ENABLE_WEBASSEMBLY=OFF - CMake succeeds, build fails
2. ✅ Patching LLIntOfflineAsmConfig.h - Headers re-copied
3. ✅ Removing WASM headers - Re-copied by build system
4. ❌ Patching CMakeLists.txt - Too complex

**Impact**: Cannot build working Bun binary for RISC-V.

## Building for RISC-V

### Prerequisites

```bash
# Install dependencies
sudo apt-get update
sudo apt-get install -y \
    cmake ninja-build ruby python3 \
    libicu-dev libxml2-dev libxslt1-dev \
    libsqlite3-dev libssl-dev

# Install Zig
wget https://ziglang.org/download/0.13.0/zig-linux-x86_64-0.13.0.tar.xz
tar xf zig-linux-x86_64-0.13.0.tar.xz
export PATH="$PWD/zig-linux-x86_64-0.13.0:$PATH"
```

### Build Steps

```bash
# Clone Bun
git clone https://github.com/oven-sh/bun.git
cd bun

# Configure for RISC-V
mkdir -p build
cd build
cmake .. \
    -DZIG_TARGET=riscv64-linux-gnu \
    -DCMAKE_SYSTEM_PROCESSOR=riscv64 \
    -DCMAKE_BUILD_TYPE=Release

# Build (will fail at JSC linking stage)
cmake --build . --target bun -j$(nproc)
```

## Architecture Changes

### Files Modified

1. **src/env.zig** - Added `isRiscv64` constant and `riscv64` architecture enum
2. **cmake/tools/SetupZig.cmake** - Added RISC-V detection
3. **src/compile_target.zig** - Updated compilation targets
4. **src/Global.zig** - Added riscv64 arch_name
5. **src/cli/upgrade_command.zig** - Added riscv64 arch_label
6. **src/analytics.zig** - Added riscv64 platform_arch
7. **src/analytics/schema.zig** - Added riscv64 to Architecture enum
8. **src/crash_handler.zig** - Added riscv64 arch_display_string
9. **src/bun.js/api/ffi.zig** - Added RISC-V system paths
10. **src/workaround_missing_symbols.zig** - Added RISC-V stat functions

### RISC-V Calling Convention

- **Integer arguments**: a0-a7 (x10-x17)
- **Floating-point arguments**: fa0-fa7 (f10-f17)
- **Return values**: a0-a1 (x10-x11)
- **Stack**: 16-byte aligned

## Testing

Tests cannot be run until JSC build issue is resolved. See `.sisyphus/notepads/bun-riscv-execution-plan/phase4-test-plan.md` for planned tests.

## Future Work

### Option 1: WebKit Upstream Fix
- Work with WebKit community to fix ENABLE_WEBASSEMBLY=OFF
- Timeline: Weeks to months
- Success probability: Medium

### Option 2: QuickJS Integration
- Replace JSC with QuickJS for RISC-V builds
- Timeline: 4-6 weeks
- Success probability: High

### Option 3: Alternative JavaScript Engines
- Evaluate Deno or Node.js for RISC-V
- Timeline: Unknown
- Success probability: Varies

## Resources

- **RISC-V Machine**: openkylin@192.168.5.205
- **Plan**: `.sisyphus/plans/bun-riscv-execution-plan.md`
- **Progress**: `.sisyphus/notepads/bun-riscv-execution-plan/progress-summary.md`
- **Blocker Details**: `.sisyphus/notepads/bun-riscv-execution-plan/blocker-task-2-1.md`

## Contributing

To help with the RISC-V port:

1. **WebKit Expertise**: Help fix ENABLE_WEBASSEMBLY=OFF support
2. **Alternative Engines**: Implement QuickJS backend for Bun
3. **Testing**: Once build works, test on RISC-V hardware

## License

Same as Bun - MIT License
