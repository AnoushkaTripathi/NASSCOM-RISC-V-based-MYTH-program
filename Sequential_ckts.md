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

   - The **>>1** (previous) and **>>2** (previous previous) notations are used to refer to past values for computation.
# Labs
11. **Exercise: Free-Running Counter:**
    ![image](https://github.com/user-attachments/assets/6d025fcd-ae17-482a-a925-73406f09e167)

   - A free-running counter starts with a value of zero.
   - Each clock cycle increments the count by 1.
     ![image](https://github.com/user-attachments/assets/87054e9d-4462-470e-9303-2094ba62bc8f)

     ![image](https://github.com/user-attachments/assets/d7569c90-ca84-4ea2-8245-d16cf8705121)

###  Expressing Values in Verilog

1. **Verilog Value Notation:**
   - In hardware, values can have varying bit widths, unlike software where values are typically 32 or 64 bits.
   - Verilog allows you to explicitly specify the bit width of a value:
     - The format is: `<number of bits>'<format><value>`.
     - Formats include:
       - `d` for decimal.
       - `h` for hexadecimal.
       - `b` for binary.
     - Example: `8'd15` (8-bit decimal value 15), `4'b1010` (4-bit binary value 1010).
   ![image](https://github.com/user-attachments/assets/6405d6ee-367d-4897-9d25-12be0554d41f)

2. **Shortcuts in Verilog:**
   - Using a single quote and 0 (`'0`) lets the compiler determine the number of bits required for the value.
   - `x` represents a "don't care" value, which can propagate through the circuit to indicate an unknown state. This is helpful for debugging as the `x` can signify issues in logic if it appears downstream.

3. **Handling Values Without Specific Bit Width:**
   - If no bit width is provided, Verilog assumes a 32-bit integer value.
   - This is typically acceptable in logic for simple calculations, like in the Fibonacci series or counter circuit examples.

4. **Synthesis and Simulation Considerations:**
   - Tools for synthesis (hardware design) and simulation (testing) may handle value assignment and bit width differently.
   - In the tools we use (MakerChip with Verilator), values may be extended or truncated automatically to fit the bit width of the signal, without generating warnings.
   - This could lead to issues where bits are lost or values are incorrectly assigned, so it's important to carefully manage bit widths.

5. **Verilator Simulator:**
   - Verilator is the open-source simulator used in MakerChip.
   - It supports only two-state simulations (0s and 1s) and does not simulate "don't care" (x) values.

---

### Calculator Example with Flip-Flop (Sequential Circuit)
![image](https://github.com/user-attachments/assets/f2cd35c0-9ea5-457b-aaa1-8e1a4042f7cc)

1. **Enhancing the Calculator Circuit:**
   - The existing calculator will be updated to mimic a real calculator, which can store and use previous values for new operations.
   - To implement this, a flip-flop will be introduced to remember the last computed value.

2. **Sequential Logic with Flip-Flop:**
   - A flip-flop will store the result of the previous computation, serving as the memory.
   - The new value in the calculator will be computed using the stored value from the flip-flop.
   - For example, if the previous value is 10 and you want to add 2, the flip-flop will store 10, and the next cycle will compute 10 + 2.

3. **Reset in Calculator Circuit:**
   - A reset signal will be added to clear the stored value, just like the "Clear" button on a traditional calculator.
   - On reset, the value held by the flip-flop will return to 0, and the next operations will start from a zeroed-out state.

4. **Lab Exercise Solution:**
   - Modify the calculator to store and use the last computed value using a flip-flop.
   - Ensure the reset functionality is properly implemented to initialize the circuit to 0 when needed.



