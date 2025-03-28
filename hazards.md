# Solutions to Pipeline Hazards:

### 1. **Objective: Transition from Three-Cycle Cadence to Back-to-Back Instruction Execution**:
   - The goal is to move the CPU, which is operating with a **three-cycle cadence**, to a more efficient execution model where instructions are executed **back-to-back** where possible, improving overall performance.
   - Two hazard scenarios need to be tackled:
     1. **Register file dependency**
     2. **Branch target dependency**
   - The focus is currently on **register file dependency**.

### 2. **Register File Dependency Hazard (Read-After-Write Hazard)**:
   - **Issue**: The CPU cannot write to the register file in one instruction and then read from it in the next, when instructions are back-to-back and depend on the same register.
   - This situation occurs when the **previous instruction writes to a register**, and the **current instruction tries to read from that same register**.
   - If the register is not involved, the register file has no issues and can be read as usual.
   - To address this hazard, a **bypass path** is introduced.

### 3. **Solution: Register Bypass**:
   - The **register bypass** path allows the CPU to bypass the register file and directly pass the result from the **previous instruction’s ALU** to the current instruction, instead of waiting for the result to be written to the register file and read back.
   - This solution eliminates the **read-after-write hazard** in back-to-back instruction scenarios.

### 4. **Implementation Steps**:
   1. **Step 1: Adjust the Register File Read Path**:
      - The **read path** for the register file is now modified to use the register file value written **two instructions ago**.
      - This is done using the **ahead by two operator**, ensuring the register file read uses the state from two instructions back.

   2. **Step 2: Introduce Multiplexers (Muxes)**:
      - Add **muxes** to handle two potential sources for the ALU input:
        - The register file value.
        - The bypass path (value from the previous instruction).
      - The **mux** will select the correct value based on the following conditions:
        - If the **destination register** written by the previous instruction matches the **source register** needed by the current instruction, the mux selects the **bypass value**.
        - Otherwise, the **register file value** is selected.

### 5. **Ensuring Compatibility with Current Three-Cycle Cadence**:
   - The CPU is still operating in a **three-cycle cadence**, so the newly introduced **bypass logic** should **not be exercised** yet.
   - The logic is designed for back-to-back valid instructions but will **not be active** during the three-cycle cadence.
   - This ensures that the new logic has **no impact** on the current behavior of the CPU, and it does not affect the simulation at this stage.

### 6. **Simulation and Debugging**:
   - After implementing the changes:
     - **Run the simulation** to check that the CPU still passes and behaves as expected.
     - Ensure the **bypass path** and **mux logic** have been implemented correctly.
     - Debug any issues that arise from this change, ensuring that the CPU’s behavior remains consistent.
   - Once the register file bypass is verified, the next step will involve addressing the **branch target dependency**.

---
### 1. **Objective: Correct the Branch Target Path**:
   - The goal is to handle **branch instructions** in the pipeline efficiently while maintaining the current three-cycle cadence.
   - The machine speculates by **incrementing the PC (Program Counter)** each cycle, assuming the next instruction is valid.
   - When a **branch instruction** is taken, the pipeline must correct this speculation and update the PC accordingly.

### 2. **Branch Instruction Dependency**:
   - **Three-cycle cadence**: Branch instructions have a three-cycle path. The CPU cannot avoid this dependency because it needs to:
     1. Decode the branch instruction.
     2. Perform a **register file read**.
     3. Compute the **branch target** based on the PC and immediate value.
   - Only after these steps can the CPU determine whether the branch is **taken or not** and then redirect the PC.
   - There are **two dead cycles** where the CPU speculatively executes instructions before determining if the branch was taken.

### 3. **Handling Branch Penalty**:
   - Since **branch prediction** is not being implemented, the CPU will suffer a **two-cycle penalty** whenever a branch is taken.
   - Branch prediction could shorten or eliminate this penalty by using **heuristics** to guess whether a branch will be taken, but this approach is beyond the scope of this current design.
   - The **two-cycle penalty** remains in place, meaning that the CPU will **execute two incorrect instructions** before redirecting to the correct branch target.

### 4. **Correcting the PC after a Taken Branch**:
   - The machine assumes that **PC + 4** is the correct address for the next instruction.
   - If a branch is taken, the CPU must **kill two instructions** in the pipeline and redirect the PC to the correct **branch target**.
   - The **PC redirect logic** has a three-cycle loop, which remains unchanged.

### 5. **Implementation Steps for Branch Handling**:
   1. **Step 1: Update the Valid Signal Logic**:
      - Implement new logic for **valid** based on whether the previous two instructions (in **stage 4** and **stage 5**) were **branch instructions** that were **not taken**.
      - As long as these two previous instructions are **not taken branches**, the next instruction is valid.
   
   2. **Step 2: Update the PC Loop**:
      - The **PC loop** is updated to reflect a **one-cycle loop** for regular instruction fetching.
      - The **PC redirect** for branches will remain as a **three-cycle loop**, as it already fits the branch handling mechanism.

### 6. **Debugging and Final Steps**:
   - Once the branch handling logic is updated:
     - **Debug the CPU** with the **almost one instruction per cycle** model.
     - Ensure both the **register bypass logic** (for register dependencies) and the **branch target logic** (for branch instructions) are functioning correctly.
     - The CPU should now handle **valid instructions** correctly while accounting for both register dependencies and branch mispredictions.

   ---

## **Overview of Decode Logic**:
   - **Decode logic** is responsible for interpreting the machine-level instructions and converting them into control signals that the CPU uses to perform the intended operations.
   - So far, the decode logic has been partially implemented for a **subset of instructions** in the RISC-V instruction set.
   - Now, the goal is to **complete the decode logic** for the remaining instructions to support the base RISC-V instruction set (RV32I).

### 2. **Base Instruction Set (RV32I)**:
   - The **RV32I** instruction set includes several types of instructions, such as arithmetic, control, and memory access.
   - The implementation will focus on the main instructions, with the exception of a few control-oriented instructions that won't be implemented:
     - **Fence**: Memory fence, used to synchronize memory operations.
     - **Ecall**: Environment call, used for system calls.
     - **Ebreak**: Breakpoint, used for debugging.
   - These instructions aren't crucial for the current focus of the design, so they are skipped.

### 3. **Load Instructions**:
   - The RISC-V instruction set supports **multiple types of load instructions** (e.g., load word, load byte, etc.).
   - Instead of decoding each type of load instruction individually, the plan is to:
     - Create an **`is_load` signal**, which will indicate whether an instruction is a load, regardless of the specific type.
     - This simplifies the design by treating all load instructions the same.
   - The **opcode** field is unique to load instructions, so the decode logic can rely on the opcode without worrying about the **funct3** field (which typically distinguishes between different flavors of load).

### 4. **Bogus Use for Signal Initialization**:
   - As you add new control signals for these instructions, some of these signals might not yet be used in the design.
   - To prevent **compiler warnings** about unused signals, you can use a technique called **bogus use**:
     - This involves assigning the signals in a way that quiets the warnings, even though they aren’t being used yet.
     - When the signals are later connected to the **ALU logic** or other parts of the design, you can remove the bogus use.

### 5. **Next Steps**:
   - **Complete the decode logic**:
     - Implement the remaining instructions in the RV32I instruction set by adding the necessary decoding logic.
     - Create an **`is_load` signal** for load instructions, based solely on the opcode.
     - Use **bogus use** for signals that are not yet connected.
   - Once the decode logic is complete, the next step is to move on to the **Arithmetic Logic Unit (ALU)**:
     - The ALU will consume the decoded signals and perform the actual computation for arithmetic and logic instructions.
---
# Completing the ALU
### 1. **Focus on the ALU**:
   - The ALU is responsible for performing arithmetic and logic operations on the data based on the instructions decoded from the instruction set.
   - The **remaining instructions** that need to be implemented in the ALU are mostly straightforward and involve basic operations.

### 2. **Expression for `result`**:
   - The ALU produces an output called **`result`**, which holds the value computed by the ALU for each instruction.
   - You’ll go back to the Verilog expression for `result` and **fill in the logic** for the instructions that haven’t been implemented yet.
   - Many of these instructions directly correspond to **basic Verilog operators**, making the implementation relatively simple.

### 3. **Common Instructions**:
   - Most of the remaining instructions involve operations like addition, subtraction, bitwise logic, and comparisons. These are easily implemented using built-in Verilog operators.
   - A list of **expressions** is provided for these instructions, which you can directly implement in the ALU.

### 4. **Special Instructions (SLT, SLTI, SLTU)**:
   - A few instructions, such as **SLT (Set Less Than)** and **SLTI (Set Less Than Immediate)**, are slightly more complex because they involve comparisons.
   - The **SLT and SLTI instructions** share a common term with another instruction called **SLTU (Set Less Than Unsigned)**.
     - To simplify the design, the SLTU logic can be **separated out into its own intermediate signal**.
     - This intermediate signal will hold a **32-bit** value that can be used in the expressions for SLT and SLTI.

### 5. **Step-by-Step Implementation**:
   - The process is essentially a **typing exercise**, where you:
     1. Go back to the **`result`** assignment in the ALU.
     2. Add the provided expressions for each instruction.
     3. Separate out the **SLTU** logic as its own signal and reuse it in SLT and SLTI.
   - This will allow you to complete the **entire instruction set** in the ALU.









