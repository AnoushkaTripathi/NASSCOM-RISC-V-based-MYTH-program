# Validity Concept in TL-Verilog

#### 1. **Introduction to Validity**
   - **Validity** is a concept that helps manage when values of signals are **meaningful** in a circuit. This is different from traditional RTL languages, where there’s no direct way to convey whether a signal's value is valid or meaningful.
   - In hardware, not every cycle involves a meaningful operation. Sometimes, signals are driven with values, but those values have no real purpose during certain cycles. Validity helps **identify** and **convey** when values hold meaning, which is especially useful for managing power and ensuring correct behavior.

#### 2. **Example: Calculator Circuit**
   - In the previous exercise with the **calculator circuit**, we saw that the calculator was performing meaningful operations **every other cycle**. On alternate cycles, the gates were still active, driving values, but those values didn’t mean anything in the context of the circuit’s functionality.
   - Validity gives us a way to indicate which cycles the output is **valid** or **invalid**, providing a cleaner and more efficient circuit design.

#### 3. **Applying Validity to Pythagorean Theorem Circuit**
   - Take a circuit that computes the **Pythagorean theorem**. In a broader system, sometimes you need the result, and sometimes you don’t. A lot of the time, hardware might be idle, but still consuming power and resources.
   - With **validity**:
     - A **valid signal** is created and assigned in **stage 1** of the pipeline. This signal dictates when the values are valid and meaningful.
     - The computation is only considered when the **valid signal** is `true`. When `valid` is `false`, the signal is treated as **invalid** (also known as a "don't care" state).

#### 4. **Benefits of Validity**
   - **Easier Debugging**:
     - When examining the waveform in simulations, validity helps you focus on the meaningful values.
     - Without validity, the waveform would show a "sea of values" even when some values are irrelevant. With validity, you can clearly see when meaningful operations occur.
   - **Cleaner Design**:
     - It adds clarity to the design, making it easier to understand and model complex circuits.
   - **Error Checking**:
     - Validity helps catch errors that might occur if invalid signals are consumed by downstream logic.
     - During simulation, if an invalid (don’t care) signal is fed into a gate, the don’t care value propagates through the circuit. This will raise errors if invalid data is inadvertently being used.
   
#### 5. **Don’t Care Values and Error Checking**
   - **Don’t Care Values (X)**: In simulation, **don’t care values** (denoted as `X`) represent values that could be `0` or `1`, but are **irrelevant**.
   - If a signal is invalid, the simulator marks it as `X` (don’t care), and any downstream logic that consumes this signal will also output don’t care values. This helps **detect bugs** early in the design, as invalid values will trigger errors if used improperly.
   - **High Impedance Values (Z)**: Sometimes, a signal may also be in a **high impedance (Z)** state, meaning that it’s undriven.

#### 6. **Clock Gating for Power Savings**
   - **Clock Gating**: Clock gating is a technique used to save **power** in hardware circuits. Since clock signals transition twice per cycle, they consume a significant amount of power as they drive flip-flops across the chip.
   - With validity, you know exactly when a signal is **meaningful**. When the signal is invalid, you can **gate off** the clock, meaning that no clock pulse is delivered to the flip-flops, saving power.
   - **How it Works**:
     - If the signal is invalid, the **clock gate** prevents the clock from toggling the flip-flop, reducing power consumption.
     - If the signal is valid, the clock operates normally, driving the flip-flop and propagating the value through the pipeline.
     - The little **blue boxes** in the clock diagram represent the clock gates, which turn off the clock pulse whenever the signal is invalid.
   
#### 7. **Advantages of Clock Gating with TL-Verilog**
   - In traditional RTL, clock gating is often an **afterthought** or an optimization added later in the design process.
   - With **TL-Verilog**, clock gating can be part of the design process **from the beginning**, thanks to the concept of validity.
   - This integrated approach provides power savings **throughout the design lifecycle**, helping manage clock power more efficiently.

#### 8. **Conclusion**
   - Validity is a powerful concept in **TL-Verilog** that enhances signal management, makes design cleaner, simplifies debugging, and leads to better power efficiency.
   - By controlling when values are meaningful and combining that with **clock gating**, designers can create more efficient and error-resilient circuits while reducing unnecessary power consumption.

## Pythagoras Theorem Accumulation in Makerchip

#### **Development Process**
1. **Initial Setup:**
   - **Pipeline Stages:** The design uses a 3-stage pipeline to perform calculations based on the Pythagorean theorem.
   - **Inputs:** The variables A and B are defined as **4-bit values** to ensure clarity in the waveform during simulation. These values are randomly generated by the environment, as there are no specific assignments.
   - **Data Width Management:** Since A and B are multiplied (4x4 bits = 8 bits), the result is automatically extended with zeros to maintain proper bit-width alignment.

2. **File Structure:**
   - The first line of the file specifies the **TL-Verilog version (1d)**.
   - A URL link points to the **TL-Verilog documentation** at `tlx.org`.
   - **M4 Macro Processing** is enabled using `m4_tlv_version`, which allows macro definitions and expansions within the code.
   - **Macro Definition:** The `m4_makerchip_module` macro is used to define a **SystemVerilog** module with inputs (clock, reset, etc.) and outputs (`pass`, `fail` signals). These control simulation behavior based on specific conditions, such as cycle count exceeding 30.

3. **Pythagorean Theorem Computation:**
   - The initial code calculates the hypotenuse (C) using A² + B².
   - **Waveform Simulation:** The computation is visible in the waveform, where values of A and B result in a corresponding distance (C).
   - **Cycle Count:** The simulation runs for 30 cycles, after which the `pass` signal is asserted.

4. **Distance Accumulation Circuit:**
   - The circuit is extended to accumulate the total distance by repeatedly performing the Pythagoras computation and adding each new result to a running total.
   - **State Retention:** The **total distance** is stored as a state and updated with each new calculation.
   - **Adder Logic:** An **adder** adds the new distance to the total distance whenever a valid cycle is detected.
   - **Reset Logic:** The total distance is reset to zero at the start (using the Verilog module's reset signal).

5. **Validity Condition:**
   - The computation only proceeds in **valid cycles**, controlled by a `valid` signal.
   - The `when` condition ensures the computation occurs only when the cycle is valid, indicated by the valid signal.
   - **Platform-Generated Signals:** The platform generates random valid signals for simplicity, allowing computation to proceed based on simulated conditions.

6. **Code Structure:**
   - After the version line, an **`sv` statement** tells the TL-Verilog compiler to pass the SystemVerilog code without modification.
   - A **Verilog include statement** is used to include a predefined **square root module**.
   - The main logic is implemented within the TL-Verilog block, which gets compiled into **SystemVerilog**.

7. **Waveform Analysis:**
   - You can zoom in on specific cycles in the waveform to inspect the computed values for A, B, and C.
   - The **computation logic** is applied in the valid cycles, and the accumulated distance is updated accordingly.

---

#### **Key Takeaways:**
- **TL-Verilog Macros:** Simplifies module definitions, allowing for easier integration into platforms like Makerchip.
- **State and Reset Management:** The total distance is treated as state, requiring careful handling of reset and validity conditions to ensure proper accumulation.
- **Valid Cycles:** Computation happens only during valid cycles, controlled by a platform-generated `valid` signal.
- **SystemVerilog Compilation:** TL-Verilog code is ultimately compiled into SystemVerilog, allowing it to be used in a hardware description environment.

---

#### **Additional Tips:**
- **Keyboard Shortcuts:**
   - **Indenting Code:** Use `Ctrl + ]` to indent and `Ctrl + [` to un-indent sections of code.
   - **Control Waveform:** Control-click on waveform signals to zoom in and inspect individual values.

---

## **Pipeline Stage and Total Distance Computation Breakdown**

1. **Objective**:
   - The task is to compute the **total distance** over multiple pipeline stages, ensuring that the accumulated value is retained even when the pipeline is in an invalid state.
   - The value must be meaningful across cycles, requiring that the computation be independent of the validity of transactions in a given pipeline stage.

2. **Stage Involvement**:
   - The total distance computation occurs in **Stage 4** of the pipeline, but it interacts with values from **Stage 5** due to the pipelined nature of the design.
   - Total distance depends on accumulating values across multiple clock cycles, so the implementation must ensure that the value is retained from previous stages.

3. **Width Expansion**:
   - Initially, total distance was defined as a 32-bit value (`31:0`), but given that the value is meant to **accumulate over time**, you identified the need for a **wider bit-width**.
   - The width was expanded to **64 bits** (`63:0`), ensuring that the total distance can handle a larger range of accumulated values without overflow.

4. **Conditional Assignment and MUX Implementation**:
   - You used a **ternary expression** to conditionally assign the value of the total distance based on reset, validity, and the existing total distance.
   - The ternary expression acts as a **MUX** (multiplexer), selecting between multiple possible values for the total distance:
     - **On reset**: The total distance is reset to `0` (all 64 bits set to zero). The reset signal is referred to as `*reset` in TL-Verilog, and the ternary expression checks this condition first.
     - **Valid condition**: When the current cycle is valid, the **previous total distance** is updated by adding the current **cycle count value** (`cc value`) to it.
     - **Invalid condition**: If the cycle is invalid, the total distance is simply retained. This means the previous value is carried forward to the next cycle without any modification.

5. **Handling Previous Values in the Pipeline**:
   - To achieve the accumulation of values from prior cycles, you referenced the total distance value from **Stage 5** in **Stage 4**.
   - This is done using the **ahead-by-one operator** (`$`), which allows accessing the total distance value from a stage one cycle ahead (i.e., from Stage 5 to Stage 4).
   - The calculation in Stage 4 uses the value from Stage 5 to ensure that the correct total distance is retained and added to.

6. **Retaining the Total Distance**:
   - If the cycle is invalid, the pipeline retains the previously computed total distance.
   - Instead of writing the same logic repeatedly, TL-Verilog provides a **retain keyword**. This simplifies the process of retaining the value without explicit reassignment.
   - The retain keyword is functionally equivalent to assigning the value from the previous stage (i.e., it carries forward the old value of total distance).

7. **Simulation and Compilation**:
   - The design was compiled successfully, and the Verilator simulation ran without errors, confirming that the pipeline behaved as expected.
   - During compilation, you encountered **warnings about unassigned signals**. For instance, the signal `valid` was not assigned explicitly, which led to a warning in the logs. This is expected, given the intentional design where certain signals are allowed to remain unassigned.
   - The waveform output from the Verilator simulation was generated, showing that the total distance was correctly calculated and retained across cycles.

8. **Waveform Analysis**:
   - Upon analyzing the waveform, you confirmed that the total distance computation was working as expected.
   - For example:
     - The first non-reset cycle had an initial value of `f` (hexadecimal 15).
     - In the subsequent valid cycles, this value was accumulated: `f + a` resulted in `19` (hexadecimal 25).
     - This process continued over multiple cycles, and the total distance was correctly updated with each new value.

9. **Reset Handling**:
   - You encountered some confusion during the initial waveforms, especially with the behavior of the first few computations.
   - To address this and ensure a **cleaner reset** methodology, you proposed defining a **pipelined reset**.
     - The idea is to propagate the reset signal through the pipeline stages, so that each stage can reset its internal state in a synchronized manner.
   - By hooking the global reset signal to the pipeline's reset, and using it consistently throughout the pipeline, all stages will have transactions either during or after reset, ensuring proper computation.
     - The reset "marches" through the pipeline, aligning each stage with the global reset.
     - This ensures that the total distance (and any other states in the pipeline) are consistently reset, regardless of which stage the transaction is in.

10. **Pipeline Reset Methodology**:
    - In a simple design, it may not matter whether the reset is applied to each stage or not, but for a **complex pipeline**, where multiple states are involved, it is essential to reset all relevant states simultaneously.
    - By using a **pipelined reset**, you ensure that any state across multiple stages is reset consistently, making the pipeline design more robust and easier to manage in terms of timing and logic.

11. **Error Handling and Log Checking**:
    - During development, you encountered an error related to **indentation** while defining the pipeline reset.
    - This was quickly corrected, and upon recompilation, the pipeline design passed all checks.
    - You emphasized the importance of always checking the log files after compilation to ensure that no warnings or errors go unnoticed. 

# Calculator With Validity and Viz
### 1. **Running Calculator Every Other Cycle Using Validity**
   - Instead of zeroing out the output value every alternate cycle, the new approach involves using a **validity signal** to control whether the input is valid or not.
   - This allows the calculator to operate only when the input is valid, meaning calculations are performed conditionally. This is useful because:
     - If the input is not valid, the calculation is skipped without zeroing the output.
     - When inputs are valid, the computation is carried out normally.

### 2. **Reset and Validity in the Pipeline**
   - Reset handling is important because we need to ensure that the pipeline is in a known state (usually zeroed out) when reset occurs.
   - You emphasized that **validity must also be considered during reset** because the logic in the pipeline needs to be reset to a known state, especially if the output is being used as feedback (re-input into the system).
   - To achieve this, you propose generating a combined **valid or reset signal**, which acts as the condition for computation. 
     - This signal ensures that during a reset, the logic behaves correctly and transitions to the appropriate reset state.

### 3. **Avoiding Zeroing the Output on Invalid Cycles**
   - In the earlier approach, you zeroed out the output during invalid cycles for clarity in waveforms. However, zeroing the output wasn’t necessary.
   - The new approach uses **don't care values** during invalid cycles instead. This simplifies the circuit and reduces unnecessary conditions, while also making the waveforms cleaner by showing explicit "don't care" values when inputs are invalid.

### 4. **Reference to the Pythagoras Theorem Example**
   - You referenced the **Pythagoras theorem example** to illustrate the new changes in the pipeline. By applying the validity condition on the logic, you control whether the computation happens, instead of manually zeroing out the result every other cycle.
   - The pipeline will now only compute when valid inputs are present, streamlining the design and avoiding unnecessary computations.

### 5. **Interactive Labs and Visualization in Makerchip**
   - The subsequent part discusses the **reference solutions** in Makerchip, which are interactive visual aids designed to help students understand the pipeline and logic behavior.
   - You can step through the solution, examining the internal calculations in real-time, seeing how the pipeline performs operations based on valid inputs.
   - The **visualization tab** shows calculations in the form of a diagram, which is especially useful when debugging pipeline designs and ensuring that stages and logic are working as expected.

### 6. **Calculator Example with Visualization**
   - The reference solution for the calculator shows how values are computed and fed back into the system. The calculator performs simple arithmetic operations (like addition and multiplication), and the results are displayed in the visualization.
   - For example:
     - `0 + 5 = 5`, then `5` is used as the next input for subsequent calculations.
   - The visualization allows you to see how each stage in the pipeline behaves, helping you track valid inputs, outputs, and computations.

### 7. **Handling Special Cases**
   - The discussion on **negative numbers** points out a case where the system might underflow (producing unexpected large values). This is because the calculator isn't interpreting numbers as signed, so negative results can appear as large unsigned values.
   - This is a reminder that in certain cases, additional logic might be needed to protect against underflows or handle signed arithmetic properly.

### 8. **Supporting Resources for Labs**
   - To assist students with more complex labs, there's **shell code** available, which includes predefined structures for pipelining and visualization.
   - This is especially useful in later labs, such as those focused on RISC-V, where you can uncomment parts of the code to enable functionality and start adding logic for the calculator, memory recall, or other operations.

---

### Key Takeaways:
- Instead of explicitly zeroing out outputs on invalid cycles, use **validity conditions** to ensure that computation only happens when inputs are valid.
- During **reset**, logic needs to be enabled to transition to the correct state, hence the need for a combined **valid or reset** signal.
- **Visualization tools** in Makerchip can help in understanding how the pipeline works and provide real-time feedback on calculations.
- Avoid manual conditions for zeroing outputs—use **don't care values** when inputs are invalid to simplify the logic and focus on valid operations.

