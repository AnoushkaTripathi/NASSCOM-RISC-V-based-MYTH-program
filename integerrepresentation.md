#  Integer Number Representation

### Introduction to 64-bit Binary Numbers:
- **Decimal vs. Binary**: Computers understand binary, while humans understand decimal. A conversion mechanism is required to translate between these two systems.
- **Example**: A 64-bit binary number is used to represent data in computer systems, such as instructions processed by a CPU.
- **Importance**: Understanding binary number representation is crucial for understanding how data is stored, transferred, and manipulated in a system.

### Basic Structure of 64-bit Representation:
- **Word Definition**: A word in computer systems refers to a fixed-size unit of data, commonly 32 or 64 bits.
- **Least Significant Bit (LSB)**: The lowest bit in a binary number.
- **Most Significant Bit (MSB)**: The highest bit in a binary number.
- **Byte**: A byte consists of 8 bits. Four bytes make up a word, and two words make up a double word.

### Unsigned Binary Representation:
- **Formula**: Using `n` bits, the number of possible patterns is `2^n`. For example, using 3 bits, the maximum number is 2^3 = 8 patterns, ranging from 0 to 7.
- **64-bit Representation**: For a 64-bit system, there are `2^64` possible patterns, which results in a huge range of numbers.

### Understanding Large Numbers:
- **Representation**: Large binary numbers are broken down into their individual bit values and summed to represent the decimal equivalent.
- **Overflow Flag**: When performing arithmetic on unsigned numbers in a 64-bit system, an overflow flag will be triggered if the result exceeds the maximum representable number.

### Maximum Unsigned Number in 64-bit:
- The largest number that can be represented in a 64-bit unsigned system is `2^64 - 1`.

### Signed vs. Unsigned Numbers:
- **Unsigned Numbers**: These are always positive and range from 0 to a maximum value (e.g., `2^64 - 1`).
- **Signed Numbers**: Signed numbers can represent both positive and negative values, and a special mechanism (like two's complement) is used to distinguish between the two. The discussion of signed numbers is introduced but will be covered in more detail later.

### Negative Numbers and Two's Complement:
1. **Two's Complement Representation**: 
   - A standard method to represent negative numbers.
   - To represent a negative number, complement all the bits (invert 0s to 1s and 1s to 0s) and then add 1.

2. **MSB (Most Significant Bit)**:
   - In the two's complement system, the **MSB** (most significant bit) indicates the sign of the number.
   - **MSB = 0**: Positive number.
   - **MSB = 1**: Negative number.

3. **Converting Negative Numbers**:
   - Example: For a 64-bit number, find its complement, invert all bits, and then add 1.
   - Negative numbers are computed by inverting bits (1’s complement) and then adding 1.
   - The final step is to check the MSB and apply the relevant steps to determine if the number is negative.

4. **Binary Representation**:
   - Positive binary numbers are represented as they are.
   - Negative binary numbers involve taking the two's complement.

5. **Range of Positive and Negative Numbers**:
   - For a 64-bit system, the range of positive and negative numbers is determined by the number of bits.
   - The largest **positive** number that can be represented is `2^63 - 1`.
   - The largest **negative** number is `-2^63`.

6. **Understanding Binary to Decimal Conversion**:
   - For positive numbers, convert the binary representation directly.
   - For negative numbers (identified by MSB = 1), apply the two’s complement process and multiply by `2^(n-1)` (where `n` is the number of bits) to get the final negative value.

7. **Special Observations**:
   - The MSB is crucial in identifying if the number is positive or negative.
   - Positive numbers are directly represented, while negative numbers follow the two’s complement approach.

8. **Range of Numbers in RV64 Architecture**:
   - Positive numbers: Start from 0 up to `2^63 - 1`.
   - Negative numbers: Start from `-2^63` to `-1`.

### Application in Integer Instructions (RISC-V Architecture):
1. **Handling Integers in RISC-V**:
   - RISC-V follows these principles for integer instructions.
   - The system distinguishes between positive and negative numbers using the MSB.
   - Integer arithmetic instructions work with signed and unsigned numbers by leveraging the two's complement system.
