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

# Task 5: ABI & Register Cheat-Sheet



<details><summary><b>Step 1: List all 32 RV32 integer registers with their ABI names</b></summary><br>


| Register | ABI Name | Typical Role                      |
|----------|----------|---------------------------------|
| x0       | zero     | Hard-wired zero (always 0)      |
| x1       | ra       | Return address                  |
| x2       | sp       | Stack pointer                  |
| x3       | gp       | Global pointer                 |
| x4       | tp       | Thread pointer                 |
| x5       | t0       | Temporary / caller-saved       |
| x6       | t1       | Temporary / caller-saved       |
| x7       | t2       | Temporary / caller-saved       |
| x8       | s0/fp    | Saved register / frame pointer |
| x9       | s1       | Saved register                 |
| x10      | a0       | Function argument / return     |
| x11      | a1       | Function argument / return     |
| x12      | a2       | Function argument              |
| x13      | a3       | Function argument              |
| x14      | a4       | Function argument              |
| x15      | a5       | Function argument              |
| x16      | a6       | Function argument              |
| x17      | a7       | Function argument              |
| x18      | s2       | Saved register                 |
| x19      | s3       | Saved register                 |
| x20      | s4       | Saved register                 |
| x21      | s5       | Saved register                 |
| x22      | s6       | Saved register                 |
| x23      | s7       | Saved register                 |
| x24      | s8       | Saved register                 |
| x25      | s9       | Saved register                 |
| x26      | s10      | Saved register                 |
| x27      | s11      | Saved register                 |
| x28      | t3       | Temporary / caller-saved       |
| x29      | t4       | Temporary / caller-saved       |
| x30      | t5       | Temporary / caller-saved       |
| x31      | t6       | Temporary / caller-saved       |

</details>

<details><summary><b>Step 2: Calling Convention Summary</b></summary>

- **a0–a7 (x10–x17):** Used to pass function arguments and return values.  
- **s0–s11 (x8, x9, x18–x27):** Callee-saved registers. The function must save and restore these if it modifies them.  
- **t0–t6 (x5–x7, x28–x31):** Caller-saved temporaries. The caller must save these if needed after a function call.  
- **ra (x1):** Return address. Used to store the address to return to after a function call.  
- **sp (x2):** Stack pointer. Points to the current top of the stack.  
- **gp (x3):** Global pointer. Used for accessing global variables in some environments.  
- **tp (x4):** Thread pointer. Used for thread-local storage.  
- **zero (x0):** Always zero. Writes have no effect.

</details>

# Task 6: Emulating and Debugging hello.elf in QEMU using GDB


  
 <details> <summary><strong>Step 1: Start QEMU with debugging enabled</strong></summary>

```bash
qemu-system-riscv32 -nographic -machine sifive_e -kernel hello.elf -S -gdb tcp::1234
```
</details>

 <details> <summary><strong>Step 2: Connect GDB to QEMU</strong></summary>

  Open another terminal and run:

```bash
riscv32-unknown-elf-gdb hello.elf
```

Then inside GDB:

```gdb

(gdb) target remote :1234
```
You should see:

```cpp

Remote debugging using :1234
0x00001000 in ?? ()
```

</details> 

<details> <summary><strong>Step 3: Set breakpoint at <code>main</code> and continue</strong></summary>
  
```gdb
(gdb) break main
Breakpoint 1 at 0x1016a: file hello.c, line 5.
(gdb) continue
```
 Problem:
At this point, QEMU consistently froze after hitting "continue".

Attempting to interrupt using Ctrl+C resulted in:

```ruby
Program received signal SIGINT, Interrupt.
0x00000000 in ?? ()
```
Stepping through with stepi stayed stuck at 0x00000000:

```scss
(gdb) stepi
0x00000000 in ?? ()
```

This behavior indicates that debugging failed due to missing debug symbols or QEMU not progressing correctly.

</details>

<details><summary><b> MY OUTPUT: </b></summary>

![WhatsApp Image 2025-06-05 at 17 24 46_cf10cf24](https://github.com/user-attachments/assets/9bd0c138-c8d7-4ca1-b83a-e2cd2723191a)


![WhatsApp Image 2025-06-05 at 17 24 47_d0c7d13e](https://github.com/user-attachments/assets/ef8e7746-85c8-4c9f-838e-ab3d769045f8)


</details>


# Task 7: Running Under an Emulator

<b>Goal</b>

Run a bare-metal RISC-V ELF program using an emulator (Spike or QEMU) and see output via UART console.  
This is essential if you do not have access to real hardware yet.

---

<details>
<summary><b>Step 1: Compile your bare-metal program with debug symbols</b></summary>

Use the following command to compile your C program (`hello.c`) with the linker script (`linker.ld`) and include debug info:

```bash
riscv32-unknown-elf-gcc -g -nostdlib -nostartfiles -T linker.ld -o hello.elf hello.c
```
</details>

<details> <summary><b>Step 2: Run the ELF using QEMU</b></summary>
Use QEMU's RISC-V system emulator to run your ELF and get UART output:

```bash

qemu-system-riscv32 -nographic -machine sifive_e -kernel hello.elf
```
</details>

<details> <summary><b>Step 3: Debugging using GDB with QEMU</b></summary><br>
  
Start QEMU with GDB server enabled:

```bash
qemu-system-riscv32 -nographic -machine sifive_e -kernel hello.elf -S -gdb tcp::1234
```
-S tells QEMU to start paused (waits for GDB)

-gdb tcp::1234 opens TCP port 1234 for GDB remote debugging

In another terminal, start GDB:

```bash

riscv32-unknown-elf-gdb hello.elf
```

Connect to QEMU's GDB server:

```gdb
(gdb) target remote :1234
```

Use GDB commands:

- info registers
Shows the current values of CPU registers (e.g., ra, sp, gp, a0, etc.)

![WhatsApp Image 2025-06-05 at 19 32 28_5e9db6a7](https://github.com/user-attachments/assets/7211a277-8224-4018-a7c8-cb1950437f2d)


- disassemble or disassemble <function>
Shows the assembly instructions around the program counter or for a specific function

![WhatsApp Image 2025-06-05 at 19 39 10_72e3359e](https://github.com/user-attachments/assets/94bbad30-e94c-44ad-a6c3-efdb0c021c97)

</details>
