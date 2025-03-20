# Introduction to Logic gates

🚀 Digital Logic Fundamentals

Digital logic boils down to ones and zeros — the simplest form of information. Let’s explore how we manipulate these bits using logic gates.

### 🔧 Logic Gates

These are the core building blocks of all digital circuits. Each gate performs a basic logic function.
![image](https://github.com/user-attachments/assets/5d48018c-4dd3-4a65-889d-0a6a7d8cddef)
### 🔨 Combinational Logic

*Combinational logic circuits* produce outputs based solely on the current inputs. No memory or state is involved.

Example: Full Adder

Let’s build a simple 3-input full adder that computes:

Inputs: A, B, Cin (carry-in)

Outputs: S (sum), Cout (carry-out)
![image](https://github.com/user-attachments/assets/e9e4b6a4-2215-41da-9295-cddc36df31da)
![image](https://github.com/user-attachments/assets/ebfbee27-6328-45d7-a812-4633c05d55be)

### ⏳ Sequential Logic

Unlike combinational logic, sequential circuits rely on clocks. This introduces state and allows data to flow through time.

Registers store values on clock edges
![image](https://github.com/user-attachments/assets/beaad135-9637-4e22-8c87-133ed5fa214c)



### 🛠️ Pipeline Logic

Pipeline logic distributes computation over multiple cycles, enhancing performance. Each stage processes part of the data, and outputs flow from stage to stage on clock edges.

### 🏗️ Hierarchical Design

We can build small modules (like the full adder) and reuse them as building blocks to create more complex circuits — think LEGO for hardware design!
![image](https://github.com/user-attachments/assets/8555eb5c-1343-452d-b4ee-093040893933)

### TL-Verilog Syntax Overview

Depending on whether you operate on 1-bit or multi-bit data, syntax may vary. Here’s a quick comparison:
![image](https://github.com/user-attachments/assets/4c3e7d89-d0fc-480e-bdbf-93ae009e3940)
