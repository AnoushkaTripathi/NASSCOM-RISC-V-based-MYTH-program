# ABI Basics


- **Application Binary Interface (ABI)**: 
   - The ABI defines the interface between the operating system and the application programs, enabling interaction between the software and the processor’s hardware.
   - It is critical from both a processor design and user-programmer perspective.
   - It includes system calls that allow applications to access hardware resources.

- **Comparison with Architecture**: 
   - The ABI functions similarly to a building's appearance being the primary interface users interact with, while implementation details (plumbing, bricks) are hidden unless needed.
   - In computers, users care more about the functionality (such as pressing buttons and receiving output) than details like specific processors or microarchitectural choices.

- **Layered Interaction**: 
   - Application programs interact with hardware through multiple software layers.
   - These layers include:
     - Application programming interface (API): Allows interaction between the application and standard libraries (e.g., `stdio.h` in C, Java classes in Java).
     - ABI: Connects the operating system to machine language (ISA—Instruction Set Architecture).

- **ISA and RTL**: 
   - The ISA is the interface between the operating system and the processor’s machine language (e.g., RISC-V, ARM, x86).
   - The ISA implementation is performed via Register-Transfer Level (RTL), defining how the ISA is realized on hardware.

- **Accessing System Resources**: 
   - The ABI allows application programs to access hardware resources via system calls, defining how the software interfaces with the processor's architecture.

- **Registers in RISC-V**: 
   - RISC-V defines registers, and in the case of a 64-bit architecture, it uses 64-bit registers (XLEN 64).
   - The discussion includes:
     - Why there are 32 registers in RISC-V.
     - The role of these registers in accessing system resources via system calls through the ABI.
       
### Memory Allocation for Double words
- **32 Registers in RISC-V**: 
   - The architecture defines 32 registers, sufficient for most operations, especially when handling 64-bit values in a 64-bit system.
   - The lecture explores why 32 registers are optimal and how they handle data efficiently.

- **Registers Example with XLEN 64 (64-bit architecture)**:
   - In a 64-bit system, all registers are 64 bits wide.
   - The data stored in these registers can be either directly placed or loaded from memory.
   - Registers hold only limited amounts of data, so larger data sets must be handled by memory.

- **Loading Data into Registers**:
   - Data can be loaded directly into registers or stored in memory first, then loaded into the registers as needed.
   - Memory operates on a **byte-addressable** system, meaning each memory address points to a single byte.
   - To load a 64-bit value, multiple bytes must be read from consecutive memory locations and assembled in the register.

- **Memory Structure**:
   - Memory organizes data in chunks (8 bytes for a 64-bit number).
   - In **Little Endian** systems (common in RISC-V), the least significant byte (LSB) is stored at the lowest memory address.
   - Example: A 64-bit number is divided across 8 memory addresses, with the lowest byte stored first.
   - The order of bytes is reversed in **Big Endian** systems.

- **Addressing Data in Memory**:
   - Byte-addressable memory means each address points to 1 byte.
   - For example, `M[0]` holds one byte, `M[1]` holds another, and so on.
   - A 64-bit number would occupy 8 consecutive memory addresses.

- **Little Endian Memory System**:
   - The least significant byte is stored first.
   - The 64-bit number is broken down into bytes, and the memory stores them starting from the least significant byte up to the most significant byte.
   - The register then reads these bytes to reconstruct the original 64-bit value.

- **Double-Word (64-bit) Access**:
   - Memory addresses of 64-bit values are multiples of 8 (e.g., `M[0]`, `M[8]`, `M[16]`).
   - For example, if a number is stored starting at `M[0]`, the next number would start at `M[8]`.

- **Loading Data with Instructions**:
   - The lecture concludes by stating that the next section will dive into the specific instructions needed to load data into the 64-bit registers.
   - These instructions help manage the memory-to-register data flow efficiently.

Here are the notes based on the provided text:

## Representation of Load store and add 

1. **64-bit Registers and Memory Access**:
   - Registers in a 64-bit system are 64 bits wide, meaning they can hold 64-bit data.
   - To load data into registers, it can either be directly loaded from memory or stored in registers.

2. **Memory Addressing**:
   - Each memory address holds 1 byte (8 bits).
   - For example, M(0) holds a byte, M(1) holds another byte, and so on.
   - When loading a 64-bit number (8 bytes), the memory follows a *little-endian* memory system:
     - The least significant byte is stored at the lowest memory address, and the most significant byte is stored at the highest memory address.
     - The bytes are stored sequentially in increasing memory addresses.
   - *Big-endian* is another format where the most significant byte is stored first.

3. **Loading Data from Memory to Registers**:
   - To load a 64-bit number from memory, we use specific instructions such as `ldw` (load double word).
   - Example:
     - Suppose `X23` contains the base memory address, and we want to load data from memory starting from an offset of 16. The data is then loaded into the destination register `X0` (64-bit register).
     - The instruction format is: `ldw X0, 16(X23)`
     - This instruction adds the offset `16` to the base address in `X23` to load the data into register `X0`.

4. **Instruction Representation**:
   - The load instruction (like `ldw`) is represented in a 32-bit format:
     - First 7 bits represent the opcode.
     - Following bits represent additional functional fields, source and destination registers, and memory offsets.
     - The instruction is divided into several fields, each representing a part of the instruction.

5. **Addition Example with Registers**:
   - An instruction such as `add X0, X1, X2` adds the contents of register `X1` and `X2` and stores the result in `X0`.
   - Instruction representation:
     - The opcode, function bits, and the destination and source registers are represented by specific fields within the instruction.

6. **Storing Data Back to Memory**:
   - Data can also be stored from a register back into memory using a store instruction, such as `stw` (store double word).
   - Example:
     - `stw X0, 8(X23)` stores the content of register `X0` into the memory location pointed to by `X23 + 8`.
   - Similar to loading, the instruction is represented in a 32-bit format with opcode, functional bits, and source/destination register fields.

7. **Managing Limited Registers**:
   - Registers are limited, and thus, storing data back to memory and reloading as necessary is important to manage register space efficiently.

###  RISC-V Instruction Types and Registers

1. **RISC-V Basic Instructions**:
   - The instructions displayed operate on signed or unsigned integers.
   - These are integer instructions and form part of the RISC-V 64I instruction set.
   - Any processor implementing the RISC-V 64 architecture needs to implement these integer instructions.
   - Example instructions: add, load (LW), and store.

2. **Types of Instructions**:
   - **R-Type Instructions**: Operate only on registers.
     - Example: `add` instruction operates on three registers.
   - **I-Type Instructions**: Operate on registers and immediate values.
     - Example: `load` (LW) instruction involves two registers and an immediate value.
   - **S-Type Instructions**: Used for storing data, with one register holding the data to be stored and another used for addressing.
     - Example: `store` (SW) instruction.

3. **Instruction Set Classifications**:
   - RISC-V has various instruction types based on their format and function.
   - These include:
     - **I-Type**: Immediate value with registers.
     - **R-Type**: Registers only.
     - **S-Type**: Used for store operations (memory).
   - Instructions may also be classified as floating-point or other types based on functionality.

4. **Register Characteristics**:
   - Registers used by R-Type, I-Type, and S-Type instructions are all 5-bit.
   - The 5-bit register size allows up to 32 registers (2^5 = 32).
   - Common register names follow the **ABI (Application Binary Interface)**.
     - Registers: `x0` to `x31` have specific roles in the system.

5. **Register Naming Conventions**:
   - Registers in RISC-V architecture have specific names and roles:
     - **x0**: Hardwired to 0.
     - **x1**: Return address (RA).
     - **x2**: Stack pointer (SP).
     - **x3**: Global pointer (GP).
     - **x4**: Thread pointer (TP).
     - **x5-x7**: Temporaries.
     - **x10-x17**: Argument registers (a0-a7).
     - **x18-x27**: Saved registers (s1-s11).
     - **x28-x31**: Temporaries (t3-t6).

6. **ABI (Application Binary Interface)**:
   - The ABI defines how the registers are used by the application and system.
   - It also defines the names for the registers to be used in system-level programming, such as function calls.

