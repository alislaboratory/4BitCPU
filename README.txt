My Mk1 4-bit CPU. This is meant to be a precursor to my more complex Mk2 CPU.

-- INTRODUCTION --
This is my first experience with low-level (machine code and assembly and logic gates) computing. I had only learnt logic gates a few weeks prior to beginning this project, and by then had a vested interest in creating my own bits and pieces. So I started with a 1-bit full adder. This became a 4-bit adder, which became a 4-bit subtractor, which then became an ALU, which then was added to a control unit to make a CPU... etc. Note I had nearly 0 clue about CPU architecture other than a very high-level understanding of the fetch-execute cycle and registers, ALU and control unit. I made pretty much every module myself just from the beginning logic gates, only through my own understanding and built my knowledge as I went through. I had no book, no YouTube tutorial, no course, just Logisim and a knowledge of boolean logic. The entire CPU architecture, instruction set and structure is completely my own, hence all the weird wiring patterns and naming schemes everywhere.

About half-way through the project, I realised how fun it was, and how accomplishing it felt to create the components, and eventually the whole CPU. But it isn't really usable, so my next project that is currently in progress, is my Mk2 16-bit CPU, with keyboard capability, I2C, a display, audio, and more. 
-- INSTRUCTION SET --
	This 4-bit CPU will have a capacity of 16 instructions (4-bit opcodes). There are 2 4-bit operands. Overflow and borrowing are disregarded in operations, only in JMPC, JMPB and JMP0 .
	The ALU will have 3-bit opcodes, so 8 comparator and arithmetic instructions. The first 8 instructions belong to the ALU to make space for new ALU functionality.
	i.e any opcode with 0 as MSB (most significant bit) is an ALU instruction. any opcode with 1 is a pure CPU instruction.
	Instructions passed to the ALU are marked with '(ALU)'.
	Active operations are marked with an (ACT).
	Undefined operations have been marked with (UDEF), they will result in no operation.

	Control unit: for operations requiring access to external memory, 2 inner bits will be used as select lines for the 16 to 4 mux.

	NOP 0000 0 (ALU) - idle operation. Result will always be 0000. (done)
	ADD 0001 1 (ALU)(ACT) - addition operation. Results will be A+B. (done)
	SUB 0010 2 (ALU)(ACT) - subtraction operation. Results will be A-B . (done)
	SHR 0011 3 (ALU)(ACT) - shift A right by 1. (done)
	SHL 0100 4 (ALU)(ACT) - shift A left by 1. (done)
	UDEF 0101 5
	UDEF 0110 6
	UDEF 0111 7
	
	SWP 1000 8 (MEMOP) - Swap contents of registers A and B. 
	WAB 1001 9 (MEMOP) - Write operand A to register A and operand B into register B.
	LDA 1010 a (MEMOP) - Fetch address at register A and write into register A.
	JMA 1011 b - Increment/decrement the program counter by value of operand A.
	WRX 1100 c - Write contents of accumulator to memory address in operand A.
	JMZ 1101 d - Decrement program counter by value of operand A IF accumulator is currently 0000. 
	HLT 1110 e - Halt CPU. Stops program counter from counting.
	UDF 1111 f 

	Temporary fix for small jump values: add opA + opB



	There can be a capacity for future operations such as multiplication and dividing, but at the moment these are user defined. In the Mk2, there should be more capacity for complex operations, as in this computer there are only 4 bits, so multiplication and division is not necessary and using while loops with ADD and JMP is more efficient for myself.
	There is also a capacity for JMPC (jump if carry) and JMPB (jump if borrow) operations but these are not implemented yet. If they are, they will be at the last two undefined instructions.

	Note: an 'active' operation refers to an operation that requires an output. For example, ADD is an active operation, while MOV and the LD operations are active. A list of active operations is defined above. The X accumulator register and the B and C ALU flags will not be overwritten until there is a new active operation. All ALU operations are active.

	30/06: Control unit rehaul. Previous instruction set found in previous versions of the document.

	03/10/32: Added something for both JMP and JMZ that OpA + OpB is now the jump location, allowing up to -32 relative jumping.


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

	Instruction memory: It is now 12-bit address and 12-bit data width - this allows for 2^12 = 4096 instructions. Each instruction is 12 bits, so this way it prevents having to use 3 clock cycles to fetch a single instruction if there were 4-bits - each address is
	split into 3 digit hex: xxx, where the MSB is the opcode and the the next two bits are operands 1 and 2 respectively. These will be split and fed into the control unit as necessary.

	I just had a slight problem with the program counter - since the output is a register, it is still performing the jump subtraction when it shouldn't be. A simple fix is to only allow through the subtrahend when the write pin, while also allowing program counter write
	on high write pin. Let's see if this fixes it. It did fix it, although I had to use my own twelve bit buffer since the inbuilt one is a tri-state so when off it is high impedance and breaks the subtractor.

	Note for future:  I may have to modify the instruction memory circuitry if I cannot find any 16-bit EEPROMs. A way around that is to use 8 bit EEPROM, with a module that handles writing (6-bits into one, 6 to the next). This will lower the amount of memory for
	ease of circuit creation. Then reading is the same process, reading the current address bits 5-0, then adding that with current address + 1 bits 5-0. For now, it can be ignored.

	There is a slight problem where loading requires 2 clock cycles to fetch the value of the address in memory. This can be fixed by using a counter that activates the program counter HALT for a single cycle while the value is fetched, then disabled the next clock cycle.

	3/10/23 - A quick temporary fix has been made with the jump instruction to utilise summing operands A and B, so now relative jumping can be up to -32.



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

	Program counter is done - uses consecutive D flip flops and then the S and R inputs are used for setting (jump instructions).


-- I/O --
No I/O has been implemented yet, except for a basic hex character at the output accumulator. Plans at the moment are: dot matrix display, and a key input. However, this is unlikely as the creation of the more versatile and actually practical 16-bit CPU has begun.

-- PROGRAMS --
Now that the CPU is finished (2/07/23), it's time to make some programs. The first ever one I made to test is as follows:
971 100 930 b30. This repetitively adds 1+7 and outputs. (the general purpose memory was not connected at this point).

Program 1: We are going to test a basic for loop which repetitively adds 0x2 in hex. This is the working code.
WAB 0 2 ; write 2 to register B
LDA 1 0 ; load the counter variable
NOP 0 0 ; wait for load
ADD 0 0 ; calculate the sum
WRX 1 0 ; write sum
WAB 6 0 
JMA 6 0 ; jump to start

Machine code: 902 a10 000 100 c10 960 b60
12-bits (3 hex digits) per instruction.

This is the first, functional, fully working program! It overflows at 0xF back to the beginning! I have worked so hard to get to this position!

Program 2: Making basic multiplication using a for loop. (work in progress)
Let's do a program for e.g 3 x 2 . Best programming practice should have the lowest number second to lower the number of cycles required.
Pseudocode:
Load the 1st multiplicand (3) into RAM | 0x0
Load the 2nd multiplicand (2) into RAM. | 0x1
Reserve a RAM address for counting. | 0x2
Reserve a RAM address for the final product. | 0x3
Jump to the end of the program if 0x1-0x2 == 0 (JMZ)
Add 3 to itself. -
Store this value to the accumulator
Increment counter
Unconditional jump back to 146

Can also halt at the end so it doesnt loop infinitely

930 WAB 3 0 ; write 1st multiplicand to 0x0
100 ADD 0 0 ;
c00 WRX 0 0 ;
920 WAB 2 0 ; write 2nd multiplicand to 0x1
100 ADD 0 0 ;
c10 WRX 1 0 ;
910 WAB 1 0 ; write 1 to 0x4
100 ADD 0 0 ;
a20 LDA 2 0 ; checking if reached the final var
000 NOP 0 0 ;
800 SWP 0 0 ;
a10 LDA 1 0 ;
000 NOP 0 0 ;
200 SUB 0 0 ;
df0 JMZ e 0 ; jump to end and halt
a00 LDA 0 0 ; add 3 to the current sum
000 NOP 0 0 ;
800 SWP 0 0 ;
a30 LDA 3 0 ;
000 NOP 0 0 ;
100 ADD 0 0 ;
c30 WRX 3 0 ;
LDA 2 0 ;
SWP 0 0 ;
WAB 
bf3 JMA f 3 ; jump back 18 to the beginning of the loop


	

-- TODO --
	- Create B subtraction borrowing.
	- Create a bidirectional shift register
	- Implement high write on new active operation.
	- Implement active on control unit so perpetual swapping doesn't occur.
	- Make GP and progmem memory
	- Fix progcounter for non-rising edge clock pulse so one clock pulser per count
	- Fix progcounter for set
	- Create bidirectional relative jumping
	- Implement jump instruction
	- Fix registers breaking for a split second 





