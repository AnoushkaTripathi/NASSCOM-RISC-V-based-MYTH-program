 # Pipelining
![image](https://github.com/user-attachments/assets/6b0332a9-dccb-4840-89cc-483c5472393b)
### Pipeline Logic and Hardware Computation (Pythagoras Theorem Example)

#### 1. **Computation Goal: Pythagoras Theorem**
   - The problem is to compute the distance `c` using the Pythagoras theorem:  
    ![image](https://github.com/user-attachments/assets/d8551168-7b33-4a1a-af04-5f999aef2c37)
   - This requires hardware to:
     - Square values `a` and `b`.
     - Add the squared results.
     - Take the square root of the sum.
   - In a modern processor running at, say, 1 GHz, we can only perform a limited amount of logic (around 20 gates deep) within a single clock cycle. If the logic is deeper, it won't complete before the next clock cycle begins, leading to incorrect behavior.

#### 2. **Pipeline Concept: Distributing Logic Across Clock Cycles**
   - The computation is too deep to fit in a single clock cycle, so we distribute the operations across **multiple clock cycles**.
   - Breakdown of the pipeline stages:
     - **Cycle 1**: Square `a` and `b` and store these intermediate results in flip-flops.
     - **Cycle 2**: Add the squared values and store the result in another set of flip-flops.
     - **Cycle 3**: Take the square root of the sum. (In reality, calculating the square root might take multiple cycles, but for simplicity, it's assumed to take one cycle here.)
   - Each clock cycle advances the computation one step further, with intermediate values being captured in flip-flops to prevent loss of data across cycles.

#### 3. **RTL Approach (Register Transfer Level)**:
   - In traditional **RTL design**, you explicitly define the logic operations and the **flip-flops** that capture intermediate results.
   - For example:
     - **Cycle 1**: The squaring of `a` and `b` is computed, and the results are stored in flip-flops.
     - **Cycle 2**: The addition of `a^2` and `b^2` is performed, and the sum is stored in flip-flops.
     - **Cycle 3**: The square root is computed and stored.
   - While functional, RTL requires careful management of timing to ensure that each stage captures the correct results at the right clock edge, leading to complex timing management.

#### 4. **TL-Verilog: Pipeline Abstraction and Simplification**
   - **TL-Verilog** introduces a higher-level abstraction called **pipeline stages**. Instead of manually coding flip-flops between stages, the pipeline abstraction in TL-Verilog automatically implies the presence of flip-flops.
   - For example, in TL-Verilog:
     - You define a pipeline called `calc`.
     - The pipeline has multiple stages: `stage 1`, `stage 2`, and `stage 3`.
     - In each stage, you specify the logic, and TL-Verilog automatically handles the flip-flops between stages.
   - **Pipeline abstraction** allows you to focus on the operations (e.g., squaring, adding, square rooting) without worrying about the low-level implementation details like flip-flop placement.

#### 5. **Example: Code and Logic Comparison**
   - The code in TL-Verilog mirrors the conceptual diagram of a pipeline. You define the pipeline stages, and within those stages, you describe the operations.
   - **Traditional RTL Code**:
     - You have to explicitly manage when the squaring, adding, and square-rooting happens, and manually add flip-flops between stages.
   - **TL-Verilog Code**:
     - The stages are defined abstractly, and the flip-flops are implied based on how you structure the stages.
   - This leads to **code reduction**, as TL-Verilog eliminates the need to manually code flip-flops and timing management. This reduction in code improves readability, decreases bugs, and speeds up the design process.

#### 6. **Timing Abstraction and Flexibility in TL-Verilog**:
   - The **main benefit** of TL-Verilog's timing abstraction is the separation of **function** from **timing**.
     - **Function**: The operations you want to perform (squaring, adding, square rooting) stay the same.
     - **Timing**: The pipeline stages that define when these operations occur (e.g., squaring in stage 1, adding in stage 2) can be adjusted independently without changing the logic of the circuit.
   - This abstraction provides the flexibility to **change the timing** of operations (i.e., adjust the number of pipeline stages) without affecting the overall behavior of the design.
   - In traditional RTL, changing the number of cycles in a design would require **significant rewiring** of the flip-flops and logic, increasing the chance of introducing bugs.

#### 7. **Why Adjust Pipeline Staging?**
   - **Design Adaptation**: Imagine the signal `a` is located on one corner of the chip, and the result `c` is needed on the opposite corner. In modern silicon, it can take **multiple clock cycles** (e.g., 25 cycles) for a signal to propagate across the die.
   - **Timing Flexibility**: TL-Verilog allows you to stretch the pipeline by **adding stages** to account for this signal propagation delay. For example, instead of completing the operation in 3 stages, you might need 5 stages to handle the delay.
   - The key point is that **the logic stays the same** (you are still performing the same squaring, adding, and square-rooting). The only difference is the **timing**—how long it takes for the operation to complete and how many stages are used.

#### 8. **Guarantee of Circuit Behavior**:
   - With TL-Verilog, changes to pipeline stages **guarantee that the behavior of the circuit remains the same**. The only risk is that timing might break if you try to consume data before it's been fully produced.
   - However, **the logic itself is unchanged**, and this abstraction reduces the chances of introducing bugs related to timing changes (which are common in traditional RTL design).

#### 9. **Challenges in Traditional RTL Design**:
   - In traditional RTL, changing pipeline timing (e.g., adding extra cycles) requires significant manual effort:
     - **Rewiring flip-flops**.
     - Adjusting logic to match the new timing requirements.
     - Ensuring the circuit continues to behave correctly after the changes.
   - These changes are **error-prone** and time-consuming, often introducing bugs if not carefully managed.
   - **TL-Verilog** simplifies this process by making timing changes much easier through **pipeline abstraction**, saving design time and reducing opportunities for mistakes.

---

### Key Takeaways:
- **Pipeline logic** helps break down deep computations into manageable stages spread over multiple clock cycles.
- **TL-Verilog** provides a **timing abstraction** that simplifies the design of pipelines, reducing code and bugs compared to traditional RTL.
- **Flexibility**: TL-Verilog allows for easy changes in the pipeline staging without affecting the functional behavior of the circuit.
- **Timing abstraction** separates the function of the logic from the timing, making the design more adaptable to different hardware environments (e.g., signal propagation delays).

#  Understanding Pipeline Benefits and Waveform Analysis

#### 1. **Pipelining for High Clock Frequency & Performance**
   - When your clock is running too fast for your logic to execute within a single clock cycle, **pipelining** is needed. 
   - **Pipelines** not only solve timing issues but also provide **performance benefits** by allowing designs to run at **higher clock frequencies**.
   - **Combinational logic** between flip-flops limits the clock speed. The **time between flip-flops** defines the **clock frequency**.
   - **Pipelining** divides the computation into smaller, manageable tasks spread across **multiple stages**, which allows the clock to run faster. By **shortening the logic path** between flip-flops, you can present a **new set of inputs** every clock cycle.

#### 2. **Throughput vs. Latency**
   - While pipelining increases the number of clock cycles (stages) it takes to compute a result (increased **latency**), it enables **higher throughput**.
   - Throughput improves because a **new set of data** can be processed every clock cycle. Therefore, **more data** is handled per second, even though individual results take longer to compute (due to multiple stages).

#### 3. **Pipeline Example: Pythagoras Theorem**
   - In this example, we are computing the distance `c` using the **Pythagoras theorem** (`c = sqrt(a^2 + b^2)`).
   - The logic is distributed across stages. For example:
     - **Stage 1**: Square values `a` and `b`.
     - **Stage 2**: Add the squared results.
     - **Stage 3**: Compute the square root of the sum.

#### 4. **Waveform Viewer in Makerchip**
   - Makerchip provides a **waveform viewer** to visualize the pipeline behavior over time.
   - In a pipeline, **data is distributed across time**, meaning that inputs at an earlier stage impact outputs at a later stage.
   - The **waveform viewer** lets you track how inputs (e.g., `a` and `b`) at different stages of the pipeline correspond to outputs (e.g., `c`) a few clock cycles later.

#### 5. **Understanding Timing and Signal Alignment**
   - **Combinational Logic (Single Cycle)**: In the example, if all computations are done in **one cycle** (non-pipelined), the logic to compute `c` (squaring `a` and `b`, adding, and taking the square root) occurs in a single clock cycle.
     - In the waveform viewer, you would see that inputs `a = 9` and `b = 12` (represented as `C` in hexadecimal) result in an output `c = F` (hexadecimal) in the same cycle.
   - **Pipelined Logic**: When pipelined, the logic is spread across **multiple stages**.
     - For example, in a 3-stage pipeline, you would see:
       - **Stage 1**: Inputs `a` and `b`.
       - **Stage 2**: The intermediate result from squaring `a` and `b`.
       - **Stage 3**: The final output `c` two clock cycles later.
     - The pipeline adds **flip-flops** between stages to store intermediate results and propagate data.

#### 6. **Tagging Signals by Stage**
   - In the waveform, each signal is **tagged** with the stage at which it was generated (e.g., `@1` for stage 1, `@3` for stage 3).
   - For instance:
     - Input `a` in **Stage 1** (tagged `@1`) affects the output `c` in **Stage 3** (tagged `@3`) two clock cycles later.
   - This **stage tagging** helps you understand how **intermediate results** progress through the pipeline and when final results appear.

#### 7. **Pipelining in TL-Verilog**
   - **TL-Verilog** simplifies pipeline design by allowing you to focus on the **logical operations** at each stage without manually managing flip-flops.
   - In TL-Verilog, a signal like `a_squared` in stage 1 automatically gets propagated to later stages (e.g., stage 2) through flip-flops, which are implied by the pipeline structure.
   - The **timing abstraction** in TL-Verilog treats these signals as part of a **single pipeline**, but under the hood (SystemVerilog level), they are **separate signals** for each stage.

#### 8. **Example: Retiming**
   - **Retiming** adjusts the **position of flip-flops** in the pipeline to better distribute the logic across stages.
   - For example, if `a_squared` is computed in stage 1 and needs to be used in stage 2, a flip-flop is placed between the stages to hold the result of `a_squared` and propagate it to the next stage.

#### 9. **Sequential Logic and Feedback in Pipelining**
   - Beyond pipelining, **sequential logic** involves **feeding back** data from later stages to earlier stages (e.g., Fibonacci sequence).
   - For instance, if a feedback loop is added to the pipeline (e.g., feeding back the value of `a`), it allows the current stage’s logic to depend on results computed **several cycles earlier**.
   - In this case, **delayed versions** of signals (e.g., `a_4`, `a_12`) are used, indicating data that has passed through the pipeline for several stages.

#### 10. **Visualizing Feedback and Pipeline Depth**
   - In the Makerchip platform, you can visualize **feedback loops** and track how signals are staged through multiple flip-flops.
   - The feedback loop might refer to the version of a signal from **4, 12, or more cycles ahead**, creating complex interactions across different stages.

#### 11. **Benefits of Pipeline Diagrams**
   - **Pipeline diagrams** help in visualizing how **flip-flops** are implied in your design and how data flows across different stages.
   - These diagrams are useful for understanding:
     - Where logic operations are occurring.
     - How flip-flops store and propagate data.
     - How the feedback mechanism works, especially in sequential logic.

### Key Takeaways:
- **Pipelining** allows designs to run at **higher clock frequencies** by dividing computations into smaller tasks across multiple stages.
- The **throughput** of the design improves with pipelining, even though individual computations take longer (increased latency).
- **TL-Verilog** simplifies pipeline design by abstracting the timing of operations, reducing the complexity of managing flip-flops.
- The **waveform viewer** in Makerchip helps correlate inputs and outputs at different stages of the pipeline, allowing designers to track data flow over time.
- **Feedback loops** enable more complex sequential logic, where later-stage results are fed back into earlier stages for future computations.

## Pipeline Calculator Circuit Example in TL-Verilog

#### 1. **Starting Point: Calculator and Counter in Pipeline**
   - **Objective**: Integrate the calculator circuit you previously developed into a **named pipeline** (`calc`), specifically in **stage 1** of that pipeline. Also, incorporate the **counter** you developed into the same pipeline, **stage 1**.
   - Once both the **calculator** and **counter** are in the same pipeline stage, you can simulate the circuit to verify that both components work as expected in the context of the pipeline.

#### 2. **Next Steps: High-Frequency Operation Adjustments**
   - When dealing with a **high-frequency circuit**, you may need to break down computations over multiple clock cycles for better timing closure.
   - **Calculator Circuit Modifications**:
     - Split the calculator operation into **two stages**:
       - **Stage 1**: Perform the actual **arithmetic operation** (e.g., addition, subtraction, multiplication, or division).
       - **Stage 2**: Use a **multiplexer** to select the correct result based on the selected operation.
   - By breaking up the logic into two stages, the circuit can handle **higher frequencies**, ensuring that the timing of each stage is met.

#### 3. **Introducing Two-Cycle Latency**
   - Since the circuit now takes **two cycles** to perform a computation, you must handle the **input-output loopback** appropriately.
   - Modify the design so that:
     - The **output** is **looped back** to the input with a **two-cycle latency** (rather than one cycle).
     - This ensures that the computation’s result is available after two cycles and used as the next input, maintaining continuity in the iterative process.

#### 4. **Step-by-Step Breakdown of Circuit Changes**
   - **Step 1**: **Output Loopback**
     - Modify the **alignment of the output** so that it loops back with a two-cycle latency.
     - Initially, the **multiplexer** is still in the same stage as the arithmetic operations (addition, subtraction, etc.).
     - The loopback incorporates two **staging flops**, which hold intermediate results and recirculate them back to the input after two cycles.

   - **Step 2**: **Single-Bit Counter for Cycle Tracking**
     - You need to track whether the current cycle is a **computation cycle** or a **meaningless cycle** (where no valid computation occurs).
     - Use a **single-bit counter** to toggle between **0** and **1**:
       - This counter alternates between 0 and 1, keeping track of **even and odd cycles**.
       - This acts as an **oscillator** to determine which cycles should perform calculations (even cycles) and which cycles are idle (odd cycles).
     - The output of this circuit serves as a **valid signal**, which is `1` during computation cycles and `0` during meaningless cycles.

   - **Step 3**: **Valid Signal and Reset Logic**
     - Connect the **valid signal** along with the **reset signal** to determine when to drive the output with a **zero value**:
       - If the system is in reset or the cycle is invalid (odd cycle), the output should be driven to `0`.
       - This ensures that during idle cycles, the circuit produces a zero output, keeping the system stable during non-computation cycles.

   - **Step 4**: **Re-Timing the Multiplexer**
     - Move the **multiplexer** logic from **stage 1** to **stage 2**, finalizing the two-stage pipeline design.
     - By shifting the multiplexer, the selection of the arithmetic operation result happens in **stage 2**, while the actual operation occurs in **stage 1**.
     - After making these changes, you can verify the circuit’s behavior in the simulation.

#### 5. **Expected Behavior in Simulation**
   - The resulting pipeline will now perform a **calculation every other cycle**:
     - During **even cycles** (valid cycles), the circuit performs computations.
     - During **odd cycles** (invalid cycles), the inputs are meaningless, and the output is driven to zero.
   - In the **waveform viewer**, you should observe the calculator executing operations on **valid cycles** and outputting results every other cycle, with zero output during invalid cycles.

#### 6. **Key Concepts Reinforced in This Example**
   - **Pipeline Staging**: Dividing the computation into two pipeline stages allows the circuit to operate at **higher frequencies** by splitting the workload across clock cycles.
   - **Two-Cycle Latency**: The output is looped back with a two-cycle delay, reflecting the fact that the computation now takes two cycles to complete.
   - **Single-Bit Counter**: The use of a single-bit counter creates a simple oscillation to track which cycles are valid for computation.
   - **Valid Signal**: The **valid signal** ensures that computations only happen on designated cycles, and the circuit outputs zero during idle cycles.
   - **Multiplexer Re-Timing**: Moving the multiplexer to the second stage separates the operation from the selection, ensuring that timing constraints are met without overloading a single pipeline stage.
# Labs
![image](https://github.com/user-attachments/assets/9ee50ba5-da48-45a0-bb2c-eae6304d4e08)
![image](https://github.com/user-attachments/assets/90682776-171a-4910-8794-ecb26ca01006)
![image](https://github.com/user-attachments/assets/9ff380e3-fb31-4299-955f-4339d7023c60)
![image](https://github.com/user-attachments/assets/de0a94aa-9026-462f-b5ee-d6b186c03938)
![image](https://github.com/user-attachments/assets/d76909fb-3a7f-471b-8290-d85e07d4f133)
![image](https://github.com/user-attachments/assets/7eb0357c-a8d1-4088-9b06-e12921b7687d)
![image](https://github.com/user-attachments/assets/f7920203-dff4-45b4-9e1b-eb356442678a)



