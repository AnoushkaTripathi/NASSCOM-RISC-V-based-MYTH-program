### Notes on Sequential Logic and D Flip-Flop

1. **Introduction to Sequential Logic:**
   ![image](https://github.com/user-attachments/assets/4ffdcfe5-a341-4845-8279-a15fd21d5dda)

   - Sequential logic introduces a clock to sequence the operations.
   - The clock helps to propagate the logic, making the timing of operations deterministic.

3. **D Flip-Flop:**
   - Simplest type of flip-flop, often referred to simply as a "flip-flop."
   - Holds on to a state, either a 1 or a 0, based on the clock signal.
   - On the rising edge of the clock, the value propagates through to the next state.
   - The flip-flop holds onto this value until the next clock cycle.
   
4. **Importance of Reset in Sequential Circuits:**
   - Reset signal is used to bring the circuit into a known state (like booting or restarting a system).
   - Many flip-flops have a reset input, which sets the state to either 1 or 0.
   - After reset, the circuit can continue its operations.

5. **Sequential Circuits as State Machines:**
   ![image](https://github.com/user-attachments/assets/e2052cd6-e69f-47ea-8686-a976286d3e9e)

   - Sequential circuits can be viewed as a big state machine.
   - Combinational logic propagates the input bits to output bits.
   - These output bits are then interpreted as the next state.
   - With every clock cycle, the state propagates, and the circuit performs computations based on the updated state.

7. **Fibonacci Series Circuit Example:**
   - The Fibonacci series is computed by sequentially summing two previous values.
     ![image](https://github.com/user-attachments/assets/9030c3d2-a2d4-4ef6-844b-a37017ef0f16)

   - The circuit holds two values in flip-flops (e.g., 3 and 2), sums them, and propagates the sum on each clock cycle.
   - The result of this summation is used to compute the next value in the Fibonacci series.

8. **Reset in Fibonacci Circuit:**
   ![image](https://github.com/user-attachments/assets/7d621313-7942-4815-b73d-0142c5499782)

   - Reset injects initial values (ones) into the circuit to start the Fibonacci sequence.
   - When reset is asserted, the circuit is initialized with the values needed for the sequence.

10. **TL Verilog Representation of the Fibonacci Circuit:**
   - In TL Verilog, the circuit can be expressed using logic expressions.
   - If reset is asserted, inject a one; otherwise, propagate the sum of the current value and the previous one.
     ![image](https://github.com/user-attachments/assets/2c060622-083b-44ef-a6fa-61090f1ccb13)

   - The **>>1** and **>>2** notations are used to refer to past values for computation.
# Labs
11. **Exercise: Free-Running Counter:**
    ![image](https://github.com/user-attachments/assets/6d025fcd-ae17-482a-a925-73406f09e167)

   - A free-running counter starts with a value of zero.
   - Each clock cycle increments the count by 1.
   - This exercise allows students to practice building a simple sequential circuit similar to the Fibonacci circuit but with incrementing values.

11. **Maker Chip Tool:**
   - Students can use the Maker Chip platform to simulate and test their circuits.
   - The Fibonacci circuit is provided as a reference, and students are encouraged to create their own adder circuit using the tool.
