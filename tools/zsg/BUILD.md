# Build Instructions

## Quick Start

### Linux
```bash
cd zephyr-settings-generator
make static          # Build fully static binary
```

Binary location: `build-static/src/zsg`

### macOS
```bash
cd zephyr-settings-generator
make static          # Build with static libraries where possible
```

Binary location: `build-static/src/zsg`

**Note:** macOS does not support full static linking. The binary will use static libraries where possible but will still link dynamically to system frameworks.

### Windows
```cmd
cd zephyr-settings-generator
build_static.bat     # Build static binary
```

Binary location: `build-static\src\zsg.exe` or `build-static\src\Release\zsg.exe`

---

## Requirements

### All Platforms
- **CMake** >= 3.13
- **C++ Compiler** with C++17 support
- **Build Tool**: Ninja (recommended) or Make

### Linux
```bash
# Rocky Linux / RHEL / CentOS
sudo dnf install gcc-c++ cmake ninja-build git

# Ubuntu / Debian
sudo apt-get install build-essential cmake ninja-build git
```

### Windows
- **Visual Studio 2019/2022** with "Desktop development with C++"
- **CMake** (https://cmake.org/download/)
- **Ninja** (optional, recommended)

### macOS
```bash
xcode-select --install
brew install cmake ninja
```

---

## Build Methods

### Method 1: Using Build Scripts (Recommended)

**Linux/macOS:**
```bash
./build_static.sh
```

**Windows:**
```cmd
build_static.bat
```

### Method 2: Using Makefile (Linux/macOS only)

```bash
make static          # Build static binary
make clean           # Clean build artifacts
sudo make install    # Install to /usr/local/bin
```

### Method 3: Manual CMake

**Linux/macOS:**
```bash
mkdir build-static
cd build-static
cmake -G Ninja -DCMAKE_BUILD_TYPE=Release -DBUILD_STATIC=ON ..
cmake --build .
```

**Windows:**
```cmd
mkdir build-static
cd build-static
cmake -G "Visual Studio 17 2022" -DCMAKE_BUILD_TYPE=Release -DBUILD_STATIC=ON ..
cmake --build . --config Release
```

---

## CMake Options

| Option | Default | Description |
|--------|---------|-------------|
| `BUILD_STATIC` | OFF | Enable static linking |
| `ATMOSIC_ZSG` | ON | Enable YAML support |
| `CMAKE_BUILD_TYPE` | Release | Debug or Release |

Example:
```bash
cmake -DBUILD_STATIC=ON -DCMAKE_BUILD_TYPE=Release ..
```

---

## Verification

### Test the Binary

**Linux/macOS:**
```bash
./build-static/src/zsg --help
./test_binary.sh
```

**Windows:**
```cmd
build-static\src\zsg.exe --help
```

### Verify Static Linking

**Linux:**
```bash
ldd build-static/src/zsg
# Should show "not a dynamic executable"

file build-static/src/zsg
# Should show "statically linked"
```

**Windows:**
```cmd
dumpbin /dependents build-static\src\zsg.exe
REM Should only show system DLLs (KERNEL32.dll, etc.)
REM Should NOT show MSVCP140.dll or VCRUNTIME140.dll
```

---

## Troubleshooting

### Linux: "cannot find -lstdc++fs"

**Problem:** GCC version < 9.0 requires explicit linking of filesystem library.

**Solution:** Upgrade GCC:
```bash
# Rocky Linux / RHEL
sudo dnf install gcc-toolset-11
scl enable gcc-toolset-11 bash

# Ubuntu
sudo apt-get install g++-11
export CXX=g++-11
```

### Linux: "GLIBC_2.XX not found"

**Problem:** Binary built on newer system won't run on older systems.

**Solution:** Use static linking (already enabled with `BUILD_STATIC=ON`).

### Windows: "Binary requires MSVCP140.dll"

**Problem:** Dynamic linking to C++ runtime.

**Solution:** Rebuild with static linking:
```cmd
rmdir /s /q build-static
build_static.bat
```

### Windows: "CMake could not find Visual Studio"

**Problem:** Visual Studio not installed or not detected.

**Solution:**
1. Install Visual Studio 2019/2022 with "Desktop development with C++"
2. Or specify generator manually:
   ```cmd
   cmake -G "Visual Studio 17 2022" ..
   ```

### All Platforms: "yaml-cpp not found"

**Problem:** yaml-cpp dependency not available.

**Solution:** CMake will automatically fetch yaml-cpp. Ensure internet connection is available during first build.

---

## Cross-Distribution Compatibility (Linux)

### Problem

Tools built on newer systems (e.g., urchin with glibc 2.34) won't run on older systems (e.g., Rocky Linux with glibc 2.17).

### Solution

Use static linking with `BUILD_STATIC=ON`:

```bash
make static
```

This creates a fully static binary that runs on any Linux distribution.

### Verification

```bash
ldd build-static/zsg
# Should output: "not a dynamic executable"
```

---

## Advanced Topics

### Building for Specific Distribution

**Option 1: Build on target system**
```bash
# On Rocky Linux
make static
```

**Option 2: Use Docker**
```bash
docker run --rm -v $(pwd):/work -w /work rockylinux:8 bash -c "
  dnf install -y gcc-c++ cmake ninja-build git &&
  make static
"
```

### Custom Installation Path

```bash
cmake -DCMAKE_INSTALL_PREFIX=/opt/zsg ..
make
sudo make install
```

### Debug Build

```bash
cmake -DCMAKE_BUILD_TYPE=Debug -DBUILD_STATIC=ON ..
cmake --build .
```

---

## CI/CD Integration

### GitHub Actions Example

```yaml
name: Build

on: [push, pull_request]

jobs:
  build-linux:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install dependencies
        run: sudo apt-get install -y build-essential cmake ninja-build
      - name: Build
        run: make static
      - name: Test
        run: ./test_binary.sh build-static/zsg

  build-windows:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build
        run: build_static.bat
      - name: Test
        run: build-static\zsg.exe --version
```

### Jenkins Example

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'make static'
            }
        }
        stage('Test') {
            steps {
                sh './test_binary.sh build-static/zsg'
            }
        }
    }
}
```

---

## Binary Size Comparison

| Platform | Dynamic | Static | Notes |
|----------|---------|--------|-------|
| Linux | ~1-2 MB | ~8 MB | Includes glibc, libstdc++ |
| Windows | ~500 KB | ~2-3 MB | Includes C++ runtime |
| macOS | ~1 MB | ~5 MB | System frameworks still dynamic |

Static binaries are larger but have no external dependencies.

---

## See Also

- [USAGE.md](USAGE.md) - How to use zsg
- [README.md](README.md) - Project overview

