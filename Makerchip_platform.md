# Makerchip Platform
Understanding the Multiplexer
## Multiplexer (MUX): A Fundamental Digital Circuit
A Multiplexer (or MUX) is a combinational circuit that selects one of multiple input signals and forwards it to the output. It acts like a digital switch, controlled by select lines.

#### 🛠️ 2-to-1 Multiplexer (Basic MUX)
Inputs:

x1, x2: Two data inputs (single-bit values)
s: Select signal (1 bit)
Output:

f: The chosen output, based on the select signal
#### Boolean Expression 
![image](https://github.com/user-attachments/assets/ddd00e90-3e08-4bf5-ac00-9f990564d064)

![image](https://github.com/user-attachments/assets/5cb7277d-da50-4c41-b7fa-06b4ab04fb4c)
### 👉 Verilog Implementation (Ternary Operator):
```
assign f = s ? x1 : x2;
```

![image](https://github.com/user-attachments/assets/cf9b5582-d0b9-4ac1-af9c-29555a493452)

The ternary operator (? :) acts like an if-else expression:

If s is true (1), take x1
Else take x2
 #### 4-to-1 Multiplexer (Extended MUX)
We can scale a MUX to handle more inputs. A 4-to-1 MUX selects one of four inputs (a, b, c, d) based on a 2-bit select signal.

Inputs:

a, b, c, d: Data inputs
sel[1:0]: 2-bit select signal
Output:

f: Chosen input, routed to output
#### 👉 Why Ternary Chains Work Best:

- Compact syntax
- Readable formatting
- Efficient synthesis in hardware
# 🚀 Makerchip IDE: TL-Verilog Integration
Makerchip is an online IDE designed for TL-Verilog — a streamlined hardware design language. It supports coding, visualization, and debugging, all in one place.

#### 🔧 Makerchip Features Overview
✅ Code Editor: Write TL-Verilog directly in your browser.

✅ Live Simulation: Runs your design on the cloud and generates waveforms.

✅ Circuit Diagram Generator: Visualizes your circuit from TL-Verilog code

✅ Waveform Viewer: Allows you to trace signals in time.

✅ Debugging Tools:

Trace errors in the code editor
Navigate to problematic expressions
✅ Save & Clone Projects: Generates a shareable URL (bookmark it to save progress).

✅ TL-Verilog Documentation: Built-in resources and examples.
![image](https://github.com/user-attachments/assets/759e7cc8-4d9b-4ac2-9e04-e51306d5938b)
### Navigate to examples
![image](https://github.com/user-attachments/assets/670fb0a2-f054-4f8a-942e-4d7e5a883c85)
### After this
![image](https://github.com/user-attachments/assets/0536dee3-7be3-47ac-bb07-17d5e8f2e11b)
![image](https://github.com/user-attachments/assets/f66b48dd-f518-4bab-bac8-54415bb15a68)
![image](https://github.com/user-attachments/assets/a544bbbd-4d2e-4cdf-b25a-90bc9653fa9d)
![image](https://github.com/user-attachments/assets/872b5c9f-a581-4072-ab06-b758e902e0a4)
