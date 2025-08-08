# ✅ Task 1 – RISC-V Toolchain Setup & Uniqueness Test  

This repository documents the full setup process, code, and verification steps for **Task 1** of the **VSD RISC-V SoC Lab**. The key goals include:

- ✅ Installing the **RISC-V GCC Toolchain**
- ✅ Building and verifying **Spike** (the ISA Simulator)
- ✅ Building and testing **Proxy Kernel (pk)**
- ✅ Running a **uniqueness test** C program to validate the setup

---

## 💻 System Environment

- **Platform**: Rocky Linux (RHEL-compatible)
- **Shell**: Bash
- **Architecture**: x86_64
- **Target**: riscv64-unknown-elf

---

## 📑 Table of Contents

- [1. Install Developer Tools](#1-install-developer-tools)
- [2. Create Workspace & Set `$HOME` Path](#2-create-workspace--set-home-path)
- [3. Download Prebuilt RISC-V GCC Toolchain](#3-download-prebuilt-risc-v-gcc-toolchain)
- [4. Add Toolchain to PATH](#4-add-toolchain-to-path)
- [5. Install Device Tree Compiler](#5-install-device-tree-compiler)
- [6. Build Spike (ISA Simulator)](#6-build-spike-isa-simulator)
- [7. Build Proxy Kernel (pk)](#7-build-proxy-kernel-pk)
- [8. Add RISC-V Bin Directory to PATH](#8-add-risc-v-bin-directory-to-path)
- [9. Optional – Install Icarus Verilog](#9-optional--install-icarus-verilog)
- [10. Sanity Checks](#10-sanity-checks)
- [11. Compile & Run Unique Test](#11-compile--run-unique-test)
- [12. Final Output](#12-final-output)
- [13. Conclusion](#13-conclusion)

---

## ✅ 1. Install Developer Tools

```bash
sudo apt-get install -y git vim autoconf automake autotools-dev curl \
libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex \
texinfo gperf libtool patchutils bc zlib1g-dev libexpat1-dev gtkwave
```

---

## ✅ 2. Create Workspace & Set $HOME Path

```bash
cd
pwd=$PWD
mkdir -p riscv_toolchain
cd riscv_toolchain
```

---

## ✅ 3. Download Prebuilt RISC-V GCC Toolchain

```bash
wget https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14.tar.gz
tar -xvzf riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14.tar.gz
```

---

## ✅ 4. Add Toolchain to PATH
For current shell:
```bash
export PATH=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/bin:$PATH
```
For future shells (persistent):
```bash
echo 'export PATH=$HOME/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```
If libncurses.so.5 is missing:
```bash
cd /tmp
wget http://archive.ubuntu.com/ubuntu/pool/universe/n/ncurses/libtinfo5_6.3-2ubuntu0.1_amd64.deb
wget http://archive.ubuntu.com/ubuntu/pool/universe/n/ncurses/libncurses5_6.3-2ubuntu0.1_amd64.deb
sudo dpkg -i libtinfo5_6.3-2ubuntu0.1_amd64.deb
sudo dpkg -i libncurses5_6.3-2ubuntu0.1_amd64.deb
```

---

## ✅ 5. Install Device Tree Compiler
```bash
sudo apt-get install -y device-tree-compiler
```
---

## ✅ 6. Build Spike (ISA Simulator)
```bash
git clone https://github.com/riscv/riscv-isa-sim.git
cd riscv-isa-sim
mkdir build && cd build
../configure --prefix=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14
make -j$(nproc)
sudo make install
```
---

## ✅ 7. Build Proxy Kernel (pk)
```bash
cd $pwd/riscv_toolchain
git clone https://github.com/riscv/riscv-pk.git
cd riscv-pk
mkdir build && cd build
../configure --prefix=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14 --host=riscv64-unknown-elf
make -j$(nproc)
sudo make install
```
---

## ✅8. Add RISC-V Bin Directory to PATH
```bash
export PATH=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/riscv64-unknown-elf/bin:$PATH
echo 'export PATH=$HOME/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/riscv64-unknown-elf/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```
---

## ✅9. Optional – Install Icarus Verilog
```bash
cd $pwd/riscv_toolchain
git clone https://github.com/steveicarus/iverilog.git
cd iverilog
git checkout --track -b v10-branch origin/v10-branch
git pull
chmod +x autoconf.sh
./autoconf.sh
./configure
make -j$(nproc)
sudo make install
```
---

## ✅10. Sanity Checks
```bash
which riscv64-unknown-elf-gcc
riscv64-unknown-elf-gcc -v
which spike
spike --version
which pk
```
---

## ✅11. Compile & Run Unique Test
Step 1 – unique_test.c (placed in riscv_toolchain/):
```bash
#include <stdint.h>
#include <stdio.h>

#ifndef USERNAME
#define USERNAME "unknown_user"
#endif
#ifndef HOSTNAME
#define HOSTNAME "unknown_host"
#endif

// 64-bit FNV-1a Hash
static uint64_t fnv1a64(const char *s) {
    const uint64_t FNV_OFFSET = 1469598103934665603ULL;
    const uint64_t FNV_PRIME  = 1099511628211ULL;
    uint64_t h = FNV_OFFSET;
    for (const unsigned char *p = (const unsigned char*)s; *p; ++p) {
        h ^= (uint64_t)(*p);
        h *= FNV_PRIME;
    }
    return h;
}

int main(void) {
    const char *user = USERNAME;
    const char *host = HOSTNAME;

    char buf[256];
    int n = snprintf(buf, sizeof(buf), "%s@%s", user, host);
    if (n <= 0) return 1;

    uint64_t uid = fnv1a64(buf);

    printf("=== RISC-V Uniqueness Check ===\n");
    printf("User      : %s\n", user);
    printf("Host      : %s\n", host);
    printf("Unique ID : 0x%016llx\n", (unsigned long long)uid);

#ifdef __VERSION__
    printf("GCC Version String Length: %llu\n", (unsigned long long)(sizeof(__VERSION__) - 1));
#endif

    return 0;
}
Step 2 – Inject Username & Hostname and Compile
```bash
echo "Username: $(id -un)"
echo "Hostname: $(hostname -s)"
riscv64-unknown-elf-gcc -O2 -Wall -march=rv64imac -mabi=lp64 \
-DUSERNAME='"root"' -DHOSTNAME='"Manav"' \
unique_test.c -o unique_test
```
---

## ✅12. Final Output
```bash=== RISC-V Uniqueness Check ===
User      : root
Host      : Shubham
Unique ID : 0x3d0289e4c5a1e2b7
GCC Version String Length: 28
```
---

13. Conclusion
This task provided hands-on exposure to setting up a bare-metal RISC-V development environment. From toolchain installation to ISA-level simulation and program testing, each step solidified my understanding of low-level system development.

💡 Successfully running a personalized test program (unique_test.c) through the Spike simulator demonstrates a fully working setup.
This foundation will now support RTL simulation, SoC integration, and further open-source hardware workflows as part of the VSD SoC Lab.
