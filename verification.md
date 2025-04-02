### Execution Process
During execution:
1. SPIKE loads program code and data into simulated memory
2. Instructions are fetched, decoded, and executed
3. Register values change according to instructions
4. Memory is read/written as needed
5. System calls are handled through the proxy kernel

## Debugging with SPIKE
SPIKE includes a built-in debugger accessed with:

```bash
spike -d pk sum1to9
```

### Common Debug Commands
- **Run to location**: `until pc 0 10098`
- **View registers**: `reg 0` (all) or `reg 0 a0` (specific)
- **Examine memory**: `mem 0 2020`
- **View program counter**: `pc 0`
- **Single-step**: `step`
- **Disassemble**: `disasm 0 10098 20`

### Debug Workflow Example
Tracing through our sum1to9 loop:

```
# Find the loop
(spike) disasm 0 10098 20

# Set breakpoint at loop comparison
(spike) until pc 0 100a8

# Examine variables
(spike) reg 0 s0    # sum variable
(spike) reg 0 s1    # i variable

# Step through one iteration
(spike) step 10

# Continue to next iteration
(spike) until pc 0 100a8
```

## Advanced Usage
### Performance Analysis
Track instruction count and other metrics:

```bash
spike --ic pk sum1to9
```

### Multi-file Projects
For larger programs with multiple source files:

```bash
# Compile individual files
riscv64-unknown-elf-gcc -c -O2 -march=rv64i file1.c -o file1.o
riscv64-unknown-elf-gcc -c -O2 -march=rv64i file2.c -o file2.o

# Link all objects
riscv64-unknown-elf-gcc file1.o file2.o -o program
```

---

This toolchain enables us to develop and test RISC-V software without physical hardware, making it ideal for exploring CPU architecture concepts and testing custom RISC-V CPU core implementations.

```
#include<stdio.h>
#include<math.h>
int main()
{
	long long int max = (long long int)(pow(2,63) -1);
	long long int min = (long long int)(pow(2,63) * -1);
	printf("Highest Signed Number of 4 bits is %lld\n",max);
	printf("Lowest Signed Number of 4 bits is %lld\n",min);
	return 0;
}
```
```
#include<stdio.h>

extern int sum1to9_ASS (int x, int y);

int main()
{
	int result = 0;
	int count = 9;
	result = sum1to9_ASS(0x0, count+1);
	printf("Sum of number from 1 to %d is %d\n", count, result);
	return 0;
}
```
Convert to asm

```
.section .text
.global load
.type load, @function

load:
        add     a4, a0, zero //Initialize the sum register a4 with 0x0
        add     a2, a0, a1   //Store the count of 10 in register a2. Register a1 is loaded with ax0 from main
        add     a3, a0, zero //Initialize the intermediate sum regsiter a3 by 0x0

loop:
        add     a4, a3, a4   //Increament addition
        addi    a3, a3, 1    //Increament intermediate register by 1
        blt     a3, a2, loop //If a3 is less than a2, go to the branch named as <loop>
        add     a0, a4, zero //Store the final result to a0 register which will be read by main program
        ret
```
![image](https://github.com/user-attachments/assets/f9b4f4dd-2008-41f5-baa7-f33c4cb5a456)
