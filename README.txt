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
	
	SWP 1000 
	WAB 1001 
	LDA 1010 
	JMF 1011
	WRX 1100
	UDF 1101- Jump to instruction at memory address A if contents at memory address B is 0000.
	UDF 1110- Write the contents of result register X to memory at address in input register A. (done()
	UDEF 1111
	


	There can be a capacity for future operations such as multiplication and dividing, but at the moment these are user defined. In the Mk2, there should be more capacity for complex operations, as in this computer there are only 4 bits, so multiplication and division is not necessary and using while loops with ADD and JMP is more efficient for myself.
	There is also a capacity for JMPC (jump if carry) and JMPB (jump if borrow) operations but these are not implemented yet. If they are, they will be at the last two undefined instructions.

	Note: an 'active' operation refers to an operation that requires an output. For example, ADD is an active operation, while MOV and the LD operations are active. A list of active operations is defined above. The X accumulator register and the B and C ALU flags will not be overwritten until there is a new active operation. All ALU operations are active.

	30/06: Control unit rehaul. Previous instruction set found in previous versions of the document


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

	Jump instructions: since I am making my instruction memory now 12 bits, I cannot address it with 4-bits in a single clock pulse. It would take 16 clock pulses
	for one jump instructions. So, I have two solutions:
	1: Relative Addressing
	I can have jump instructions that simply go 5/10/n instructions before and after. I can design my programs around this and it would give a unique challenge 
	for creating cool programs.
	2: Jump table
	I can have a memory table that is customisable for programs so that 16 12-bit addresses can be fetched. I can even use this on top of the relative addressing,
	as loops are a big part of software programs.

	For now, I will implement the relative addressing instruction. I'll do a 5 instruction jump. I can make one instruction that handles forward and backward jumping,
	as the instruction only uses one register. The second can control forward or backward - if the LSB is 0, jump back. Otherwise, jump forwards. This way, I could even 
	save memory by using other addresses already being used for other operations that will control forward and backward jumping in instruction memory.



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

	= PROGRAM COUNTER =
	I decided to redesign my program counter again. I've decided to my instruction memory 12-bits now, so capable of holding 4096 bits and therefor around 340 instructions.
	So, an adder circuit would now be stupid as there would be literally hundreds or even thousands of gates. I think I should use a standard binary counter with JK flip-flops.
	It doesn't mean its not homemade, as I can still learn about it. But first, I think it's always good to take a shot before we copy online. 
	I had a look at the clock pulse diagram, and for each Q_n, it pulses halve the amount in x clock pulses for Q_(n-1). This way, we can use two flip flops before each consecutive output.

-- PATHWAYS --
Concurrently with designing my Mk2 CPU, I would like to attempt to create my 4-bit CPU on breadboards. To do this, I will need testing equipment such as oscilloscopes and logic
analysers. A supposed course of action is to get in touch with CSU, UTS or some other university. Ms. Rainger also suggested Core Electronics or Paktronics but I don't think they have actual chip fabs. They could have connections or sponsor equipment for me.
First of all, I should get the CPU fully working. Once this is done, I will begin work on my
Mk2 CPU while beginning the hardware step of this project. Mrs. Rainger also had an old contact who used to work for a chip fab company.
	


	

-- TODO --
	- Create B subtraction borrowing.
	- Create a bidirectional shift register
	- Jump instructions
	- Implement high write on new active operation.
	- Implement active on control unit so perpetual swapping doesn't occur.
	- Make GP and progmem memory
	- Implement jump instructions
	- Fix progcounter for non-rising edge clock pulse so one clock pulser per count
	- Fix progcounter for set





