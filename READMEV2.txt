Mk1 4-Bit CPU - Datasheet V0.2

== Instruction Set ==
	There are 4-bit opcodes, so there is a capacity for 16 unique instructions.
	Note: ACT refers to an active operation, that changes the value of the accumulator
	Note: MEMOP refers to an operation that changes the value of registers A or B.

	BINA HX NME 	DESC			(ID)
	0000 00 NOP Idle operation. No changes in any variables.
	0001 01 ADD Addition operation. Add registers A and B (A+B) (ACT)
	0010 02 SUB Subtraction operation. Subtract B from A (A-B) (ACT)
	0011 03 SHR Shift A left by one. (ACT)
	0100 04 SHL Shift A right by one. (ACT)
	0101 05 UDF 
	0110 06 UDF 
	0111 07 UDF

	1000 08 SWP Swap contents of registers A and B. (MEMOP)
	1001 09 WAB Write operand 1 into A and operand 2 into B. (MEMOP)
	1010 0A LDA Load contents at memory address in register A. (MEMOP)
	1011 0B UDF
	1100 0C JMF Decrement/increment program counter by 5
	1101 0D UDF 
	1110 0E WRX Write the contents of accumulator to memory address in register A.


