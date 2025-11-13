# Zephyr Settings Generator (zsg)

A command-line tool for generating and managing Zephyr RTOS NVS (Non-Volatile Storage) settings binaries.

## Features

- ✅ Generate settings binaries from YAML configuration files
- ✅ Read and export existing NVS binaries to YAML
- ✅ Support for multiple data types (string, u8, u16, u32, u64, binary)
- ✅ Cross-platform support (Linux, Windows, macOS)
- ✅ Static linking for maximum compatibility

## Quick Start

### Linux/macOS
```bash
cd zephyr-settings-generator
make static                  # Build static binary
./build-static/src/zsg --help # Test
```

### Windows
```cmd
cd zephyr-settings-generator
build_static.bat             # Build static binary
build-static\src\zsg.exe --help
```

## Basic Usage

### Generate Binary from YAML
```bash
zsg write factory.yml 1024 2048 factory.bin
```

### Read Binary to YAML
```bash
zsg read factory.bin FACTORY 1024 2048 output.yml
```

### Show Binary Contents
```bash
zsg show factory.bin FACTORY 1024 2048 show_pretty
```

### Scan ATE Information
```bash
zsg scan factory.bin 1024 2048
```

## Documentation

- **[BUILD.md](BUILD.md)** - Build instructions for all platforms
- **[USAGE.md](USAGE.md)** - Complete usage guide with examples

## YAML Example

```yaml
settings:
  - tag: "device/serial_number"
    value: "ABC12345"
    type: "string"

  - tag: "device/id"
    value: 123456
    type: "u32"
```

See [sample.yml](../sample.yml) for more examples.

## License

This software is licensed under [BSD-2-Clause Plus Patent License](https://spdx.org/licenses/BSD-2-Clause-Patent).



