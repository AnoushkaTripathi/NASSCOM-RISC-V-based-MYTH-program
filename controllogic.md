# Control Logic
## **Register File Read Process:**

#### **1. Purpose of Register File Read:**
- After decoding the instruction, the next step is to **read from the register file** based on the **source registers** specified by the instruction.
- The fields decoded earlier (like `rs1` and `rs2`) determine which registers in the register file need to be read.

#### **2. Register File Setup:**
- The register file is defined through a **macro instantiation**, which has been **commented out** in the provided shell.
  - **Action**: Uncomment the register file instantiation to enable the register file functionality.

#### **3. Register File Interface:**
- The macro provides a register file with the following interface signals:
  - **Inputs**: Control signals to handle read and write operations.
  - **Outputs**: Data read from the registers.
- The register file supports:
  - **Two reads per cycle** (for source operands).
  - **One write per cycle** (for the destination register).

#### **4. Focus on Read Signals:**
- You'll need to **hook up the read signals** to the appropriate fields in the instruction.
  - **Read Sources (rs1, rs2)**: These are the **source registers** in the instruction, and you’ll connect their values to the register file to control the read operations.

#### **5. Steps to Hook Up Register File:**
1. **Uncomment the Register File Instantiation**:
   - Make sure the register file is active in your design.

2. **Hook Up the Inputs**:
   - **Reset**: Connect the reset signal to initialize the register file.
   - **RS1 and RS2**: These signals represent the indices of the source registers. Connect `rs1` and `rs2` (decoded fields from the instruction) to the inputs of the register file to specify the registers that should be read.

3. **Enable Read Operation**:
   - **RS1 and RS2 Validity**: Only enable the read operation if `rs1` and `rs2` are valid for the current instruction. This validity is determined by the decoded instruction type.
   
4. **Destination Register Confusion (RD)**:
   - **RD Signal**: In RISC-V, **RD** refers to the **destination register**, which is written to, not read from. However, keep in mind that **RD** is commonly used to refer to a **read operation** in general terminology, so don’t confuse it with read signals.
   - **Action**: Ensure the **destination register (RD)** is written to only when the instruction specifies a valid destination.

#### **6. Debugging Tip:**
- To assist with debugging, the **register file macro initializes the register file** such that each register contains its own index after reset.
  - For example:
    - **R5 (X5)** will contain the value **5** after reset.
    - This allows you to easily verify in simulation whether the correct register values are being read based on the indices.

#### **7. Simulation:**
- After connecting the register file signals, run the simulation to ensure that the correct values are being read from the register file.
- Verify that the register values match the expected indices, and debug any discrepancies.

#### **8. Next Step:**
- Once the register file is correctly reading values, the next step will be to **connect the outputs of the register file** to the appropriate parts of the CPU to use the read values.

## **Connecting Register File Read to ALU:**

#### **1. Purpose:**
- After successfully reading values from the register file, the next step is to connect these **read values** to the **Arithmetic Logic Unit (ALU)**. 
- The ALU uses these values as **source operands** to execute the decoded instruction.

#### **2. Hooking Up Register File Outputs:**
- The register file provides **two outputs** (for `rs1` and `rs2`).
- These outputs represent the values read from the source registers (`rs1_val` and `rs2_val`).

#### **3. Signal Connections:**
- **Action**: Connect the **output signals** from the register file (the values read) to the **input signals** of the ALU.
  - These signals should be **clearly and meaningfully named** (e.g., `src1`, `src2` for ALU inputs) for clarity.
  - Ensure that the register file outputs match with the respective ALU inputs for accurate operation.

#### **4. Verifying the Connections:**
- Once the signals are hooked up, **run a simulation** to confirm that the values read from the register file are properly reaching the ALU inputs.
- The ALU can now use these values to perform the required operations (e.g., addition, subtraction, etc.) based on the decoded instruction.

### **Arithmetic Logic Unit (ALU):**

#### **1. Purpose of the ALU:**
- The **Arithmetic Logic Unit (ALU)** performs computations based on the instruction type (e.g., `add`, `addi`, `blt`).
- The result of the computation is stored in a signal called **`result`**.

#### **2. Structure of the ALU:**
- The ALU can be visualized as a **multiplexer (mux)**. It selects the correct computation based on the instruction, similar to how a mux selects inputs based on control signals.
  
#### **3. Ternary Expression for Computation:**
- A **ternary operator** is used to select the output based on the instruction.
  - Example: For `addi`, the result is the sum of the **first source register (`rs1`)** and the **immediate value**.
  - Expression: `result = (instruction == addi) ? (src1 + immediate) : (next_instruction_check);`
  
#### **4. Instructions to Implement:**
- **Current Focus**: Only implement the instructions necessary for the test program. This includes:
  - **`addi`**: Sum of the first source register (`rs1`) and immediate.
  - **`add`**: Sum of two source registers (`rs1` and `rs2`).
  - **`blt` (Branch Less Than)**: Does not produce a result value since it's a conditional branch.
  
#### **5. Action Items:**
- Code up the logic for **`add`** in a similar manner to **`addi`**.
  - **For `add`**: The result is the sum of **`src1` + `src2`**.
- No need to assign a result for **branch instructions** (e.g., `blt`), as they don't output a destination value.

#### **6. Next Steps:**
- After coding the ALU for **`addi`** and **`add`**, simulate to verify correctness.
- The next step will involve moving to the **register file write-back** phase.

## **Register File Array and Pipeline:**

#### **1. Register File Array in Context:**
- The **array** in this context refers to the register file for RISC-V, which has **32 entries**, each storing a register value.
- **Arrays are typically provided by standard libraries** and instantiated rather than coded up manually, but understanding their internal behavior is important, especially when considering timing in pipeline design.

#### **2. Internal Structure of the Array (Register File):**
- Each entry in the array checks whether the **write index** corresponds to its own index. If it does, the entry **selects the data to write** via a multiplexer (mux).
- **Write enable** is triggered when the write index matches the entry index, allowing the value to be written into the respective entry's flip-flops.
- After a reset, the register file values are initialized (e.g., to **zero**).

#### **3. Register File Write and Read Logic:**
- **Write Logic:** Controls writing into the register file, ensuring that only the correct entry gets updated when a specific register is targeted.
  - The write process involves selecting the correct data and enabling the write for the specific register.
  
- **Read Logic:** The **read index** determines which value is output from the register file. The logic allows for selecting the value based on the index of the read operation.
  - In RISC-V, two registers are often read simultaneously, so this logic must handle two reads per cycle.
  - Using **TL Verilog's 'when' expressions**, outputs are only valid when a read operation is enabled.

#### **4. Timing and Pipelining Implications:**
- In a pipeline, the instruction that writes to the register file may also be reading from it in the same cycle.
  - If not timed correctly, this could result in the instruction **reading the value it just wrote**, which is incorrect behavior for a CPU.
  
- The **ideal behavior** is for the CPU to read the values written by the **previous instruction** (from the prior cycle). In a one-instruction-per-cycle pipeline, this ensures that the **updated state** is correctly read by the next instruction.
  
#### **5. Pipeline Staging:**
- In a typical pipeline setup:
  - **Stage 1** might handle the instruction that is writing to the register file.
  - **Stage 2** then reads the register values written in Stage 1 for the subsequent instruction.
  
- **Ahead by one reference:** The read logic must account for the fact that the values it is reading are **one stage ahead** in the pipeline, as they were updated by the previous instruction.

#### **6. Summary of the Register File Timing in the Pipeline:**
- The values stored in the register file represent **state**.
  - The state is updated by one instruction (in one pipeline stage) and read by the following instruction (in the next stage).
  
- Understanding this **write-read separation** in the pipeline helps ensure that the CPU operates correctly, with each instruction reading the correct, most recently updated values.


## **Branch Instructions in RISC-V CPU:**

#### **1. Branch vs. Jump in RISC-V:**
- **Branch Instructions**: Conditional, meaning the branch will be taken based on a condition evaluated between two registers.
- **Jump Instructions**: Unconditional, always taken regardless of any condition.

#### **2. Branch Types in RISC-V:**
- RISC-V offers several conditional branch instructions based on specific conditions between source registers:
  - **Branch if Equal (BEQ)**: Branch if the two source registers are equal.
  - **Branch if Not Equal (BNE)**: Branch if the two source registers are not equal.
  - **Branch if Less Than (BLT)**: Branch if the first source register is less than the second.
  - **Branch if Greater Than (BGT)**: Branch if the first source register is greater than the second.
  - **Unsigned Comparisons**: Branch if Less Than Unsigned (BLTU) and Branch if Greater Than Unsigned (BGTU).

#### **3. Conditional Branch Logic:**
- **Condition Computation**: The condition for each branch is computed using a comparison of two source registers.
  - For example, in BEQ, the condition checks if the values in the two registers are equal.
  - Signed and unsigned comparisons are handled, with the signed comparisons requiring special attention to the sign bit (the upper bit of the value).

## ** Completing Branch Instruction Support:**

#### **1. Branch Target Calculation:**

- **Branch Target PC**: 
  - In RISC-V, the **branch target PC** is computed by adding the **current program counter (PC)** to the **immediate value** specified by the branch instruction.
  - The branch immediate value represents the **offset** from the current PC and can be positive (forward branch) or negative (backward branch).
  - This addition involves the **signed immediate** value but, thanks to **two's complement arithmetic**, you don't need to explicitly handle signed vs. unsigned numbers when performing this addition. It’s simply a standard arithmetic addition in hardware.
  - The formula for the branch target address is:
  ![image](https://github.com/user-attachments/assets/22ff3280-2e00-446e-9c5a-e1e3f953c16f)

  - **Two's complement arithmetic** naturally handles both forward (positive immediate) and backward (negative immediate) jumps, as the binary representation of negative numbers in two's complement already integrates with the addition logic.

#### **2. Modifying the PC Multiplexer (PC Mux):**

- The **PC Mux** is responsible for selecting the address of the **next instruction** to be fetched and executed.
  - Normally, the PC is incremented sequentially to fetch the next instruction.
  - However, when a **branch is taken**, the **next PC** should be the **branch target PC** rather than the next sequential address.

- **Adding Branch Target to PC Mux Input**:
  - Modify the **PC Mux** to include the **branch target PC** as an input option.
  - When a branch instruction is executed and the condition for the branch is satisfied (branch taken), the PC Mux will select the branch target PC as the next value for the PC.
  - The PC Mux needs to have the following inputs:
    - **Incremented PC**: This is the usual next PC value for sequential instructions.
    - **Branch Target PC**: This is the value to select when a branch instruction is executed and the branch is taken.

- **PC Mux Selection Logic**:
  - The selection between the incremented PC and the branch target PC depends on whether the **previous instruction** was a branch and if it was **taken**.
  - If the branch is taken, select the **branch target PC**; otherwise, select the **incremented PC** for the next instruction.
  
#### **3. Handling the Pipeline and Timing:**

- **Pipeline Timing and Instruction Execution**:
  - In pipelined designs, branch instructions need special attention due to the way instructions flow through the pipeline stages.
  - The branch decision (whether to take the branch or not) is made in the **branch instruction cycle**, but the next instruction in the pipeline (at the next PC) depends on this decision.
  - This means that the **PC update** resulting from a branch instruction will affect the next instruction in the pipeline, creating a **timing difference** known as **"ahead by one"**.
    - For instance, the branch instruction is in **cycle 1**, but its result (whether the branch is taken) needs to affect the next instruction that is in **cycle 0**.
  - To handle this correctly, ensure that the PC Mux is properly synchronized with this timing, so that when the branch is taken, the **next instruction fetched** will be at the branch target.

#### **4. Completing the Branch Logic:**

- **Conditions for Branching**:
  - In RISC-V, branches are **conditional**. The branch will be taken based on the result of a comparison between two source registers.
  - The conditions for branch instructions can be:
    - **Branch Equal (BEQ)**: The branch is taken if the two source registers are equal.
    - **Branch Not Equal (BNE)**: The branch is taken if the two source registers are not equal.
    - **Branch Less Than (BLT)**: The branch is taken if the first source register is less than the second.
    - **Branch Greater Than or Equal (BGE)**: The branch is taken if the first source register is greater than or equal to the second.
    - **Unsigned Variants (BLTU, BGEU)**: These perform the same comparisons but treat the values as unsigned.

- **Taken Branch Signal**:
  - You need to compute whether a branch is **taken** or **not taken** based on the result of the condition evaluation.
  - The **taken branch** signal can be created as a **ternary expression**, which checks whether the instruction is a branch and whether the branch condition is met.
  - If the branch condition is satisfied, the **taken branch** signal should be asserted, instructing the PC Mux to select the branch target PC.
  
- **Is-Branch Type Signal**:
  - You can use the **is_branch** signal to indicate whether the current instruction is a branch instruction. This signal is typically derived during the instruction decode phase and used to trigger the PC update logic if a branch is taken.

#### **5. Final Implementation Steps:**

- **Simulating the Branch Logic**:
  - After implementing the branch target calculation and modifying the PC Mux to handle branch instructions, simulate the design to ensure that the branches behave as expected.
  - The program should correctly iterate and branch, for example, in a loop that sums values (e.g., summing values from 1 to 9).
  - Check that:
    - Branches are correctly taken or not taken based on the conditions.
    - The PC correctly updates to the branch target when a branch is taken.
    - The program continues to execute correctly after the branch is taken.

- **Debugging**:
  - If the branch is not being taken when expected, check the condition logic (e.g., comparison between source registers).
  - Verify that the **PC Mux** is correctly selecting between the incremented PC and the branch target PC.
  - Ensure that the **"ahead by one"** timing is handled properly, so that the PC Mux selects the right value for the next instruction.

- **Saving Work**:
  - Once you have confirmed that the branch logic is working as expected, save your code and simulation results outside of the development environment to avoid losing any work.
  
#### **6. Key Takeaways:**

- **Branch Target Calculation**: Add the current PC to the branch’s immediate value (signed addition).
- **PC Mux Update**: Modify the PC Mux to select the branch target PC when a branch is taken.
- **Pipeline Timing**: Ensure the correct timing so that the next instruction uses the updated PC when a branch is taken.
- **Simulation**: Test the branch logic in simulation and verify that the program executes correctly.
- **Debugging**: Ensure that the branch condition logic and PC Mux are functioning properly.

These detailed steps ensure that branch instructions are correctly implemented, allowing the CPU to handle conditional branches in a pipelined architecture.

#### **4. Handling Signed vs. Unsigned Comparisons:**
- In Verilog, comparisons by default are **unsigned**. 
  - For signed comparisons, the sign bit (the most significant bit) must be considered to ensure the correct behavior.

#### **5. Step-by-Step Implementation:**
1. **Code the Branch Conditions**: 
   - Write expressions for the conditions (equal, not equal, less than, greater than, etc.) that determine whether the branch is taken.
   
2. **Combine Conditions**:
   - The branch instruction is only taken if:
     - The instruction is a **branch type** (use the `is_b_type` signal).
     - The condition for the specific branch type (e.g., equal, not equal) is met.

3. **Taken Branch Signal**:
   - Create a signal `taken_branch` to indicate whether the branch should be taken.
   - This signal should be a combination of:
     - The branch condition.
     - The fact that the current instruction is indeed a branch instruction (`is_b_type`).

#### **6. Default Behavior for Non-Branch Instructions:**
- For non-branch instructions, the `taken_branch` signal should default to zero to ensure no unintended branches occur.


## Explanation of Creating a Test Bench to Verify CPU Functionality:

#### **Objective**:
- You now have a working **test program** that runs in simulation, and the goal is to verify whether the program executed correctly. To do this, we will create a **test bench** that automatically checks whether the test program has passed or failed. This test bench will monitor specific signals during the simulation.

#### **Key Components**:

1. **Test Bench Functionality**:
   - A **test bench** is essentially a piece of code that verifies the behavior of the CPU by checking certain conditions during simulation.
   - In this case, the test bench will monitor the value in **register x10**, which stores the summation result of numbers from 1 to 9.
   
2. **Pass/Fail Signals**:
   - The **test bench** will use two signals:
     - **Passed**: Indicates that the test has successfully completed, i.e., the CPU has computed the correct sum.
     - **Failed**: Indicates that the test has not passed or something went wrong in the computation.
   - These signals communicate with the **Makerchip platform** and allow the simulation to be controlled. If the test is successful, the simulation will stop, and a "Pass" message will be logged. If the test fails, the simulation will stop with a "Fail" message.

3. **Monitoring Register x10**:
   - The register **x10** in RISC-V holds the result of the summation (sum of numbers from 1 to 9).
   - The test bench will constantly check the value stored in **x10** to verify whether it matches the expected result (which, for summing numbers from 1 to 9, should be **45**).
   - The syntax `x_reg[10].value` refers to accessing the value in **register 10** (which is x10) from the register file.

4. **Waiting Before Stopping Simulation**:
   - To avoid stopping the simulation immediately after reaching the correct value, we introduce a delay using a concept called **"ahead by 5"**. This delays the test by 5 cycles, allowing more time for the waveform to be visible before halting the simulation.
   - This is useful for debugging and analyzing the behavior of the system, as you can observe the result before the simulation ends.

5. **Pass/Fail Expression**:
   - The expression used for checking if the test passed is based on the value in **x10**. If the value matches the expected summation, the test is marked as passed.
   - For example:
     ```verilog
     pass = (x_reg[10].value == 45)
     ```
   - If this condition is true, the simulation logs a **Pass** message and stops.

#### **Steps to Implement**:

1. **Create the Test Bench**:
   - Write the logic to monitor **x10** and compare it with the expected result.
   - Set the condition for the **Pass** signal if the summation in **x10** is correct.
   - Set the condition for the **Fail** signal if the simulation runs too long without getting the correct result.

2. **Run the Simulation**:
   - Execute the simulation and monitor the log file to see whether the test passed or failed.

3. **Verify in the Waveform**:
   - The delay introduced by **"ahead by 5"** allows you to visually inspect the waveform for a few cycles after the correct value is reached. This can help in debugging and understanding the CPU behavior.

#### **Expected Outcome**:
- If the summation program correctly calculates the sum of numbers from 1 to 9, the value in **x10** will be **45**. The test bench will recognize this and trigger the **Pass** signal, stopping the simulation and displaying a success message in the log file.
- If something goes wrong (e.g., an incorrect summation), the **Fail** signal will be triggered, and the simulation will stop with a failure message.

