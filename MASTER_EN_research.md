# Universal Linux Installer for Apple Silicon — Research Master Document

> Single source of truth · v0.5 · English research edition
> Faithful translation and restructuring of the project's original Spanish MASTER.md (65,385 bytes). Sections 1–29 carry no findings added, removed or altered; §§30–32 are clearly-labelled editorial appendices compiled from the project history. Wording has been adapted to a research-report register. Where the source records a negative or "not tested" result, it is preserved as such.
> Convention: this document is to be updated at the start and end of every working session.

---

## 1. Current Project Status

| Field | Value |
|---|---|
| Phase | 2 — AMX driver integrated and validated in a VM |
| Test hardware | MacBook Air M2 (Mac14,2 · T8112) · macOS 15.6 · Asahi **not yet installed** |
| Development environment | Debian Linux |
| Last recorded activity | 21 Jul 2026 — QEMU validation complete · kernel-panic bug fixed |
| Next step | Install Asahi on the M2 Mac (the only step that requires real hardware) |

### 1.1 What Has Been Tested and What Has Not

| Aspect | Where tested | Status |
|---|---|---|
| AMX instructions execute | Real M2, userspace under macOS | TESTED |
| 5120-byte state, save/restore | Real M2, userspace under macOS | TESTED |
| Context switch 0.31 µs | Real M2, userspace under macOS | TESTED |
| Driver does not break boot without AMX | QEMU cortex-a72 | TESTED |
| MIDR detection (0x61 = Apple) | QEMU cortex-a72 | TESTED |
| Symbols linked at runtime | QEMU, /proc/kallsyms | TESTED |
| Driver in kernel space on the M2 | — | NOT TESTED |
| amx_thread_init from EL1 | — | NOT TESTED |
| TPIDR2_EL0 from the Linux kernel | — | NOT TESTED |

**Validity caveat.** The 5/5 test results on the M2 were obtained in *userspace under macOS*, not in kernel space under Linux. The complete driver path (kzalloc + TPIDR2_EL0 access from EL1) has never been executed. Doing so requires Asahi to be installed on the machine.

---

## 2. Working Protocol

### 2.1 Roles
| Party | Responsibility |
|---|---|
| Human researcher | Final decisions · hardware validation · real-hardware tests · direction |
| AI assistant | Analysis · code · documentation · research · gap detection |

### 2.2 Decision-Making
- Every important step is decided jointly.
- The AI proposes, with technical justification.
- The human approves, corrects or redirects.
- Nothing critical is implemented without consensus.

### 2.3 Continuity Between Sessions
1. The AI reads this MASTER in full at the start.
2. Status and pending items are reviewed.
3. Work continues from where it stopped.
4. At the end, this document is updated.

### 2.4 Team Rules
- Everything discovered is documented here immediately.
- Every technical decision is recorded together with its rationale.
- Errors are documented — they are as valuable as successes.
- The AI proposes; the developer approves.

---

## 3. Project Vision

A Rust CLI installer plus a universal compatibility layer that allows any ARM64 Linux distribution to be installed natively on Macs with Apple Silicon chips, with 100 % native performance, no virtualization, supporting M1/M2/M3/M4/M5.

**The software acts as a universal translator.** Regardless of the architecture or how the Mac speaks internally, any ARM64 Linux ISO runs natively — comparable to what UEFI does on conventional PCs, but for Apple Silicon.

**Philosophy (preserved verbatim in spirit):** do not hack or breach Apple — understand the system, talk to it, and adapt to it. Passive introspection, not brute force.

---

## 4. Decisions Taken

| # | Decision | Rationale |
|---|---|---|
| 1 | Language: Rust | Native speed + memory safety + full control |
| 2 | Interface: CLI first | Fast iteration, complete debugging; GUI later |
| 3 | Target: official ARM64 ISOs | Real native performance — x86 on ARM never reaches 100 % |
| 4 | v1.0 distributions: Ubuntu, Debian, Fedora, Kali | The only ones with a complete official ARM64 ISO today |
| 5 | Initial hardware: MacBook Air M2 (Mac14,2) | Hardware available — Asahi has an M2 base |
| 6 | Future GUI: Swift + SwiftUI | Native to macOS, same Rust backend via FFI |
| 7 | Decisions: joint | Better architecture, fewer errors |
| 8 | Continuity: single master document | Avoid losing context between sessions |
| 9 | AMX reverse engineering: first component | Virgin territory — greatest potential impact |
| 10 | AMX observation: via Accelerate + DTrace | Do not breach; only listen |
| 11 | AMX driver: in the base kernel | AMX is integrated in kernel.release.t8112, not a kext |
| 12 | AMX context switch: lazy save | Apple works in massive blocks — save only when necessary |

---

## 5. System Architecture

```
┌─────────────────────────────────────────────────────────┐
│              Rust CLI (installer)                       │
│   menu · distro selection · progress · error handling   │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│              Rust Core Modules                          │
│  detect_hardware()      prepare_partitions()            │
│  download_iso()         install_bootchain()             │
│  select_devicetree()    verify_install()                │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│         Universal compatibility layer                   │
│    m1n1 (stage 1) → U-Boot (stage 2) → GRUB            │
│         Device Trees per Mac model                      │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│              Apple Silicon hardware                     │
│    CPU · GPU · AMX · Secure Enclave · UMA               │
└─────────────────────────────────────────────────────────┘
```

---

## 6. Technical Stack

| Component | Technology |
|---|---|
| Installer | Rust CLI |
| Bootloader stage 1 | m1n1 (Asahi, open source) |
| Bootloader stage 2 | U-Boot |
| Device Trees | Upstream Asahi DTBs |
| Future GUI | Swift + SwiftUI |
| Static analysis | Ghidra |
| Dynamic tracing | m1n1 hypervisor + DTrace |
| Development emulation | QEMU ARM64 |
| Cross-compiler | aarch64-linux-gnu-gcc |
| AMX reverse engineering | corsix/amx + dyld_info + Python memory scan |

---

## 7. Confirmed Hardware — MacBook Air M2

| Field | Value | Source |
|---|---|---|
| Model | MacBook Air (Mac14,2) | mprobe.sh + system_profiler |
| Chip | Apple M2 (T8112) | mprobe.sh + XNU kernel |
| macOS | 15.6 (Darwin 24.6.0) | sw_vers |
| RAM | 16 GB unified memory | sysctl hw.memsize |
| SSD | APPLE SSD AP0256Z 256 GB · 138 GB free | system_profiler SPNVMeDataType |
| GPU | Apple AGX G14 · 8 cores | system_profiler SPDisplaysDataType |
| CPU | 4 P-cores + 4 E-cores = 8 total | sysctl hw.perflevel* |
| Page size | **16 KB** ← critical for the Linux kernel | sysctl hw.pagesize |
| Cache line | 128 bytes | sysctl hw.cachelinesize |
| L1i | 128 KB · L2: 4 MB | sysctl hw.l1icachesize |
| cpu_type | 16777228 = ARM64 Apple Silicon | sysctl hw.cputype |
| cpu_subtype | 2 = ARM64e with PAC | sysctl hw.cpusubtype |
| APFS containers | 4 | diskutil list |
| SIP | enabled | csrutil status |
| ARM64 features | AES · SHA512 · BF16 · FP16 · DotProduct · CRC32 · Atomics v8.1 | sysctl hw.optional.* |

---

## 8. Linux Development Environment (Debian)

| Field | Value |
|---|---|
| Project path | `/home/user/Documents/Projects/Linux` |
| Python virtualenv | `/home/user/Documents/Projects/Linux/mini-venv` |
| Ghidra | `/home/user/tools/ghidra/ghidraRun` |
| QEMU script | `/home/user/Documents/Projects/Linux/scripts/qemu-arm64.sh` |
| Rust | installed via rustup · target aarch64-unknown-linux-gnu |
| Cross-compiler | aarch64-linux-gnu-gcc |
| OpenSSL | vendored mode in Cargo.toml |

---

## 9. Rust Repository Structure

```
/home/user/Documents/Projects/Linux/
├── Cargo.toml
├── src/
│   ├── main.rs                  ← IMPLEMENTED — complete flow
│   ├── hardware/
│   │   ├── mod.rs               ← IMPLEMENTED
│   │   ├── detect.rs            ← IMPLEMENTED (real macOS + simulated M2 on Linux)
│   │   └── models.rs            ← IMPLEMENTED (Chip, MacModel, DTB M1-M4, Mac14,2)
│   ├── cli/
│   │   ├── mod.rs               ← IMPLEMENTED
│   │   ├── menu.rs              ← IMPLEMENTED (interactive selection + compatibility)
│   │   └── progress.rs          ← placeholder
│   ├── distro/
│   │   ├── mod.rs               ← IMPLEMENTED
│   │   ├── catalog.rs           ← IMPLEMENTED (4 distros, URLs, checksums)
│   │   ├── download.rs          ← IMPLEMENTED (streaming + SHA256 + cache)
│   │   └── install.rs           ← empty
│   ├── partitions/
│   │   ├── mod.rs               ← empty ← requires complete RE first
│   │   ├── apfs.rs              ← empty ← MOST CRITICAL MODULE
│   │   └── layout.rs            ← empty
│   ├── bootchain/
│   │   ├── mod.rs               ← empty
│   │   ├── m1n1.rs              ← empty
│   │   ├── uboot.rs             ← empty
│   │   └── devicetree.rs        ← empty
│   └── utils/
│       ├── mod.rs               ← IMPLEMENTED
│       ├── checksum.rs          ← IMPLEMENTED (SHA256, SHA256SUMS parsing)
│       └── network.rs           ← IMPLEMENTED (HTTP client)
├── assets/devicetrees/          ← empty (DTBs go here)
├── scripts/
│   └── qemu-arm64.sh            ← IMPLEMENTED
└── mini-venv/                   ← Python virtualenv for m1n1
```

---

## 10. Test Status

| Module | Tests | Status |
|---|---|---|
| hardware | 7/7 | pass |
| distro/catalog | 9/9 | pass |
| distro/download | 4/4 | pass |
| utils/checksum | 6/6 | pass |
| cli | 2/2 | pass |
| **Total** | **28/28** | pass |

---

## 11. Partition Map — MacBook Air M2

```
disk0 (251 GB — physical NVMe APPLE SSD AP0256Z)
├── disk0s1 → Apple_APFS_ISC (524 MB)  ← iBoot System Container — NEVER TOUCH
├── disk0s2 → Apple_APFS main (245.1 GB) ← main macOS container
│   ├── disk3s1  Macintosh HD       14.5 GB  (sealed macOS system — read-only)
│   ├── disk3s1s1 APFS snapshot     14.5 GB  (signed system snapshot)
│   ├── disk3s2  Preboot            13.3 GB  ← the Linux stub goes here
│   ├── disk3s3  Recovery            2.2 GB  ← do not modify
│   ├── disk3s5  Data               59.8 GB  (user home — do not touch)
│   └── disk3s6  VM/Swap             5.4 GB  (macOS virtual memory)
└── disk0s3 → Apple_APFS Recovery (5.4 GB) ← Apple Recovery

Free space for Linux: ~138 GB inside disk0s2
Strategy:
  1. Shrink disk0s2, freeing ~40-80 GB
  2. Create a new APFS container in the freed space (Linux stub)
  3. Install m1n1 + U-Boot + Linux partition in the stub
  4. Register in Preboot (disk3s2) so it appears in the boot picker
```

---

## 12. Roadmap

### Phase 0 — Completed
- [x] Debian development environment set up
- [x] setup-dev-env.sh functional
- [x] Rust project compiling on x86_64 and ARM64
- [x] hardware/detect.rs — simulated M2 on Linux
- [x] hardware/models.rs — DTB table M1-M4 + Mac14,2
- [x] distro/catalog.rs — 4 ARM64 distributions
- [x] distro/download.rs — streaming + SHA256
- [x] cli/menu.rs — interactive selection
- [x] utils/checksum.rs + network.rs
- [x] 28/28 tests passing
- [x] Complete CLI flow working

### Phase 1 — AMX reverse engineering completed
- [x] mprobe.sh — Mac M2 hardware probe
- [x] amx_probe.c — enable/disable AMX: 5 ns confirmed
- [x] amx_observe.sh — 1.3 TFLOPS via Accelerate confirmed
- [x] amx_disasm.sh — dyld shared cache discovered
- [x] extract_blas.sh — libBLAS in memory: 0x18a57d000
- [x] amx_opcodes.sh — cblas_sgemm is a wrapper of _APL_sgemm
- [x] dump_apl_sgemm.py — 139,331 AMX instructions mapped
- [x] ARM64e PAC finding — root cause of all crashes
- [x] Complete GEMM pattern documented

### Phase 2 — In progress
- [ ] ARM64 context around AMX_ENABLE (0x18a9b00c4) ← NEXT (as recorded at this point of the source)
- [x] Implement the Linux AMX driver: amx.h + amx.c + amx_test.c COMPLETE
- [ ] partitions/apfs.rs — safe partitioning
- [ ] bootchain/m1n1.rs + uboot.rs + devicetree.rs

### Phase 3 — Pending
- [ ] distro/install.rs
- [ ] OpenBLAS integration with AMX
- [ ] Swift + SwiftUI GUI
- [ ] M3/M4/M5 support

---

## 13. Identified Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Apple breaks boot (macOS 27 already did) | High | Boot resilience independent of the boot picker |
| Badly written APFS partition → brick | Critical | Dry-run + exhaustive validation + backup |
| Wrong DTB → dead hardware | High | Exhaustive table + verification before install |
| M4/M5 changes AMX architecture | Medium | Per-chip-generation modules |
| SEP firmware update | High | Always treat the SEP as a black box |

---

## 14. Executed Scripts — Complete Registry

> Everything that has been run on the Mac and on Linux, with a description of what each script does.

### 14.1 Scripts Executed on the MacBook Air M2

#### 14.1.1 `mprobe.sh` — General hardware probe
**Path:** `~/Documents/InvestigacionAlex/mprobe.sh`
**Command:**
```bash
chmod +x mprobe.sh
./mprobe.sh --pretty --output=m2_probe_v2.json
```
**What it does:** Reads the Mac's hardware information using native macOS tools (`system_profiler`, `ioreg`, `sysctl`, `vm_stat`, `diskutil`, `pmset`, `csrutil`). It installs nothing, modifies nothing and makes no network connections. It produces a JSON file with: model, chip, OS, CPU (cores, caches, page size), memory, NVMe storage, IORegistry (GPU, SEP, DART), boot (SIP), battery, ARM64 features, board-id and partitions.
**Key output:**
- Model: MacBook Air (Mac14,2)
- Chip: Apple M2 (T8112)
- Page size: **16 KB** ← critical
- GPU: Apple M2 · 8 cores
- board_id: Mac14,2 (via hw.model)
- 4 APFS containers · 138 GB free

#### 14.1.2 `amx_probe.c` (v1, v2, v3, v4) — Direct AMX coprocessor probe
**Path:** `~/Documents/InvestigacionAlex/amx_probe.c`
**Command:**
```bash
clang -O2 -o amx_probe amx_probe.c
./amx_probe
```
**What it does:** A C program that attempts to enable AMX directly using undocumented ARM64 instructions. It measures enable/disable latency and checks whether the data instructions (LDX, STX, STZ) work from userspace.
**Versions and results:**
- v1: enable/disable OK (5 ns) · LDX crashes with SIGILL
- v2: same result · opcodes corrected
- v3: same result · corsix/amx encoding
- v4: AMX_ENABLE OK · latency loop crashes
**Conclusion:** Enable/disable works (~5 ns). The data instructions fail because macOS 15 ARM64e requires a PAC tag in the operands.

#### 14.1.3 `amx_check.sh` — AMX environment diagnostic
**Path:** `~/Documents/InvestigacionAlex/amx_check.sh`
**Command:**
```bash
chmod +x amx_check.sh
./amx_check.sh
```
**What it does:** Verifies available tools, binary architecture, macOS version, SIP status, AMX-related sysctl flags, binary entitlements and kernel version. Confirms ARM64e with SIP enabled and a T8112 kernel.
**Key output:** Kernel `xnu-11417.140.69~1/RELEASE_ARM64_T8112` · native arm64 binary · SIP enabled · no AMX keys in sysctl

#### 14.1.4 `amx_observe.sh` — Observing AMX through the Accelerate framework
**Path:** `~/Documents/InvestigacionAlex/amx_observe.sh`
**Command:**
```bash
pip3 install numpy
chmod +x amx_observe.sh
./amx_observe.sh 2>&1 | tee amx_observe_output.txt
```
**What it does:** Passively observes AMX through Apple's Accelerate framework. It looks for AMX instructions in vecLib, maps libraries in memory, benchmarks matrix multiplication with numpy, traces calls with `sample`, checks for AMX kexts, and examines the T8112 kernel.
**Key output:**
- 273–1,309 GFLOPS confirmed (AMX active via Accelerate)
- libBLAS.dylib @ 0x18a57d000 (in the dyld shared cache)
- cblas_sgemm @ 0x18abceda0
- Accelerate stack: libBLAS · libBNNS · libLAPACK · libvDSP · libvMisc · libSparse
- kernel.release.t8112 is ARM64e — AMX lives in the base kernel, not a separate kext

#### 14.1.5 `amx_disasm.sh` — Attempt to disassemble Accelerate
**Path:** `~/Documents/InvestigacionAlex/amx_disasm.sh`
**Command:**
```bash
chmod +x amx_disasm.sh
./amx_disasm.sh 2>&1 | tee amx_disasm_output.txt
```
**What it does:** Attempts to disassemble vecLib and libBLAS with `otool` and `dyld_info`. Searches for AMX opcodes on disk and in memory.
**Key output:** libBLAS.dylib is a broken symlink — the real file is in the dyld shared cache. `dyld_info` has a `-disassemble` flag. Confirmed instructions in libBLAS that build AMX operands with PAC tag `0x8000` in the high bits.

#### 14.1.6 `find_blas.sh` — Locating libBLAS in the dyld shared cache
**Path:** `~/Documents/InvestigacionAlex/find_blas.sh`
**Command:**
```bash
chmod +x find_blas.sh
./find_blas.sh 2>&1 | tee find_blas_output.txt
```
**What it does:** Locates libBLAS in the filesystem and in the dyld shared cache. Uses Python to load Accelerate via `ctypes` and obtain the real in-memory address of `cblas_sgemm`. Lists the 316 loaded dyld images and filters those related to Accelerate.
**Key output:**
- cblas_sgemm @ 0x18abceda0
- libBLAS.dylib @ dyld image 186, base 0x18a57d000
- Homebrew not installed · dyld-cache-extractor unavailable
- `dyld_info` available with flags: `-disassemble`, `-exports`, `-function_starts`

#### 14.1.7 `amx_opcodes.sh` — Searching for AMX opcodes in libBLAS memory
**Path:** `~/Documents/InvestigacionAlex/amx_opcodes.sh`
**Command:**
```bash
chmod +x amx_opcodes.sh
./amx_opcodes.sh 2>&1 | tee amx_opcodes_output.txt
```
**What it does:** Uses `dyld_info -exports` and `dyld_info -disassemble` to analyse libBLAS. Uses Python to read process memory directly and search for AMX byte patterns. Reads the first 512 bytes of cblas_sgemm and 1 MB of libBLAS looking for `20 12 20 00` (AMX_ENABLE).
**Key output:**
- cblas_sgemm: 1 BL instruction → calls _APL_sgemm (thin wrapper)
- AMX_ENABLE not found in cblas_sgemm nor in the first 1 MB of libBLAS
- libBLAS export table: _APL_sgemm @ offset 0x000F9AB8

#### 14.1.8 `dump_apl_sgemm.py` — Complete AMX instruction map of libBLAS
**Path:** `~/Documents/InvestigacionAlex/dump_apl_sgemm.py`
**Command:**
```bash
python3 dump_apl_sgemm.py 2>&1 | tee apl_sgemm_output.txt
```
**What it does:** Reads the functions `_APL_sgemm`, `_APL_dgemm`, `_APL_sgemm_LU` and `_SGEMM` from memory. It then scans the first 7 MB of libBLAS in full for AMX instructions (patterns `0x002010xx` and `0x002012xx`). It generates the report `libblas_amx_report.txt` with every instruction found, its address and type.
**Key output:**
- 139,331 AMX instructions in libBLAS
- AMX_ENABLE @ 0x18a9b00c4 · AMX_DISABLE @ 0x18a9b0124
- ENABLE→DISABLE distance: 96 bytes (24 ARM64 instructions)
- Most-used instruction: AMX_LDY (19,384) and AMX_LDX (19,207)
- AMX_OP_25 (0x00201324) — undocumented instruction
- 7,832 AMX blocks · largest block: 748 instructions

### 14.2 Scripts Executed on Debian Linux

#### 14.2.1 `amx_context.py` — ARM64 context around AMX_ENABLE/DISABLE
**Path:** `~/Documents/InvestigacionAlex/amx_context.py`
**Command:**
```bash
python3 amx_context.py 2>&1 | tee amx_context_output.txt
```
**What it does:** Reads process memory at runtime around the exact addresses of AMX_ENABLE (0x18a9b00c4) and AMX_DISABLE (0x18a9b0124) in libBLAS. It decodes every ARM64 instruction in the context in order to reconstruct the complete code of Apple's `amx_enable()` and `amx_disable()` functions. It saves the binaries `AMX_ENABLE_context.bin` and `AMX_DISABLE_context.bin`.
**Key output:**
- amx_enable() @ 0x18a9b0084 — PACIBSP + setup + flag check + BL init_amx_hw + NOP ×3 + AMX_ENABLE + counter increment + RET
- amx_disable() @ 0x18a9b00e4 — PACIBSP + setup + check + counter decrement + if refs>0 skip + NOP ×3 + AMX_DISABLE + RET
- init_amx_hw() @ 0x18a99b16c — called via BL #-1133256 before the enable
- gemm_kernel() @ 0x18a9b0134 — massive GEMM function with 139,329 AMX instructions
- Reference counting confirmed in real Apple production code
- The 3 NOPs before AMX_ENABLE and AMX_DISABLE confirmed in production

#### 14.2.2 `setup-dev-env.sh` — Complete development-environment setup
**Path:** `/home/user/Documents/Projects/Linux/setup-dev-env.sh`
**Command:**
```bash
chmod +x setup-dev-env.sh
./setup-dev-env.sh
```
**What it does:** Installs the entire development environment in one step: system dependencies (build-essential, git, clang, cmake), the ARM64 cross-compiler (`aarch64-linux-gnu-gcc`), QEMU ARM64, Rust via rustup with the ARM64 target, Rust tooling (cargo-watch, cargo-audit), dependencies for building the Linux kernel, Ghidra, a Python virtualenv for m1n1, and creates the complete Rust project structure under `/home/user/Documents/Projects/Linux`.
**Errors resolved during setup:**
- `hexdump` → replaced by `bsdextrautils`
- `dtc` → replaced by `device-tree-compiler`
- Python venv externally managed → use `python3 -m venv mini-venv`
- OpenSSL ARM64 cross-compile → use `openssl = { features = ["vendored"] }` in Cargo.toml
- `cargo new` in an existing directory → use `cargo init`

---

## 15. AMX Reverse Engineering — Apple Matrix eXtensions (M2 · T8112)

### 15.1 What Is AMX?
A matrix-multiplication coprocessor integrated in every Apple M-series chip. There is no official documentation — everything known comes from reverse engineering. It is the accelerator used internally by macOS's `Accelerate` framework. Linux has no support for it — everything built here is original work.

### 15.2 Hardware Architecture

**Per-process AMX state:**
```
amx0 (X registers): 8 × 64 bytes =  512 bytes  — column inputs
amx1 (Y registers): 8 × 64 bytes =  512 bytes  — row inputs
amx2 (Z registers):64 × 64 bytes = 4096 bytes  — outputs / accumulator
─────────────────────────────────────────────────────────────────
Total per process:                 5120 bytes
Total on M2 (2 instances):       10240 bytes
```

**Placement in the M2 SoC:**
- One AMX instance per cluster of 4 P-cores = 2 instances on the base M2
- Accesses the cluster's L2 and unified memory directly
- Has no separate bus — shares the interconnect fabric

**Versions:**
- M1: AMX v2 (9-bit writemasks)
- M2: AMX v2 + bf16 + minor tweaks
- M3: AMX v2 + extra mode in ldx/ldy/matint
- M4: migrates to standard ARM SME (already supported in Linux)

### 15.3 Supported Data Types
- IEEE 754: f16, f32, f64
- f16 × f32 (f16 multiplicands, f32 accumulator)
- Integers: int8×int8→int16/int32, int16×int16→int32
- **M2+:** bfloat16 → bf16 or f32

### 15.4 AMX ISA — Instructions Confirmed in libBLAS

**Base encoding:**
```
.word (0x00201000 + (op << 5) + operand_bits)
Operand is always in register x0 BEFORE the instruction
```

**Instruction table (verified across 139,331 occurrences):**
```
op=0  LDX   — load X register (64 bytes from memory)
op=1  LDY   — load Y register (64 bytes from memory)
op=2  STX   — store X register to memory
op=3  STY   — store Y register to memory
op=4  LDZ   — load Z register (64 bytes from memory)
op=5  STZ   — store Z register to memory
op=6  LDZI  — load Z interleaved
op=7  STZI  — store Z interleaved
op=8  EXTRX — extract row/column from Z → X
op=9  EXTRY — extract row/column from Z → Y
op=10 MAC16 — multiply-accumulate FP16
op=11 FMA16 — fused multiply-add FP16
op=12 FMS16 — fused multiply-subtract FP16
op=13 FMA32 — fused multiply-add FP32
op=14 FMS32 — fused multiply-subtract FP32
op=15 FMA64 — fused multiply-add FP64
op=16 FMS64 — fused multiply-subtract FP64
op=17 SET   — enable (operand=0) / disable (operand=1)
op=18 VECINT — integer vector operations
op=19 VECFP  — FP vector operations
op=20 MATINT — integer matrix multiplication
op=21 MATFP  — FP matrix multiplication
op=22 GENLUT — lookup table
op=25 ???    — undocumented instruction (0x00201324) — only once in libBLAS
```

**Operand encoding for LD/ST:**
```
bits[62]    = mode (0 = single row, 1 = multiple rows)
bits[61:56] = row number (X/Y: 0-7, Z: 0-63)
bits[55: 0] = memory address
```

**Enable/Disable:**
```c
// Always preceded by 3 NOPs (required by Apple Accelerate)
AMX_ENABLE:  nop; nop; nop; mov x0, #0; .word 0x00201220
AMX_DISABLE: nop; nop; nop; mov x0, #1; .word 0x00201221
```

### 15.5 Critical Reverse-Engineering Findings

**Finding 1 — ARM64e PAC tag in AMX operands (root cause of all crashes).**
The MacBook Air M2 uses ARM64e, the ARM64 variant with Pointer Authentication Codes. In macOS userspace, AMX operands carry a PAC tag `0x8000` in bits[63:48]:

```asm
; How Apple builds AMX operands inside libBLAS:
mov  x3, #0x12010000        ; AMX operand base
movk x3, #0x8000, lsl #48   ; ← PAC authentication tag
orr  x4, x3, x15            ; combine with memory address
```

On bare-metal Linux PAC does not apply — the driver uses standard operands.

**Finding 2 — Only 2 AMX_SET instructions in 7 MB of libBLAS.**
- AMX_ENABLE  @ `0x18a9b00c4`
- AMX_DISABLE @ `0x18a9b0124`
- Distance: 96 bytes = 24 ARM64 instructions between them
- Apple enables AMX ONCE and works in massive bursts
- Implication for the Linux driver: lazy enable + lazy save

**Finding 3 — cblas_sgemm is a thin wrapper.**
- Begins with `PACIBSP` (ARM64e signature)
- Contains exactly 1 `BL` instruction → calls `_APL_sgemm`
- The real AMX opcodes live in low-level internal functions

**Finding 4 — GEMM pattern on AMX (matrix-multiplication kernel).**
```
AMX_ENABLE (once at the start)
  └─ Loop over matrix blocks:
      ├─ LDX × N  (load columns of A into the X register)
      ├─ LDY × N  (load rows of B into the Y register)
      ├─ MAC16/FMA16 × N  (multiply and accumulate into Z)
      ├─ EXTRY/EXTRX × N  (reorganise results in Z)
      └─ STZ × N  (store result block C to memory)
AMX_DISABLE (once at the end)
```

**Finding 5 — Distribution of the 139,331 AMX instructions in libBLAS.**
```
AMX_LDY  : 19,384 (13.9%) — load row B
AMX_LDX  : 19,207 (13.8%) — load column A
AMX_FMS16: 16,392 (11.8%) — FP16 fused multiply-subtract
AMX_MAC16: 16,203 (11.6%) — FP16 multiply-accumulate
AMX_STZ  : 16,043 (11.5%) — store result
AMX_EXTRY: 14,095 (10.1%) — reorganise Z → Y
AMX_FMA16:  9,708 ( 7.0%) — FP16 fused multiply-add
AMX_EXTRX:  8,774 ( 6.3%) — reorganise Z → X
AMX_VECFP:  6,354 ( 4.6%) — FP vector operations
AMX_FMA32:  5,836 ( 4.2%) — FP32 fused multiply-add
AMX_LDZ  :  4,211 ( 3.0%) — load Z
AMX_STY  :  1,907 ( 1.4%) — store Y
AMX_STX  :    710 ( 0.5%) — store X
AMX_MATFP:    504 ( 0.4%) — FP matrix multiplication
AMX_SET  :      2 ( 0.0%) — enable + disable
AMX_OP_25:      1 ( 0.0%) — undocumented instruction
```

**Finding 6 — 7,832 AMX code blocks; largest block: 748 instructions.**
Confirms that AMX works in intense bursts. A context switch in the middle of a burst would be expensive — this supports the lazy-save strategy.

**Finding 7 — Confirmed performance: 1.3 TFLOPS on the M2.**
```
64×64   matrices: 273  GFLOPS
256×256 matrices: 1,061 GFLOPS
512×512 matrices: 1,309 GFLOPS  ← peak
1024×1024:        1,250 GFLOPS  ← sustained
```
Linux without an AMX driver would run this ~10–50× slower.

### 15.6 ARM System Registers for the Driver (as recorded in the source)
```
AMX_CONFIG_EL1   — enable/disable AMX per process (EL1 = kernel)
AMX_STATE_T_EL1  — save state on context switch
AMX_CONTEXT_EL1  — restore state on context switch
```

---

## 16. AMX — Linux Driver Design

### 16.1 Analogy with the Linux FPU
The AMX driver follows exactly the same pattern as `arch/arm64/kernel/fpsimd.c`.

```
Linux FPU (existing):            Linux AMX (our driver):
─────────────────────            ────────────────────────────
State: 512 bytes                 State: 5120 bytes
Trap: SIGFPE                     Trap: SIGILL (AMX without context)
Enable: CPACR_EL1.FPEN           Enable: AMX_CONFIG_EL1
Flag: TIF_FOREIGN_FPSTATE        Flag: TIF_AMX (new)
```

### 16.2 AMX State Lifecycle
```
Process born → thread.amx_state = NULL (zero overhead)
     │
     ▼ first use of an AMX instruction
SIGILL trap → amx_trap_handler()
     ├─ kzalloc(5120 bytes) → thread.amx_state
     ├─ amx_enable() via AMX_CONFIG_EL1
     ├─ set_thread_flag(TIF_AMX)
     └─ return → process retries → OK

Context switch OUT:
     └─ if TIF_AMX → amx_save_state() → STX×8 + STY×8 + STZ×64

Context switch IN:
     └─ if TIF_AMX → amx_restore_state() → LDX×8 + LDY×8 + LDZ×64

Process exits → kfree(thread.amx_state)
```

### 16.3 Linux Kernel Files to Create/Modify
```
arch/arm64/kernel/amx.c               ← NEW — main driver
arch/arm64/kernel/entry.S             ← add AMX SIGILL handler
arch/arm64/kernel/process.c           ← add amx_thread_free()
arch/arm64/include/asm/processor.h    ← add amx_state to thread_struct
arch/arm64/include/asm/thread_info.h  ← add TIF_AMX flag
arch/arm64/include/asm/amx.h          ← NEW — public declarations
arch/arm64/Kconfig                    ← add CONFIG_ARM64_AMX
```

### 16.4 Driver Implementation (initial draft)
```c
/* arch/arm64/include/asm/amx.h */
struct amx_state {
    u8 x[8][64]  __attribute__((aligned(64)));  /* 512 bytes */
    u8 y[8][64]  __attribute__((aligned(64)));  /* 512 bytes */
    u8 z[64][64] __attribute__((aligned(64)));  /* 4096 bytes */
} __attribute__((aligned(64)));                 /* Total: 5120 bytes */

/* arch/arm64/kernel/amx.c */
#define AMX_INST(op) \
    ".word (0x00201000 + (" #op " << 5))\n"

static void amx_enable(void) {
    asm volatile("nop\nnop\nnop\n"
                 "mov x0, #0\n"
                 AMX_INST(17) ::: "x0", "memory");
}

static void amx_save_state(struct amx_state *s) {
    for (int i = 0; i < 8;  i++) AMX_STX(s->x[i], i);
    for (int i = 0; i < 8;  i++) AMX_STY(s->y[i], i);
    for (int i = 0; i < 64; i++) AMX_STZ(s->z[i], i);
    amx_disable();
}

int amx_trap_handler(struct pt_regs *regs, unsigned int esr) {
    struct task_struct *tsk = current;
    if (tsk->thread.amx_state) return -EFAULT;
    tsk->thread.amx_state = kzalloc(sizeof(struct amx_state), GFP_KERNEL);
    if (!tsk->thread.amx_state) return -ENOMEM;
    amx_enable();
    set_thread_flag(TIF_AMX);
    return 0;  /* kernel retries the instruction */
}
```
*Editorial note: this is the original draft. The final implementation differs (see §21 onward and the driver sources `amx.h` / `amx.c`).*

---

## 17. Research Sources

| Source | What it covers | URL |
|---|---|---|
| corsix/amx | Complete ISA M1–M4, opcodes, writemasks | github.com/corsix/amx |
| MIT thesis 2025 (Jonathan Zhou) | M2 performance analysis, throughput | commit.csail.mit.edu |
| PQC-AMX paper 2024 | Use in post-quantum cryptography | eprint.iacr.org/2024/195 |
| meekolab blog 2025 | EL1 registers, enable/disable | research.meekolab.com |
| Asahi Linux | m1n1, kernel patches, DTBs | asahilinux.org |
| Dougall Johnson | Original AMX + GPU RE | dougallj.github.io/applecpu |

---

## 18. Session Log (early sessions)

### Session 1
Project definition · Technical stack · Chip component analysis · RE tooling · Working protocol

### Session 2
Complete development-environment setup · setup-dev-env.sh script · Errors resolved · Hardware confirmed: MacBook Air M2

### Session 3
Modules implemented: hardware/detect.rs · models.rs · catalog.rs · download.rs · menu.rs · checksum.rs · network.rs · main.rs · 28/28 tests passing · Complete CLI flow working

### Session 4
mprobe.sh · MacBook Air M2 probe · Findings: 16 KB page size · Mac14,2 · T8112 · 4 APFS containers

### Session 5
Complete AMX RE:
- amx_probe.c: enable/disable OK, 5 ns · data instructions: SIGILL (ARM64e PAC)
- amx_observe.sh: 1.3 TFLOPS confirmed · libBLAS mapped
- amx_disasm.sh: dyld shared cache discovered · PAC tag in operands
- find_blas.sh: libBLAS @ 0x18a57d000 · cblas_sgemm @ 0x18abceda0
- amx_opcodes.sh: cblas_sgemm = thin wrapper → _APL_sgemm
- dump_apl_sgemm.py: 139,331 AMX instructions mapped
- ARM64e PAC finding: root cause of all crashes
- Complete GEMM pattern documented
- Linux AMX driver design finalised

**Pending for the next session (as recorded then):**
- ARM64 context around AMX_ENABLE @ 0x18a9b00c4
- Implement arch/arm64/kernel/amx.c

---

## 19. Context for an AI Resuming the Project

Any AI can resume this project by reading this document in full.

**The project:** a Rust CLI installer that natively installs any ARM64 Linux distribution on Macs with Apple Silicon. The software acts as a universal translator between the Apple world and the Linux world.

**Who develops it:** an engineer experienced in C, pointers and Linux drivers, with no prior experience in analysis tooling. Hardware: MacBook Air M2 (Mac14,2, T8112, macOS 15.6) + Debian Linux.

**Code status:** 28/28 tests passing · complete CLI flow working in simulated-M2 mode.

**RE completed:** AMX (Apple Matrix eXtensions) fully mapped. 139,331 instructions in libBLAS analysed. GEMM pattern documented. Linux driver design ready. Cause of crashes = ARM64e PAC tag in operands.

**Next (as recorded then):** ARM64 context around AMX_ENABLE to understand the complete initialisation → implement the driver.

**Rules:** joint decisions · document everything · do not breach Apple · talk to the hardware, do not force it.

---

## 20. AMX — Final Findings from the ARM64 Context

### 20.1 The Four Functions Discovered in libBLAS
```
0x18a99b16c  init_amx_hw()    <- initialises the AMX hardware
0x18a9b0084  amx_enable()     <- reference counting + AMX_ENABLE
0x18a9b00e4  amx_disable()    <- reference counting + AMX_DISABLE
0x18a9b0134  gemm_kernel()    <- massive GEMM kernel (139,329 AMX instructions)
```

### 20.2 amx_enable() — Exact Confirmed Pattern
```
amx_enable @ 0x18a9b0084:
  PACIBSP                     ; ARM64e signature
  STP x20,x19 / STP x29,x30  ; save registers
  ADD x29, sp                 ; frame setup

  ; Check whether AMX is already active
  LDR x8, [amx_state]        ; load global state pointer
  BLR x8                     ; indirect call (vtable)
  MOV x19, x0                ; save result

  LDR w8, [x19, #ref_count]  ; read reference counter
  CBZ w8, .init_needed        ; if (refs == 0) -> initialise

  MOV w0, #0                 ; already active -> return 0
  B   .done

.init_needed:
  BL  init_amx_hw            ; @ 0x18a99b16c -- HARDWARE INIT
  NOP / NOP / NOP            ; 3 mandatory NOPs before AMX
  .word 0x00201220           ; AMX_ENABLE

  LDR w8, [x19, #ref_count]  ; read counter
  MOV w0, #1                 ; return 1 (success)
  ADD w8, w8, #1             ; increment counter
  STR w8, [x19, #ref_count]  ; store new counter

.done:
  LDP x29,x30 / LDP x20,x19
  RETAB                      ; return with PAC (ARM64e)
```

### 20.3 amx_disable() — Exact Confirmed Pattern
```
amx_disable @ 0x18a9b00e4:
  PACIBSP
  STP x29,x30 / ADD x29, sp

  ; Reference counting
  LDR x8, [amx_state]
  BLR x8                     ; indirect check
  LDR w8, [x0, #ref_count]   ; read counter
  SUBS w8, w8, #1            ; decrement
  STR w8, [x0, #ref_count]   ; store
  B.NE .still_active         ; if (refs > 0) do not disable

  MOV w0, #0
  NOP / NOP / NOP            ; 3 mandatory NOPs
  .word 0x00201221           ; AMX_DISABLE

  MOV w0, #1                 ; return 1 (success)
  LDP x29,x30
  RETAB

.still_active:
  RET                        ; other refs active, do not disable
```

### 20.4 Reference Counting — Implication for the Linux Driver

Apple uses a reference counter for AMX. The Linux driver must implement the same pattern:

```c
static atomic_t amx_ref_count = ATOMIC_INIT(0);

void amx_get(void) {
    if (atomic_inc_return(&amx_ref_count) == 1) {
        amx_hw_init();
        asm volatile("nop\nnop\nnop\n"
                     "mov x0, #0\n"
                     ".word (0x00201000 + (17 << 5))\n"
                     ::: "x0", "memory");
    }
}

void amx_put(void) {
    if (atomic_dec_and_test(&amx_ref_count)) {
        asm volatile("nop\nnop\nnop\n"
                     "mov x0, #1\n"
                     ".word (0x00201000 + (17 << 5))\n"
                     ::: "x0", "memory");
    }
}
```

### 20.5 AMX RE Completeness Matrix

| Component | Status |
|---|---|
| Complete ISA (op=0 to op=22) | COMPLETE |
| Operand encoding | COMPLETE |
| 3 NOPs before SET | CONFIRMED in production |
| Reference counting | COMPLETE |
| Lazy-save context switch | DESIGNED |
| Per-process 5120 B state | CONFIRMED |
| ARM64e PAC causes crashes | EXPLAINED |
| 1.3 TFLOPS performance | MEASURED |
| amx_enable() pattern | COMPLETE |
| amx_disable() pattern | COMPLETE |
| init_amx_hw() address | KNOWN @ 0x18a99b16c |
| init_amx_hw() content | COMPLETE — TPIDR2_EL0 + SVE init + no SVC |
| Complete GEMM pattern | COMPLETE |
| Linux driver design | COMPLETE and evidence-based |

---

## 21. AMX_OP_25 — Complete Investigation and Definitive Conclusion

> **Priority note recorded by the researcher:** AMX_OP_25 (0x00201324) is not documented by anyone previously. It must be captured with every detail. This section, together with §22, is that record.

### 21.1 Result of the 46 MB Scan (8 Accelerate libraries)

| Library | Size | AMX_OP_25 |
|---|---|---|
| libBLAS.dylib | 7 MB | **1 occurrence** @ 0x18ac0950c |
| libBNNS.dylib | 11 MB | 0 |
| libLAPACK.dylib | 18 MB | 0 |
| libvDSP.dylib | 1 MB | 0 |
| libSparse.dylib | 1 MB | 0 |
| libvMisc.dylib | 512 KB | 0 |
| libLinearAlgebra.dylib | 128 KB | 0 |
| libSparseBLAS.dylib | 256 KB | 0 |
| **TOTAL** | **46 MB** | **1 single occurrence** |

### 21.2 Complete ARM64 Context Around AMX_OP_25

```
; -- Before (loads and extractions) --
0x18ac094e4: 061fff64  [SVE/AMX prefix=0x06 -- possible EXTRV variant]
0x18ac094e8: 081fffbc  [SVE/AMX prefix=0x08 -- possible EXTRH variant]
0x18ac094ec: 00200344  AMX_OP_26 (op=26, low=4) -- ALSO NEW
0x18ac094f0: 13200358  [SVE prefix=0x13]
0x18ac094f4: 00200774  AMX_OP_27 (op=27, low=20) -- ALSO NEW
0x18ac094f8: 13200788  [SVE prefix=0x13]
0x18ac094fc: 06200ba4  [SVE/AMX prefix=0x06]
0x18ac09500: 08200bf8  [SVE/AMX prefix=0x08]
0x18ac09504: 06200f70  [SVE/AMX prefix=0x06]
0x18ac09508: 08200fc4  [SVE/AMX prefix=0x08]

; -- AMX_OP_25 --
0x18ac0950c: 00201324  AMX_OP_25 (op=25, low=4)  <<<<< TARGET

; -- After (stores with increasing offsets) --
0x18ac09510: 03000014  [prefix=0x03, offset=0x14]
0x18ac09514: 0300005c  [prefix=0x03, offset=0x5c]
0x18ac09518: 530000c4  [SVE ST1, offset=0xc4]
0x18ac0951c: 53000150  [SVE ST1, offset=0x150]
0x18ac09520: 530001d8  [SVE ST1, offset=0x1d8]
0x18ac09524: 53000260  [SVE ST1, offset=0x260]
0x18ac09528: 030002fc  [prefix=0x03, offset=0x2fc]
0x18ac0952c: 53000354  [SVE ST1, offset=0x354]
0x18ac09530: 530003d8  [SVE ST1, offset=0x3d8]
0x18ac09534: 53000458  [SVE ST1, offset=0x458]
```

### 21.3 Additional Discovery — AMX_OP_26 and AMX_OP_27

Analysis of the context revealed TWO further undocumented instructions:

| Instruction | Opcode | op | low | Occurrences |
|---|---|---|---|---|
| AMX_OP_25 | 0x00201324 | 25 | 4 | 1 time in libBLAS |
| AMX_OP_26 | 0x00200344 | 26 | 4 | 1 time (same region) |
| AMX_OP_27 | 0x00200774 | 27 | 20 | 1 time (same region) |

All three instructions appear in the same code region (a 48-byte range). This suggests they form a GROUP of related instructions.

### 21.4 Definitive Conclusion on AMX_OP_25

**1. It is not a general-purpose instruction.**
A single occurrence in 46 MB rules this out. LDX appears 19,207 times.

**2. It is a specialised, edge-case instruction.**
It appears only in libBLAS — not in libBNNS (neural networks) nor in libLAPACK. This rules out use in ML inference or general linear algebra.

**3. Main hypothesis: a GENLUT variant on the M2.**
```
corsix/amx documents:
  op=22 GENLUT -- Generate Lookup Table (documented, M1+)
  op=23 ???    -- Undocumented
  op=24 ???    -- Undocumented
  op=25 ???    -- Undocumented  <-- our finding
  op=26 ???    -- Undocumented  <-- additional finding
  op=27 ???    -- Undocumented  <-- additional finding
```
The pattern (load data → AMX_OP_25 → stores with table-like offsets) is consistent with lookup-table generation or access. *This remains a hypothesis; the exact function is unresolved.*

**4. Impact on the Linux driver.**
```c
/* In amx_save_state() -- include op=25/26/27 as compute instructions */
/* The Z-register state (4096 bytes) already captures the results */
/* There are no additional registers to save */

/* Document in the code: */
/* AMX_OP_25 (0x00201324) - unknown function, possible GENLUT variant on M2 */
/* AMX_OP_26 (0x00200344) - unknown function, same region as OP_25 */
/* AMX_OP_27 (0x00200774) - unknown function, same region as OP_25 */
/* Covered by the context switch via Z-register save/restore */
```

**5. Historical significance.**
First known public documentation of AMX op=25, op=26 and op=27 on the M2. An original contribution to knowledge of Apple's AMX ISA. To be reported to corsix/amx and to the Apple Silicon RE community.

### 21.5 Script That Produced This Finding
```bash
# On the Mac:
python3 scan_op25.py 2>&1 | tee scan_op25_output.txt
```
Generated files:
- `amx_op25_full_report.txt` — complete scan report

---

## 22. init_amx_hw() — Complete Analysis (0x18a99b16c)

### 22.1 The Five Critical Findings

#### Finding 1 — No syscalls (0 SVC calls)
Apple makes NO syscall into the kernel before enabling AMX. AMX is fully accessible from EL0 (userspace) without special privileges.

**Implication for the Linux driver (as recorded):**
- No SIGILL trap handler is needed in the kernel
- No prior registration of the process with the kernel is needed
- The driver is simpler than anticipated
- AMX_ENABLE/DISABLE can be performed directly from EL0

*Editorial note: this finding supersedes the SIGILL-trap flow sketched in §16.2, which was the initial draft.*

#### Finding 2 — TPIDR2_EL0 is the key mechanism

Two accesses to register `S3_3_C13_C0_5`:
```
0x18a99b434: MRS x8, S3_3_C13_C0_5   <- read current context
0x18a99b440: MSR S3_3_C13_C0_5, xzr  <- clear context (NULL)
```

Decoding: `S3_3_C13_C0_5` = **TPIDR2_EL0**
```
op0=3  -> EL0/EL1 accessible
op1=3  -> TPIDR register space
CRn=13 -> Thread registers
CRm=0  -> base
op2=5  -> TPIDR2 (op2=4 = TPIDR, op2=5 = TPIDR2)
```

TPIDR2_EL0 was introduced in ARMv8.7-A for SME / Streaming SVE. Apple reuses it for the AMX context state.

**Exact flow in init_amx_hw():**
```
1. MRS x8, TPIDR2_EL0     <- read the pointer to the current AMX state
2. CBZ x8, .skip           <- if (pointer == NULL) there is no previous state
3. BL  0x18abfb480         <- save_amx_state() <- save the existing state
4. MSR TPIDR2_EL0, xzr    <- clear the pointer (NULL = no context)
```

#### Finding 3 — SVE/SME instructions to initialise registers

The unrecognised instructions (0x25xxxxxx, 0xc0xxxxxx, 0xc1xxxxxx) are SVE/SME:
```
0x25xxxxxx -> SVE instructions (prefix 0x25 = SVE encoding space)
0xc0xxxxxx -> SVE/SME operations
0xc1xxxxxx -> SVE/SME operations
```

Apple uses SVE instructions to zero the AMX registers before the enable. The pattern repeats 12 times → it initialises the 12 registers of the same kind. This guarantees isolation between processes without leaving data from earlier sessions.

#### Finding 4 — Function structure

```
0x18a99b16c  [no PACIBSP] SVE init block (low-level routine)
               -> SVE instructions to clear AMX registers
               -> block-wise initialisation loop
               -> B to 0x18a99b3b4

0x18a99b3e0  Data table (loop constants)

0x18a99b420  [PACIBSP] Second, standard ARM64e function
               -> MRS TPIDR2_EL0 (read previous context)
               -> CBZ (if NULL, skip)
               -> BL save_amx_state() (save if there was a context)
               -> MSR TPIDR2_EL0, xzr (clear the pointer)
               -> more SVE initialisation code
```

#### Finding 5 — save_amx_state() @ 0x18abfb480

Called from init_amx_hw() when TPIDR2_EL0 != NULL. This function saves the existing AMX state before overwriting it. It is Apple's userspace context-save mechanism. Address: libBLAS base (0x18a57d000) + offset ~0x484480.

### 22.2 Final Linux Driver Design — Based on Real Evidence

```c
/* arch/arm64/kernel/amx.c */

/* TPIDR2_EL0 points to the AMX state of the current process */
#define read_tpidr2()  ({ u64 __v; asm("mrs %0, tpidr2_el0" : "=r"(__v)); __v; })
#define write_tpidr2(v) asm("msr tpidr2_el0, %0" :: "r"((u64)(v)))

/* First use of AMX by this process */
int amx_thread_init(void) {
    struct amx_state *state;

    /* If it already has state, do nothing */
    if (read_tpidr2() != 0)
        return 0;

    /* Allocate 5120 bytes aligned to 64 bytes */
    state = kzalloc(sizeof(struct amx_state), GFP_KERNEL);
    if (!state)
        return -ENOMEM;

    /* Register in TPIDR2_EL0 (Apple's mechanism) */
    write_tpidr2((u64)state);

    /* Enable AMX (3 mandatory NOPs first) */
    asm volatile("nop\nnop\nnop\n"
                 "mov x0, #0\n"
                 ".word (0x00201000 + (17 << 5))\n"
                 ::: "x0", "memory");

    return 0;
}

/* Context switch -- save the outgoing process's state */
void amx_thread_switch_to(struct task_struct *prev,
                           struct task_struct *next) {
    u64 prev_state = read_tpidr2();

    /* Save the outgoing process's state if it has AMX active */
    if (prev_state) {
        amx_save_state((struct amx_state *)prev_state);
        /* Disable AMX */
        asm volatile("nop\nnop\nnop\n"
                     "mov x0, #1\n"
                     ".word (0x00201000 + (17 << 5))\n"
                     ::: "x0", "memory");
    }

    /* Restore the incoming process's state */
    u64 next_state = (u64)next->thread.amx_state;
    write_tpidr2(next_state);

    if (next_state) {
        /* Enable AMX */
        asm volatile("nop\nnop\nnop\n"
                     "mov x0, #0\n"
                     ".word (0x00201000 + (17 << 5))\n"
                     ::: "x0", "memory");
        amx_restore_state((struct amx_state *)next_state);
    }
}

/* Cleanup when the process exits */
void amx_thread_free(struct task_struct *tsk) {
    struct amx_state *state = tsk->thread.amx_state;
    if (state) {
        write_tpidr2(0);  /* clear TPIDR2_EL0 */
        kfree(state);
        tsk->thread.amx_state = NULL;
    }
}
```
*Editorial note: this is the design-stage code. The shipped driver later replaced the symbolic `tpidr2_el0` operand with the numeric encoding `S3_3_C13_C0_5` and added MIDR-based hardware detection and guards (see §26).*

### 22.3 Linux Already Has Infrastructure for TPIDR2_EL0

Linux has supported TPIDR2_EL0 since kernel 6.1 for SME (Streaming SVE). The code is in `arch/arm64/kernel/fpsimd.c`. The AMX driver can SHARE and EXTEND this existing infrastructure.

Relevant Linux files:
```
arch/arm64/kernel/fpsimd.c         <- TPIDR2_EL0 save/restore for SME
arch/arm64/include/asm/fpsimd.h    <- declarations
arch/arm64/kernel/entry.S          <- context-switch hooks
```

### 22.4 Script That Produced This Finding
```bash
# On the Mac:
python3 dump_init_amx.py 2>&1 | tee init_amx_output.txt
```
Generated file: `init_amx_hw_@_0x18a99b16c.bin`

---

## 23. Linux AMX Driver — Complete Implementation

### 23.1 Generated Files

| File | Destination in the kernel | Status |
|---|---|---|
| `amx.h` | `arch/arm64/include/asm/amx.h` | COMPLETE |
| `amx.c` | `arch/arm64/kernel/amx.c` | COMPLETE |
| `amx_test.c` | userspace (Mac) | COMPLETE |
| `README.md` | documentation | COMPLETE |

### 23.2 Compiling and Running the Tests on the Mac
```bash
# On the Mac:
clang -O2 -o amx_test amx_test.c
./amx_test
```

The tests validate:
- T1: State size (5120 bytes, 64-byte aligned)
- T2: Enable/Disable latency (must be < 100 ns)
- T3: Save/Restore of the X, Y, Z registers
- T4: Isolation between contexts (Z = 0 in a new context)
- T5: Context-switch performance (complete save + restore)

### 23.3 Kernel Integration — Exact Steps
```
1. Copy amx.h to arch/arm64/include/asm/amx.h
2. Copy amx.c to arch/arm64/kernel/amx.c
3. Add to arch/arm64/include/asm/processor.h:
   struct amx_state *amx_state;  // in thread_struct
4. Add to arch/arm64/include/asm/thread_info.h:
   #define TIF_AMX 27
5. Hook in arch/arm64/kernel/process.c:
   exit_thread() -> amx_thread_free(tsk)
   copy_thread() -> p->thread.amx_state = NULL
6. Hook in arch/arm64/kernel/fpsimd.c:
   fpsimd_thread_switch() -> amx_thread_switch(prev, next)
7. Hook in arch/arm64/kernel/entry.S:
   illegal_instruction handler -> amx_handle_trap()
8. Add CONFIG_ARM64_AMX in arch/arm64/Kconfig
9. Add amx.o to arch/arm64/kernel/Makefile
```
*Editorial note: step 6 was later corrected — Asahi's `fpsimd_thread_switch` takes only `next`, so the call became `amx_thread_switch(next)` (see §26).*

---

## 24. AMX Driver — Validation Results on the MacBook Air M2 (T8112)

### 24.1 Date: 2026-07-05

| Test | Description | Result | Value |
|---|---|---|---|
| T1 | AMX state size | PASS | 5120 bytes · 64-byte aligned |
| T2 | Enable/Disable latency | PASS | 11.80 ns total · 6 ns per side |
| T3 | Save/Restore X[0] | PASS | Identical data before and after |
| T4 | Z-register isolation | PASS | Z[0] = 0x00 in a new context |
| T5 | Context-switch performance | PASS | 0.31 µs · 33,246 GB/s effective |

### 24.2 Analysis of the Results

**T1 — 5120-byte state confirmed.**
The `amx_state_t` struct occupies exactly 5120 bytes with 64-byte alignment. This confirms the driver design is correct for the real hardware.

**T2 — Enable/disable latency: 11.80 ns.**
6 ns per enable or disable operation. Slightly higher than the earlier measurement (5 ns) because of loop overhead. Entirely acceptable for production context switching. Comparison (source's estimate): an ARM64 FPU context switch is typically ~50–200 ns; AMX is 8–33× faster than the FPU for enable/disable.

**T3 — Save/Restore works.**
Data loaded into X[0] is saved and restored identically. The STX/LDX mechanism works correctly on the M2. A solid basis for the kernel context switch.

**T4 — Safe isolation.**
Z[0] is 0x00 in a new AMX context. The Z registers hold no data from earlier sessions. The driver does NOT need to clear Z in init — the hardware already does it. This simplifies `amx_thread_init()`: no memset of Z is needed.

**T5 — Exceptional performance.**
Complete context switch (save 5120 B + restore 5120 B): 0.31 microseconds. Effective bandwidth: 33,246 GB/s. The source attributes this to AMX accessing the cluster's L2 cache directly without passing through main memory, so real memory latency does not apply.
*Methodological caveat (editorial): an "effective bandwidth" of 33,246 GB/s exceeds any plausible physical memory path and should be read as a derived figure (bytes moved ÷ measured time), not as a sustained hardware bandwidth. The 0.31 µs timing is the meaningful result.*

**Scope reminder.** These five tests ran in userspace under macOS. They do not validate the kernel-space path (see §1.1).

**Comparison against a reference (as recorded in the source):**
```
Linux FPU context switch: ~1-5 us
AMX context switch:       ~0.31 us  <- 3-16x faster than the FPU

The AMX driver is more efficient than the existing FPU driver.
```
*Editorial note: the source quotes two different FPU reference ranges (~50–200 ns in §24.2 for enable/disable; ~1–5 µs here for a full context switch). They refer to different operations and are not directly comparable.*

### 24.3 Conclusion

The AMX driver is ready for integration into the Linux kernel. All 5 tests pass on real hardware (MacBook Air M2 · T8112 · macOS 15.6).

Next step (as recorded): clone the Asahi kernel and perform the integration.

---

## 25. Integrating the AMX Driver into the Asahi Kernel

### 25.1 Automatic Script
```bash
# PATH: /home/user/Documents/Projects/Linux/integrate_amx.sh
chmod +x integrate_amx.sh
./integrate_amx.sh
```

The script performs 12 steps in order:
1. Verifies dependencies (flex, bison, libelf, gcc-aarch64, etc.)
2. Clones the Asahi Linux kernel (branch `asahi`) into Linux/asahi-linux/
3. Copies amx.h to arch/arm64/include/asm/
4. Copies amx.c to arch/arm64/kernel/
5. Adds amx.o to the Makefile
6. Patches processor.h — `amx_state` in thread_struct
7. Patches thread_info.h — TIF_AMX = 27
8. Patches process.c — amx_thread_free + NULL init
9. Patches fpsimd.c — amx_thread_switch hook
10. Adds CONFIG_ARM64_AMX to Kconfig
11. Configures the kernel (apple_m1_defconfig as the base)
12. Builds with the ARM64 cross-compiler

### 25.2 Files Modified in the Kernel

| File | Change |
|---|---|
| `arch/arm64/include/asm/amx.h` | NEW — complete header |
| `arch/arm64/kernel/amx.c` | NEW — implementation |
| `arch/arm64/kernel/Makefile` | amx.o added next to fpsimd.o |
| `arch/arm64/include/asm/processor.h` | amx_state in thread_struct |
| `arch/arm64/include/asm/thread_info.h` | TIF_AMX = 27 |
| `arch/arm64/kernel/process.c` | amx_thread_free + NULL init |
| `arch/arm64/kernel/fpsimd.c` | amx_thread_switch hook |
| `arch/arm64/kernel/entry.S` | illegal_instruction handler (manual) |
| `arch/arm64/Kconfig` | CONFIG_ARM64_AMX |

### 25.3 Manual Build (if the script fails)
```bash
cd /home/user/Documents/Projects/Linux/asahi-linux

# Configure for Apple Silicon
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- apple_m1_defconfig

# Enable AMX
./scripts/config --enable CONFIG_ARM64_AMX

# Build (use all cores)
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc)
```

### 25.4 entry.S — Required Manual Patch

The hook in `entry.S` cannot be patched automatically. Add it manually in the illegal-instruction handler:

```asm
/* arch/arm64/kernel/entry.S */
/* In the EL0 SIGILL handler (el0t_64_sync_handler or similar): */

/* Check whether it is an AMX instruction (encoding 0x002010xx - 0x002013xx) */
ldr    w0, [x21]          /* read the instruction that faulted */
and    w1, w0, #0xFFFF0000
cmp    w1, #0x00200000    /* AMX prefix? */
b.ne   .not_amx
and    w1, w0, #0x0000F000
cmp    w1, #0x1000        /* range 0x002010xx - 0x002013xx? */
b.lo   .not_amx
cmp    w1, #0x1400
b.ge   .not_amx

/* It is an AMX instruction -- initialise the context and retry */
bl     amx_handle_trap    /* initialise AMX state */
cbnz   x0, .not_amx       /* on failure (ENOMEM), deliver a normal SIGILL */
b      ret_to_user         /* retry the instruction */

.not_amx:
/* continue with the normal SIGILL */
```
*Status: this manual patch remains an open item (see §32).*

### 25.5 Post-Build Validation
```bash
# Verify AMX symbols in vmlinux
aarch64-linux-gnu-nm asahi-linux/vmlinux | grep -i amx

# Verify that amx.o was built
ls asahi-linux/arch/arm64/kernel/amx.o

# View the build log
tail -50 kernel_build.log
```

---

## 26. AMX Driver — Build Status and Error History

### 26.1 amx.o Built Successfully in the Asahi Kernel
```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- arch/arm64/kernel/amx.o

  CC      arch/arm64/kernel/asm-offsets.s
  CC      kernel/sched/rq-offsets.s
  CC      arch/arm64/kernel/amx.o          <- SUCCESS, NO ERRORS
```

**GCC:** aarch64-linux-gnu-gcc 14.2.0
**Clang:** 19.1.7 (available as an alternative)
**Kernel:** Asahi Linux (branch `asahi`)

### 26.2 Compile Errors Resolved (summary)

| Error | Cause | Solution |
|---|---|---|
| `unknown type name class_raw_spinlock_irqsave_t` | Include cycle: amx.h -> linux/sched.h | Remove the direct include; use the forward declaration `struct task_struct;` |
| `system register name tpidr2_el0` | The assembler needs `-march=armv8.7-a` for the symbolic name | Use the numeric encoding `S3_3_C13_C0_5` in the inline asm |
| `implicit declaration of write_tpidr2_el0` | A global replace renamed the macros | Restore the names `read/write_tpidr2_el0()` while keeping the numeric encoding inside |

### 26.3 Next Steps for the Full Kernel Build (as recorded)
```bash
cd /home/user/Documents/Projects/Linux/asahi-linux

# Build the full kernel
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc) Image

# Verify AMX symbols in vmlinux
aarch64-linux-gnu-nm vmlinux | grep " amx_"

# Build the Apple Silicon DTBs
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- dtbs
```

### 26.4 Kernel Build Error History — All Resolved

**Error 1: `unknown type name class_raw_spinlock_irqsave_t`.**
*Cause:* `amx.h` included `linux/sched.h` directly, creating an include cycle: `processor.h` → `amx.h` → `sched.h` → `processor.h`.
*Solution:* remove the direct include; use the forward declaration `struct task_struct;`.

**Error 2: `system register name tpidr2_el0 not supported`.**
*Cause:* the assembler needs `-march=armv8.7-a` for the symbolic name `tpidr2_el0`.
*Solution:* use the numeric encoding `S3_3_C13_C0_5` in the inline asm.
```c
// Before (fails without -march=armv8.7-a):
asm volatile("mrs %0, tpidr2_el0" : "=r"(__v));
// After (works on any toolchain):
asm volatile("mrs %0, S3_3_C13_C0_5" : "=r"(__v));
```

**Error 3: `implicit declaration of write_tpidr2_el0`.**
*Cause:* the global replace renamed the `read/write_tpidr2_el0` macros to `read/write_S3_3_C13_C0_5`.
*Solution:* restore the correct macro names while keeping the numeric encoding internally.

**Error 4: `prev undeclared in fpsimd_thread_switch`.**
*Cause:* the automatic patch inserted `amx_thread_switch(prev, next)`, but Asahi's `fpsimd_thread_switch` receives only `next`, not `prev`.
*Solution:* change the signature to `amx_thread_switch(struct task_struct *next)` and use `current` for the outgoing process.

**Error 5: `BUILD_BUG_ON sizeof_field thread_struct.uw`.**
*Cause:* the automatic patch inserted `amx_state` INSIDE the `uw` sub-struct (line 163) as well as outside it (line 182). The `BUILD_BUG_ON` checks the exact size of `uw`.
*Solution:* delete the duplicated line inside `uw`:
```bash
sed -i '163d' arch/arm64/include/asm/processor.h
```
`amx_state` then remains only in `thread_struct` directly (line 181), outside `uw`.

**State at the time of the source entry: compiling.**
```
CC      arch/arm64/kernel/amx.o   <- PASS with no errors
CC      fs/fserror.o              <- ordinary kernel files
CC      lib/fdt_strerror.o        <- not compile errors
...
```
Build in progress — estimated 20–40 minutes in total.

---

## 27. Milestone — AMX Driver Integrated into the Linux Kernel

### 27.1 Kernel Built Successfully
```
Image: 40 MB
arch/arm64/boot/Image  Sun Jul 5 11:43:41 2026
Build time: 28 minutes 10 seconds
GCC: aarch64-linux-gnu-gcc 14.2.0
Kernel: Asahi Linux (branch asahi)
```

### 27.2 AMX Symbols in vmlinux — Confirmed
```
ffff80008003668c T amx_handle_trap
ffff8000800363a0 T amx_is_available
ffff800080036460 T amx_restore_state
ffff8000800363a8 T amx_save_state
ffff8000800365c0 T amx_thread_free
ffff800080036520 T amx_thread_init
ffff800080036624 T amx_thread_switch
```
`T` = code in the kernel's text segment (real functions, not declarations). Seven AMX driver functions are present in the Linux kernel for Apple Silicon.

### 27.3 What This Means
The first Linux driver for Apple Silicon's AMX coprocessor exists.

It is based on:
- Reverse engineering of 139,331 AMX instructions in libBLAS.dylib
- Analysis of init_amx_hw() @ 0x18a99b16c (TPIDR2_EL0 confirmed)
- 0 SVC in init_amx_hw (AMX accessible from EL0)
- 5/5 tests passing on real MacBook Air M2 hardware (T8112) — *in userspace under macOS*
- Context switch validated: 0.31 µs, ~33 TB/s effective bandwidth (derived figure; see §24.2)
- Original discovery: AMX_OP_25/26/27 (first public documentation)

### 27.4 Next Step — Test on the M2 Mac (as recorded)
```bash
# Copy the kernel to the M2 Mac (requires Asahi installed as a base)
scp arch/arm64/boot/Image user@macbook:/boot/

# On the M2 Mac (with Asahi):
# Replace the kernel and reboot
sudo cp /boot/Image /boot/Image.amx
sudo reboot

# Verify that AMX is available
cat /proc/kallsyms | grep amx_
dmesg | grep -i amx
```

### 27.5 Final AMX Project Status (at that point)

| Component | Status |
|---|---|
| Complete AMX ISA RE | COMPLETE |
| init_amx_hw() RE | COMPLETE |
| amx_enable/disable RE | COMPLETE |
| GEMM pattern RE | COMPLETE |
| AMX_OP_25/26/27 discovered | FIRST PUBLIC DOCUMENTATION |
| Driver amx.h | COMPLETE |
| Driver amx.c | COMPLETE |
| Tests 5/5 on real M2 | PASS |
| Asahi kernel integration | COMPLETE |
| 40 MB kernel built | COMPLETE |
| 7 symbols in vmlinux | CONFIRMED |
| Test on the M2 Mac | PENDING |

---

## 28. VM Testing — History

### 28.1 Attempt 1: QEMU on Debian with the Asahi kernel
**Date:** 5 July 2026
**Result:** the kernel boots but freezes.
**Cause:** the Asahi kernel, built with an Apple-specific config, has no drivers for virtual hardware (virtio, pl011, etc.).

### 28.2 Attempt 2: UTM on the M2 Mac with the Asahi kernel
**Date:** 21 July 2026
**Result:** black screen on the serial console.
**Cause:** same reason — an Apple-specific kernel without QEMU/virt drivers.

### 28.3 Attempt 3: UTM with Boot from ISO (Ubuntu Server)
**Date:** 21 July 2026
**Result:** WORKS — the Ubuntu installer appears correctly.
**Conclusion:** UTM works fine; the problem is the Asahi kernel.

### 28.4 Solution: Build a Generic ARM64 Kernel for QEMU
**Date:** 21 July 2026
**Status at the time:** compiling.

Config applied on top of the Asahi kernel (branch `asahi`):
```
make mrproper
make defconfig           <- generic ARM64 config
CONFIG_ARM64_AMX         <- our AMX driver
CONFIG_SERIAL_AMBA_PL011 <- serial console for QEMU/UTM
CONFIG_SERIAL_AMBA_PL011_CONSOLE
CONFIG_VIRTIO            <- virtual disk and network
CONFIG_VIRTIO_BLK
CONFIG_VIRTIO_NET
CONFIG_BLK_DEV_SD
CONFIG_ISO9660_FS        <- mount ISO
CONFIG_EXT4_FS           <- filesystem
```

Once built:
1. Transfer the new Image to the Mac
2. Configure UTM with the new kernel
3. Verify that it boots and has a serial console
4. Verify AMX symbols: `cat /proc/kallsyms | grep amx_`

---

## 29. QEMU Validation — COMPLETE (21 July 2026)

### 29.1 Bug Found and Fixed: Panic on Hardware Without AMX

**Initial symptom (QEMU, cortex-a72):**
```
Internal error: Oops - Undefined instruction: 0000000002000000
pc : amx_thread_switch+0x58/0x68
lr : fpsimd_thread_switch+0x78/0x198
Code: a8c17bfd d50323bf d65f03c0 d2800000 (d51bd0a0)
```

**Root cause:** `amx_thread_switch()` executed `msr S3_3_C13_C0_5` (TPIDR2_EL0) unconditionally. The Cortex-A72 is ARMv8.0 and does not implement that register (introduced in ARMv8.7-A). Undefined instruction → panic on every context switch.

The `d51bd0a0` in the dump is exactly `msr S3_3_C13_C0_5, x0`.

**Real impact:** the kernel would have panicked on QEMU, on M1, and on any ARM64 that was not M2+. A critical bug, found thanks to the VM test.

*Methodological note: this bug was discovered by deliberately testing the case in which the hardware has NO AMX. The project regards this negative-case testing as one of its most valuable sources of findings.*

**Fix applied in amx.c:**

1. `amx_probe_hw()` — reads MIDR_EL1 and checks implementer == 0x61 (Apple)
   ```c
   midr = read_cpuid_id();
   implementer = (midr >> 24) & 0xFF;
   if (implementer != 0x61) { amx_available = 0; return; }
   ```
2. `early_initcall(amx_init)` — detection once at boot, not on every switch
3. Early-exit guards in `amx_thread_switch()`:
   ```c
   if (!amx_is_available()) return;          // not Apple
   if (!prev_amx && !next_amx) return;       // no task uses AMX
   ```
4. Equivalent guards in `amx_thread_init()`, `amx_thread_free()`, `amx_handle_trap()`

### 29.2 Validation Result

**Command:**
```bash
qemu-system-aarch64 \
    -machine virt -cpu cortex-a72 -smp 2 -m 2G \
    -kernel asahi-linux/arch/arm64/boot/Image \
    -initrd initrd-amx.img \
    -append "console=ttyAMA0 rdinit=/init" \
    -nographic
```

**Boot log (key excerpts):**
```
[0.000000] Booting Linux on physical CPU 0x0000000000 [0x410fd083]
[0.042968] AMX: not an Apple CPU (implementer 0x41), disabling
[1.234486] Run /init as init process
~ #
```

Complete boot to a shell. Zero oopses. Zero panics.

**Symbols in /proc/kallsyms (runtime, not just vmlinux):**
```
T amx_is_available
T amx_save_state
T amx_restore_state
T amx_thread_init
T amx_thread_free
T amx_thread_switch
T amx_handle_trap
t amx_init
d __initcall__kmod_amx__295_342_amx_initearly
d amx_available
```

### 29.3 What Was Validated on Linux (without needing the Mac)

| Aspect | Status |
|---|---|
| The kernel with the driver boots | VALIDATED |
| MIDR detection works | VALIDATED (0x41 = ARM, rejected) |
| No panic on hardware without AMX | VALIDATED |
| Symbols linked at runtime | VALIDATED (10 symbols) |
| early_initcall registered | VALIDATED |
| Context switch without AMX | VALIDATED (thousands of switches, no oops) |

### 29.4 What Still Requires the Real M2 Mac

| Aspect | Why it cannot be done in QEMU |
|---|---|
| AMX instructions execute | QEMU does not emulate the AMX coprocessor |
| TPIDR2_EL0 responds | The Cortex-A72 is ARMv8.0 and lacks the register |
| 1.3 TFLOPS performance | Requires real silicon |
| amx_thread_init with real state | Requires AMX to be present |

### 29.5 Test-Environment Notes
- Minimal initramfs with ARM64 busybox (1 MB)
- Required dependencies: ld-linux-aarch64.so.1, libc.so.6, libresolv.so.2
- Debian's busybox is dynamic; the 3 libraries must be copied into the initramfs
- Useful symlinks: sh, cat, ls, mount, grep, sed, awk, dmesg, uname
- Without a symlink, use `busybox <applet>` directly

### 29.6 On UTM on the Mac
UTM was tried before QEMU and gave no useful output:
- With Apple virtualization: black screen (different backend, console not exposed)
- Without Apple virtualization: UTM passes the cmdline to QEMU incorrectly (`Could not open 'rw'` = it treats the kernel args as QEMU args)

Conclusion: for kernel tests, QEMU run directly on Linux gives full control and is the correct environment. UTM is discarded for this phase.

---

## 30. Editorial Appendix A — Evidence Standing (consolidated by the editors, not in the source MASTER)

| Claim | Evidence level |
|---|---|
| AMX ISA mapping (op 0–22) | Static analysis of 139,331 instructions in one library (libBLAS) on one machine (M2, macOS 15.6) |
| AMX_OP_25/26/27 exist | Observed once each, in one 48-byte region, in one library; function **unknown** (GENLUT-variant is a hypothesis) |
| 5120-byte state, enable/disable ≈ 6 ns per side, save/restore, isolation | Measured on real M2, **userspace under macOS** |
| Context switch 0.31 µs | Measured on real M2, **userspace under macOS** (STX/LDX round-trip, not a kernel context switch) |
| Driver compiles and links into the Asahi kernel | Reproduced (amx.o; 7 T symbols in vmlinux; 40 MB Image) |
| Driver is inert and safe on non-Apple CPUs | Validated in QEMU (cortex-a72): clean boot, MIDR guard works |
| Driver functions correctly in kernel space on Apple silicon | **NOT YET TESTED** — requires Asahi installed on the M2 |

---

## 31. Editorial Appendix B — Reproducibility Notes (compiled from project history, not in the source MASTER)

- Cross-compiler: `aarch64-linux-gnu-gcc` 14.2.0; Clang 19.1.7 available as an alternative.
- Kernel base: Asahi Linux, branch `asahi` (version 7.0.13 per the project's records).
- Kernel build time recorded: 28 min 10 s; resulting `Image` ≈ 40 MB.
- Development environment is Debian (not Fedora, despite a `qubes.fc41` kernel string on the Qubes host).
- Files that must be re-obtained to rebuild: the Asahi tree (re-cloned by `integrate_amx.sh`), the compiled `Image`, the initramfs, and Cargo.toml/Cargo.lock.
- Raw RE data (dumps `*.bin`, the 4.6 MB `libblas_amx_report.txt` with the 139,331-instruction listing) are not regenerable without the Mac and must be preserved.

---

## 32. Editorial Appendix C — Open Items (compiled from project history, not in the source MASTER)

- Test the driver in kernel space on the M2 (requires Asahi installed) — the only step that genuinely requires the Mac.
- Decide the bootchain strategy (lean on Asahi vs. reimplement). Open.
- Implement the empty installer modules: `partitions/apfs.rs`, `partitions/layout.rs`, `bootchain/m1n1.rs`, `bootchain/uboot.rs`, `bootchain/devicetree.rs`, `distro/install.rs`.
- OpenBLAS integration so that applications actually use the AMX driver.
- Manual `entry.S` trap-handler patch (§25.4).
- Report AMX_OP_25/26/27 to corsix/amx and the Apple Silicon RE community.
- Determine the actual function of AMX_OP_23–27 (currently hypothesis only).

---

*End of document. Translated and restructured from the Spanish MASTER.md; no findings were added or removed. Editorial notes are marked as such and do not alter the source's claims.*
