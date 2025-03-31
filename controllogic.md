# Control Logic
## **Register File Read Process:**
![image](https://github.com/user-attachments/assets/0147b811-c926-4e09-9d52-184febb74da6)

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

## Labs
![image](https://github.com/user-attachments/assets/30095a17-9a93-492b-a14e-e212a36d029b)
### Solution
![image](https://github.com/user-attachments/assets/b7e1a6dc-168b-4772-802d-566c52f7bd68)
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
         
         //register file interfacing
         //I'm reusing field validity check signals
         
         //INPUT SIGNALS - reg file
         //write signals
         $rf_wr_en = $rd_valid;  //write into the register file only if rd is a valid field
         $rf_wr_index[4:0] = $rd; //address of the dest reg
         //$rf_wr_data[31:0] = //this is actually the output of the alu - the result, so it can't be assigned now
         
         //read signals
         $rf_rd_en1 = $rs1_valid; //read rs1 from reg file only if rs1 is a valid field in the instr
         $rf_rd_index1[4:0] = $rs1; //address of source reg 1
         $rf_rd_en2 = $rs2_valid; //read rs2 from reg file only if rs2 is a valid field
         $rf_rd_index2[4:0] = $rs2; //address of source reg 2
         
         //OUTPUT SIGNALS - reg file - inputs to the ALU
         $src1_value[31:0] = $rf_rd_data1; //rs1 value
         $src2_value[31:0] = $rf_rd_data2; //rs2 value

```
## Lab 2 Register file
![image](https://github.com/user-attachments/assets/17f20f82-208c-4509-a9d5-aa1a03b4e01f)
### Solution
![image](https://github.com/user-attachments/assets/58fb4164-214e-4d3e-a74a-e57bae8f6768)
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
         
         //REGISTER FILE interfacing
         //I'm reusing field validity check signals
         
         //INPUT SIGNALS - reg file
         //write signals 
         //NOTE: x0 reg in risc v is hardwired to 0, so writes are ignored and read is always 0
         //wr_en = 0 if rd = x0 - changing wr_en along is not disabling write (reg is still written into when r0 is rd - check rf_wr_data field in waveform)
         $rf_wr_en = ($rd == 5'd0) ? 1'b0 : $rd_valid;  //write into the register file only if rd is a valid field
         $rf_wr_index[4:0] = $rd; //address of the dest reg
         //check the wr_en signal - if its 0, don't write into the register, else, write
         $rf_wr_data[31:0] =  $rf_wr_en ? $result : 32'd0; //this is actually the output of the alu - the result
         
         //read signals
         $rf_rd_en1 = $rs1_valid; //read rs1 from reg file only if rs1 is a valid field in the instr
         $rf_rd_index1[4:0] = $rs1; //address of source reg 1
         $rf_rd_en2 = $rs2_valid; //read rs2 from reg file only if rs2 is a valid field
         $rf_rd_index2[4:0] = $rs2; //address of source reg 2
         
         //OUTPUT SIGNALS - reg file - inputs to the ALU
         $src1_value[31:0] = $rf_rd_data1; //rs1 value
         $src2_value[31:0] = $rf_rd_data2; //rs2 value
         
         //ALU 
         //operations: add, addi (others are added later)
         $result[31:0] = $is_addi ? ($src1_value + $imm) : //rd <-- rs1 + imm
                         $is_add ? ($src1_value + $src2_value) : //rd <-- rs1 + rs2
                         32'bx ; //if none of the above instrs, result is don't care


```

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
## Lab for Branch instruction
![image](https://github.com/user-attachments/assets/ca1c0b5d-9131-42d6-9336-8fb65cb55ee7)
### Solution
![image](https://github.com/user-attachments/assets/a1284a4b-fb41-4ac9-b90d-a4f8a409abc9)
```
|cpu
      @0
         $reset = *reset;
         
         
         $pc[31:0] = (>>1$reset) ? 32'd0 : 
                     (>>1$taken_br) ? (>>1$br_tgt_pc): 
                     (>>1$pc + 32'd4);
         //curr pc = (is prev reset value true) ? if yes, curr pc = 0 : if not, is branch taken? yes, pc = branch target addr of prev instr : no, curr pc = prev pc + 4 (4 locations = 1 instr)
         //i need to consider the prev taken br and prev br target addr, cuz pc operates @0, but all computation is done @1 --> so curr pc always gives next instr addr , hence consider prev instr for branching
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
         
         //REGISTER FILE interfacing
         //I'm reusing field validity check signals
         
         //INPUT SIGNALS - reg file
         //write signals 
         //NOTE: x0 reg in risc v is hardwired to 0, so writes are ignored and read is always 0
         //wr_en = 0 if rd = x0 - changing wr_en along is not disabling write (reg is still written into when r0 is rd - check rf_wr_data field in waveform)
         $rf_wr_en = ($rd == 5'd0) ? 1'b0 : $rd_valid;  //write into the register file only if rd is a valid field
         $rf_wr_index[4:0] = $rd; //address of the dest reg
         //check the wr_en signal - if its 0, don't write into the register, else, write
         $rf_wr_data[31:0] =  $rf_wr_en ? $result : 32'd0; //this is actually the output of the alu - the result
         
         //read signals
         $rf_rd_en1 = $rs1_valid; //read rs1 from reg file only if rs1 is a valid field in the instr
         $rf_rd_index1[4:0] = $rs1; //address of source reg 1
         $rf_rd_en2 = $rs2_valid; //read rs2 from reg file only if rs2 is a valid field
         $rf_rd_index2[4:0] = $rs2; //address of source reg 2
         
         //OUTPUT SIGNALS - reg file - inputs to the ALU
         $src1_value[31:0] = $rf_rd_data1; //rs1 value
         $src2_value[31:0] = $rf_rd_data2; //rs2 value
         
         //ALU 
         //operations: add, addi (others are added later)
         $result[31:0] = $is_addi ? ($src1_value + $imm) : //rd <-- rs1 + imm
                         $is_add ? ($src1_value + $src2_value) : //rd <-- rs1 + rs2
                         32'bx ; //if none of the above instrs, result is don't care
                         
         //logic to specify if a branch is taken or not
         $taken_br = (! $is_b_instr) ? 1'b0 :
                      $is_beq ? ($src1_value == $src2_value) :
                      $is_bne ? ($src1_value != $src2_value) :
                      $is_blt ? ( ($src1_value < $src2_value) ^ ($src1_value[31] != $src2_value[31]) ) : // for signed comparisons
                      $is_bge ? ( ($src1_value >= $src2_value) ^ ($src1_value[31] != $src2_value[31]) ) : //for signed comparison
                      $is_bltu ? ($src1_value < $src2_value) : //unsigned comparisons only
                      $is_bgeu ? ($src1_value >= $src2_value) : //usigned comparisons
                      1'b0 ; //default condition
    
         //if branch is taken for curr instr, then next pc is changed
         $br_tgt_pc[31:0] = $pc + $imm ; 
```

## Single cycle risc v that computes the sum of numbers from 1 to 9
### Solution
![image](https://github.com/user-attachments/assets/301cd7a9-9844-496b-8e26-2e12cfd6e3cf)
```
\m4_TLV_version 1d: tl-x.org
\SV
   // This code can be found in: https://github.com/stevehoover/RISC-V_MYTH_Workshop
   
   m4_include_lib(['https://raw.githubusercontent.com/BalaDhinesh/RISC-V_MYTH_Workshop/master/tlv_lib/risc-v_shell_lib.tlv'])

\SV
   m4_makerchip_module   // (Expanded in Nav-TLV pane.)
\TLV

   // /====================\
   // | Sum 1 to 9 Program |
   // \====================/
   //
   // Program for MYTH Workshop to test RV32I
   // Add 1,2,3,...,9 (in that order).
   //
   // Regs:
   //  r10 (a0): In: 0, Out: final sum
   //  r12 (a2): 10
   //  r13 (a3): 1..10
   //  r14 (a4): Sum
   // 
   // External to function:
   m4_asm(ADD, r10, r0, r0)             // Initialize r10 (a0) to 0.
   // Function:
   m4_asm(ADD, r14, r10, r0)            // Initialize sum register a4 with 0x0
   m4_asm(ADDI, r12, r10, 1010)         // Store count of 10 in register a2.
   m4_asm(ADD, r13, r10, r0)            // Initialize intermediate sum register a3 with 0
   // Loop:
   m4_asm(ADD, r14, r13, r14)           // Incremental addition
   m4_asm(ADDI, r13, r13, 1)            // Increment intermediate register by 1
   //to check if taken_br logic works, check instr 6 in waveform
   m4_asm(BLT, r13, r12, 1111111111000) // If a3 is less than a2, branch to label named <loop>
   m4_asm(ADD, r10, r14, r0)            // Store final result to register a0 so that it can be read by main program
   //to check the final result, see the waveform (in the last 7 value of imm_rd_addr, rd (x10) will have 2d (45 in dec)
   
   // Optional:
   // m4_asm(JAL, r7, 00000000000000000000) // Done. Jump to itself (infinite loop). (Up to 20-bit signed immediate plus implicit 0 bit (unlike JALR) provides byte address; last immediate bit should also be 0)
   m4_define_hier(['M4_IMEM'], M4_NUM_INSTRS)

   //NOTE: WE'RE IMPLEMENTING RV32I means each reg = 32 bit long
   //code for Program counter (PC) and getting instr from instr mem (instr mem interfacing)
   //adding decode unit - opcode, imm, other fields - modify such that a field will be valid based on instr type (eg. imm is invalid for rtype)
   //decoding individual instrs
   //register file read/write - interfacing
   //add alu operations + add branch instrs + modify pc
   
   |cpu
      @0
         $reset = *reset;
         
         
         $pc[31:0] = (>>1$reset) ? 32'd0 : 
                     (>>1$taken_br) ? (>>1$br_tgt_pc): 
                     (>>1$pc + 32'd4);
         //curr pc = (is prev reset value true) ? if yes, curr pc = 0 : if not, is branch taken? yes, pc = branch target addr of prev instr : no, curr pc = prev pc + 4 (4 locations = 1 instr)
         //i need to consider the prev taken br and prev br target addr, cuz pc operates @0, but all computation is done @1 --> so curr pc always gives next instr addr , hence consider prev instr for branching
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
         
         //REGISTER FILE interfacing
         //I'm reusing field validity check signals
         
         //INPUT SIGNALS - reg file
         //write signals 
         //NOTE: x0 reg in risc v is hardwired to 0, so writes are ignored and read is always 0
         //wr_en = 0 if rd = x0 - changing wr_en along is not disabling write (reg is still written into when r0 is rd - check rf_wr_data field in waveform)
         $rf_wr_en = ($rd == 5'd0) ? 1'b0 : $rd_valid;  //write into the register file only if rd is a valid field
         $rf_wr_index[4:0] = $rd; //address of the dest reg
         //check the wr_en signal - if its 0, don't write into the register, else, write
         $rf_wr_data[31:0] =  $rf_wr_en ? $result : 32'd0; //this is actually the output of the alu - the result
         
         //read signals
         $rf_rd_en1 = $rs1_valid; //read rs1 from reg file only if rs1 is a valid field in the instr
         $rf_rd_index1[4:0] = $rs1; //address of source reg 1
         $rf_rd_en2 = $rs2_valid; //read rs2 from reg file only if rs2 is a valid field
         $rf_rd_index2[4:0] = $rs2; //address of source reg 2
         
         //OUTPUT SIGNALS - reg file - inputs to the ALU
         $src1_value[31:0] = $rf_rd_data1; //rs1 value
         $src2_value[31:0] = $rf_rd_data2; //rs2 value
         
         //ALU 
         //operations: add, addi (others are added later)
         $result[31:0] = $is_addi ? ($src1_value + $imm) : //rd <-- rs1 + imm
                         $is_add ? ($src1_value + $src2_value) : //rd <-- rs1 + rs2
                         32'bx ; //if none of the above instrs, result is don't care
                         
         //logic to specify if a branch is taken or not
         $taken_br = (! $is_b_instr) ? 1'b0 :
                      $is_beq ? ($src1_value == $src2_value) :
                      $is_bne ? ($src1_value != $src2_value) :
                      $is_blt ? ( ($src1_value < $src2_value) ^ ($src1_value[31] != $src2_value[31]) ) : // for signed comparisons
                      $is_bge ? ( ($src1_value >= $src2_value) ^ ($src1_value[31] != $src2_value[31]) ) : //for signed comparison
                      $is_bltu ? ($src1_value < $src2_value) : //unsigned comparisons only
                      $is_bgeu ? ($src1_value >= $src2_value) : //usigned comparisons
                      1'b0 ; //default condition
         //how does the expression for signed comparison work? 
         //a<b or a>=b does unsigned comparison, so we need another expression to account for signed values
         //31st bit = msb, consider a, b, if msb(a) = 1 and msb(b) = 0, then term 2 in the expression = 1
         //the expression now becomes: term1 xor 1 (property : 1 xor A = A' - Abar/ complement), so whatever the term1 evaluates, final output is opposite of that
         //eg: if a(-ve) and b(+ve) , then a<b is 0 (unsigned comparison)  but considering the signs: a<b = 1 cuz -ve < +ve always
         //same logic can be applied when a(+ve) and b(-ve) and for bge
         
         //calculate the branch target address (for current pc)
         //if branch is taken for curr instr, then next pc is changed
         $br_tgt_pc[31:0] = $pc + $imm ; 
         
         


   
   // Assert these to end simulation (before Makerchip cycle limit).
   //*passed = *cyc_cnt > 40;
   //how to tell makerchip that the simulation has passed? 1. manually check the waveforms, 2. do the following (a simple check)
   *passed = |cpu/xreg[10]>>5$value == (1+2+3+4+5+6+7+8+9) ; //monitors xreg[10], when the value reaches the RHS at say nth cycle, it tells it on (n+5)th cycle
   //tell - means, passed msg appears in LOG
   *failed = 1'b0;
   
   // Macro instantiations for:
   //  o instruction memory
   //  o register file
   //  o data memory
   //  o CPU visualization
   |cpu
      //these are all the arrays required for building the cpu
      //this is the instr mem
      m4+imem(@1)    // Args: (read stage) 
      //this is the register file
      m4+rf(@1, @1)  // Args: (read stage, write stage) - if equal, no register bypass is required 
      //m4+dmem(@4)    // Args: (read/write stage)

   //m4+cpu_viz(@4)    // For visualisation, argument should be at least equal to the last stage of CPU logic. @4 would work for all labs.
\SV
   endmodule
```
