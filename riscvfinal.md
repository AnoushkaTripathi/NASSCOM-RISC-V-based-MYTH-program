# Final Phase of RISC-V CPU Design


### 1. **Introduction to Load and Store**:
   - In RISC-V, there are different types of **load instructions** that support both signed and unsigned loads of varying data widths (bytes, half-words, full words).
   - **Signed loads** extend the upper bit based on its value (sign extension), while **unsigned loads** zero-extend the upper bits.
   - However, in this implementation, only **full word load** and **store** instructions will be supported.

### 2. **Decoding Load Instructions**:
   - For simplicity, all five types of loads are decoded to a single **is_load** signal, without distinguishing between the different types of loads (e.g., byte, half-word).
   - This approach uses the **opcode** field, which is unique to load instructions, to simplify the decoding process.

### 3. **Operation of Load Instructions**:
   - In RISC-V, the **load instruction** specifies the memory address from which data should be loaded into a destination register.
   - The address is calculated as the sum of a register value (**rs1**) and an **immediate** value (an offset).
   - This address calculation is the same as that used for the **addi** instruction.

### 4. **Operation of Store Instructions**:
   - The **store instruction** writes data from a source register (**rs2**) to a memory address, which is calculated similarly using **rs1** and the **immediate**.

### 5. **Timing and Hazard in Load Instructions**:
   - A **two-cycle delay** exists between when the load address is presented and when the data is available for loading.
   - This introduces a **hazard**, as the CPU can’t write the data to the register file immediately, and must **invalidate two subsequent instructions** (to avoid conflicts in writing to the register file).

### 6. **Handling Load Hazards (Waterfall Diagram)**:
   - The earliest point at which the CPU can access the load data is **two cycles** after presenting the address.
   - In the **load shadow**, the subsequent two instructions are **invalidated** (similar to handling branch hazards) to ensure the load data can be correctly written to the register file.

### 7. **PC Redirection for Load**:
   - To handle load operations, the CPU must **redirect the PC** (program counter) to the instruction following the load, and **replay** the instruction after the load.
   - This ensures that the **load data** is written to the register file properly, and subsequent instructions operate with the correct data.

### 8. **Bypassing Load Data**:
   - Due to the register file **bypass mechanism**, the CPU must ensure that the **load data** being written to the register file is also available for bypass.
   - This allows subsequent instructions to use the **load result** immediately, without waiting for it to be written to the register file.

### 9. **Implementation Steps**:
   - **First step**: Implement the **PC redirect** for load instructions, similar to the approach used for branches. This involves clearing the **valid** signal for instructions in the **shadow** of the load.
   - **Second step**: Ensure that the **incremented PC** from three cycles ago is selected as the next PC.
   - **Next step**: Debug and test the logic to ensure the **load shadow** and redirect behave as expected.
   - After completing this, the next step would be to handle the actual **load data** and ensure it's correctly written to the register file.
## Implementation of **load data handling** and **memory address calculation** for load/store instructions in the RISC-V architecture:

### 1. **Load Data Path Implementation**:
   - After implementing the **load shadow** mechanism, the next step is to handle the actual load data.
   - We're assuming the availability of a **load data signal** from memory (though memory is not yet implemented).
   - This **load data** will be written to the **register file** two cycles after the load instruction execution.

### 2. **Steps to Implement Load and Store Logic**:
   - **Step 1: Compute Address for Loads/Stores**:
     - Both loads (identified by **is_load**) and stores (identified by **is_s_type**) need to compute an address.
     - Use the **ALU** to compute the memory address, which is the result of adding **rs1** (source register 1) and an **immediate** value (similar to the **addi** instruction).
   
   - **Step 2: Add Mux for Result and Load Data**:
     - Introduce a **mux** in front of the **result** signal.
     - This mux selects between the **ALU result** (for valid instructions) and the **load data** (for invalid instructions in the load shadow).
     - The easiest selection logic is: if the instruction is **invalid**, use the **load data**; otherwise, use the **ALU result**.
   
   - **Step 3: Control Write Enable Signal**:
     - Ensure that the **write enable** signal is asserted two cycles after the load instruction to write the load data to the register file.
     - De-assert the write for the load instruction itself, as the **ALU result** (address) should not be written to the register file.
     - Optionally, **clean up** by ensuring the load instruction does not have a valid destination register (**rd**) in the decode logic.
   
   - **Step 4: Correct Register File Index**:
     - Add a **mux** to select the correct **register file index**.
     - Two cycles after the load instruction, write the load data into the **destination register** specified by the load instruction’s **rd** field from two cycles ago.

### 3. **Important Considerations**:
   - Ensure the **load data** is written only after two cycles.
   - The **destination register index** for the load should be selected correctly based on the load instruction two cycles prior.
   - **Save** your work frequently, and verify in simulation.

### Instantiating Data Memory and Hooking it to the Logic:

1. **Data Memory Instantiation**:
   - A generic macro for **data memory** has been provided, similar to the instruction memory and register file.
   - The data memory is designed to support either **one read** or **one write** operation per cycle.
   - It has **16 entries**, each 32-bits wide (i.e., **word-wide** memory).
   - The macro is a simplified toy CPU memory setup, not a full-blown memory system.

2. **Data Memory Interface**:
   - The interface signals provided by the macro need to be hooked up appropriately for load and store operations.
   - You’ll be using:
     - **Address** signals (provided by the ALU result)
     - **Write enable** for stores
     - **Read enable** for loads
     - **Data for store** comes from **rs2** (the second source register).

3. **Address Consideration**:
   - The address provided is a **byte address**.
   - Since only **word** operations are supported, you’ll assume that the **byte address** is aligned on a word boundary:
     - The lower two bits of the byte address are ignored (they are assumed to be zero).
     - To **index the memory** with a word index, consider **address bits 5 to 2** (since the memory has 16 entries).
   
4. **Connecting Control Bits**:
   - **Write Enable**: Assert this signal for **store** operations (when there’s valid data to store).
   - **Read Enable**: Assert this signal for **load** operations (when valid data needs to be read).
   - Ensure that the memory control bits (write/read) are **only asserted for valid instructions**, preventing unintended operations.

5. **Data Memory Read**:
   - The data memory provides a **read data signal** (for load operations), which should be connected to the register file.
   - Remember to implement a **two-cycle delay** for the load data path, as it takes two cycles for load data to be written back to the register file.
   - Ensure that the **data from two cycles ago** is correctly looped back into the register file for valid instructions.

6. **Write Conditioning**:
   - Properly condition the write signal so that it does not assert during invalid cycles or when a load operation is mistakenly treated as a write.

## Testing Load and Store Instructions:

1. **Basic Testing of Load and Store**:
   - It's recommended to test load and store instructions by adding simple test cases at the end of your program.
   - This test will **store** the value in **r10** after the loop completes and then **load** that same value into **r15**.

2. **Test Process**:
   - After the **loop completes**, **r10** should hold the final summation value.
   - The summation value in **r10** will be **stored** at **address 4** (a binary address value).
   - Immediately after, the value at **address 4** is **loaded** into **r15**.

3. **Verifying the Test**:
   - The passing condition of the test will now be based on the value in **r15** (x-reg15) instead of **r10**.
   - Modify the **passing signal condition** (`star passing star past`) to check **r15** instead of **r10**.

4. **Additional Benefit**:
   - This test also verifies that the loop ends correctly:
     - Previously, the loop could continue beyond the expected iteration (if branch conditions were incorrect), and the issue wouldn't have been detected.
     - Now, by **storing** the value after the loop and **loading** it into a new register (**r15**), the loop's proper termination can be verified.
     - If the loop doesn’t terminate properly, this test may reveal a bug in the **branch less than condition**.

5. **Final Steps**:
   - Run the test to ensure the program ends properly and passes the conditions.
   - After confirming the load and store functionality, you can proceed to the next step (jump instructions).

### Key Tasks:
   - Add **store** instruction to store the value from **r10** at **address 4**.
   - Add **load** instruction to load the value from **address 4** into **r15**.
   - Modify the **passing condition** to check **r15** instead of **r10**.
   - Run the program to test the load and store instructions and verify the proper termination of the loop.
     
## Implementing Jump Instructions:

1. **Overview of Jump Instructions**:
   - There are two types of jumps to implement: 
     - **Jump and Link (JAL)**
     - **Jump and Link Register (JALR)**
   - **JAL** functions like a **branch** but without a condition, jumping to a target calculated as **PC + immediate**.
   - **JALR** calculates the target relative to a **register value (source 1)** instead of the **PC**.

2. **Implementation of JAL**:
   - **JAL** works similarly to a branch:
     - No condition needs to be evaluated.
     - Ensure the **taken branch signal** is asserted for jumps.
     - The target address is computed as **PC + immediate** (just like a branch).
     - Redirect the **PC** after the jump and insert **invalid cycles** (similar to branches and loads).

3. **Implementation of JALR**:
   - **JALR** computes the target based on the **source 1** value (the register value) instead of the **PC**.
   - A separate adder is used to compute **source 1 + immediate**.
   - To handle this properly, ensure the source 1 value comes after the **bypass mux** (to account for potential bypassing).
   - To optimize, the same adder can be reused by adding a **mux** to choose between the **PC** (for branches/JAL) and **source 1** (for JALR).
     - This requires moving the adder and introducing a control signal to select the appropriate input (PC or source 1).
   - A simpler approach is to use a separate adder for **JALR** computation, avoiding the complexity of muxes and shared resources.

4. **Steps for Implementation**:
   - **Step 1**: Define the **is jump** signal, which is the logical OR of **JAL** and **JALR** instructions.
     - For jumps, redirect the **PC** (just like for taken branches).
     - Insert **invalid cycles** after the jump (similar to what was done for branches and loads).
   
   - **Step 2**: Perform the target computation for **JAL**:
     - Compute **PC + immediate** (already done for branches).
     - This value will be used to update the **PC** when **JAL** is executed.

   - **Step 3**: Implement the computation for **JALR**:
     - Add the **source 1 value** (from the bypass mux) to the immediate value to compute the new PC for JALR.
     - Ensure this is handled in **stage 3**, as it depends on the **source 1** value.

   - **Step 4**: Select the appropriate PC for **JAL** and **JALR**:
     - For **JAL**, you can reuse the same logic as branches to update the PC.
     - For **JALR**, introduce a new path to handle the **JALR target PC**.

5. **Final Considerations**:
   - Ensure proper selection of the PC for each jump type.
   - Handle the jump instruction by creating invalid cycles after execution, similar to previous operations.
   - Once implemented, test to confirm the correct jump behavior, ensuring the program counter updates as expected.

