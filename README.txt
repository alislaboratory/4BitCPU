My Mk1 4-bit CPU. This is meant to be a precursor to my more complex Mk2 CPU.

-- INSTRUCTION SET --
This 4-bit CPU will have a capacity of 8 instructions (3-bit opcodes). There are 2 4-bit operands. Overflow and borrowing are disregarded in operations.
NOP 000 - idle operation. Result will always be 0000. (done)
ADD 001 - addition operation. Results will be A+B. (done)
SUB 010 - subtraction operation. Results will be A-B . (done)
SHR 011 - shift A right by B. Will take B clock cycles for result (TBD)
SHL 100 - shift A left by B. Will take B clock cycles for result (TBD)
MOV 101 - move contents of A in memory to B in memory. (TBD)
LDA 110 - Load contents of B in memory to register A. (TBD)
LDB 111 - Load contents of A in memory to register B.

One of the last two operations could be removed and left for other operations. This will depend on their usage when all operations are finished. 


-- MEMORY --
The A and B registers exist in the CPU for when instructions are executed. Regarding external memory, there are 16 4-bit registers for general purpose access. Registers are constructed of D flip flops (or latches if necessary).
The RAM module will have a 4-bit register input, a 1-bit mode select, a 4-bit data input, and a 4-bit data output.

When the mode select is low (default read mode), the data stored at the address in given by the register input will be at the 4-bit data output. The data input is disregarded - it has no effect on the output.
When the mode select is high (write mode), the address at the register input will be overwritten with the data from the data input at the (rising edge or high) clock pulse. The output is (0000 or the new/old output).



