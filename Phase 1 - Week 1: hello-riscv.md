# 1. RISC-V Toolchain Setup Log

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

 EXPECTED OUTPUT:

![WhatsApp Image 2025-06-03 at 15 42 08_2e7d8809](https://github.com/user-attachments/assets/989c9f8e-7087-48fb-878b-e650cc827d4f)


```bash
riscv32-unknown-elf-objdump --version
```

EXPECTED OUTPUT:

![WhatsApp Image 2025-06-03 at 16 28 38_e4dec069](https://github.com/user-attachments/assets/0c5f2e27-7f32-4b8e-8dbc-69f7d4381c46)


```bash
riscv32-unknown-elf-gdb --version
```
 EXPECTED OUTPUT:

![WhatsApp Image 2025-06-04 at 16 56 06_c0a8bc80](https://github.com/user-attachments/assets/8168147f-6f4e-4a9e-a268-8e5178a5789a)

</details>

# 2. Compile “Hello, RISC-V”

The goal is to write a minimal C "Hello, World!" program, compile it using the RISC-V toolchain for the RV32IMC architecture, and verify the output ELF file.

---

<details>
<summary><b>Step 1: Write a minimal C program</b></summary>

Create a file named `hello.c`:

```bash
nano hello.c
```

Paste this code inside:

```c
#include <stdio.h>

int main() {
    printf("Hello, RISC-V\n");
    return 0;
}
```
Save and exit the editor (Ctrl + O, Enter, then Ctrl + X in nano).

</details>

<details> <summary><b>Step 2: Compile using the RISC-V cross-compiler</b></summary>
Run this command in the terminal:

```bash

riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -o hello.elf hello.c
```
 If you get an error like "cannot find suitable multilib", it means your toolchain doesn't support rv32imc. You can try replacing with:

```bash
riscv32-unknown-elf-gcc  -o hello.elf hello.c
```
</details>

<details> <summary><b>Step 3: Verify the output ELF file</b></summary>
Check the type of the file using:

```bash

file hello.elf
```
Expected output (also displaying the possible error you may encounter):

![WhatsApp Image 2025-06-04 at 18 10 47_6ea5a008](https://github.com/user-attachments/assets/f3d84b2a-7628-4d71-b93d-99b7e4499c55)

</details>

# 3. From C to Assembly

### Goal
Take the same `hello.c` program and generate the assembly code (`.s` file), then understand what's happening in the function prologue/epilogue.

---

### Step-by-Step Instructions (Terminal Only)

<details>
<summary><strong> 1. Ensure your C file still exists</strong></summary>

Check your C source file:

```bash
cat hello.c
```

![WhatsApp Image 2025-06-04 at 17 10 59_b874a9f3](https://github.com/user-attachments/assets/2be19e58-941e-4686-a683-b661f835e159)
</details>

<details> <summary><b>Step 2: Generate the assembly code</b></summary>
Use the compiler to generate a .s file (assembly code) from the C source:

```bash
riscv32-unknown-elf-gcc -S -O0 hello.c
```
This will produce a file named hello.s.

</details>

<details> <summary><b>Step 3: View the generated assembly</b></summary>
View the output using:

```bash

cat hello.s
```
You should see something like this:

![WhatsApp Image 2025-06-04 at 17 19 39_32f9dfb7](https://github.com/user-attachments/assets/e4e233a0-7572-41fd-8d9e-460e7c77a6d8)

</details>

<details> <summary><b>Step 4: Understand each assembly instruction</b></summary>

![image](https://github.com/user-attachments/assets/efa29835-f2a8-4a90-b82e-c2fc675b60cc)

The first two correspond to the prologue and the bottom three to the epilogue

 These instructions are automatically inserted to manage function call/return safely.

</details>


  
# Task 4: Hex Dump & Disassembly

The goal is to turn your compiled RISC-V ELF file (`hello.elf`) into:
1. A human-readable disassembly (addresses + instructions).
2. A raw hexadecimal file (Intel HEX format) that can be used for flashing or simulation.

---

<details>
<summary><b>Step 1: Disassemble the ELF file</b></summary>

Use `objdump` to convert your ELF binary into readable RISC-V assembly with addresses:

```bash
riscv32-unknown-elf-objdump -d hello.elf > hello.dump
```
To view it:

```bash
cat hello.dump
```
PARTIAL OUTPUT:

![WhatsApp Image 2025-06-04 at 19 13 14_4f0828e8](https://github.com/user-attachments/assets/3c7ab543-6980-43a5-89f1-de0461a9a887)

100b4          -> Memory address(hex)  
1141           -> Raw machine code  
addi           -> mnemonic  
sp,sp,-16      -> operands  
addi sp,sp,-16 -> opcode  

</details>

<details> <summary><b>Step 2: Generate the raw hex format</b></summary>
  
Use objcopy to convert the ELF file into Intel HEX format:

```bash

riscv32-unknown-elf-objcopy -O ihex hello.elf hello.hex
```

To view the hex contents:

```bash
cat hello.hex
```

PARTIAL OUTPUT:

![WhatsApp Image 2025-06-04 at 19 25 04_c4b66b75](https://github.com/user-attachments/assets/52cb54f5-4367-4a84-9b1b-c23f44631cfb)

This file represents your binary's memory layout and can be loaded into emulators or flashed to hardware.

</details>
</details> 
