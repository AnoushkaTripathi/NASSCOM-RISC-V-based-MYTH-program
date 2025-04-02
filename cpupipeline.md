### Pipelining the CPU:

#### **Overview**:
- After creating a functioning CPU and verifying its test environment, the next step is to improve performance by **pipelining** the CPU core. 
- **Pipelining** is a technique where multiple instructions are executed in an overlapping manner, increasing throughput and allowing for higher clock frequencies.

#### **Key Concepts**:

1. **Waterfall Representation**:
   - To understand pipelining, it's helpful to visualize the CPU logic in a **waterfall diagram**.
   - This diagram shows how each instruction moves through different stages of the pipeline, with the logic being replicated for each instruction as it moves through the pipeline.
   - The **top instruction** represents the instruction in stage 1 of the pipeline, and instructions behind it are in earlier stages.
   - This approach highlights how logic is shared between instructions and how **dependencies** are formed from one instruction to the next.

2. **Instructions Feeding into Each Other**:
   - In a **waterfall diagram**, each stage of the pipeline for one instruction feeds into the corresponding stage of the next instruction.
   - For example:
     - The **PC mux** (Program Counter multiplexer) of the first instruction feeds into the second instruction.
     - The **branch target calculation** for one instruction feeds into the next instruction's PC calculation.
     - The **register file read** values for one instruction are used by the following instruction in the next stage.

3. **Pipeline Stages**:
   - The goal of pipelining is to split the CPU logic into stages, allowing each stage to execute less logic in a single cycle.
   - This reduces the critical path delay, enabling the CPU to run at a higher clock frequency.
   - By dividing the work into smaller steps, the CPU can execute instructions more efficiently, as each stage completes part of the instruction in parallel with other stages.

4. **Increasing Performance**:
   - In a pipelined CPU, instructions are **overlapped** during execution, so a new instruction starts every cycle. This creates a **higher performance pipeline**, where each cycle represents progress for multiple instructions.
   - However, this introduces potential problems due to **dependencies** between instructions.

#### **Pipeline Hazards**:
1. **Control Flow Hazard**:
   - When a **branch instruction** is executed, the branch target address is not known until **stage 3** of the first instruction.
   - However, the next instruction needs the branch target address for its **PC calculation** in **stage 1**, two cycles earlier.
   - This creates a **control flow hazard**, where the CPU doesn't know which instruction to fetch next until the branch address is computed.

2. **Data Hazard (Read After Write Hazard)**:
   - Similarly, the CPU may write a value to the **register file** in **stage 4** of one instruction, but the next instruction may need to read that value in **stage 2**, a cycle earlier.
   - This creates a **data hazard**, specifically a **read after write (RAW) hazard**, where the second instruction is trying to read a value before it has been written.

#### **Challenges in Pipelining**:
- **Pipelining** creates performance improvements by increasing the instruction throughput, but it introduces **hazards** that need to be carefully managed.
- These hazards occur because certain instructions depend on results from previous instructions, which might not be available at the time they are needed.
  
#### **Summary**:
- The **waterfall diagram** provides a helpful visualization of how instructions move through a pipeline.
- Pipelining divides the CPU logic into stages to allow higher clock frequencies and increased performance.
- The two main hazards introduced by pipelining are **control flow hazards** (due to branches) and **data hazards** (due to read after write dependencies).
- The next steps in pipelining will involve strategies to mitigate these hazards, allowing the CPU to operate efficiently without stalling or incorrect behavior.
  
## Pipelining and Waterfall Logic Diagram

#### **Pipelining Overview**:
- You have a functioning CPU, and now the goal is to **pipeline** it to improve performance by increasing the clock speed.
- **Pipelining** splits the CPU's logic into stages, allowing each stage to perform a portion of the work, and allowing multiple instructions to be processed simultaneously in an overlapping manner.

#### **RTL Methodology vs. TLVerilog**:
- In traditional **RTL (Register Transfer Level)** methodology, designing and managing pipelines can be complex and error-prone.
- However, with **TLVerilog**, managing pipelines becomes simpler. The framework allows for more flexibility in modifying and experimenting with pipeline configurations.

#### **Waterfall Representation**:
- A **Waterfall Logic Diagram** is a way to visualize the flow of instructions through the pipeline.
- In this diagram, each instruction is replicated across the pipeline stages, showing how logic flows from one instruction to the next.
- **Key idea**: The diagram focuses on how each stage of logic interacts with instructions at different pipeline stages.

#### **How It Works**:
1. **Stages in the Waterfall Diagram**:
   - Each stage (such as **PC Mux** or **branch target calculation**) is drawn for each instruction in the pipeline.
   - For example, **Instruction 1** in stage 1 has a **PC Mux**. **Instruction 2** (behind Instruction 1) is in stage 0, but will soon move to stage 1 and execute its own PC Mux logic.
   - The logic from one instruction can **feed** into the next instruction as it moves through the pipeline.

2. **PC Calculation**:
   - The **PC (Program Counter)** Mux computes the next instruction's address.
   - In the waterfall diagram, the PC Mux logic for the first instruction feeds into the PC calculation for the second instruction.
   - The waterfall diagram allows you to see how this flow of logic is organized across different pipeline stages.

3. **Branch Target Calculation**:
   - For a branch instruction, the **branch target** is computed and then fed into the PC calculation for the following instruction.
   - This illustrates how control flow is handled in the pipeline, as branch instructions affect what happens next.

4. **Register File**:
   - In the pipeline, instructions read from and write to the **register file**.
   - The register file logic produces values that are read by the next instruction, forming a dependency from one stage to the next.
   
#### **Why Pipelining Improves Performance**:
- The key idea behind pipelining is to **overlap** the execution of instructions. By splitting the work across stages, you can execute multiple instructions simultaneously.
- For example:
   - **Instruction 1** is in stage 3, while **Instruction 2** is in stage 2, and **Instruction 3** is in stage 1.
   - This means you're executing multiple instructions at once, with each instruction making progress through its stages every clock cycle.

#### **Challenges of Pipelining – Hazards**:
While pipelining increases performance, it introduces some **challenges**, particularly **hazards**, which occur when there are dependencies between instructions.

1. **Control Flow Hazard**:
   - When a branch instruction is executed, the **branch target** is computed in stage 3.
   - However, the next instruction (which may need this branch target) is in stage 1 or earlier.
   - The pipeline cannot compute the branch target fast enough for the next instruction, creating a **control flow hazard**. This is because the branch target is only known after several stages of execution.

2. **Data Hazard (Read After Write Hazard)**:
   - A **read after write (RAW) hazard** occurs when one instruction writes to a register, but the next instruction tries to read from that register before the value is written.
   - For example, **Instruction 1** writes to the register file in stage 4, but **Instruction 2** might need to read that value in stage 2. Since the value is not yet available, this creates a data hazard.

#### **Summary**:
- Pipelining allows the CPU to process multiple instructions in parallel, increasing the instruction throughput and clock speed.
- **Waterfall Logic Diagrams** help visualize how logic flows between instructions in the pipeline, showing dependencies between stages.
- However, pipelining introduces hazards, such as **control flow hazards** (due to branch instructions) and **data hazards** (due to register file dependencies).
- Managing these hazards is crucial for ensuring the pipeline runs smoothly without stalling or incorrect behavior.
---

### 1. **Transition from Waterfall Diagram to Logic Diagram**:
   - When you switch from a **Waterfall Logic Diagram** (visualizing logic flow across instructions) to a **Logic Diagram** (representing the hardware connections), you will observe the following:
     - If you use **TLVerilog** to change the **@ stages** and re-bucket logic, the logic diagram may imply wires going "backward" in time.
     - For example, if an instruction needs the **branch target PC** computed by the previous instruction, and the next instruction is executed one cycle later, this creates a challenge: the value needed for the next PC is not available until a later cycle.
     - This situation creates an issue similar to requiring **negative flip-flops**, which are impossible because they would involve reversing time.

### 2. **Hazards in the Waterfall Diagram**:
   - These diagrams also highlight **read-after-write hazards**:
     - For instance, if **Instruction 1** writes to register `a3` and **Instruction 2** (a branch instruction) needs to read `a3`, the branch may try to access the value before it’s written.
     - This is a **data hazard**, where one instruction needs to read a value that hasn't been written yet.
   - Similarly, **control flow hazards** occur when a branch instruction requires a branch target PC before it's available, causing delays in the pipeline.

### 3. **Simple Solution: Three-Cycle Latency**:
   - A straightforward way to handle these hazards is by introducing a **three-cycle latency** between instructions.
     - This means that each instruction will begin execution **three cycles** after the previous one, providing enough time to avoid hazards.
     - By adding this three-cycle gap, there are no backward time dependencies (such as trying to access the branch target before it’s available), and the logic becomes **self-consistent**.
   
### 4. **Creating a Three-Cycle Cadence**:
   - To implement this three-cycle latency, you'll need to use a **valid signal**:
     - The pipeline will carry this **valid signal** forward, and every three cycles, it will pulse again.
     - This creates a three-cycle loop of valid pulses, allowing each instruction to execute at the correct cadence.
   - This approach is easier to implement than using a counter, especially since the cycle count is not a power of two.

### 5. **Start Signal After Reset**:
   - After a **reset**, you need to initialize the CPU with a **start signal**.
     - The start signal will pulse after reset is de-asserted (when reset goes low after being high).
     - The valid pulsing begins based on this start signal, and the valid signal will then propagate through the pipeline every three cycles.

### 6. **Generating the Valid Signal**:
   - After initialization, the valid signal will follow a pattern of `low, low, high`.
     - This pattern repeats itself every three cycles, maintaining the three-cycle cadence throughout the pipeline.
     - This ensures that the instructions are spaced correctly and that hazards are avoided.

### 7. **Simulation**:
   - Once you've implemented the valid signal and the three-cycle cadence, run a **simulation**.
     - The waveform should show the valid signal pulsing every third cycle, ensuring that the pipeline operates correctly.
     - Check the waveform carefully to confirm that the pattern matches the expected behavior.

### **Summary**:
- By introducing a **three-cycle latency**, the pipeline can avoid hazards, ensuring that each instruction has enough time to complete its necessary computations before the next instruction executes.
- The **valid signal** maintains the correct timing for the pipeline, and the **start signal** ensures that the CPU begins executing instructions correctly after a reset.


## **Handling Invalid Instructions in the Pipeline**
   - **Three-Cycle Cadence**: Now that we’ve implemented a valid signal to control the execution of instructions in a three-cycle cadence, the next step is handling **invalid instructions**—instructions that don't occur every cycle.
   - The pipeline needs to account for the fact that some cycles will have invalid instructions, and we must ensure that these invalid instructions do not affect the **state** of the CPU.

### 2. **Avoiding State Updates for Invalid Instructions**:
   - **State in the CPU**: 
     - The **register file** is a critical part of the CPU state, and we need to ensure that invalid instructions do not write to the register file.
     - Similarly, the **program counter (PC)** must not be updated by invalid instructions.
   - **Step-by-Step Approach**:
     1. **Register File Protection**: Ensure that invalid instructions do not write to the register file by setting the **write enable** signal to low (deasserted) for invalid instructions.
     2. **PC Update Protection**: Prevent the program counter from being updated by invalid instructions, particularly in cases where the instruction looks like a branch.
         - Introduce a new signal, **valid taken branch**, that factors in the validity of the branch instruction before it affects the PC.
     3. **Three-Cycle Latency**: Adjust the PC update logic so that instructions result in the PC being updated with a three-cycle latency (i.e., the PC is updated by an instruction three cycles after it begins execution).

### 3. **Managing Instruction Timing**:
   - **PC Updates**: The PC must now reflect a **three-cycle update**:
     - If a branch occurs, the next instruction’s PC is redirected three cycles later.
     - Regular **PC increments** also need to be applied with the same three-cycle delay.
   - **Register File Write**: For the register file, while the PC update is delayed, the write operation to the register file can happen with **one-cycle latency**. The next instruction (if invalid) won’t update the register file, ensuring the written value is preserved.

### 4. **Simulation and Waveform Analysis**:
   - **Coding and Simulation**: Once the handling of invalid instructions and state updates is coded, run the design through a **simulation** to verify correctness.
   - In the **waveform**:
     - You should observe a **three-cycle cadence** where valid instructions occur every third cycle.
     - Invalid instructions will fill the gaps in between.
     - Even without using `when` expressions (which could clean up the waveform), the behavior should still be regular and predictable.
   - **Pass/Fail Check**: The simulation should include a pass/fail check, verifying if the instruction pipeline operates correctly.
   - **Invalid Instruction Gaps**: In the waveform, two out of every three cycles should show invalid instructions, consistent with the three-cycle cadence.



## 1. **Objective: Logic Distribution in the Pipeline**:
   - The next step involves **distributing the logic** across pipeline stages, leveraging the **three-cycle cadence** that offers flexibility in organizing the logic.
   - The goal is to move logic around to appropriate pipeline stages and ensure it aligns with the desired design, without worrying about back-to-back instruction execution.

### 2. **Step 1: Partitioning the Logic**:
   - **Partitioning Process**: 
     - The logic must be divided into different pipeline stages using the `@stage` syntax.
     - To implement this, cut and paste the logic across stages, structuring it according to the pipeline in a readable, logical manner.
     - In **Verilog**, it is possible to repeat pipeline stages. For example, logic could be defined in `stage 1`, then logic in `stage 2`, and then more logic back in `stage 1`. However, for readability, it’s best to **organize logic sequentially** within the stages.
   
### 3. **Step 2: Register File Setup**:
   - The **register file macro** (RF macro) takes two parameters: 
     1. **Read Timing**: Set to **stage 2** (`@2`).
     2. **Write Timing**: Set to **stage 3** (`@3`).
   - The register file is set up with a **two-cycle dependency loop**. This means:
     - Instead of the previous **one-cycle** loop, we are now stretching the dependency to **two cycles**.
     - As long as the register file is updated within **three cycles**, the updates will be visible to the subsequent read operation.
   - **Macro Constraints**: The RF macro ensures that:
     - The read happens immediately after the register file flops are written.
     - It forces the timing of register file reads and prevents the staging of flip-flops unnecessarily for multiple cycles.

### 4. **Handling Logic in TL-Verilog**:
   - In **TL-Verilog**, the logic can be easily distributed and structured.
   - By specifying arguments for register files and their respective stages, the code remains **clean and straightforward**.
   - Moving logic around across pipeline stages is a relatively simple task, and TL-Verilog provides **flexibility** in manipulating the RTL with minimal changes.

### 5. **Debugging and Simulation**:
   - Once the partitioning and register file setup is done:
     - **Debug as needed**, ensuring that all logic behaves correctly and the timing is preserved.
     - Run a **simulation** to verify the correctness of the logic distribution.
     - Make sure the **pipeline stages** operate smoothly and adhere to the timing requirements (three-cycle cadence).

### 6. **Impact of Simple Changes in TL-Verilog**:
   - Even though the steps involve simple operations, **TL-Verilog** can significantly alter the RTL.
   - Minimal changes can lead to widespread updates in the RTL logic, making the development process efficient.

## Test Bench for Simulation Verification
![image](https://github.com/user-attachments/assets/4cb0c1c1-39d6-4a34-9d90-e1e539f90b61)

### Purpose
The test bench is used to verify whether the test program has been executed correctly in simulation. It monitors specific signals and ensures the expected results are obtained before stopping the simulation.

### Signals Used
- **`passed` and `failed`**: These signals communicate with the MakerChip platform to indicate whether the test has passed or failed. The simulation halts accordingly and reports the result.

### Monitoring Register for Validation
- The summation result is stored in **register x10**.
- The test bench checks the value inside this register to verify if the correct sum (sum of numbers from 1 to 9) has been computed.

### Expression for Passing Condition
- The expression **`x_reg[10].value`** is used to read the value from register x10.
- A delay of **5 cycles** is introduced before stopping the simulation to allow better observation of waveforms.
- The test verifies that register x10 contains the correct summation result.

### Steps to Implement
1. Define the condition to check if the value in `x10` matches the expected sum.
2. Allow the simulation to run for a few extra cycles before stopping.
3. Check the simulation log to confirm the presence of a **pass** message at the bottom.
4. If the message appears, the test is successful.


