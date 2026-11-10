# Begonia Kernel Configuration

This repository builds kernel variants from JSON config files in the `repos/` folder. The GitHub workflow reads every file matching `repos/repos*.json`, combines them into a build matrix, and compiles one kernel per entry.

> The config format below reflects the current build system in this repo and is the source of truth for `repos/*.json`.

## Quick overview

Each config entry describes:

- the kernel source repository and branch
- the target device and defconfig
- root integration (`KernelSU`, `KernelSU-Next`, `SukiSU`, or none)
- optional SUSFS and KPM patches
- the compiler/toolchain used for the build
- whether to package the output with AnyKernel3

## File layout

```text
repos/
  repos.power.json
```

A config file is usually an array of objects like this:

```json
[
  {
    "kernelName": "power",
    "kernelSource": {
      "device": "begonia",
      "defconfig": "begonia_user_defconfig",
      "name": "Power-KSU-Next-SUSFS",
      "repo": "https://github.com/Saikrishna1504/kernel_xiaomi_mt6785",
      "branch": "bka"
    },
    "withRoot": "ksun",
    "withSUSFS": true,
    "whichSUSFS": "susfs-159",
    "withKPM": false,
    "toolchains": [
      {
        "repo": "https://gitlab.com/crdroidandroid/android_prebuilts_clang_host_linux-x86_clang-r547379",
        "branch": "15.0",
        "name": "clang-crDroid",
        "binPath": ["./bin"]
      }
    ],
    "ccache": true,
    "params": {
      "ARCH": "arm64",
      "CC": "clang-crDroid/bin/clang",
      "externalCommand": {
        "CROSS_COMPILE": "clang-crDroid/bin/aarch64-linux-gnu-",
        "CROSS_COMPILE_ARM32": "clang-crDroid/bin/arm-linux-gnueabi-",
        "LD": "clang-crDroid/bin/ld.lld",
        "LLVM": "1",
        "LLVM_IAS": "1",
        "AR": "clang-crDroid/bin/llvm-ar",
        "NM": "clang-crDroid/bin/llvm-nm",
        "OBJCOPY": "clang-crDroid/bin/llvm-objcopy",
        "OBJDUMP": "clang-crDroid/bin/llvm-objdump",
        "READELF": "clang-crDroid/bin/llvm-readelf",
        "OBJSIZE": "clang-crDroid/bin/llvm-size",
        "STRIP": "clang-crDroid/bin/llvm-strip",
        "LDGOLD": "clang-crDroid/bin/aarch64-linux-gnu-ld.gold",
        "LLVM_AR": "clang-crDroid/bin/llvm-ar",
        "LLVM_DIS": "clang-crDroid/bin/llvm-dis"
      }
    },
    "AnyKernel3": {
      "use": true,
      "release": false,
      "repo": "https://github.com/cvnertnc/AnyKernel3/",
      "branch": "begonia"
    }
  }
]
```

## Field reference

### `kernelName`

Type: `string`

A short name for the build target. This is used in the workflow for file naming and matrix labeling.

Example:

```json
"kernelName": "power"
```

### `kernelSource`

Type: `object`

Contains the kernel source repository metadata.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `device` | string | yes | Device codename, e.g. `begonia` |
| `defconfig` | string | yes | Defconfig used by `make`, e.g. `begonia_user_defconfig` |
| `name` | string | yes | Friendly kernel name shown in build output and release naming |
| `repo` | string | yes | Git URL of the kernel source repository |
| `branch` | string | yes | Source branch to clone |

Example:

```json
"kernelSource": {
  "device": "begonia",
  "defconfig": "begonia_user_defconfig",
  "name": "Power-KSU-Next-SUSFS",
  "repo": "https://github.com/Saikrishna1504/kernel_xiaomi_mt6785",
  "branch": "bka"
}
```

### `withRoot`

Type: `string`

Which root solution to integrate into the kernel.

Supported values:

- `none`
- `ksu`
- `sukisu`
- `ksun`

Example:

```json
"withRoot": "ksun"
```

### `withSUSFS`

Type: `boolean`

Enable SUSFS patching when a supported root solution is selected.

Example:

```json
"withSUSFS": true
```

### `whichSUSFS`

Type: `boolean|string`

Used to select the SUSFS variant. In current configs, this is often a string such as `"susfs-159"`.

When not used, set it to `false`.

Example:

```json
"whichSUSFS": "susfs-159"
```

### `withKPM`

Type: `boolean`

Enable KPM injection for supported kernels / root setups.

Example:

```json
"withKPM": false
```

### `toolchains`

Type: `array`

Each item defines a compiler or toolchain repo to clone and add to `PATH`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `repo` | string | conditional | Git repo URL for a toolchain |
| `url` | string | conditional | Direct download URL for a tar/zip archive |
| `branch` | string | conditional | Branch for Git-based toolchain |
| `name` | string | yes | Local folder name |
| `binPath` | array | yes | Relative path(s) to add to `PATH` |

Examples:

```json
"toolchains": [
  {
    "repo": "https://gitlab.com/crdroidandroid/android_prebuilts_clang_host_linux-x86_clang-r547379",
    "branch": "15.0",
    "name": "clang-crDroid",
    "binPath": ["./bin"]
  }
]
```

Archive download example:

```json
"toolchains": [
  {
    "url": "https://android.googlesource.com/platform/prebuilts/clang/host/linux-x86/+archive/refs/heads/master-kernel-build-2022/clang-r450784d.tar.gz",
    "name": "clang",
    "binPath": ["bin"]
  }
]
```

### `ccache`

Type: `boolean`

Enable ccache for faster incremental kernel builds.

Example:

```json
"ccache": true
```

### `params`

Type: `object`

Build arguments passed into the kernel make command.

Common keys:

- `ARCH`: target architecture, usually `arm64`
- `CC`: compiler path, such as `clang-crDroid/bin/clang`
- `externalCommand`: extra flags passed to `make`

Example:

```json
"params": {
  "ARCH": "arm64",
  "CC": "clang-crDroid/bin/clang",
  "externalCommand": {
    "CROSS_COMPILE": "clang-crDroid/bin/aarch64-linux-gnu-",
    "CROSS_COMPILE_ARM32": "clang-crDroid/bin/arm-linux-gnueabi-",
    "LD": "clang-crDroid/bin/ld.lld",
    "LLVM": "1",
    "LLVM_IAS": "1"
  }
}
```

### `AnyKernel3`

Type: `object`

Controls packaging and optional release generation for the flashable zip.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `use` | boolean | yes | Whether to package with AnyKernel3 |
| `release` | boolean | yes | Whether to upload it as a GitHub release |
| `repo` | string | yes | AnyKernel3 repository URL |
| `branch` | string | yes | Branch to clone |

Example:

```json
"AnyKernel3": {
  "use": true,
  "release": false,
  "repo": "https://github.com/cvnertnc/AnyKernel3/",
  "branch": "begonia"
}
```

## Minimal example

```json
[
  {
    "kernelName": "power",
    "kernelSource": {
      "device": "begonia",
      "defconfig": "begonia_user_defconfig",
      "name": "Power-Non-Patches",
      "repo": "https://github.com/Saikrishna1504/kernel_xiaomi_mt6785",
      "branch": "bka"
    },
    "withRoot": "none",
    "withSUSFS": false,
    "whichSUSFS": false,
    "withKPM": false,
    "toolchains": [
      {
        "repo": "https://gitlab.com/crdroidandroid/android_prebuilts_clang_host_linux-x86_clang-r547379",
        "branch": "15.0",
        "name": "clang-crDroid",
        "binPath": ["./bin"]
      }
    ],
    "ccache": true,
    "params": {
      "ARCH": "arm64",
      "CC": "clang-crDroid/bin/clang",
      "externalCommand": {
        "CROSS_COMPILE": "clang-crDroid/bin/aarch64-linux-gnu-",
        "CROSS_COMPILE_ARM32": "clang-crDroid/bin/arm-linux-gnueabi-",
        "LD": "clang-crDroid/bin/ld.lld"
      }
    },
    "AnyKernel3": {
      "use": true,
      "release": false,
      "repo": "https://github.com/cvnertnc/AnyKernel3/",
      "branch": "begonia"
    }
  }
]
```

## How the workflow uses the config

The build workflow in `.github/workflows/build.yml` does this:

1. reads every `repos/repos*.json`
2. merges them into a JSON matrix
3. loops through each item
4. clones the kernel source repository and toolchains
5. applies root-related patches, SUSFS, and KPM when enabled
6. runs `make ${defconfig}`
7. builds the kernel
8. repackages it with AnyKernel3 if enabled

## Notes

- Use valid JSON syntax. Do not leave trailing commas.
- Keep the `name` values unique if you plan to build multiple variants.
- When you enable root integration, make sure the repo source and patches are compatible with the kernel branch you target.
- `AnyKernel3.release` only matters when `AnyKernel3.use` is `true`.

## Recommended pattern for new builds

For a new kernel variant, copy an existing entry in `repos/repos.power.json` and update only the values that differ:

- `kernelName`
- `kernelSource.name`
- `kernelSource.repo`
- `kernelSource.branch`
- `kernelSource.device`
- `kernelSource.defconfig`
- `withRoot`
- `withSUSFS` / `whichSUSFS`
- `toolchains`
- `params`
- `AnyKernel3`

This lets you quickly add new builds without changing the workflow logic.
