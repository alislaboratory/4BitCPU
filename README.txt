My Mk1 4-bit CPU. This is meant to be a precursor to my more complex Mk2 CPU.

-- INSTRUCTION SET --
	This 4-bit CPU will have a capacity of 16 instructions (4-bit opcodes). There are 2 4-bit operands. Overflow and borrowing are disregarded in operations, only in JMPC, JMPB and JMP0 .
	The ALU will have 3-bit opcodes, so 8 comparator and arithmetic instructions. The first 8 instructions belong to the ALU to make space for new ALU functionality.
	i.e any opcode with 0 as MSB (most significant bit) is an ALU instruction. any opcode with 1 is a pure CPU instruction.
	Instructions passed to the ALU are marked with '(ALU)'.
	Active operations are marked with an (ACT).
	Undefined operations have been marked with (UDEF), they will result in no operation.

	Control unit: for operations requiring access to external memory, 2 inner bits will be used as select lines for the 16 to 4 mux.

	NOP 0000 (ALU) - idle operation. Result will always be 0000. (done)
	ADD 0001 (ALU)(ACT) - addition operation. Results will be A+B. (done)
	SUB 0010 (ALU)(ACT) - subtraction operation. Results will be A-B . (done)
	SHR 0011 (ALU)(ACT) - shift A right by 1. (done)
	SHL 0100 (ALU)(ACT) - shift A left by 1. (done)
	UDEF 0101
	UDEF 0110
	UDEF 0111
	
	SWP 1000 - Swap values held in registers A and B. (done)
	LDA 1001- Load contents of B in memory to register A. (done)
	LDB 1010- Load contents of A in memory to register B. (done)
	UDEF 1011
	JMP 1100- Set program counter (jump) to instruction at memory address A.
	JMP0 1101- Jump to instruction at memory address A if contents at memory address B is 0000.
	WRX 1110- Write the contents of result register X to memory at address in input register A. (done()
	UDEF 1111
	


	There can be a capacity for future operations such as multiplication and dividing, but at the moment these are user defined. In the Mk2, there should be more capacity for complex operations, as in this computer there are only 4 bits, so multiplication and division is not necessary and using while loops with ADD and JMP is more efficient for myself.
	There is also a capacity for JMPC (jump if carry) and JMPB (jump if borrow) operations but these are not implemented yet. If they are, they will be at the last two undefined instructions.

	Note: an 'active' operation refers to an operation that requires an output. For example, ADD is an active operation, while MOV and the LD operations are active. A list of active operations is defined above. The X accumulator register and the B and C ALU flags will not be overwritten until there is a new active operation. All ALU operations are active.


-- MEMORY --
	= INTERNAL MEMORY =
	The A,B, and X registers exist in the CPU for when instructions are executed. A & B are both input registers, and X is the accumulator, storing the result of the most recent active operation. There is also a B and C flag from the ALU that will be 1 when either borrowing or carrying occurred on the most recent active operation. 
	

	Regarding external memory, there are for now 16 4-bit registers for general purpose access. However, the maximum instructions viable will realistically be 15 instructions so that at least one address
	is stored for data. Current ideas are to have separate memory modules for instructions and memory. This will work fine, as the instructions should really only be read-only at runtime, so a ROM should be made. This would actually be ideal and should've been implemented in the first place, as instructions can be modified on the go which is pretty detrimental to program safety.
	If this happens (very likely at the moment), there will be 16 general purpose registers, while program memory (PROGMEM) is held in 16 registers, so programs can only be up to 16 instructions long. This is really the maximum amount, and it would be silly to use time controls to get more registers as this is simply a stepping stone to an actual CPU that will not have these capacity issues.

	Registers are constructed of D flip flops. One of the problems I had with shift registers was that I was using D latches, not flip flops, so the values inputted would instantaneously move through all flip flops and overwrite the data. I need a flip flop, not a latch. The main difference is that a flip flop only stores the D value on the rising edge of the clock, not while the clock pulse is high.

	Behaviour of 4-bit register: When W flag is low, the value will be stored no matter what the input is. When W is high, the data in the register will be replaced on rising clock pulse.

	The RAM module will have a 4-bit register input, a 1-bit mode select, a 4-bit data input, and a 4-bit data output.

	When the mode select is low (default read mode), the data stored at the address in given by the register input will be at the 4-bit data output. The data input is disregarded - it has no effect on the output.
	When the mode select is high (write mode), the address at the register input will be overwritten with the data from the data input at the (rising edge or high) clock pulse. The output is (0000 or the new/old output).

	The architecture of the RAM look-up table will be a 4:16 multiplexer i.e there will be 4 select bits that will set the result demultiplexer which all memory units are connected to for that memory address. The write mode will work the same way but with a demux at the start that takes the 4-bit write and reroutes it to the memory address.



-- CPU BEHAVIOUR --
	= FETCH-EXECUTE CYCLE =
	This computer uses the fetch execute cycle. Each cycle, the program counter is incremented unless a jump instruction is encountered or there is a reset.
	The program counter starts at 0000. Whenever a jump instruction is encountered, the program counter is set to the address.

	The cycle will go as follows:
	Fetch & Decode:
	The value at the current program counter's address in progmem will be fetched. This is the opcode. Then, the program counter's address + 1 will be fetched from progmem. This is register A.
	Then, the program counter's address + 2 will be fetched from progmem. This is register B.
	Execute:
	The instruction is passed through either the ALU or CU as per the opcode. The resultant register X is held until the next active operation. 
	There is no store part in the cycle, this will be user defined. The value in X will be overwritten when the next active operation executes, so it is up to the user to make sure they are 
	storing their values.


	

-- TODO --
	- Create B subtraction borrowing.
	- Create a bidirectional shift register
	- Jump instrections
	- Implement high write on new active operation.
	- Create program counter registers
	- Implement active on control unit so perpetual swapping doesn't occur.
	- Decide whether there is separate or one memory module for instructions and general purpose memory.
	- Figure out program counter logic.
	- Where will opcodes and operands be stored?
	- Complete rewrite of memory architecture.
	- Learn about JK flip flops and make 4-bit counter




