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

### **7. Practical Considerations**
- You will now proceed to **code all these components**.
- The initial implementation assumes that **everything happens in a single cycle** for simplicity.
- Later, more advanced designs will introduce **pipelining**, memory delays, and other optimizations, but for now, the goal is to understand the core components and their interactions.



---


- You open **Makerchip** and load the **starting point code** for the RISC-V CPU.
- The initial shell contains several predefined elements, including **macros** and the **Makerchip module interface**.
  
#### **A. Program Code Section**
- The first significant part of the shell is a **RISC-V assembly program**.
- This section is created using a **custom assembler** provided within the workshop shell, which compiles programs to RISC-V assembly code.
- This assembler might not fully comply with the official RISC-V assembly syntax, but it is close enough for practical purposes.
  
#### **B. Example Test Program**
- The shell includes a **test program** written in RISC-V assembly.
  - This program sums numbers from 1 to 9, which is similar to the test program you developed earlier using RISC-V tools.
  - The test program is assembled using RISC-V tools, and its assembly matches closely with the compiled code from the workshop.
  
#### **C. Pipeline and Reset Signal**
- The CPU shell provides a **pipeline** with a **reset signal** in **stage 0**, allowing you to develop the rest of the CPU from this starting point.
- The pipeline structure helps in modularizing different stages of the CPU’s operation.

---

### **Key Components Provided in the Shell**

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

### **Working withthe Shell**

#### **A. Code Insertion**
- The shell provides spaces where you can **insert your code** to implement the CPU components and logic.
- As you progress through the workshop, you'll gradually develop the CPU by adding logic to the shell.

#### **B. Testing with Control Signals**
- The shell initializes some basic control signals such as **passed** and **failed** status flags.
- Initially, the test program is set to pass after **40 clock cycles** by default, which can be used to test your implementation.
  
#### **C. Compilation and Execution**
- You can compile the program and the CPU design using the shortcut **Ctrl + Enter**.
- After compilation, Makerchip will generate a **visual diagram** that shows the state of the instruction memory and other components.

#### **D. Customization of Components**
- The components provided, such as the **register file** and **data memory**, are customized for the lab, but they represent **generic elements** that can be reused in various CPU designs.
  
#### **E. Accessing Help and Reference Solutions**
- If you encounter difficulties during the implementation, you can refer to the **help section** or access **reference solutions** for the RISC-V design.
- These solutions provide guidance on the **syntax** and logic required to implement the necessary components.

---

### **Simulation and Visualization**

- Once your design is compiled, the simulation will show how the different components of your CPU behave in each clock cycle.
- The **instruction memory** diagram will display the program, and you can see how the **program counter (PC)** steps through the instructions.
- By monitoring the **visualization output**, you can debug and ensure your CPU is fetching, decoding, and executing instructions correctly.

---

### **Future Development and Pipelining**

- In the early stages of the workshop, the focus is on building a **basic single-cycle CPU**.
- As you progress, the design will evolve to handle more complex features, such as **pipelining** and **multi-cycle operations**.
- These advanced features are not yet part of the current shell but will be added in subsequent exercises.

---

### **Summary of Key Points**

1. **Starting Code**: The shell includes a predefined environment with the basic structure, macros, and a test program.
2. **Assembler**: You work with a RISC-V assembler embedded in the shell that pre-populates the instruction memory with a test program.
3. **Register File and Memory**: Basic components like the register file and memory are provided, ready for you to utilize in your CPU design.
4. **Visualization**: A built-in visualization tool helps you monitor the CPU’s operations and debug the design.
5. **Reference Help**: If you are stuck, reference solutions and syntax help are available for guidance.
6. **Future Enhancements**: The current design is simplified, and future stages will introduce more complex features like pipelining and memory delays.

---

### **1. Lab-Specific Instantiation**
- The provided file contains an instantiation of the **solution** for a specific lab exercise.
- The lab number corresponds to a **slide number** from the workshop’s presentation. If this mechanism changes, you’ll be instructed to read any comments that may appear in the code.
- The slide number ties the lab exercise to the appropriate stage of the CPU development, ensuring that you work on the correct solution for each part of the lab.

### **2. CPU Visualization**
- The **visualization** in Makerchip helps you see the internal operations of the RISC-V CPU.
- Once the design code is substantial, the **visualization diagram** may take some time to generate. This is expected since the complexity of the design increases as you proceed through the lab exercises.
  
#### **A. Instruction Memory**
- The **instruction memory** stores and displays the instructions that the CPU is executing.
- The **disassembled form** of the instructions is shown, making it easier to understand what each instruction is doing.
- A visual **Program Counter (PC)** is represented by an arrow, which steps through the instructions during simulation.

#### **B. Program Counter and Branch Behavior**
- The **PC** increments over each instruction during execution.
- If the CPU encounters a **branch instruction**, the **PC** may initially continue executing the next instruction, but the pipeline will later detect that a branch was taken.
- The CPU will **jump back** to the correct **branch target** and continue execution from the correct instruction, reflecting **branch prediction** and handling in the pipeline.
  
#### **C. Instruction Binary and Decode Logic**
- Each instruction is displayed in its **binary representation**, allowing you to relate the binary values to their corresponding decoded logic.
- You can see how the **instruction decode logic** works as the CPU processes each instruction.
  
#### **D. Bad Path Detection**
- When the CPU detects that it is on a **wrong path** (due to branch misprediction or other issues), the incorrect instructions will be shown in **gray**.
- After detecting the error, the CPU will correct itself by jumping to the correct instruction and resuming proper execution.

#### **E. Register File Updates**
- As the instructions execute, the **register file** is updated in real time.
- For example, if an **ADD instruction** is executed, the result will be stored in a register. The visualization will show the updated values in the registers.
- In this case, the registers that originally held the value **3** are updated to **6** after the ADD operation, and this change is reflected in the register file visualization.

---

### **3. Early Labs and Progressive Development**
- In the **early stages** of the lab exercises, the visualization will not show any significant behavior because the functionality is not yet implemented.
- As you add more logic, you’ll start seeing the **PC moving**, instructions being **decoded**, and the **register file updating** based on the instructions in the program.

---

### **4. Potential Issues with Visualization**
- There might be times when the **diagram doesn’t generate** due to platform load (especially with many students working simultaneously) or technical issues with Makerchip.
- In these cases, you can use the **waveform** to debug and track the behavior of your design.

#### **A. Debugging with Waveform**
- Even if the visual diagram times out, the **waveform** provides a detailed view of the signals used in the simulation.
- The **CPU pipeline signals** are all represented in the waveform, allowing you to debug your design by checking these signals.

#### **B. Reference Solutions**
- If you're stuck on a specific part of the lab exercise, you can refer to the **reference solutions** for hints about the **syntax** or the design logic.
- By comparing the **visualization** or **waveform** from the reference solution to your own design, you can pinpoint differences and understand what might be going wrong.

---

### **5. Protecting Code and Output**
- The solution code is often **hidden** for certain parts of the lab exercises, especially in **protected sections** of the lab.
- Even though you can’t see the **protected code**, you can still use the **visualization** to understand the expected behavior.
  
#### **A. Pipeline Signals for Debugging**
- The **CPU visualization pipeline** in the waveform provides all the signals used in the visualization, so you can use these signals to debug your design.
- By analyzing these signals, you can figure out how the instructions are flowing through the pipeline and identify potential errors in your implementation.

---

### **Summary**
- The **visualization tool** helps you track the progress of your CPU as you develop it through the lab exercises.
- Early on, the visualization won’t show much until you add more functionality, but as you progress, the **PC**, **instruction decode**, and **register updates** will become visible.
- The **waveform** can be used for more detailed debugging, especially when the visual diagram fails to generate.
- **Reference solutions** are available to guide you if you get stuck, and you can compare your design with the correct behavior using both the visualization and the waveform.

# Labs
## Program Counter
![image](https://github.com/user-attachments/assets/615c6c1d-a319-4b7c-86c2-4079c00e6cfa)
![image](https://github.com/user-attachments/assets/dde10a03-d509-40d9-bfbe-a395870a9f27)
![image](https://github.com/user-attachments/assets/3f2e035f-923a-4a56-85ef-bd8e76cf1b0d)

