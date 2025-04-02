## Execution Process
When running a program in SPIKE:
1. The simulator loads the program and its data into memory.
2. Instructions go through the fetch-decode-execute cycle.
3. Register values update according to the executed instructions.
4. Memory operations occur as needed.
5. System calls are processed via the proxy kernel.

## Debugging in SPIKE
SPIKE comes with a built-in debugger, which you can launch using:

```bash
spike -d pk sum1to9
```

### Essential Debugging Commands
- **Run to a specific address**: `until pc 0 10098`
- **Inspect registers**: `reg 0` (all) or `reg 0 a0` (specific register)
- **Check memory content**: `mem 0 2020`
- **View current instruction pointer**: `pc 0`
- **Step through execution**: `step`
- **Disassemble instructions**: `disasm 0 10098 20`

### Debugging Walkthrough
Example of debugging the `sum1to9` loop:

```
# Identify loop location
(spike) disasm 0 10098 20

# Pause execution at loop condition
(spike) until pc 0 100a8

# View key variables
(spike) reg 0 s0    # sum value
(spike) reg 0 s1    # loop counter

# Execute a few steps
(spike) step 10

# Continue execution to next loop iteration
(spike) until pc 0 100a8
```

## Advanced Features
### Performance Tracking
To monitor instruction counts and other metrics, run:

```bash
spike --ic pk sum1to9
```

### Handling Larger Projects
For programs with multiple source files:

```bash
# Compile individual files
riscv64-unknown-elf-gcc -c -O2 -march=rv64i file1.c -o file1.o
riscv64-unknown-elf-gcc -c -O2 -march=rv64i file2.c -o file2.o

# Link compiled files into a final executable
riscv64-unknown-elf-gcc file1.o file2.o -o program
```

---
This toolchain allows developers to write and test RISC-V applications without needing actual hardware. It’s a valuable resource for exploring CPU design and optimizing custom RISC-V implementations.

## Sample Programs
### Finding the Max and Min 64-bit Signed Integer
```c
#include <stdio.h>
#include <math.h>

int main() {
    long long int max = (long long int)(pow(2, 63) - 1);
    long long int min = (long long int)(-pow(2, 63));
    printf("Maximum Signed 64-bit Integer: %lld\n", max);
    printf("Minimum Signed 64-bit Integer: %lld\n", min);
    return 0;
}
```

### Summing Numbers from 1 to 9 in C
```c
#include <stdio.h>

extern int sum1to9_ASS(int x, int y);

int main() {
    int total = 0;
    int upper_limit = 9;
    total = sum1to9_ASS(0x0, upper_limit + 1);
    printf("Sum of numbers from 1 to %d is %d\n", upper_limit, total);
    return 0;
}
```

### Assembly Implementation of Summing 1 to 9
```assembly
.section .text
.global load
.type load, @function

load:
        add     a4, a0, zero  # Initialize sum register a4 to 0
        add     a2, a0, a1    # Load the loop count into a2
        add     a3, a0, zero  # Initialize intermediate register a3 to 0

loop:
        add     a4, a3, a4    # Add intermediate sum to total
        addi    a3, a3, 1     # Increment loop counter
        blt     a3, a2, loop  # Repeat if counter is below limit
        add     a0, a4, zero  # Store final result in a0
        ret
```


![image](https://github.com/user-attachments/assets/f9b4f4dd-2008-41f5-baa7-f33c4cb5a456)
