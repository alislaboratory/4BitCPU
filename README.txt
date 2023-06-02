My Mk1 4-bit CPU. This is meant to be a precursor to my more complex Mk2 CPU.

-- INSTRUCTION SET --
	This 4-bit CPU will have a capacity of 16 instructions (4-bit opcodes). There are 2 4-bit operands. Overflow and borrowing are disregarded in operations, only in JMPC, JMPB and JMP0 .
	The ALU will have 3-bit opcodes, so 8 comparator and arithmetic instructions. 
	Instructions passed to the ALU are marked with '(ALU)'.
	Active operations are marked with an (ACT).
	Undefined operations have been marked with (UDEF), they will result in no operation.

	NOP 0000 (ALU) - idle operation. Result will always be 0000. (done)
	ADD 0001 (ALU)(ACT) - addition operation. Results will be A+B. (done)
	SUB 0010 (ALU)(ACT) - subtraction operation. Results will be A-B . (done)
	SHR 0011 (ALU)(ACT) - shift A right by 1. (TBI)
	SHL 0100 (ALU)(ACT) - shift A left by B. Will take B clock cycles for result (TBD)
	MOV 0101 - move contents of A in memory to B in memory. (TBD)
	LDA 0110 - Load contents of B in memory to register A. (TBD)
	LDB 0111 - Load contents of A in memory to register B.
	WRX 1000 - Write the contents of result register X to memory at address in input register A.
	JMP0 1001 - Jump to instruction at memory address A if data at memory address B is 0000.
	JMPC 1001 - Jump to instruction at memory address A if ALU in most recent active operation resulted in addition overflow (carry).
	JMPB 1010 - Jump to instruction at memory address A if ALU in most recent active operation resulted in subtraction overflow (borrowing).

	UDEF 1011
	UDEF 1100
	UDEF 1101
	UDEF 1110
	UDEF 1111



	There can be a capacity for future operations such as multiplication and dividing, but at the moment these are user defined. In the Mk2, there should be more capacity for complex operations, as in this computer there are only 4 bits, so multiplication and division is not necessary and using while loops with ADD and JMP is more efficient for myself.

	Note: an 'active' operation refers to an operation that requires an output. For example, ADD is an active operation, while MOV and the LD operations are active. A list of active operations is defined above. The X accumulator register and the B and C ALU flags will not be overwritten until there is a new active operation. All ALU operations are active.


-- MEMORY --
	= INTERNAL MEMORY =
	The A,B, and X registers exist in the CPU for when instructions are executed. A & B are both input registers, and X is the accumulator, storing the result of the most recent active operation. There is also a B and C flag from the ALU that will be 1 when either borrowing or carrying occurred on the most recent active operation.

	

	Regarding external memory, there are 16 4-bit registers for general purpose access. 

	Registers are constructed of D flip flops. One of the problems I had with shift registers was that I was using D latches, not flip flops, so the values inputted would instantaneously move through all flip flops and overwrite the data. I need a flip flop, not a latch. The main difference is that a flip flop only stores the D value on the rising edge of the clock, not while the clock pulse is high.

	Behaviour of 4-bit register: When W flag is low, the value will be stored no matter what the input is. When W is high, the data in the register will be replaced on rising clock pulse.

	The RAM module will have a 4-bit register input, a 1-bit mode select, a 4-bit data input, and a 4-bit data output.

	When the mode select is low (default read mode), the data stored at the address in given by the register input will be at the 4-bit data output. The data input is disregarded - it has no effect on the output.
	When the mode select is high (write mode), the address at the register input will be overwritten with the data from the data input at the (rising edge or high) clock pulse. The output is (0000 or the new/old output).

	The architecture of the RAM look-up table will be a 4:16 multiplexer i.e there will be 4 select bits that will set the result demultiplexer which all memory units are connected to for that memory address. The write mode will work the same way but with a demux at the start that takes the 4-bit write and reroutes it to the memory address.

	

-- TODO --
	- Create B subtraction borrowing.
	- Create X register
	- Create lookup table architecture (4:16 multiplexer)
	- Create a bidirectional shift register



