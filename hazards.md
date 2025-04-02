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



![image](https://github.com/user-attachments/assets/bb48ecc0-6313-4afd-b853-ef0b906eaa17)


```
   |cpu
      @0
         $reset = *reset;
         
         //use a valid signal that becomes high every 3 clock cycles, 
         //we already have 2 stages of pipeline, just add one more, and check that value
         //this is a better approach than implementing a counter, cuz 1. pipeline already there, 
         //make use of it, 2. every 3 cycles, 3 is odd, so harder to implement a counter (even number is easier)
         //valid signal must be initialised using start signal
         //start = 1, right after reset is asserted, and = 0 o/w
         //this start signal seems a bit diff from what is shown in the vid, in that - it is 1 throughout the period reset = 1 
         
         $start = $reset ? 1'b0 : (>>1$reset) ? 1'b1 : 1'b0 ; //if curr reset = 1, start = 0, else, prev reset = 1, start = 1, else start = 0
         
         //remove this valid signal @0, cuz now 1 instr needs to be supported every cycle
         //$valid = $reset ? 1'b0 :
         //         $start ? 1'b1 :
         //         (>>3$valid) ; //valid value 3 cycles before current cycle
         ////how to use this? 
         ////1. avoid writing into reg file when invalid (valid = 0)
         ////2. update pc every 3 cycles
         
         
         //curr pc = if reset, =0, else if 3 cycles ago, valid branch taken, = branch target addr calc 3 cycles ago, else curr pc = 3 cycles ago pc + 4
         //this is done because the cpu executes 1 instr every 3 cycles, so prev instr requires pc value 3 cycles ago
         //now, support inc of pc every cycle
         //it is assumed that branch is taken 3 cycles ago only - cuz it will be computed @3
         $pc[31:0] = (>>1$reset) ? 32'd0 : 
                     (>>3$taken_br) ? (>>3$br_tgt_pc): 
                     (>>1$pc + 32'd4);
      
      @1
         //INSTRUCTION FETCH - START
         $imem_rd_en = ! $reset;  //active low reset 
         $imem_rd_addr[7:0] = $pc / 4 ; 
         //get the instruction from the instr mem
         $instr[31:0] = $imem_rd_data[31:0]; 
         //INSTRUCTION FETCH - END

         //INSTRUCTION DECODE - START
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

         //imm field is not valid for rtype
         $imm_valid = $is_r_instr ? 1'b0 : 1'b1 ; 
         ?$imm_valid
            $imm[31:0] = $is_i_instr ? { {21{$instr[31]}}, $instr[30:20] } : //12 bits for i type 
                        $is_s_instr ? { {21{$instr[31]}}, $instr[30:25], $instr[11:7] } : //12 bits for s type 
                        $is_b_instr ? { {20{$instr[31]}}, $instr[7], $instr[31:25], $instr[11:8], 1'b0 } : //13 bits for b type 
                        $is_u_instr ? { $instr[31:12] , 12'b0 } : //20 bits for u type 
                        $is_j_instr ? { {12{$instr[31]}}, $instr[19:12], $instr[20], $instr[30:21], 1'b0 } : //21 bits for j type 
                        32'b0 ; 


            //funct7 is valid only for rtype
         $funct7_valid = $is_r_instr ? 1'b1 : 1'b0 ; // condition ? if true : if false
         ?$funct7_valid
            $funct7[6:0] = $instr[31:25]; 

         $u_or_j = $is_u_instr || $is_j_instr ; 

         //funct3 is invalid for u type and j type
         $funct3_valid = $u_or_j ? 1'b0 : 1'b1 ;
         ?$funct3_valid
            $funct3[2:0] = $instr[14:12];

         //rs1 is invalid for u type and j type
         $rs1_valid = $u_or_j ? 1'b0 : 1'b1 ;
         ?$rs1_valid
            $rs1[4:0] = $instr[19:15]; 

         $rs2_valid = ($u_or_j || $is_i_instr) ? 1'b0 : 1'b1; 
         ?$rs2_valid
            $rs2[4:0] = $instr[24:20]; //source reg 2

         //rd is invalid for stype and btype
         $rd_valid = ($is_s_instr || $is_b_instr) ? 1'b0 : 1'b1; 
         ?$rd_valid
            $rd[4:0] = $instr[11:7]; //destination reg

         $opcode[6:0] = $instr[6:0]; 

         $dec_bits[10:0] = { $funct7[5], $funct3, $opcode }; 

         //arithmetic instrs, shift and logical 
         $is_add = $dec_bits == 11'b0_000_0110011; 
         $is_sub = $dec_bits ==? 11'b1_000_0110011;
         $is_sll = $dec_bits ==? 11'b0_001_0110011;
         $is_slt = $dec_bits ==? 11'b0_010_0110011;
         $is_sltu = $dec_bits ==? 11'b0_011_0110011;
         $is_xor = $dec_bits ==? 11'b0_100_0110011;
         $is_srl = $dec_bits ==? 11'b0_101_0110011;
         $is_sra = $dec_bits ==? 11'b1_101_0110011;
         $is_or = $dec_bits ==? 11'b0_110_0110011;
         $is_and = $dec_bits ==? 11'b0_111_0110011;

         //branch variants 
         $is_beq = $dec_bits ==? 11'bx_000_1100011;
         $is_bne = $dec_bits ==? 11'bx_001_1100011;
         $is_blt = $dec_bits ==? 11'bx_100_1100011;
         $is_bge = $dec_bits ==? 11'bx_101_1100011;
         $is_bltu = $dec_bits ==? 11'bx_110_1100011;
         $is_bgeu = $dec_bits ==? 11'bx_111_1100011;
         
         //u type instrs
         $is_lui = $dec_bits ==? 11'bx_xxx_0110111;
         $is_auipc = $dec_bits ==? 11'bx_xxx_0010111;
         
         //jump instrs - j type/i type
         $is_jal = $dec_bits ==? 11'bx_xxx_1101111;
         $is_jalr = $dec_bits ==? 11'bx_xxx_1100111;
         
         //load - all load variants treated the same, so check opcode only
         $is_load = $opcode ==? 7'b0000011; 
         
         //stores - stype instructions
         $is_sb = $dec_bits ==? 11'bx_000_0100011;
         $is_sh = $dec_bits ==? 11'bx_001_0100011;
         $is_sw = $dec_bits ==? 11'bx_010_0100011;
         
         //set less than varaints
         $is_slti = $dec_bits ==? 11'bx_010_0010011;
         $is_sltiu = $dec_bits ==? 11'bx_011_0010011; 
         
         //logical and shift instructions - itype
         $is_addi = $dec_bits ==? 11'bx_000_0010011;
         $is_xori = $dec_bits ==? 11'bx_100_0010011;
         $is_ori = $dec_bits ==? 11'bx_110_0010011;
         $is_andi = $dec_bits ==? 11'bx_111_0010011;
         $is_slli = $dec_bits ==? 11'b0_001_0010011;
         $is_srli = $dec_bits ==? 11'b0_101_0010011;
         $is_srai = $dec_bits ==? 11'b1_101_0010011;
         
         
         

            //INSTRUCTION DECODE - END
      @2
         //REGISTER FILE interfacing
         //INPUT SIGNALS - reg file

         //$rf_wr_data[31:0] = $rf_wr_en ? (>>1$result) : 32'd0; //this can't be done @2, it has to be done after alu calc the result, so @3
         //if this stays here, no error, but you'll never get the answer - it never passes

         //read signals
         $rf_rd_en1 = $rs1_valid; 
         $rf_rd_index1[4:0] = $rs1; //address of source reg 1
         $rf_rd_en2 = $rs2_valid; 
         $rf_rd_index2[4:0] = $rs2; //address of source reg 2


         //OUTPUT SIGNALS - reg file - inputs to the ALU
         //adding reg file bypass to handle read after write hazard
         //if previously rd was written into reg file AND previous instr rd == rs1 or rs2, then select previous result (2 cycles ago) as src1 or src2 for this instr
         //why 2 cycles ago? refer to the diagram to understand (video)

         $select1 = >>2$rf_wr_en && (>>1$rd == $rs1); 
         $select2 = >>2$rf_wr_en && (>>1$rd == $rs2);
         $src1_value[31:0] = $select1 ? (>>1$result) : $rf_rd_data1; //rs1 value
         $src2_value[31:0] = $select2 ? (>>1$result) : $rf_rd_data2; //rs2 value
         
         $br_tgt_pc[31:0] = $pc + $imm ; 
         //REGISTER FILE - END
      @3
         //ALU - START
         //operations: add, addi (others are added later)
         
         $sltu_res[31:0] = $src1_value < $src2_value ;
         $sltiu_res[31:0] = $src1_value < $imm ; 
         $srai_res[31:0] = { {32{$src1_value[31]}}, $src1_value } >> $imm[4:0]; 
         $sra_res[31:0] = { {32{$src1_value[31]}}, $src1_value } >> $src2_value[4:0]; 
         $slt_res[31:0] = ($src1_value[31] == $src2_value[31]) ? $sltu_res : {31'b0, $src1_value[31]};
         $slti_res[31:0] = ($src1_value[31] == $imm[31]) ? $sltiu_res : {31'b0, $src1_value[31]};
         
         $result[31:0] = $is_addi ? ($src1_value + $imm) : //rd <-- rs1 + imm
                         $is_add ? ($src1_value + $src2_value) : //rd <-- rs1 + rs2
                         $is_andi ? ($src1_value & $imm) :
                         $is_ori ? ($src1_value | $imm) :
                         $is_xori ? ($src1_value ^ $imm) :
                         $is_slli ? ($src1_value << $imm[5:0]) : //why 5:0 ?
                         $is_srli ? ($src1_value >> $imm[5:0]) :
                         $is_and ? ($src1_value & $src2_value) :
                         $is_or ? ($src1_value | $src2_value) :
                         $is_xor ? ($src1_value ^ $src2_value) :
                         $is_sub ? ($src1_value - $src2_value) :
                         $is_sll ? ($src1_value << $src2_value[4:0]) :
                         $is_srl ? ($src1_value >> $src2_value[4:0]) :
                         $is_sltu ? $sltu_res :
                         $is_sltiu ? $sltiu_res :
                         $is_lui ? {$imm[31:12], 12'b0} :
                         $is_auipc ? ($pc + $imm) :
                         ($is_jal || $is_jalr) ? ($pc + 32'd4) :
                         $is_srai ? $srai_res :
                         $is_sra ? $sra_res :
                         $is_slt ? $slt_res :
                         $is_slti ? $slti_res :
                         32'bx ; //if none of the above instrs, result is don't care
         //ALU - END
         
         $valid =  ! (>>1$taken_br || >>2$taken_br);
         //write signals 
         //include the valid signal in the wr_en signal - disable write when valid = 0
         $rf_wr_en = ($rd == 5'd0 || (! $valid)) ? 1'b0 : $rd_valid;  
         $rf_wr_index[4:0] = $rd; //address of the dest reg
         //valid signal shifted here
         //given prev two instructions are not branch, valid signal will be high
         
         $rf_wr_data[31:0] = $rf_wr_en ? ($result) : 32'd0; //write result into reg file write after calc it
         
         //logic to specify if a branch is taken or not
         $taken_br = (! $is_b_instr) ? 1'b0 :
                     $is_beq ? ($src1_value == $src2_value) :
                     $is_bne ? ($src1_value != $src2_value) :
                     $is_blt ? ( ($src1_value < $src2_value) ^ ($src1_value[31] != $src2_value[31]) ) : // for signed comparisons
                     $is_bge ? ( ($src1_value >= $src2_value) ^ ($src1_value[31] != $src2_value[31]) ) : //for signed comparison
                     $is_bltu ? ($src1_value < $src2_value) : //unsigned comparisons only
                     $is_bgeu ? ($src1_value >= $src2_value) : //usigned comparisons
                     1'b0 ; //default condition
         //to accomodate valid signal - removed now
         //$valid_taken_br = ($valid) && ($taken_br);





   
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
```


