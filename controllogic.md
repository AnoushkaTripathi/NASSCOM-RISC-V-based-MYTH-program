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

