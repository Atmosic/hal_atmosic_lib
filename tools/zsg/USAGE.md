# Usage Guide

## Overview

`zsg` (Zephyr Settings Generator) operates in two modes:

1. **YAML Mode** - For complex configurations with YAML files
2. **Command-line Mode** - For simple configurations via command-line arguments

---

## Commands

| Command | Purpose | Syntax |
|---------|---------|--------|
| `write` | Generate binary from YAML | `zsg write <yaml> <sector_size> <partition_size> <output.bin>` |
| `read` | Export binary to YAML | `zsg read <bin> <type> <sector_size> <partition_size> <output.yml>` |
| `show` | Display NVS contents | `zsg show <bin> <type> <sector_size> <partition_size> <mode>` |
| `scan` | Scan ATE information | `zsg scan <bin> <sector_size> <partition_size>` |
| `-o` | Command-line mode | `zsg -o <output.bin> <tag:type:value> ...` |

---

## YAML Mode

### 1. Write: Generate Binary from YAML

Convert a YAML configuration file to a binary NVS image.

```bash
zsg write <yaml_file> <sector_size> <partition_size> <output.bin>
```

**Example:**
```bash
zsg write factory.yml 1024 2048 factory.bin
```

**Output:**
- `factory.bin` - Binary NVS image
- `factory_attr.yml` - Attribute file (stores type information)

### 2. Read: Export Binary to YAML

Convert a binary NVS image back to YAML format.

```bash
zsg read <bin_file> <FACTORY|SETTINGS> <sector_size> <partition_size> <output.yml>
```

**Example:**
```bash
zsg read factory.bin FACTORY 1024 2048 factory_readback.yml
```

**Note:** Requires the corresponding `_attr.yml` file for proper type conversion.

### 3. Show: Display NVS Information

Display the contents of a binary NVS image.

```bash
zsg show <bin_file> <FACTORY|SETTINGS> <sector_size> <partition_size> <show_pretty|show_verbose>
```

**Examples:**
```bash
# Pretty format (human-readable)
zsg show factory.bin FACTORY 1024 2048 show_pretty

# Verbose format (with hex dumps)
zsg show factory.bin FACTORY 1024 2048 show_verbose
```

### 4. Scan: Scan ATE Information

Scan and display ATE (Automatic Test Equipment) information from binary.

```bash
zsg scan <bin_file> <sector_size> <partition_size>
```

**Example:**
```bash
zsg scan factory.bin 1024 2048
```

---

## YAML File Format

### Basic Structure

```yaml
settings:
  - tag: "device/serial_number"
    value: "ABC12345"
    type: "string"

  - tag: "device/mac_address"
    value: "AA:BB:CC:DD:EE:FF"
    type: "string"

  - tag: "config/baudrate"
    value: 115200
    type: "u32"

  - tag: "config/enabled"
    value: 1
    type: "u8"
```

### Field Descriptions

- **tag** - Settings key (path-like structure with `/` separator)
- **value** - The actual value to store
- **type** - Data type (see [Data Types](#data-types))

### Complete Example

See [`sample.yml`](../sample.yml) in the repository root.

---

## Command-line Mode

For simple configurations without YAML files:

```bash
zsg -o <output.bin> <tag:type:value> [<tag:type:value> ...]
```

**Example:**
```bash
zsg -o settings.bin \
  data/serial:string:ABC12345 \
  data/mac:string:AA:BB:CC:DD:EE:FF \
  data/id:u32:123456
```

---

## Data Types

| Type | Description | Size | Example Value |
|------|-------------|------|---------------|
| `string` | ASCII string | Variable | `"Hello World"` |
| `u8` | Unsigned 8-bit integer | 1 byte | `255` |
| `u16` | Unsigned 16-bit integer | 2 bytes | `65535` |
| `u32` | Unsigned 32-bit integer | 4 bytes | `4294967295` |
| `u64` | Unsigned 64-bit integer | 8 bytes | `18446744073709551615` |
| `binary` | Raw binary data (hex) | Variable | `48656C6C6F` (hex for "Hello") |

**Notes:**
- Strings are automatically null-terminated
- Integers are stored in little-endian format
- Binary data is specified as hexadecimal (without `0x` prefix)
- Maximum value length: 1024 bytes

---

## Examples

### Device Provisioning

```yaml
settings:
  - tag: "device/serial_number"
    value: "DEV-2024-12345"
    type: "string"

  - tag: "device/mac_address"
    value: "00:11:22:33:44:55"
    type: "string"

  - tag: "device/hw_version"
    value: 2
    type: "u8"
```

Generate binary:
```bash
zsg write device.yml 4096 8192 device.bin
```

### WiFi Configuration

```yaml
settings:
  - tag: "wifi/ssid"
    value: "MyNetwork"
    type: "string"

  - tag: "wifi/password"
    value: "SecurePassword123"
    type: "string"

  - tag: "wifi/channel"
    value: 6
    type: "u8"
```

Generate binary:
```bash
zsg write wifi.yml 1024 2048 wifi.bin
```

### Verification

```bash
# Generate binary
zsg write config.yml 1024 2048 config.bin

# Verify by reading back
zsg read config.bin FACTORY 1024 2048 config_verify.yml

# Compare
diff config.yml config_verify.yml
```

---

## Tips

- Use YAML mode for complex configurations (>5 settings)
- Keep YAML files in version control
- Partition size should be multiple of sector size
- Allow room for growth (2-3x current data size)
- Keep `*_attr.yml` files for accurate read-back
- Use `/` separator in tag names (e.g., `device/serial_number`)

---

## See Also

- [BUILD.md](BUILD.md) - Build instructions
- [sample.yml](../sample.yml) - Complete YAML example
- [Zephyr Settings Documentation](https://docs.zephyrproject.org/latest/services/settings/index.html)

