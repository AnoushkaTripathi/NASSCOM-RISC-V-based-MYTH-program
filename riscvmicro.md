# **1. RISC-V Microarchitecture Overview**
- **RISC-V** is an **Instruction Set Architecture (ISA)** that defines the set of instructions a CPU can execute.
- The **microarchitecture** refers to the internal design and logic that implements the functionality defined by the ISA.
- We are building a **simple RISC-V CPU microarchitecture** where the components will work together to execute RISC-V instructions.

---

### **2. Key Components in the Microarchitecture**
![image](https://github.com/user-attachments/assets/f467d11e-60f3-4e84-af70-8576a895a78b)

#### **A. Program Counter (PC)**
- The **Program Counter (PC)** is a register that holds the address of the **next instruction** to be executed.
- It serves as a pointer into the instruction memory.
- Every clock cycle, the PC is incremented (unless a branch or jump occurs) to point to the next instruction.
- The PC is sent to the **instruction memory** as the address from which the instruction is fetched.

#### **B. Instruction Memory**
- **Instruction Memory** is where all the program's instructions are stored.
- The **PC value** serves as the index or address to this memory, fetching the next instruction to execute.
- The data returned from the instruction memory is the **fetched instruction**, which is passed on to the **decode logic** for further interpretation.

#### **C. Instruction Decode Logic**
- The fetched instruction needs to be interpreted, and that's the role of the **decode logic**.
- The **decode logic** splits the instruction into different fields:
  - **Opcode**: Specifies the type of operation (e.g., load, store, add, branch).
  - **Source registers**: Identifies the registers containing the operands for the operation.
  - **Destination register**: Where the result will be written (in case of arithmetic/logical operations).
  - **Immediate values**: Constants encoded directly into the instruction, used in operations like branches or memory addressing.
- In branch instructions, the immediate value typically represents an **offset** that tells how far to jump from the current PC.

#### **D. Register File**
- The **Register File** is a small, fast storage unit that holds the **operands** for instructions.
- It contains a set of **general-purpose registers**.
- The CPU needs to **read from** and **write to** registers during execution.
- For arithmetic operations, two registers are usually read simultaneously (called a **two-port register file**).
- The output from the **ALU** (Arithmetic Logic Unit) or memory is written back into the register file.

#### **E. Arithmetic Logic Unit (ALU)**
- The **ALU** performs arithmetic and logical operations (such as addition, subtraction, and bitwise operations).
- It takes two inputs, typically from the source registers, and computes the result.
- Examples of operations:
  - **Addition (ADD)**: Takes two numbers from the source registers and adds them.
  - **Subtraction (SUB)**: Takes two numbers and subtracts the second from the first.
  - **Logical operations**: Bitwise AND, OR, XOR, etc.
- The ALU's result is then written back to the **destination register** in the register file.
- The ALU is conceptually similar to a simple **calculator** where values are fetched, an operation is performed, and the result is stored.

---
![image](https://github.com/user-attachments/assets/a3bda26c-1bc6-44ac-8f47-8b4f1d2c01a9)
Link to repo : https://github.com/stevehoover/RISC-V_MYTH_Workshop

 **Starting Point Code**: 
   - The GitHub repository for the workshop contains starting point code for RISC-V design.
   - The shell includes some basic content like macro definitions and program instantiation using Makerchip's module interface.
![image](https://github.com/user-attachments/assets/2775d8b7-3dc8-4d80-a693-d5d81844f5bc)


   **RISC-V Assembler**: 
   - The shell uses a RISC-V assembler based on TL-Verilog, which is unlocked through a macro preprocessor called `m4`.
   - This assembler is not fully compliant with the RISC-V standard syntax, but it's close enough for use in most cases.
   
 **Pre-Defined Test Program**:
   - The provided code matches the test program created earlier during the workshop, summing up numbers from 1 to 9.
   - The CPU pipeline is already set up with a reset signal connected in stage 0, and there's a placeholder for custom code.
![image](https://github.com/user-attachments/assets/3bf2a93d-ba1d-41c7-bb24-5ecd388ca2d9)

 **Pipeline Components**:
   - The shell contains a pre-configured instruction memory populated with assembly instructions.
   - Components like the register file (for reading/writing) and data memory are also provided.

 **Platform Communication**:
   - The code communicates with the platform to indicate pass/fail after 40 cycles.
   - Basic macros for logging and feedback are initialized to track progress.
![image](https://github.com/user-attachments/assets/4abe6bbe-bcc7-4c89-ad19-d7196fb76ca3)

 **Instruction Memory and Arrays**:
   - The instruction memory contains pre-loaded instructions.
   - The shell uses customized components for the lab exercises, including array structures like instruction memory, register file, and data memory.

 **Visualization**:
   - A visualization tool for the CPU is available, similar to the one used for the calculator example.
   - The tool helps in understanding and debugging the design visually.

 **Reference Solutions**:
   - Reference solutions are available under the 'Help' section in case users get stuck with their design implementation.
### **3. Branching and Control Flow**

#### **A. Branch Instructions**
- **Branching** is how the CPU changes the flow of execution (e.g., conditional jumps).
- A **branch instruction** includes:
  - **Immediate value** (offset): Specifies how far to jump relative to the current instruction.
  - **Condition**: The branch may depend on a condition (e.g., if a register is equal to zero).
- The **PC** is updated based on the result of the condition:
  - If the condition is met, the **PC** is updated with the **branch target** (PC + offset).
  - If the condition is not met, the PC continues to increment normally.

#### **B. Branch Target Calculation**
- The **next PC value** for a branch instruction is computed by adding the **PC** (current instruction address) to the **immediate value** (offset).
- This is done using an **adder**, which computes the branch target and updates the PC.
- The **updated PC** will point to the next instruction to execute if the branch is taken.

---

### **4. Memory Access (Load and Store Instructions)**

#### **A. Load Instructions**
- **Load instructions** transfer data from memory into the CPU's registers.
- The memory address is calculated by adding a **base address** from a register and an **immediate offset** encoded in the instruction.
- The CPU reads the data from this calculated memory address and stores it in a **destination register**.

#### **B. Store Instructions**
- **Store instructions** write data from the CPU registers to memory.
- Similar to load instructions, the **store address** is computed by adding a base register value to an immediate offset.
- The **data** to be stored comes from the source register and is written to memory at the calculated address.

#### **C. Address Calculation**
- The memory address for both load and store operations is calculated by an **adder**, which adds the **base register value** to the **immediate offset**.
- The calculated address is then sent to memory to either fetch data (in the case of load) or store data (in the case of store).

---

### **5. Pipeline and Cycle Timing**

#### **A. Single-Cycle Model**
- For simplicity, the design assumes a **single-cycle model**, meaning:
  - All the operations (instruction fetch, decode, ALU operations, memory access, etc.) happen in one clock cycle.
  - This model is highly simplified for educational purposes, and in real-world CPUs, many stages take multiple cycles.
  
#### **B. No Memory Delays Assumed**
- In this workshop, we assume that memory access is instantaneous (i.e., there is no delay in fetching or writing data from/to memory).
- This assumption simplifies the process, but in a real CPU, memory access is typically **multi-cycle** because memory is slower than the CPU.

#### **C. Feedback and PC Update**
- After each instruction is executed, the **PC is updated** for the next instruction fetch.
- For normal instructions, the PC is incremented by the instruction size (usually 4 bytes).
- For branches or jumps, the PC is updated based on the **branch target** or jump address.

---

### **6. Summary of Execution Flow**

1. **Instruction Fetch**: The PC points to the next instruction in memory, which is fetched and sent to the decode logic.
2. **Instruction Decode**: The instruction is broken into fields (opcode, source registers, destination register, immediate values, etc.).
3. **Register Read**: Source registers are read from the register file.
4. **ALU Operation**: The ALU performs the arithmetic or logical operation based on the opcode and source register values.
5. **Write Back**: The result of the ALU operation is written back to the destination register in the register file.
6. **Memory Access (if needed)**: For load and store instructions, the memory is accessed to either read or write data.
7. **PC Update**: The PC is updated to point to the next instruction, either by incrementing it or computing a branch target.

---


#### **A. Instruction Memory**
- The **instruction memory** is populated with the **RISC-V program** you write in the assembler section.
- The test program defined in the assembly is automatically stored in the **instruction memory**.

#### **B. Register File**
- The shell provides the **register file** that stores the CPU’s general-purpose registers.
- The **register file** supports both read and write operations, which will be used in later stages of the CPU’s pipeline.

#### **C. Data Memory**
- **Data memory** is provided but will be used in later parts of the exercise, particularly when you start working with **load and store instructions**.

#### **D. Macro Instantiations**
- Standard **array structures** are provided via macros, which make it easier to instantiate memory elements like the **instruction memory** and **register file**.
- These elements have been customized for the lab but are similar to **generic array libraries** used in real-world CPU designs.

#### **E. CPU Visualization**
- The shell includes a **visualization** feature for the CPU, similar to the **calculator visualization** in the earlier exercises.
- This **visual representation** helps you visualize the internal operations of your CPU, making it easier to debug and understand.
  
---

 **Visualization for RISC-V Core**:
   - Visualization is enabled for the CPU, showing the final implementation of the RISC-V core.
   - Initially, when functionality is incomplete, the program counter (PC), instructions, and register file won’t update, but this changes as labs progress.
     
![image](https://github.com/user-attachments/assets/879511ff-ee76-45d3-ab3f-f01830033c1f)

 **Instruction Memory and PC**:
   - The visualization shows the instruction memory with disassembled instructions.
   - The program counter (PC) increments, and during a branch, the machine adjusts by jumping back to the correct branch target.
![image](https://github.com/user-attachments/assets/7e6dd1d9-09ad-4ddb-850b-b49b9f5f70b0)

 **Binary Representation of Instructions**:
   - Instructions are also displayed in their binary form, aiding in understanding the decode logic.
   - The tool visualizes how each instruction, like an `add`, is decoded, showing the values in the registers and how the results are written back to the register file.

 **Register File Updates**:
   - The register file shows updated values as instructions are processed, and users can observe how values are written into the registers.
![image](https://github.com/user-attachments/assets/1781c160-1fc2-4a99-bac6-0f7c78268ee4)

 **Visualization Behavior Over Time**:
   - Early labs won’t show functionality in the visualization, but as logic is added, components like PC and instructions will start updating.
   - The visualized components (like pipeline signals) assist in debugging and understanding logic flow.

 **Timeout and Load Issues**:
   - Diagrams may time out due to high platform load or other tool limitations, especially with many users working simultaneously.
   - Users can still use the waveforms for debugging if diagrams fail to generate.

 **Waveform for Debugging**:
   - The waveform helps users debug the CPU’s behavior by showing pipeline signals.
   - Users can compare the behavior in the waveform with the expected logic to identify errors.

  
## **Instruction Memory**
- The **instruction memory** stores and displays the instructions that the CPU is executing.
- The **disassembled form** of the instructions is shown, making it easier to understand what each instruction is doing.
- A visual **Program Counter (PC)** is represented by an arrow, which steps through the instructions during simulation.

#### **Program Counter and Branch Behavior**
- The **PC** increments over each instruction during execution.
- If the CPU encounters a **branch instruction**, the **PC** may initially continue executing the next instruction, but the pipeline will later detect that a branch was taken.
- The CPU will **jump back** to the correct **branch target** and continue execution from the correct instruction, reflecting **branch prediction** and handling in the pipeline.
  
#### **Instruction Binary and Decode Logic**
- Each instruction is displayed in its **binary representation**, allowing you to relate the binary values to their corresponding decoded logic.
- You can see how the **instruction decode logic** works as the CPU processes each instruction.
  
#### **Bad Path Detection**
- When the CPU detects that it is on a **wrong path** (due to branch misprediction or other issues), the incorrect instructions will be shown in **gray**.
- After detecting the error, the CPU will correct itself by jumping to the correct instruction and resuming proper execution.

#### **Register File Updates**
- As the instructions execute, the **register file** is updated in real time.
- For example, if an **ADD instruction** is executed, the result will be stored in a register. The visualization will show the updated values in the registers.
- In this case, the registers that originally held the value **3** are updated to **6** after the ADD operation, and this change is reflected in the register file visualization.

---

---

### ** Potential Issues with Visualization**
- There might be times when the **diagram doesn’t generate** due to platform load (especially with many students working simultaneously) or technical issues with Makerchip.
- In these cases, you can use the **waveform** to debug and track the behavior of your design.

#### **Debugging with Waveform**
- Even if the visual diagram times out, the **waveform** provides a detailed view of the signals used in the simulation.
- The **CPU pipeline signals** are all represented in the waveform, allowing you to debug your design by checking these signals.

  
#### ** Pipeline Signals for Debugging**
- The **CPU visualization pipeline** in the waveform provides all the signals used in the visualization, so you can use these signals to debug your design.
- By analyzing these signals, you can figure out how the instructions are flowing through the pipeline and identify potential errors in your implementation.

---

### 2. **Next PC Logic**:
   - The **Next PC Logic** is responsible for determining the address of the next instruction. Initially, it’s designed for sequential execution (i.e., one instruction after the other).
   - The **PC (Program Counter)** is incremented with each instruction fetch to point to the next instruction in memory.

![image](https://github.com/user-attachments/assets/6f07e205-f8ed-451b-9c18-1fb4c45b0c28)
   - **Increment by 4**: Since each RISC-V instruction is 32 bits (4 bytes), the PC needs to increment by 4 after each instruction fetch. The following relationship defines the next PC value:


     This ensures that the next instruction is fetched from the correct memory address.

     
![image](https://github.com/user-attachments/assets/615c6c1d-a319-4b7c-86c2-4079c00e6cfa)

```
 $reset = *reset;
         
         //code for Program counter (PC)
         $pc[31:0] = (>>1$reset) ? 32'd0 : (>>1$pc + 32'd4);
         //curr pc = (is prev reset value true) ? if yes, curr pc = 0 : if not, curr pc = prev pc + 4 (4 locations = 1 instr)
         
```
---

### 3. **Handling PC Reset**:
   - **Reset Behavior**: When the CPU is reset, the program counter should start from 0, so the first instruction that the CPU executes is located at address `PC = 0`.
   - **Issue**: If the next PC logic is naively implemented, after the reset, the PC would increment from 0 to 1. This would result in the CPU starting from the second instruction instead of the first.

   - **Solution**:
     - The solution involves resetting the PC to 0 during the first non-reset cycle, ensuring the first instruction is properly fetched.
     - The PC value must be 0 when the CPU begins execution, meaning that **during the first instruction after reset**, the PC should remain at 0 instead of incrementing.

---

### 4. **PC Update Mechanism**:
   - **Avoiding Skipping the First Instruction**: Instead of relying on the reset signal directly, the logic needs to account for the state of the previous instruction to decide if the next PC should be 0. Specifically:
     - If the **previous instruction** was a reset instruction, then the **next instruction** should start at `PC = 0`.
     - After the reset period is over, the PC should begin incrementing by 4 for each instruction.

   - **Implementation Detail**:
     - Use the previous instruction’s reset signal (`previous_reset`) to control the PC value. This ensures that during the first instruction after reset, the PC is set to 0, and from then on, it follows the normal incrementation.

     The logic for updating the PC can be represented as:
        ![image](https://github.com/user-attachments/assets/c779bcda-afb8-4de5-bb31-0aa9a9773c3b)

     This prevents the issue of starting from the wrong instruction and ensures the first instruction is fetched and executed correctly.

---

### 5. **Simulation and Testing**:
   - **Simulating Next PC Logic**: After implementing the next PC logic, it’s critical to verify that the program counter behaves as expected. This involves:
     1. Checking that the **PC starts at 0** after reset.
     2. Ensuring that, after the first instruction is fetched, the PC increments by 4 for subsequent instructions.
     3. Verifying that, in the absence of any branch or jump instructions, the PC increments sequentially through the program.

   - **Waveform Analysis**: 
     - Using simulation tools, you can observe the waveform of the PC and other signals.
     - Confirm that after reset, the PC remains at 0 for the first instruction, and subsequent instructions increment the PC by 4.
     - Ensure that when there is a branch or jump instruction (added later), the PC reflects the correct target address instead of sequential incrementation.

---

### 6. **Branch and Jump Logic (Future Implementation)**:
   - Although branch and jump instructions are not immediately handled, it is important to design the initial next PC logic in a way that branch target computation can be added later.
   - Once branch instructions are implemented, they will modify the next PC by an offset, and the PC will need to jump to the branch target address instead of following a sequential flow.

---

### 7. **Step-by-Step Development**:
   - As the **next PC logic** is developed, test each small piece in isolation:
     1. Implement the PC incrementing by 4.
     2. Test the reset behavior to ensure the CPU starts from `PC = 0`.
     3. Simulate each step and verify using waveforms that the PC and related components (e.g., instruction fetch) are working correctly.

---


# Labs
## Program Counter
![image](https://github.com/user-attachments/assets/615c6c1d-a319-4b7c-86c2-4079c00e6cfa)
![image](https://github.com/user-attachments/assets/dde10a03-d509-40d9-bfbe-a395870a9f27)
![image](https://github.com/user-attachments/assets/3f2e035f-923a-4a56-85ef-bd8e76cf1b0d)

