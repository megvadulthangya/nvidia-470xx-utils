# Harvest Index — 2026-11-27 01:30:29 

## TL;DR

- **State:** ❗1 failed · 🔍0 suspicious · ⚠3 warnings · ✅6 clean · 🚫0 not-built
- **Action needed (1):**
    - 4 builds · 4 hits · `warning: ignoring return value of 'refcount_sub_and_test' declared with attribute 'warn_unused_result' [-Wunused-result]`

## Overview

- **Diag root:** `/build/sources`
- **Harvest:**   `/build/harvest`
- **Branches:**  470xx
- **Builds:** 10 — ❗1 failed, 🔍0 suspicious, ⚠3 warn, ✅6 clean, 🚫0 not-built
- **Warning signatures:** 1 total — 1 actionable across multiple builds, 0 informational, 0 unique to one build
- **Suspicious signatures:** 0  ·  **integrity issues:** 0 builds  ·  **integrity notes:** 0 builds

## Legend

| Symbol | Status | Meaning |
|:------:|--------|---------|
| ❗ | failed     | make.log contains real errors — **blocker** |
| 🔍 | suspicious | no errors, but suspicious lines or integrity issues |
| ⚠  | warnings   | built, no errors, non-noise warnings to review |
| ✅ | clean      | built, no errors, only known noise |
| 🚫 | not-built  | no make.log at all |
| 🧪 | utils      | userspace utils failure, mpatch debug archived |

Per-build detail lives in the harvest tree, one directory per kernel: `00-header.txt`, `01-errors.txt`, `02-warnings.txt`, `03-noise.txt`, `04-suspicious.txt`, `summary.txt`.

The raw `make.log` is also archived in every build directory as `make.log.zst` (or `.gz` / plain).

## Status table

| # | Branch | Variant | Kernel | Driver | Status | Err | Clean (uniq/total) | Susp | Noise | Integ |
|---|--------|---------|--------|--------|:------:|----:|-------------------:|-----:|------:|:-----:|
| 1 | 470xx | normal | 6.1.192-2-MANJARO | 470.256.02 | ✅ clean | 0 | 0 / 0 | 0 | 157265 |  |
| 2 | 470xx | normal | 6.12.117-1-MANJARO | 470.256.02 | ✅ clean | 0 | 0 / 0 | 0 | 63696 |  |
| 3 | 470xx | normal | 6.18.63-3-MANJARO | 470.256.02 | ⚠ warnings | 0 | 1 / 1 | 0 | 63696 |  |
| 4 | 470xx | normal | 6.6.162-2-MANJARO | 470.256.02 | ✅ clean | 0 | 0 / 0 | 0 | 63696 |  |
| 5 | 470xx | normal | 7.3.4-3-MANJARO | 470.256.02 | ⚠ warnings | 0 | 1 / 1 | 0 | 63696 |  |
| 6 | 470xx | normal | 7.3.4-3-MANJARO | 470.256.02 | ⚠ warnings | 0 | 1 / 1 | 0 | 63696 |  |
| 7 | 470xx | normal | linux-next-nvidia-470xx | — | ❗ failed | 71 | 1 / 1 | 0 | 0 |  |
| 8 | 470xx | rt | 6.1.190-2-rt70-MANJARO | 470.256.02 | ✅ clean | 0 | 0 / 0 | 0 | 157265 |  |
| 9 | 470xx | rt | 6.12.116-1-rt22-MANJARO | 470.256.02 | ✅ clean | 0 | 0 / 0 | 0 | 63696 |  |
| 10 | 470xx | rt | 6.6.159-2-rt82-MANJARO | 470.256.02 | ✅ clean | 0 | 0 / 0 | 0 | 63696 |  |

## ❗ Failures — fix these first

Full error list in each build's `01-errors.txt`. Excerpt below.

### 470xx / normal / linux-next-nvidia-470xx — ?  (71 errors)

Path: `470xx/normal/linux-next-nvidia-470xx`

```
668: nvidia/nv-pat.c:37:43: error: implicit declaration of function ‘rdmsr’; did you mean ‘rdmsrq’? [-Wimplicit-function-declaration]
674: make[4]: *** [/usr/lib/modules/7.4.0-rc4-next-20261124-1-MANJARO/build/scripts/Makefile.build:321: nvidia/nv-pat.o] Error 1
678: ././common/inc/nv-procfs-utils.h:37:29: error: ‘nv_proc_ops_t’ {aka ‘const struct proc_ops’} has no member named ‘proc_read’; did you mean ‘proc_mmap’?
690: ././common/inc/nv-procfs-utils.h:144:32: error: positional initialization of field in ‘struct’ declared with ‘designated_init’ attribute [-Werror=designated-init]
706: ././common/inc/nv-procfs-utils.h:144:32: error: initialization of ‘loff_t (*)(struct file *, loff_t,  int)’ {aka ‘long long int (*)(struct file *, long long int,  int)’} from incompatible pointer type ‘ssize_t (*)(struct file *, char *, size_t,  loff_t *)’ {aka ‘long int (*)(struct file *, char *, long unsigned int,  long long int *)’} [-Wincompatible-pointer-types]
727: ././common/inc/nv-procfs-utils.h:37:29: error: ‘nv_proc_ops_t’ {aka ‘const struct proc_ops’} has no member named ‘proc_read’; did you mean ‘proc_mmap’?
739: ././common/inc/nv-procfs-utils.h:144:32: error: positional initialization of field in ‘struct’ declared with ‘designated_init’ attribute [-Werror=designated-init]
755: ././common/inc/nv-procfs-utils.h:144:32: error: initialization of ‘loff_t (*)(struct file *, loff_t,  int)’ {aka ‘long long int (*)(struct file *, long long int,  int)’} from incompatible pointer type ‘ssize_t (*)(struct file *, char *, size_t,  loff_t *)’ {aka ‘long int (*)(struct file *, char *, long unsigned int,  long long int *)’} [-Wincompatible-pointer-types]
772: ././common/inc/nv-procfs-utils.h:37:29: error: ‘nv_proc_ops_t’ {aka ‘const struct proc_ops’} has no member named ‘proc_read’; did you mean ‘proc_mmap’?
784: ././common/inc/nv-procfs-utils.h:144:32: error: positional initialization of field in ‘struct’ declared with ‘designated_init’ attribute [-Werror=designated-init]
800: ././common/inc/nv-procfs-utils.h:144:32: error: initialization of ‘loff_t (*)(struct file *, loff_t,  int)’ {aka ‘long long int (*)(struct file *, long long int,  int)’} from incompatible pointer type ‘ssize_t (*)(struct file *, char *, size_t,  loff_t *)’ {aka ‘long int (*)(struct file *, char *, long unsigned int,  long long int *)’} [-Wincompatible-pointer-types]
817: ././common/inc/nv-procfs-utils.h:37:29: error: ‘nv_proc_ops_t’ {aka ‘const struct proc_ops’} has no member named ‘proc_read’; did you mean ‘proc_mmap’?
829: ././common/inc/nv-procfs-utils.h:144:32: error: positional initialization of field in ‘struct’ declared with ‘designated_init’ attribute [-Werror=designated-init]
845: ././common/inc/nv-procfs-utils.h:144:32: error: initialization of ‘loff_t (*)(struct file *, loff_t,  int)’ {aka ‘long long int (*)(struct file *, long long int,  int)’} from incompatible pointer type ‘ssize_t (*)(struct file *, char *, size_t,  loff_t *)’ {aka ‘long int (*)(struct file *, char *, long unsigned int,  long long int *)’} [-Wincompatible-pointer-types]
862: ././common/inc/nv-procfs-utils.h:37:29: error: ‘nv_proc_ops_t’ {aka ‘struct proc_ops’} has no member named ‘proc_read’; did you mean ‘proc_mmap’?
```

## 🔁 Cross-build warning signatures

Warnings appearing in **more than one build**. Fixing the top signature fixes the most builds at once.

### 🎯 Actionable (driver source)

| # | Builds | Hits | Signature | Branches |
|---|-------:|-----:|-----------|----------|
| 1 | 4 | 4 | `warning: ignoring return value of 'refcount_sub_and_test' declared with attribute 'warn_unused_result' [-Wunused-result]` | 470xx/normal |

### ⚙️ Informational (build system / Makefile)

_None._

## 🧩 Unique to one build

Warnings in exactly **one** build. Usually kernel- or config-specific.

_No unique-to-one-build warnings._

---
_End of index. Per-build detail files live in the harvest tree._
