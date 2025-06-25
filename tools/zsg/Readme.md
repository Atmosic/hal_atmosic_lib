<!--
SPDX-FileCopyrightText: 2023 Albert Krenz <albert.krenz@mailbox.org>
SPDX-License-Identifier: CC0-1.0
-->

# What is zsg?
zsg is the short term for "Zephyr Settings Generator". It is a simple command line utility to generate flash images (in binary format) for the zephyr settings Subsystem with NVS backend.

It can be used e.g. during the manufacturing process to create a settings binary on the fly with values for

- serial number
- MAC address
- manufacturing date
- credentials
- etc.

An example could look like

    zsg -o settings.bin data/serialnumber:string:ABC12345 data/MAC:string:AA-BB-CC-DD-EE-FF data/manufacturing-data:string:2023-09-01 data/id:u32:123456

The generated binary must be flashed at the location of the `storage` partition of your Board (e.g. by using JFlash).

For more information on the Zephyr settings subsystem see: https://docs.zephyrproject.org/latest/services/settings/index.html

# Build and installing
## How to build
To build zsg you need
- `libboost programm options` with version >= 1.67.
- `yaml-cpp` with version >= 0.8.0 (For Atmosic special application)
- `CMake` with version >= 3.6
- Compiler with C++17 support
- Ninja or Make as buildtool

If you want to build via Ninja use the following commands:

    $ mkdir build
    $ cd build
    $ cmake -GNinja -DCMAKE_RELEASE_TYPE=Release ..
    $ ninja

## How to install
You can install zsg either by one of the provided Install packages from the [Release page](https://gitlab.com/oha4/zephyr-settings-generator/-/releases), or directly via

    § ninja install

after building.

# License
This Software is licensed under [BSD-2-Clause Plus Patent License](https://spdx.org/licenses/BSD-2-Clause-Patent)

=============== For Atmosic special application ===============

Previously, configuring through Boost was difficult to maintain with large amounts of data, so the data source has been standardized to the YAML file format.
The maximum length of the Tag Value in the YAML file currently supports up to 1024 bytes.
Set 'ATMOSIC_ZSG' in CMakeLists.txt to ON to enable this feature.

# Execution example
1. To generate binary string data to the settings storage area(Data read from YAML file):
```
Usage: zsg.exe write [yaml_file] [partition_sector_size] [partition_size] [bin_output_file]
```

- Windows example:
$ cd build
§ src\\zsg.exe write ..\\factory.yml 1024 2048 ..\\factory.bin

- Linux example:
$ cd build
§ src/zsg write ../factory.yml 1024 2048 ../factory.bin

2. Read NVS information from a bin file and save it to a YAML file(Data read from NVS file).
```
Usage: zsg.exe read [bin_file] [FACTORY | SETTINGS] [partition_sector_size] [partition_size] [yaml_output_file]
```
- Windows example:
$ cd build
§ src\\zsg.exe read ..\\factory.bin FACTORY 1024 2048 ..\\factory.yml

- Linux example:
$ cd build
§ src/zsg read ../factory.bin FACTORY 1024 2048 ../factory.yml

3. Show NVS information from a bin file(Data read from NVS file).
```
Usage: zsg.exe show [bin_file] [FACTORY | SETTINGS] [partition_sector_size] [partition_size] [show_pretty | show_verbose]
```
- Windows example:
$ cd build
§ src\\zsg.exe show ..\\factory.bin FACTORY 1024 2048 show_pretty
§ src\\zsg.exe show ..\\factory.bin FACTORY 1024 2048 show_verbose

- Linux example:
$ cd build
§ src/zsg show ../factory.bin FACTORY 1024 2048 show_pretty
§ src/zsg show ../factory.bin FACTORY 1024 2048 show_verbose

# Yaml file format
Please refer to the 'sample.yml' in the directory

# Attribute Yaml file
When converting a YAML file to a binary file, the Tag Value types will be recorded in <bin_file_name>_attr.yml. Later, if the binary file needs to be converted back to a YAML file, this file will be referenced for the conversion (if the file does not exist, the values will be treated as hexadecimal by default).