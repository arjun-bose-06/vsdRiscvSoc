# RISC-V Toolchain Setup Log

**Target**: Ubuntu 64-bit  
**Toolchain**: `riscv32-unknown-elf`  
**Purpose**: Setup, environment config, and sanity checks.

---

<details>
<summary><strong>[1] Decompress the Toolchain Archive</strong></summary>

```bash
cd ~/Downloads
tar -xzf riscv-toolchain-rv32imac-x86_64-ubuntu.tar.gz
```
</details>
<details> <summary><strong>[2] Modify the User Environment</strong></summary>
Edit your shell config (~/.bashrc or ~/.zshrc) and append:

```bash
export PATH="$HOME/Downloads/riscv/bin:$PATH"
```
</details>
<details> <summary><strong>[3] Sanity Check: Toolchain Version Output</strong></summary>

  Run the following:


```bash
riscv32-unknown-elf-gcc --version
```
[img] OUTPUT:

```bash
riscv32-unknown-elf-objdump --version
```

[img] OUTPUT:

```bash
riscv32-unknown-elf-gdb --version
```
[img] OUTPUT:
