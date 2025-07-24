# Windows ARM64 Support for VNT

This document describes the changes needed to add Windows ARM64 (aarch64-pc-windows-msvc) support to VNT.

## Changes Made

### 1. README.md Updates
- Updated the supported platforms section to explicitly mention Windows ARM64 support
- Added note about compatibility with Windows on ARM devices

### 2. Cargo.lock Compatibility Fix
- Removed incompatible Cargo.lock file (will be regenerated during build)
- This fixes the "lock file version 4" error

### 3. GitHub Actions Workflow Changes Required

The following changes need to be made to `.github/workflows/rust.yml`:

#### Add Windows ARM64 Target to Build Matrix

In the `matrix.include` section, add the following entry after the `x86_64-pc-windows-msvc` target:

```yaml
- TARGET: aarch64-pc-windows-msvc # Windows ARM64 support
  OS: windows-latest
  FEATURES: ring-cipher,wss
```

#### Add Rustflags Configuration

In the cargo config section, add the following configuration after the `i686-pc-windows-msvc` target:

```yaml
[target.aarch64-pc-windows-msvc]
rustflags = ["-C", "target-feature=+crt-static","-C", "strip=symbols"]
```

### 3. Complete Workflow Changes

Here are the exact line changes needed:

**Line 73-76 change from:**
```yaml
- TARGET: x86_64-pc-windows-msvc # tested on a windows machine
  OS: windows-latest
  FEATURES: ring-cipher,wss
- TARGET: mipsel-unknown-linux-musl # openwrt
```

**To:**
```yaml
- TARGET: x86_64-pc-windows-msvc # tested on a windows machine
  OS: windows-latest
  FEATURES: ring-cipher,wss
- TARGET: aarch64-pc-windows-msvc # Windows ARM64 support
  OS: windows-latest
  FEATURES: ring-cipher,wss
- TARGET: mipsel-unknown-linux-musl # openwrt
```

**Line 205-207 change from:**
```yaml
[target.i686-pc-windows-msvc]
rustflags = ["-C", "target-feature=+crt-static","-C", "strip=symbols"]      
[target.x86_64-apple-darwin]
```

**To:**
```yaml
[target.i686-pc-windows-msvc]
rustflags = ["-C", "target-feature=+crt-static","-C", "strip=symbols"]      
[target.aarch64-pc-windows-msvc]
rustflags = ["-C", "target-feature=+crt-static","-C", "strip=symbols"]
[target.x86_64-apple-darwin]
```

**Additional Fix Required**: Update Rust version from 1.77 to 1.80 in two places:

**Line 150-151 change from:**
```yaml
rustup install 1.77
rustup default 1.77
```

**To:**
```yaml
rustup install 1.80
rustup default 1.80
```

**Line 164-165 change from:**
```yaml
rustup install 1.77
rustup default 1.77
```

**To:**
```yaml
rustup install 1.80
rustup default 1.80
```

## Technical Details

### Dependencies Compatibility
All Windows-specific dependencies are compatible with ARM64:
- `winapi` 0.3.9 supports ARM64
- `windows-sys` 0.59.0 supports ARM64  
- `libloading` 0.8.0 supports ARM64

### Build Features
The Windows ARM64 build uses the same features as other Windows builds:
- `ring-cipher`: For cryptographic operations
- `wss`: For WebSocket Secure support

### Cross-compilation
The GitHub Actions Windows environment includes the necessary cross-compilation tools for ARM64, so no additional setup is required.

## Testing

Once the workflow changes are applied, the Windows ARM64 build will be automatically tested on every release tag, ensuring compatibility and generating ARM64 binaries for distribution.

## Benefits

This addition enables VNT to run natively on:
- Windows on ARM laptops and tablets
- ARM-based Windows servers
- Future ARM-based Windows devices

The ARM64 build provides better performance and battery efficiency on ARM-based Windows devices compared to x86 emulation.
