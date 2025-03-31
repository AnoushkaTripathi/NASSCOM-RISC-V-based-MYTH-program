# Fetch & Decode 


### **1. Building the CPU Framework**
- You're working within a **framework** that will guide you through the process of constructing a CPU.
- The CPU is built in **sequential steps**, from left to right in terms of operation.
  
### **2. Steps to Implement Each CPU Component**
As you progress through the construction, you will add the following components one by one:

#### **A. Next PC Logic**
- This is the first step, where the logic for generating the **next Program Counter (PC)** is defined.
- Initially, assume that instructions are executed **sequentially**, meaning the PC will simply **increment** after each instruction.
- You’ll deal with branch instructions later, which will modify the PC by an **offset**.

#### **B. Instruction Fetch**
- You will **fetch the instruction** from the instruction memory using the current PC value.
  
#### **C. Decode**
- The fetched instruction will then be **decoded** to determine what operation the CPU should perform.
  
#### **D. Register File Read**
- The CPU reads values from the **register file**, which are needed for the execution of the decoded instruction.
  
#### **E. Arithmetic Logic Unit (ALU)**
- The **ALU** performs the arithmetic or logical operation specified by the decoded instruction.
  
#### **F. Register File Write**
- The result of the ALU operation is **written back** to the register file.
  
#### **G. Branch Target Computation**
- Finally, you’ll add logic to handle **branch instructions**, computing the correct branch target and updating the PC accordingly.

### **3. Debugging and Simulation**
- After implementing each component, you will check its outputs in **simulation** to ensure correctness.
  
### **4. Implementing Next PC Logic**
The first component to implement is the **Next PC Logic**.

#### **A. Incrementing the PC**
- The PC should increment **sequentially** to point to the next instruction in memory.
- Each instruction is **32 bits** wide, but since the PC is **byte-addressed**, you increment the PC by **4** for each instruction.
  
#### **B. Handling the First Instruction**
- You need to ensure that the first instruction executed after **reset** is the instruction at **PC=0**.
  
#### **C. Potential Issue with Reset**
- A potential issue arises: if you simply code the PC to reset to zero, when the CPU moves to the next instruction after reset, it will compute the new PC as **0 + 1**.
  - This would cause the CPU to fetch **instruction 1** instead of **instruction 0**.
  
#### **D. Correcting the Reset Behavior**
- To correct this, you can use the **reset signal** from the **previous instruction**. This means the first non-reset instruction will use **PC=0**, ensuring the correct instruction is fetched.
  
#### **E. Implementation Strategy**
- Instead of directly using the reset signal, modify the logic to account for the **previous reset** status in the **multiplexer (MUX)** that selects the next PC value.
  
### **5. Testing in Simulation**
- Once you've implemented the **Next PC Logic**, check the results in the **waveform** during simulation to confirm that:
  - The PC starts at **0** for the first non-reset instruction.
  - The PC correctly increments by **4** for each subsequent instruction.

#### **Summary of Key Points:**
- **Next PC Logic** is responsible for calculating the new PC value after each instruction.
- The PC is incremented by **4** to move from one 32-bit instruction to the next.
- Special care must be taken to ensure that the **first instruction** after reset is **PC=0**.
- You will simulate each piece after implementing it to verify its correctness, particularly making sure that the PC behaves as expected in the **waveform**.

# **Fetch Logic** in the CPU:

### **1. Overview**
- After successfully implementing the **PC logic**, the next step is to work on the **instruction fetch** logic.
- The **instruction memory** contains the test program, which you previously developed, and will now be used to fetch instructions based on the current PC.

### **2. Instruction Memory Instantiation**
- The shell you’re working with already provides an **instruction memory instantiation**.
- This memory is structured to hold the **test program** that you're using in this lab exercise.
- To use it, you will need to **uncomment** the relevant line towards the bottom of the shell.
  
### **3. CPU Visualization**
- You should also uncomment the **CPU visualization**, which is based on specific signals related to the CPU pipeline.
- The **visualization** will help you visualize the CPU behavior as you step through the program.

### **4. Compilation and Initial Log Check**
- Initially, compile the code even if you haven’t fully connected the required signals.
- When you compile, there will be **errors** in the log regarding **unassigned signals**. This is because the **instruction memory** requires specific signals to function correctly, and those signals need to be **hooked up**.

#### **Log Errors**
- The errors will tell you which signals the instruction memory needs but haven’t yet been provided. Examples include:
  - Signals that are **consumed but not driven**.
  - Signals that are **produced but not consumed**.

### **5. Hooking Up Instruction Memory Signals**
- Now that you have the errors, the next step is to **provide the necessary signals** to the instruction memory.
  
#### **Key Signals:**
- **Immediate Enable (imem_en):**
  - Enable signal for instruction memory.
- **Instruction Memory Address (imem_addr):**
  - This address is derived from the **PC**.
  - Since the PC is byte-addressed but instructions are 32 bits, you need to align the PC to the instruction boundary. This means:
    - The lower **two bits** of the PC should be **0**.
    - Effectively, you're dividing the PC by 4 to get the proper instruction index.
- **Read Data (imem_read_data):**
  - This is the **instruction** fetched from the memory.
  - You'll need to hook up the output from the memory to the **instruction signal**, which will represent the instruction being fetched.

#### **Addressing:**
- You need to use an **address expression** to map the PC to the memory’s address space. Use the constant provided in the code to index the instructions properly.
- The **address signal** for instruction memory should be **PC / 4** (since each instruction is 4 bytes long and the PC is byte-addressed).

### **6. Fetching Instructions**
- The instruction memory will output the fetched instruction via the **imem_read_data** signal.
- You need to **consume this signal** by connecting it to an **instruction signal** in your CPU.
- Rename this fetched signal appropriately (e.g., `instruction`) and use it to represent the instruction being processed by the CPU.

### **7. Testing and Simulation**
- Once all the necessary signals are hooked up, recompile and run the **simulation**.
- Check the **log** again to ensure there are no errors, and verify that everything is properly connected.
- In the **visualization**, you should now see the **PC** incrementing as instructions are fetched.
  
### **8. Expected Output**
- At this stage, you should see the **PC** increment properly during simulation.
- Depending on what default logic you have, the **instructions themselves** may not be visible yet. This will be addressed when you add **default decode logic** in the next step.
---
  # Labs Fetch Logic
![image](https://github.com/user-attachments/assets/47370ade-9759-4c96-af4f-ca8bf18d678c)
## Solution
```
      @0
         $reset = *reset;
         
         
         $pc[31:0] = (>>1$reset) ? 32'd0 : (>>1$pc + 32'd4);
         //curr pc = (is prev reset value true) ? if yes, curr pc = 0 : if not, curr pc = prev pc + 4 (4 locations = 1 instr)
         
      @1
         $imem_rd_en = ! $reset;  //active low reset - it considers the prev reset value
         
         //input the address - its not pc directly since pc is always a multiple of 4
         //$imem_rd_addr[7:0] = $pc; //gives wrong ans - in instr mem - every index is considered 4 bytes (not 1 byte) so divide pc by 4 necessary
         $imem_rd_addr[7:0] = $pc / 4 ; 
         //after observing the waveform: it seems the addr in the mem = 8 only
         //so as pc = 8*4 = 32 --> instr extracted is the first instr cuz : (32/4)mod8 = 0 (mod8 since only 8 instrs)
         
         //get the instruction from the instr mem
         $instr[31:0] = $imem_rd_data[31:0];
```
![image](https://github.com/user-attachments/assets/f33a0e5f-cb0a-4d10-94f6-4b5ef2ed161e)

### **Summary of Key Steps:**
- **Uncomment** the instruction memory instantiation and CPU visualization.
- **Compile** the code to see what errors arise related to missing signal hookups.
- **Hook up** the required signals for instruction memory, including:
  - **Enable signal**.
  - **Memory address** (derived from PC, divided by 4).
  - **Read data** (connect to instruction signal).
- **Test and simulate** to confirm that the **PC increments** and the necessary signals are properly assigned.
- Continue developing the default logic for instructions in the next steps.
Here are the detailed notes on implementing the **Decode Logic** for the RISC-V instruction set in the CPU:
# Decode Logic
### **1. Overview**
- The next step in building the CPU is implementing the **decode logic**, which interprets the bits of the fetched instruction.
- The goal of the decode stage is to determine the type of instruction and break it into its component fields based on the **RISC-V ISA** (Instruction Set Architecture).
![image](https://github.com/user-attachments/assets/e1b09f0e-519e-4811-a35e-d0bf06dee6bd)

### **2. Instruction Types in RISC-V**
- **RISC-V** defines several types of instructions, such as **R-type, I-type, S-type**, and more.
- Each type of instruction has a different format, meaning the bits are split into fields differently (e.g., opcode, funct3, funct7, etc.).
- The first step in decoding is identifying **which type** of instruction is being fetched.

### **3. Opcode Field**
- The **opcode field** is located in bits **6 down to 0** (7 bits total) in a RISC-V instruction.
- However, for the **base RISC-V instruction set**, the lower two bits (bits **1 and 0**) are always **'1'**. Therefore, we can ignore these two bits.
- This leaves us with bits **6 to 2** as the key part of the **opcode** that helps determine the type of instruction.

### **4. Instruction Type Decoding**
- The instruction types are identified based on the bits in positions **6 to 2**.
- You will write logic that decodes these bits to figure out which type of instruction you are dealing with.

#### **Logic for Instruction Type**
- You'll define a bunch of signals, each representing a specific instruction type. For example:
  - **is_I_type**: Signal to indicate if the instruction is an I-type instruction.
  - **is_R_type**: Signal to indicate if the instruction is an R-type instruction.
  - **is_S_type**: Signal to indicate if the instruction is an S-type instruction.

### **5. Table of Opcodes**
- Based on the RISC-V ISA, instructions fall into certain categories based on their **opcode** bits.
- You’ll use these **bit patterns** to determine which instruction type is being decoded.

### **6. Decoding the Opcode**
- The **comparison logic** uses an operator (`==?`) that allows for comparisons involving **don’t care** bits (bits that can be either 0 or 1).
- For example, if you’re checking whether the instruction is of type I, you’ll compare the relevant bits (6 to 2) with patterns like `00000?` or `00100?`, where `?` indicates a **don’t care** bit.

#### **Example for I-type Instruction:**
- If bits **6 to 2** are `00000` (with don’t care for the last bit), the instruction is an **I-type**.
- You’ll express this in your decode logic like:
  ```verilog
  is_I_type = (opcode[6:2] == 5'b00000?);
  ```
  
### **7. Handling Don’t Care Bits**
- The **don't care** bits (`?`) in the opcode comparison allow flexibility in the decode logic, where certain bits are ignored for specific instruction types.
- For example, in the **RISC-V** instruction format, sometimes the last bit (bit 2) in the opcode doesn’t affect the instruction type. You can use the `==?` operator to account for this.

### **8. Example of Signal Definitions:**
- Here’s how you would start to define the signals for the instruction types:
  - **is_I_type**: Decodes based on specific opcode bit patterns.
  - **is_R_type**: Similarly, decodes based on different bit patterns.
  
  Example Verilog code for I-type decoding:
  ```verilog
  is_I_type = (opcode[6:2] == 5'b00000?) || (opcode[6:2] == 5'b00100?);
  ```
- Continue adding similar logic for other instruction types (R-type, S-type, etc.).

### **9. Handling Unused Instruction Buckets**
- Some instruction categories, such as certain types of I-type instructions, may not be used in the current test program.
- You can safely **ignore** decoding logic for those instruction types, but still add the decoding for completeness.

### **10. Testing and Simulation**
- After implementing the decode logic, run your simulation to **verify** that the instruction types are being correctly decoded.
- Focus on the instructions in your **test program** to ensure the decoding matches the expected values.

### **11. Debugging in Simulation**
- Use **waveforms** during simulation to monitor:
  - The **opcode** field of each instruction.
  - The output of the decoding signals (e.g., **is_I_type, is_R_type**) to ensure they are correctly identifying the instruction type.
- Ensure that for each instruction, the correct type signal is **asserted**.

### **Summary of Key Steps:**
- Decode the **opcode** (bits 6 to 2) to determine the instruction type.
- Use the `==?` operator to compare opcode bits while handling don’t care conditions.
- Define signals for each instruction type (**is_I_type, is_R_type, etc.**).
- Test the logic in simulation and verify correct decoding of instruction types based on the **waveform output**.
- Ignore instruction types not needed for the current test program but include their logic for future completeness.


 # **Decoding the immediate fields** for different instruction types in the RISC-V ISA:

### **1. Overview**
- The next step is decoding the **immediate value** for instructions that require one. 
- Based on the **instruction type** (I-type, S-type, etc.), the immediate value is located in different parts of the instruction.
- For each instruction type, we need to **extract and construct** a **32-bit immediate value**.

### **2. Immediate Fields in RISC-V**
- Not all instructions in the RISC-V ISA use an immediate value, but for those that do, the **immediate value** is derived from specific bits in the instruction.
- Each **instruction type** has a different way of encoding the immediate. The **RISC-V spec** provides a table that defines how these immediate bits are extracted for each type.

### **3. Immediate Construction for Each Instruction Type**
- The immediate value needs to be a **32-bit** value for each instruction.
- The immediate is constructed by taking specific bits from the instruction and then sign-extending them to form a full 32-bit value.
- **Concatenation** and **bit replication** are used to build the immediate value from smaller sections of the instruction.

### **4. Example: I-Type Instruction**
- For **I-type** instructions, the immediate is derived from the instruction bits:
  - **Bits 31 to 20**: Form the immediate value.
  - **Bit 31** is the sign bit and needs to be **sign-extended** to 32 bits.
  
#### **Verilog Syntax for Immediate Construction**
- **Concatenation** is used to combine multiple bits into one value.
- **Bit replication** is used to extend the sign bit to fill the upper bits of the immediate.
  
  Example for **I-type**:
  ```verilog
  immediate_value = { {21{instruction[31]}}, instruction[30:20] };
  ```
  - **{21{instruction[31]}}**: This replicates **bit 31** (the sign bit) **21 times** to fill the upper 21 bits of the 32-bit immediate.
  - **instruction[30:20]**: This takes bits **30 to 20** as the remaining lower bits of the immediate value.

### **5. Immediate Fields for Other Instruction Types**
- For each type of instruction (I-type, S-type, B-type, U-type, J-type), the immediate value is constructed differently based on which bits hold the immediate.
- You will define a similar **concatenation and bit replication** logic for each of these instruction types:
  
#### **S-type Example**:
- For S-type, the immediate comes from two separate parts of the instruction:
  - **Bits 31 to 25**: Upper bits of the immediate.
  - **Bits 11 to 7**: Lower bits of the immediate.
  
  Example construction:
  ```verilog
  immediate_value = { {21{instruction[31]}}, instruction[30:25], instruction[11:7] };
  ```

#### **B-type Example**:
- In **B-type** instructions (branch instructions), the immediate is constructed from various scattered bits in the instruction:
  - **Bits 31, 30 to 25, 11, and 7** are used.
  
  Example:
  ```verilog
  immediate_value = { {20{instruction[31]}}, instruction[7], instruction[30:25], instruction[11:8], 1'b0 };
  ```

- Notice the addition of **1'b0** to make the immediate a **multiple of 2**, as branch offsets are word-aligned.

### **6. Complete Immediate Logic**
- For each instruction type, define the **logic** to pull the appropriate bits from the instruction and concatenate them to form a 32-bit immediate value.
- Make sure to handle **sign extension** properly to ensure that the immediate value is correctly interpreted for negative values.

### **7. Testing and Simulation**
- After implementing the immediate decode logic, simulate the CPU to verify that the immediate values are being decoded correctly.
- In the waveform, check the value of the immediate field for instructions that use it, such as I-type or S-type instructions.
- Verify that the immediate value is correctly

# **Extracting other fields of the instruction** in RISC-V ISA**

### **1. Overview**
- After decoding the **immediate fields**, the next step is to extract the **other fields** of the instruction, such as register source (rs1, rs2), destination register (rd), and function codes (funct3, funct7).
- These fields are located at fixed positions within the instruction, regardless of the instruction type (with the exception of the immediate field, which we have already handled).
  
### **2. Field Locations in RISC-V Instructions**
- Each RISC-V instruction type has several fields that are relevant for decoding the instruction. These fields include:
  - **rd (Destination Register)**: This is the register where the result of the instruction is written.
  - **rs1, rs2 (Source Registers)**: These are the registers that provide operands for the instruction.
  - **funct3, funct7 (Function Codes)**: These codes determine the specific operation (like add, subtract, etc.) for certain instructions.
  
- **RISC-V ISA** defines the positions of these fields consistently across instruction types, except for the immediate.

### **3. Consistent Extraction of Fields**
- One key observation is that the fields (rd, rs1, rs2, funct3, funct7) always appear at the **same bit positions** in the instruction, regardless of the instruction type. This makes it straightforward to extract them without needing any conditional logic.

### **4. Extracting Fields**
For each field, you need to create expressions to pull out the bits from the instruction as per the RISC-V encoding:

#### **a. rs1 (Source Register 1)**
- **Bits 19 to 15** contain the value of the **rs1** field.
  
  Example:
  ```verilog
  rs1 = instruction[19:15];
  ```

#### **b. rs2 (Source Register 2)**
- **Bits 24 to 20** contain the value of the **rs2** field.
  
  Example:
  ```verilog
  rs2 = instruction[24:20];
  ```

#### **c. rd (Destination Register)**
- **Bits 11 to 7** contain the value of the **rd** field.
  
  Example:
  ```verilog
  rd = instruction[11:7];
  ```

#### **d. funct3 (Function Code 3)**
- **Bits 14 to 12** contain the **funct3** field, which determines the specific operation within a group of instructions (e.g., add vs. subtract).

  Example:
  ```verilog
  funct3 = instruction[14:12];
  ```

#### **e. funct7 (Function Code 7)**
- **Bits 31 to 25** contain the **funct7** field, which further specifies the operation for certain instructions (e.g., determining whether an addition is signed or unsigned).
  
  Example:
  ```verilog
  funct7 = instruction[31:25];
  ```
![image](https://github.com/user-attachments/assets/b33ac6c4-c76e-44fd-b3a5-baa2af61096e)

### **5. Field Extraction Table (for Reference)**
| **Field**  | **Bit Range**  | **Example Verilog Expression**                  |
|------------|----------------|-------------------------------------------------|
| **rs1**    | 19:15          | `rs1 = instruction[19:15];`                     |
| **rs2**    | 24:20          | `rs2 = instruction[24:20];`                     |
| **rd**     | 11:7           | `rd = instruction[11:7];`                       |
| **funct3** | 14:12          | `funct3 = instruction[14:12];`                  |
| **funct7** | 31:25          | `funct7 = instruction[31:25];`                  |

### **6. Implementation Steps**
1. **Identify the relevant fields** in the instruction based on the RISC-V instruction format.
2. **Extract the bits** from the instruction using bit slicing expressions as defined above.
3. These extracted signals can now be used for further decoding and execution logic in the CPU, such as determining which registers to read, which operation to perform, and where to store the result.

### **7. Testing and Verification**
- After extracting these fields, verify their correctness in **simulation**:
  - Check that the correct values of **rs1**, **rs2**, and **rd** are being extracted for the instructions in the test program.
  - Ensure that the **funct3** and **funct7** values match the expected operation codes for each instruction.
  
- By checking the waveform in simulation, ensure that the **extracted values** match the expected register and function code values for various instructions.

### **Key Points to Remember:**
- The fields like **rs1, rs2, rd, funct3, funct7** are always in **fixed positions** in the instruction and can be extracted directly without conditional logic.
- Use simple **bit slicing** to pull out these fields, and then use them in subsequent steps of the decode logic.
- Verify the extraction logic in simulation to ensure it works correctly with your test program.

##  Improving Field Definitions with Conditional Validity

#### **1. Background**
- In the previous step, we extracted fields like **rs1**, **rs2**, **rd**, **funct3**, and **funct7** from the instruction, assuming these fields were always present in every instruction. However, not all fields are used by every instruction type in the RISC-V ISA.
  
- Now, we will improve the logic by introducing **conditional validity**. We will define these fields only when the corresponding instruction type supports them.

#### **2. Conditional Field Validity**
- Each instruction type (R-type, I-type, S-type, B-type, etc.) uses different fields. For example:
  - The **rs2** field is **valid** for **R-type, S-type, and B-type** instructions, but **invalid** for other types like I-type or J-type.
  - Similarly, fields like **funct7** are used in some instructions (R-type) but not in others (I-type, S-type, etc.).

#### **3. Defining Validity for Fields**
- To improve the logic, we first create a signal to indicate whether a field is **valid** for a given instruction type.
  
- For instance, to conditionally define **rs2**, we first check if the instruction type is **R-type**, **S-type**, or **B-type**:
  
  Example:
  ```verilog
  wire rs2_valid = (is_r_type || is_s_type || is_b_type);
  ```

- Then, we **conditionally assign** the value of **rs2** only if the instruction is one of these types:
  
  Example:
  ```verilog
  rs2 = rs2_valid ? instruction[24:20] : 5'b0;
  ```

#### **4. Applying Conditions to All Fields**
- You should apply the same logic to other fields. For each field, create a validity expression based on the instruction type and conditionally assign the field value only when the instruction uses that field.

### **Field-Wise Examples**

#### **a. rs2 (Source Register 2)**
- **Validity**: `rs2` is valid for **R-type**, **S-type**, and **B-type** instructions.
  
  Expression:
  ```verilog
  wire rs2_valid = (is_r_type || is_s_type || is_b_type);
  rs2 = rs2_valid ? instruction[24:20] : 5'b0;  // Assign rs2 if valid, else assign 0
  ```

#### **b. rs1 (Source Register 1)**
- **Validity**: `rs1` is valid for **most instruction types** including **R-type**, **I-type**, **S-type**, **B-type**, etc.
  
  Expression:
  ```verilog
  wire rs1_valid = (is_r_type || is_i_type || is_s_type || is_b_type || is_u_type);
  rs1 = rs1_valid ? instruction[19:15] : 5'b0;
  ```

#### **c. rd (Destination Register)**
- **Validity**: `rd` is valid for **R-type**, **I-type**, **U-type**, and **J-type** instructions.
  
  Expression:
  ```verilog
  wire rd_valid = (is_r_type || is_i_type || is_u_type || is_j_type);
  rd = rd_valid ? instruction[11:7] : 5'b0;
  ```

#### **d. funct3 (Function Code 3)**
- **Validity**: `funct3` is valid for instructions that perform arithmetic or control operations (e.g., **R-type**, **I-type**, **B-type**, etc.).
  
  Expression:
  ```verilog
  wire funct3_valid = (is_r_type || is_i_type || is_b_type);
  funct3 = funct3_valid ? instruction[14:12] : 3'b000;
  ```

#### **e. funct7 (Function Code 7)**
- **Validity**: `funct7` is only valid for **R-type** instructions that specify additional operation details.
  
  Expression:
  ```verilog
  wire funct7_valid = (is_r_type);
  funct7 = funct7_valid ? instruction[31:25] : 7'b0000000;
  ```

#### **6. Testing and Simulation**
- After making these changes, you should simulate your design again, paying close attention to the **waveform**:
  - Verify that the fields (**rs1, rs2, rd, funct3, funct7**) are **only defined** when the instruction type makes them valid.
  - Ensure that the waveform shows these fields as being zero or undefined when they are not valid for the instruction being executed.

#### **7. Summary**
- **Conditional Validity** ensures that fields like **rs2** and **funct7** are only defined for instructions that actually use them. This reduces the chances of unintended behavior from fields that shouldn't be used for certain instruction types.
- By implementing this logic, you ensure cleaner, more accurate decoding of instructions in the RISC-V CPU design.

## **Explanation: Decoding Individual Instructions**

#### **1. Introduction**
- Now that we've set up the basic instruction decode mechanism, the next step is to actually **decode individual instructions** based on the test program.
- We'll focus on decoding the instructions relevant to our **test program**, which include:
  - **ADD** (Addition)
  - **ADDI** (Addition with immediate)
  - **Branch if Less Than (BLT)**

  We'll also extend this to handle **all branch instructions** while we're at it, though we'll focus primarily on those circled in red in the instructions.

#### **2. The Approach**
- We’ll decode each instruction by **matching specific bit patterns** of the instruction against the **opcode**, **funct3**, and **funct7** fields.
  
- These fields collectively specify the exact instruction being decoded:
  - **Opcode**: Specifies the type of instruction (e.g., arithmetic, load, store, branch).
  - **funct3**: Provides further classification within the instruction type.
  - **funct7**: Provides additional operation details (used for certain instructions like R-type).

#### **3. The Decode Process**
- We begin by **collecting** the relevant instruction fields (opcode, funct3, funct7) into a single bit vector called `decode_bits`.
  
  Example:
  ```verilog
  wire [14:0] decode_bits = {funct7, funct3, opcode};
  ```

- After this, we compare this **`decode_bits`** vector against specific bit patterns that correspond to individual instructions.

#### **4. Example: Decoding Branch Equal (BEQ)**
- The **BEQ (Branch if Equal)** instruction is a branch instruction.
  
  The decode logic checks if `decode_bits` matches the BEQ pattern:
  
  Example:
  ```verilog
  wire is_beq = (decode_bits == 15'bxxxxxxxx_000_1100011);  // BEQ pattern
  ```

  In this case:
  - The **funct7** part of the pattern is irrelevant (denoted by `x` for don't care).
  - **funct3 = 000** corresponds to BEQ.
  - **opcode = 1100011** specifies that this is a branch instruction.

#### **5. Handling Other Instructions**
- Now that you've seen how to decode BEQ, you can apply the same technique to other instructions (like ADD, ADDI, and BLT).
  
  Example for **ADD**:
  ```verilog
  wire is_add = (decode_bits == 15'b0000000_000_0110011);  // ADD pattern
  ```

  Example for **ADDI**:
  ```verilog
  wire is_addi = (decode_bits == 15'bxxxxxxxx_000_0010011);  // ADDI pattern
  ```

  Example for **Branch if Less Than (BLT)**:
  ```verilog
  wire is_blt = (decode_bits == 15'bxxxxxxxx_100_1100011);  // BLT pattern
  ```

#### **6. Using SystemVerilog’s `==?` Operator**
- The **`==?` operator** is a SystemVerilog operator that allows us to compare values with **don't care bits (`x`)**. This is useful when parts of the instruction, such as `funct7`, aren’t relevant for certain instructions.
  
  - The **`x`** in the bit pattern tells the comparison to ignore that bit.

  Example:
  ```verilog
  wire is_beq = (decode_bits == 15'bxxxxxxxx_000_1100011);  // BEQ pattern, ignore funct7
  ```

#### **7. The `bogus_use` Macro**
- As we assign many signals (like `is_beq`, `is_add`, etc.), some of these signals may not be used immediately. Normally, the tool (e.g., SandPiper) would raise warnings about unused signals.

- To avoid these warnings and clean up the log, we use the **`bogus_use` macro**. This is a **SystemVerilog macro** that allows us to tell the tool to ignore these unused signals, preventing clutter in the log file.

  Example:
  ```verilog
  `bogus_use(is_add, is_addi, is_beq, is_blt);
  ```

  - This macro evaluates to **nothing** in the final code, but it satisfies the tool by indicating that the signals are being "used."

#### **8. Summary of the Process**
- **Step 1**: Define `decode_bits` as a concatenation of `funct7`, `funct3`, and `opcode`.
- **Step 2**: Use the `==?` operator to compare `decode_bits` with known patterns for each instruction.
- **Step 3**: Use the `bogus_use` macro to avoid cluttering the log with warnings about unused signals.

#### **9. Next Steps**
- After you’ve coded up all the necessary branches and arithmetic instructions, run the simulation.
  - Check that each instruction is correctly decoded by inspecting the waveforms.
  - Focus on ensuring that the **correct instruction is decoded** based on the bit patterns of `opcode`, `funct3`, and `funct7`.
# Labs Decode logic
![image](https://github.com/user-attachments/assets/c28db0d0-c355-4405-934f-6781ded41701)
## Solution
```
 @0
         $reset = *reset;
         
         
         $pc[31:0] = (>>1$reset) ? 32'd0 : (>>1$pc + 32'd4);
         //curr pc = (is prev reset value true) ? if yes, curr pc = 0 : if not, curr pc = prev pc + 4 (4 locations = 1 instr)
         
      @1
         $imem_rd_en = ! $reset;  //active low reset - it considers the prev reset value
         
         //input the address - its not pc directly since pc is always a multiple of 4
         //$imem_rd_addr[7:0] = $pc; //gives wrong ans - in instr mem - every index is considered 4 bytes (not 1 byte) so divide pc by 4 necessary
         $imem_rd_addr[7:0] = $pc / 4 ; 
         //after observing the waveform: it seems the addr in the mem = 8 only
         //so as pc = 8*4 = 32 --> instr extracted is the first instr cuz : (32/4)mod8 = 0 (mod8 since only 8 instrs)
         
         //get the instruction from the instr mem
         $instr[31:0] = $imem_rd_data[31:0]; 
         
         //opcode in any instr format is 7 bits long but we consider only 5 bits
         //the part of instr that we need to decode the instr format = instr[6:2]
         //instr[1:0] is always 11 for base instr set, hence ignored
         
         $is_i_instr = $instr[6:2] ==? 5'b0000x || //this takes care of 5'b00000 and 5'b00001
                       $instr[6:2] ==? 5'b001x0 || //5'b00100 and 5'b00110
                       $instr[6:2] ==? 5'b11001 ||
                       $instr[6:2] ==? 5'b00100 ; 
         
         $is_r_instr = $instr[6:2] ==? 5'b01011 ||
                       $instr[6:2] ==? 5'b011x0 || //5'b01100 and 5'b01110
                       $instr[6:2] ==? 5'b10100 ;
         
         $is_s_instr = $instr[6:2] ==? 5'b0100x ; //5'b01000 and 5'b01001
         
         $is_b_instr = $instr[6:2] ==? 5'b11000 ;
         
         $is_j_instr = $instr[6:2] ==? 5'b11011 ;
         
         $is_u_instr = $instr[6:2] ==? 5'b0x101 ; //5'b00101 and 5'b01101
```
![image](https://github.com/user-attachments/assets/86476076-f057-4b36-bcce-d729ef40cb54)
# Lab 2
![image](https://github.com/user-attachments/assets/81ed70be-063c-4ddb-a660-d7dd300bd8b4)
## Solution
![image](https://github.com/user-attachments/assets/0e84f0ea-9199-4707-8ab3-c75fd05f408f)
```
 @0
         $reset = *reset;
         
         
         $pc[31:0] = (>>1$reset) ? 32'd0 : (>>1$pc + 32'd4);
         //curr pc = (is prev reset value true) ? if yes, curr pc = 0 : if not, curr pc = prev pc + 4 (4 locations = 1 instr)
         
      @1
         $imem_rd_en = ! $reset;  //active low reset - it considers the prev reset value
         
         //input the address - its not pc directly since pc is always a multiple of 4
         //$imem_rd_addr[7:0] = $pc; //gives wrong ans - in instr mem - every index is considered 4 bytes (not 1 byte) so divide pc by 4 necessary
         $imem_rd_addr[7:0] = $pc / 4 ; 
         //after observing the waveform: it seems the addr in the mem = 8 only
         //so as pc = 8*4 = 32 --> instr extracted is the first instr cuz : (32/4)mod8 = 0 (mod8 since only 8 instrs)
         
         //get the instruction from the instr mem
         $instr[31:0] = $imem_rd_data[31:0]; 
         
         //opcode in any instr format is 7 bits long but we consider only 5 bits
         //the part of instr that we need to decode the instr format = instr[6:2]
         //instr[1:0] is always 11 for base instr set, hence ignored
         
         $is_i_instr = $instr[6:2] ==? 5'b0000x || //this takes care of 5'b00000 and 5'b00001
                       $instr[6:2] ==? 5'b001x0 || //5'b00100 and 5'b00110
                       $instr[6:2] ==? 5'b11001 ||
                       $instr[6:2] ==? 5'b00100 ; 
         
         $is_r_instr = $instr[6:2] ==? 5'b01011 ||
                       $instr[6:2] ==? 5'b011x0 || //5'b01100 and 5'b01110
                       $instr[6:2] ==? 5'b10100 ;
         
         $is_s_instr = $instr[6:2] ==? 5'b0100x ; //5'b01000 and 5'b01001
         
         $is_b_instr = $instr[6:2] ==? 5'b11000 ;
         
         $is_j_instr = $instr[6:2] ==? 5'b11011 ;
         
         $is_u_instr = $instr[6:2] ==? 5'b0x101 ; //5'b00101 and 5'b01101
         
         //getting the immediate values and sign extending them
         //Eg: {21{$instr[31]} , $instr[30:20]} --> 12 bit imm, msb = instr[31] -> sign extend = {21{msb}}
         //instr[30:20] = give lower 11 (30-20+1) bits
         //r instr doesn't have any imm values
         $imm[31:0] = $is_i_instr ? { {21{$instr[31]}}, $instr[30:20] } : //12 bits for i type - immediate is one of the operands
                      $is_s_instr ? { {21{$instr[31]}}, $instr[30:25], $instr[11:7] } : //12 bits for s type - imm gives offset for calc effective address to store value
                      $is_b_instr ? { {20{$instr[31]}}, $instr[7], $instr[31:25], $instr[11:8], 1'b0 } : //13 bits for b type - imm gives pc relative offset to branch required label/instr
                      $is_u_instr ? { $instr[31:12] , 12'b0 } : //20 bits for u type - imm gives upper 20 bits of a 32 bit value 
                      $is_j_instr ? { {12{$instr[31]}}, $instr[19:12], $instr[20], $instr[30:21], 1'b0 } : //21 bits for j type - imm gives the pc relative addr to jump to 
                      32'b0 ; //if none of the instrs mentioned (if rtype), then imm is 0 - if this condition isn't there - ternary op is not complete - throws error
```
![image](https://github.com/user-attachments/assets/3b34aa71-a98a-4b81-a1f1-e9ac927674b9)

![image](https://github.com/user-attachments/assets/42595953-f6eb-4c3f-bd1a-463ebe3da438)
# Lab
![image](https://github.com/user-attachments/assets/26427d83-c9d9-48d6-840f-1b844d4adc17)
## Solution
![image](https://github.com/user-attachments/assets/2177da3b-bdc1-4121-b508-c52a0cfef471)
```
 @0
         $reset = *reset;
         
         
         $pc[31:0] = (>>1$reset) ? 32'd0 : (>>1$pc + 32'd4);
         //curr pc = (is prev reset value true) ? if yes, curr pc = 0 : if not, curr pc = prev pc + 4 (4 locations = 1 instr)
         
      @1
         $imem_rd_en = ! $reset;  //active low reset - it considers the prev reset value
         
         //input the address - its not pc directly since pc is always a multiple of 4
         //$imem_rd_addr[7:0] = $pc; //gives wrong ans - in instr mem - every index is considered 4 bytes (not 1 byte) so divide pc by 4 necessary
         $imem_rd_addr[7:0] = $pc / 4 ; 
         //after observing the waveform: it seems the addr in the mem = 8 only
         //so as pc = 8*4 = 32 --> instr extracted is the first instr cuz : (32/4)mod8 = 0 (mod8 since only 8 instrs)
         
         //get the instruction from the instr mem
         $instr[31:0] = $imem_rd_data[31:0]; 
         
         //opcode in any instr format is 7 bits long but we consider only 5 bits
         //the part of instr that we need to decode the instr format = instr[6:2]
         //instr[1:0] is always 11 for base instr set, hence ignored
         
         $is_i_instr = $instr[6:2] ==? 5'b0000x || //this takes care of 5'b00000 and 5'b00001
                       $instr[6:2] ==? 5'b001x0 || //5'b00100 and 5'b00110
                       $instr[6:2] ==? 5'b11001 ||
                       $instr[6:2] ==? 5'b00100 ; 
         
         $is_r_instr = $instr[6:2] ==? 5'b01011 ||
                       $instr[6:2] ==? 5'b011x0 || //5'b01100 and 5'b01110
                       $instr[6:2] ==? 5'b10100 ;
         
         $is_s_instr = $instr[6:2] ==? 5'b0100x ; //5'b01000 and 5'b01001
         
         $is_b_instr = $instr[6:2] ==? 5'b11000 ;
         
         $is_j_instr = $instr[6:2] ==? 5'b11011 ;
         
         $is_u_instr = $instr[6:2] ==? 5'b0x101 ; //5'b00101 and 5'b01101
         
         //getting the immediate values and sign extending them
         //Eg: {21{$instr[31]} , $instr[30:20]} --> 12 bit imm, msb = instr[31] -> sign extend = {21{msb}}
         //instr[30:20] = give lower 11 (30-20+1) bits
         //r instr doesn't have any imm values
         $imm[31:0] = $is_i_instr ? { {21{$instr[31]}}, $instr[30:20] } : //12 bits for i type - immediate is one of the operands
                      $is_s_instr ? { {21{$instr[31]}}, $instr[30:25], $instr[11:7] } : //12 bits for s type - imm gives offset for calc effective address to store value
                      $is_b_instr ? { {20{$instr[31]}}, $instr[7], $instr[31:25], $instr[11:8], 1'b0 } : //13 bits for b type - imm gives pc relative offset to branch required label/instr
                      $is_u_instr ? { $instr[31:12] , 12'b0 } : //20 bits for u type - imm gives upper 20 bits of a 32 bit value 
                      $is_j_instr ? { {12{$instr[31]}}, $instr[19:12], $instr[20], $instr[30:21], 1'b0 } : //21 bits for j type - imm gives the pc relative addr to jump to 
                      32'b0 ; //if none of the instrs mentioned (if rtype), then imm is 0 - if this condition isn't there - ternary op is not complete - throws error
         
         //extracting other fields of instr
         //risc v is a regular ISA, which means most fields are consistent in their bit positions for all instr types except for immediate field
         //hence other fields of instr don't need to type based conditions
         $funct7[6:0] = $instr[31:25]; 
         $funct3[2:0] = $instr[14:12];
         $rs1[4:0] = $instr[19:15]; //source reg 1 - only 32 reg in total, so 5 bits to represent them
         $rs2[4:0] = $instr[24:20]; //source reg 2
         $rd[4:0] = $instr[11:7]; //destination reg
         $opcode[6:0] = $instr[6:0]; //opcode required to instruct the alu
```
# PC + instr fetch + decode unit (with validity check)
## Solution
![image](https://github.com/user-attachments/assets/ca99e5a1-f1e8-4429-9ae0-6a5e68ea2c1b)
```
 @0
         $reset = *reset;
         
         
         $pc[31:0] = (>>1$reset) ? 32'd0 : (>>1$pc + 32'd4);
         //curr pc = (is prev reset value true) ? if yes, curr pc = 0 : if not, curr pc = prev pc + 4 (4 locations = 1 instr)
         
      @1
         $imem_rd_en = ! $reset;  //active low reset - it considers the prev reset value
         
         //input the address - its not pc directly since pc is always a multiple of 4
         //$imem_rd_addr[7:0] = $pc; //gives wrong ans - in instr mem - every index is considered 4 bytes (not 1 byte) so divide pc by 4 necessary
         $imem_rd_addr[7:0] = $pc / 4 ; 
         //after observing the waveform: it seems the addr in the mem = 8 only
         //so as pc = 8*4 = 32 --> instr extracted is the first instr cuz : (32/4)mod8 = 0 (mod8 since only 8 instrs)
         
         //get the instruction from the instr mem
         $instr[31:0] = $imem_rd_data[31:0]; 
         
         //opcode in any instr format is 7 bits long but we consider only 5 bits
         //the part of instr that we need to decode the instr format = instr[6:2]
         //instr[1:0] is always 11 for base instr set, hence ignored
         
         $is_i_instr = $instr[6:2] ==? 5'b0000x || //this takes care of 5'b00000 and 5'b00001
                       $instr[6:2] ==? 5'b001x0 || //5'b00100 and 5'b00110
                       $instr[6:2] ==? 5'b11001 ||
                       $instr[6:2] ==? 5'b00100 ; 
         
         $is_r_instr = $instr[6:2] ==? 5'b01011 ||
                       $instr[6:2] ==? 5'b011x0 || //5'b01100 and 5'b01110
                       $instr[6:2] ==? 5'b10100 ;
         
         $is_s_instr = $instr[6:2] ==? 5'b0100x ; //5'b01000 and 5'b01001
         
         $is_b_instr = $instr[6:2] ==? 5'b11000 ;
         
         $is_j_instr = $instr[6:2] ==? 5'b11011 ;
         
         $is_u_instr = $instr[6:2] ==? 5'b0x101 ; //5'b00101 and 5'b01101
         
         //getting the immediate values and sign extending them
         //Eg: {21{$instr[31]} , $instr[30:20]} --> 12 bit imm, msb = instr[31] -> sign extend = {21{msb}}
         //instr[30:20] = give lower 11 (30-20+1) bits
         //r instr doesn't have any imm values
         $imm[31:0] = $is_i_instr ? { {21{$instr[31]}}, $instr[30:20] } : //12 bits for i type - immediate is one of the operands
                      $is_s_instr ? { {21{$instr[31]}}, $instr[30:25], $instr[11:7] } : //12 bits for s type - imm gives offset for calc effective address to store value
                      $is_b_instr ? { {20{$instr[31]}}, $instr[7], $instr[31:25], $instr[11:8], 1'b0 } : //13 bits for b type - imm gives pc relative offset to branch required label/instr
                      $is_u_instr ? { $instr[31:12] , 12'b0 } : //20 bits for u type - imm gives upper 20 bits of a 32 bit value 
                      $is_j_instr ? { {12{$instr[31]}}, $instr[19:12], $instr[20], $instr[30:21], 1'b0 } : //21 bits for j type - imm gives the pc relative addr to jump to 
                      32'b0 ; //if none of the instrs mentioned (if rtype), then imm is 0 - if this condition isn't there - ternary op is not complete - throws error
         
         //extracting other fields of instr
         //risc v is a regular ISA, which means most fields are consistent in their bit positions for all instr types except for immediate field
         //hence other fields of instr don't need to type based conditions
         $funct7[6:0] = $instr[31:25]; 
         $funct3[2:0] = $instr[14:12];
         $rs1[4:0] = $instr[19:15]; //source reg 1 - only 32 reg in total, so 5 bits to represent them
         $rs2[4:0] = $instr[24:20]; //source reg 2
         $rd[4:0] = $instr[11:7]; //destination reg
         $opcode[6:0] = $instr[6:0]; //opcode required to instruct the alu
```
## PC + instr fetch + decode unit (validity check + individual instr decode)
### Solution
![image](https://github.com/user-attachments/assets/1b99f839-ec0c-4228-b273-24ddadf65516)
```
   |cpu
      @0
         $reset = *reset;
         
         
         $pc[31:0] = (>>1$reset) ? 32'd0 : (>>1$pc + 32'd4);
         //curr pc = (is prev reset value true) ? if yes, curr pc = 0 : if not, curr pc = prev pc + 4 (4 locations = 1 instr)
         
      @1
         $imem_rd_en = ! $reset;  //active low reset - it considers the prev reset value
         
         //input the address - its not pc directly since pc is always a multiple of 4
         //$imem_rd_addr[7:0] = $pc; //gives wrong ans - in instr mem - every index is considered 4 bytes (not 1 byte) so divide pc by 4 necessary
         $imem_rd_addr[7:0] = $pc / 4 ; 
         //$imem_rd_addr[7:0] = $pc >> 4 ; //this works too 
         //after observing the waveform: it seems the addr in the mem = 8 only
         //so as pc = 8*4 = 32 --> instr extracted is the first instr cuz : (32/4)mod8 = 0 (mod8 since only 8 instrs)
         
         //get the instruction from the instr mem
         $instr[31:0] = $imem_rd_data[31:0]; 
         
         //opcode in any instr format is 7 bits long but we consider only 5 bits
         //the part of instr that we need to decode the instr format = instr[6:2]
         //instr[1:0] is always 11 for base instr set, hence ignored
         
         $is_i_instr = $instr[6:2] ==? 5'b0000x || //this takes care of 5'b00000 and 5'b00001
                       $instr[6:2] ==? 5'b001x0 || //5'b00100 and 5'b00110
                       $instr[6:2] ==? 5'b11001 ||
                       $instr[6:2] ==? 5'b00100 ; 
         
         $is_r_instr = $instr[6:2] ==? 5'b01011 ||
                       $instr[6:2] ==? 5'b011x0 || //5'b01100 and 5'b01110
                       $instr[6:2] ==? 5'b10100 ;
         
         $is_s_instr = $instr[6:2] ==? 5'b0100x ; //5'b01000 and 5'b01001
         
         $is_b_instr = $instr[6:2] ==? 5'b11000 ;
         
         $is_j_instr = $instr[6:2] ==? 5'b11011 ;
         
         $is_u_instr = $instr[6:2] ==? 5'b0x101 ; //5'b00101 and 5'b01101
         
         //getting the immediate values and sign extending them
         //Eg: {21{$instr[31]} , $instr[30:20]} --> 12 bit imm, msb = instr[31] -> sign extend = {21{msb}}
         //instr[30:20] = give lower 11 (30-20+1) bits
         //r instr doesn't have any imm values
         
         //imm field is not valid for rtype
         $imm_valid = $is_r_instr ? 1'b0 : 1'b1 ; 
         ?$imm_valid
            $imm[31:0] = $is_i_instr ? { {21{$instr[31]}}, $instr[30:20] } : //12 bits for i type - immediate is one of the operands
                         $is_s_instr ? { {21{$instr[31]}}, $instr[30:25], $instr[11:7] } : //12 bits for s type - imm gives offset for calc effective address to store value
                         $is_b_instr ? { {20{$instr[31]}}, $instr[7], $instr[31:25], $instr[11:8], 1'b0 } : //13 bits for b type - imm gives pc relative offset to branch required label/instr
                         $is_u_instr ? { $instr[31:12] , 12'b0 } : //20 bits for u type - imm gives upper 20 bits of a 32 bit value 
                         $is_j_instr ? { {12{$instr[31]}}, $instr[19:12], $instr[20], $instr[30:21], 1'b0 } : //21 bits for j type - imm gives the pc relative addr to jump to 
                         32'b0 ; //if none of the instrs mentioned (if rtype), then imm is 0 - if this condition isn't there - ternary op is not complete - throws error
         
         //extracting other fields of instr
         //risc v is a regular ISA, which means most fields are consistent in their bit positions for all instr types except for immediate field
         //hence other fields of instr don't need to type based conditions
         
         //funct7 is valid only for rtype
         $funct7_valid = $is_r_instr ? 1'b1 : 1'b0 ; // condition ? if true : if false
         ?$funct7_valid
            $funct7[6:0] = $instr[31:25]; 
         
         //this is the only signal that is in my code and not told in the workshop 
         //funct3, rs1, rs2 is invalid for u_type and j type in common, so define a common signal
         $u_or_j = $is_u_instr || $is_j_instr ; 
         
         //funct3 is invalid for u type and j type
         $funct3_valid = $u_or_j ? 1'b0 : 1'b1 ;
         ?$funct3_valid
            $funct3[2:0] = $instr[14:12];
         
         //rs1 is invalid for u type and j type
         $rs1_valid = $u_or_j ? 1'b0 : 1'b1 ;
         ?$rs1_valid
            $rs1[4:0] = $instr[19:15]; //source reg 1 - only 32 reg in total, so 5 bits to represent them
         
         //rs2 is valid for rtype, stype and btype only OR rs2 is invalid for u type, j type and i type - im using this variant since ive define uorj signal already
         //any of the above two is fine
         $rs2_valid = ($u_or_j || $is_i_instr) ? 1'b0 : 1'b1; 
         ?$rs2_valid
            $rs2[4:0] = $instr[24:20]; //source reg 2
         
         //rd is invalid for stype and btype
         $rd_valid = ($is_s_instr || $is_b_instr) ? 1'b0 : 1'b1; 
         ?$rd_valid
            $rd[4:0] = $instr[11:7]; //destination reg
         
         //opcode is valid for all instr types, so no validity check 
         $opcode[6:0] = $instr[6:0]; //opcode required to instruct the alu
         
         //now decode individual instrs - only few instrs from RV32I are considered - others can be added later
         //collect the bits required for decoding - funct7[5], func3, opcode
         $dec_bits[10:0] = { $funct7[5], $funct3, $opcode }; //if index range not specified - means consider all bits of that field
         
         //add, addi (arithmetic instrs)
         $is_add = $dec_bits == 11'b0_000_0110011;
         $is_addi = $dec_bits ==? 11'bx_000_0010011; //==? is verilog operation for comparing against don't cares, it works even when there is not don't care
         //for addi, funct7[5] can be 0 or 1, it doesn't matter - so x (don't care) is used
         
         //branch variants 
         $is_beq = $dec_bits ==? 11'bx_000_1100011;
         $is_bne = $dec_bits ==? 11'bx_001_1100011;
         $is_blt = $dec_bits ==? 11'bx_100_1100011;
         $is_bge = $dec_bits ==? 11'bx_101_1100011;
         $is_bltu = $dec_bits ==? 11'bx_110_1100011;
         $is_bgeu = $dec_bits ==? 11'bx_111_1100011;
         
         
```

