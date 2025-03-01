# 16-Bit-CPU-using-Verilog
Design of a 16-Bit CPU using Verilog

----
# Requirements
![image](https://github.com/user-attachments/assets/88c3290e-4e30-4884-8bfd-28f8dd252a47)


---
## Approach

To achieve a 16-Bit CPU design, we had to start by designing the individual components.
The components used were:

1. ALU
2. Arithmetic Unit
3. Logic Unit
5. Data Memory
6. Instruction memory
7. Program Counter
8. Instruction Register
9. Multiplexer
10. Controller
11. CPU

------


## The Instruction Set

The instruction set plays a very important role to determine the operation of the CPU. The Code input to the Instruction memory is 16-bit long. This is sent to the instruction register which takes the bits [15:12] as OPCODE and [11:0] as address to start with.

The instruction set are designed in a way to achieve all the necessary functions. Since the OPCODE controls the activity, the OPCODE is used as described below.

The last bit of the code input acts as a mode selection for ALU. Remaining 3 are used for the ALU operation.

<img width="223" alt="image" src="https://github.com/user-attachments/assets/46b2e802-6546-4956-b863-c92c96912454" />

------

## Flow

The Design is completed using a state machine approach. Three states are used to monitor the action of CPU. The three states used are reset, load and execute.

In reset state, we initializa all address to base address and code input to 0. All the signals going out from controller are 0.

In the load state, we initialise the Instruction register with the address and OPCODE to make sure the design is ready for execution.

In execute state, we provide the function required by the USER based in the Instruction set provided.

----

## CODE for Design

## ALU

/////////////////////////////////////////////////////////////////////  
//	This describes the ALU of the design			////  
////////////////////////////////////////////////////////////////////

module ALU( a, b, opcode, mode, outALU, za, zb, eq, gt, lt);  
input [15:0] a;  
input [15:0] b;  
input [2:0] opcode;   
input mode;   
output [31:0] outALU;   
output za, zb, eq, gt, lt;    
reg [31:0] outALU;  
reg za, zb, eq, gt, lt;  
wire [31:0] outau;   
wire [31:0] outlu;   
wire tza, tzb, teq, tgt, tlt;  

// Instantiation of the modules   
arith a1 (.a(a), .b(b), .opcode(opcode), .outau(outau));  
logic l1 (.a(a), .b(b), .opcode(opcode), .outlu(outlu), .za(tza), .zb(tzb), .eq(teq), .gt(tgt), .lt(tlt));   

// At every change of a, b, mode and opcode, we need to select the output.   
always@(a,b,mode,opcode)   
begin    
	if(mode == 0) begin    
	outALU = outau;    
	end   
	else if (mode == 1) begin   
	outALU = outlu;    
	end    
	else begin   
	outALU = 32'h00000000;   
	end   
	za = tza;  
	zb = tzb;   
	eq = teq;  
	gt = tgt;  
	lt = tlt;  
end   
endmodule   

## MUX
///////////////////////////////////////////////////////   
//// 		Design of Multiplxer		//////    
/////////////////////////////////////////////////////   

module muxA ( clk, in1, in2, sel, outA);   
input clk;   
input [31:0] in1;   
input [15:0] in2;   
input sel;   
output [31:0] outA;  
reg [31:0] outA;  

always@(posedge clk)   
begin  
	if ( sel == 1 ) begin   
	outA <= in1;  
	end  
	else if ( sel == 0) begin  
	outA <= {16'b0000000000000000 , in2};  
	end  
end  
endmodule  

## Register A  
////////////////////////////////////////////////////////////////////////////////////////////   
//	Program to realise a Register. The operation depends on Clock and load		///   
///////////////////////////////////////////////////////////////////////////////////////////   

module regA (clk, loadA, dataAin, dataAout);  
input clk;  
input loadA;  
input [15:0] dataAin;  
output [15:0] dataAout;  
reg [15:0] dataAout;  
reg [15:0] tempA;  

always@(clk,loadA)  
begin  
	@(posedge clk)   
	begin  
		if ( loadA == 1) begin  
		dataAout <= dataAin;  
		tempA <= dataAin;  
		end  
		else if (loadA == 0) begin  
		dataAout <= tempA;  
		end  
	end  
end  
endmodule  

## Regiser B
////////////////////////////////////////////////////////////////////////////////////////////  
//	Program to realise a Register. The operation depends on Clock and load		///  
///////////////////////////////////////////////////////////////////////////////////////////  

module regB (clk, loadB, dataBin, dataBout);  
input clk;  
input loadB;  
input [15:0] dataBin;  
output [15:0] dataBout;  
reg [15:0] dataBout;  
reg [15:0] tempB;  

always@(clk,loadB)  
begin  
	@(posedge clk)   
	begin  
		if ( loadB == 1) begin  
		dataBout <= dataBin;  
		tempB <= dataBin;  
		end  
		else if (loadB == 0) begin  
		dataBout <= tempB;  
		end  
	end  
end  
endmodule  

## Regiser C  
////////////////////////////////////////////////////////////////////////////////////////////  
//	Program to realise a Register. The operation depends on Clock and load		///  
///////////////////////////////////////////////////////////////////////////////////////////  

module regC (clk, loadC, dataCin, dataCout);  
input clk;  
input loadC;  
input [31:0] dataCin;  
output [31:0] dataCout;  
reg [31:0] dataCout;  
reg [31:0] tempC;  

always@(clk,loadC)  
begin  
	@(posedge clk)   
	begin  
		if ( loadC == 1) begin  
		dataCout <= dataCin;  
		tempC <= dataCin;  
		end  
		else if (loadC == 0) begin  
		dataCout <= tempC;  
		end  
	end  
end  
endmodule  

## DATA Memory 
//////////////////////////////////////////////////////////////////////  
//	Program to realise Datamemory in the CPU		/////  
////////////////////////////////////////////////////////////////////  

module datamem(clk, we_DM, dataDM, addDM, outDM);    
input clk;  
input we_DM;   
input [31:0] dataDM;   
input [11:0] addDM;  
output [31:0] outDM;  
reg [31:0] outDM;   

// Memory is an array stored at particular address  
reg [31:0] mem [0 : 31];  

always@(posedge clk)  
begin  
	if (we_DM == 1) begin  
	mem[addDM] = dataDM;  
	end  
	
       else if (we_DM == 0) begin  
	outDM = mem[addDM];  
	end  
end  
endmodule  

## PC   
///////////////////////////////////////////////////////////////  
/////	Module to realize a Program Counter. 		//////  
/////////////////////////////////////////////////////////////  
module PC(clk, loadPC, incPC, address, execadd);  
input clk;   
input loadPC;   
input incPC;  
input [11:0] address;  
output [11:0] execadd;   
reg [11:0] execadd;     
reg [11:0] temp;   
always@( posedge clk)   
begin  
	if ( loadPC == 0 && incPC == 0 ) begin  
	temp <= 12'h000;  
	end  
	else if (loadPC == 1 && incPC == 0 ) begin   
	temp <= address;  
	end   
	else if (loadPC == 0 && incPC == 1 ) begin  
	temp <= temp + 12'h001;   
	end  
	else begin  
	temp <= temp;  
	end  
	execadd <= temp;   
end   
endmodule   

## MUX1  
///////////////////////////////////////////////////////  
//// 		Design of Multiplxer		//////  
/////////////////////////////////////////////////////  

module muxB ( clk, in1, in2, sel, outB);  
input clk;  
input [11:0] in1;  
input [11:0] in2;  
input sel;  
output [11:0] outB;  
reg [11:0] outB;  

always@(posedge clk)  
begin  
	if ( sel == 1 ) begin  
	outB <= in1;  
	end  
	else if ( sel == 0) begin  
	outB <= in2;  
	end  
end  
endmodule  

## Instruction Memory 
//////////////////////////////////////////////////////////////////////  
//	Program to realise Instmemory in the CPU		/////  
////////////////////////////////////////////////////////////////////  

module instmem(clk, we_IM, dataIM, addIM, outIM);  
input clk;  
input we_IM;  
input [15:0] dataIM;  
input [11:0] addIM;  
output [15:0] outIM;  
reg [15:0] outIM;  

// Memory is an array stored at particular address  
reg [15:0] mem [0 : 15];  

always@(posedge clk)    
begin
	if (we_IM == 1) begin  
	mem[addIM] = dataIM;  
	end  

	else if (we_IM == 0) begin  
	outIM = mem[addIM];  
	end  
end 
endmodule  

## Instruction Register
////////////////////////////////////////////////////////////////  
//////	Instruction Register realization		///////  
///////////////////////////////////////////////////////////////  
module insReg ( clk, loadIR, insin, address, opcode);  
input clk;  
input loadIR;  
input [15:0] insin;  
output [11:0] address;  
output [3:0] opcode;   
reg [11:0] address;   
reg [3:0] opcode;  
reg [15:0] temp;  

always@(posedge clk)  
begin  
	if(loadIR == 1) begin  
	temp <= insin;  
	end  
address <= temp[11:0];  
opcode <= temp[15:12];  
end  
endmodule  

## CPU controller  
////////////////////////////////////////////////////////////////////////////////  
////////	Verilog Program for the controller of the CPU		///////  
//////////////////////////////////////////////////////////////////////////////  
module controller ( clk, en, opcode, loadA, loadB, loadC, loadIR, loadPC, incPC, mode, we_DM, selA, selB);  
input clk;  
input en;  
input [3:0] opcode;  
output loadA;  
output loadB;  
output loadC;  
output loadIR;  
output loadPC;  
output incPC;  
output mode;    
output we_DM;  
output selA;  
output selB;  
reg loadA;  
reg loadB;  
reg loadC;   
reg loadIR;  
reg mode;  
reg we_DM;  
reg selA;  
reg selB;  
reg loadPC;  
reg incPC;  
// Registers to hold the value of state and next state  

reg [2:0] state;  
reg [2:0] next_state;  
parameter reset = 3'b000, load = 3'b010, execute = 3'b100;  

//Code for operation of the CPU  
always@(posedge clk)  
begin  
	
	if ( en == 0 ) begin  
	state = reset;  
	end  
	else if (en == 1) begin  
	state = next_state;  
	end  
end  


// Now for the Output logic. Output logic depends on OPCODE and Enable signal   
always@(*)  
begin  
	if ( en == 0 ) begin  
	loadA = 0;  
	loadB = 0;  
	loadC = 0;  
	loadIR = 0;  
	loadPC = 0;  
	incPC = 0;  
	mode = 1'bZ;  
	we_DM = 0;  
	selA = 1'b0;  
	selB = 1'b0;  
	next_state = reset;  
	end  

	else begin  
		
		case(state)  
		// We just wait for a small duration of time in the same state to see if there is any change in input  
		reset: 	begin  
			loadA = 0;  
			loadB = 0;  
			loadC = 0;  
			loadIR = 0;  
			loadPC = 0;  
			incPC = 0;  
			mode = 1'bZ;  
			we_DM = 0;  
			selA = 1'b0;  
			selB = 1'b0;  
			next_state = load;  
			end  

		load:	begin  
			loadA = 0;  
			loadB = 0;  
			loadC = 0;  
			loadIR = 1;  
			loadPC = 1;  
			incPC = 0;  
			mode = 1'bZ;  
			we_DM = 0;  
			selA = 1'b0;  
			selB = 1'b0;  
			next_state = execute;  
			end  

		execute:begin  
			case(opcode)  
			// Mode 0, ALU operation for opcode 000  
			0000: 	begin  
				loadA = 0;  
				loadB = 0;  
				loadC = 0;  
				loadIR = 0;  
				loadPC = 0;  
				incPC = 1;  
				mode = 1'b0;  
				we_DM = 1;  
				#5 we_DM = 0;  
				selA = 1'b0;  
				selB = 1'b0;  
				end   
			// Mode 0, ALU operation for opcode 001   
			0001: 	begin   
				loadA = 0;  
				loadB = 0; 
				loadC = 0;  
				loadIR = 0;  
				loadPC = 0;  
				incPC = 1;  
				mode = 1'b0;  
				we_DM = 1;  
				#5 we_DM = 0;  
				selA = 1'b0;  
				selB = 1'b0;  
				end   
			// Mode 0, ALU operation for opcode 010   
			0010: 	begin  
				loadA = 0;   
				loadC = 0;  
				loadIR = 0;  
				loadPC = 0;  
				incPC = 1;  
				mode = 1'b0;  
				we_DM = 1;  
				#5 we_DM = 0;  
				selA = 1'b0;  
				selB = 1'b0;  
				end  
			// Mode 0, ALU operation for opcode 011   
			0011: 	begin  
				loadA = 0;  
				loadB = 0;  
				loadC = 0;  
				loadIR = 0;  
				loadPC = 0;  
				incPC = 1;  
				mode = 1'b0;  
				we_DM = 1;  
				#5 we_DM = 0;  
				selA = 1'b0;  
				selB = 1'b0;  
				end  
			// Load A operation   
			0100: 	begin  
				loadA = 1;   
				loadB = 0;   
				loadC = 0;   
				loadIR = 0;   
				loadPC = 0;   
				incPC = 1;   
				mode = 1'bZ;  
				we_DM = 1;  
				#5 we_DM = 0;  
				selA = 1'b0;  
				selB = 1'b0;  
				end  
			// Load B operation  
			0101: 	begin  
				loadA = 0;  
				loadB = 1;  
				loadC = 0;   
				loadIR = 0;   
				loadPC = 0;  
				incPC = 1;  
				mode = 1'bZ;  
				we_DM = 1;  
				#5 we_DM = 0;  
				selA = 1'b0;     
				selB = 1'b0;  
				end  
			// Load C operation   
			0110: 	begin   
				loadA = 0;  
				loadB = 0;   
				loadC = 1;  
				loadIR = 0;   
				loadPC = 0;  
				incPC = 1;  
				mode = 1'b0;  
				we_DM = 1;  
				#5 we_DM = 0;  
				selA = 1'b0;  
				selB = 1'b0;  
				end  
			// JMP translation  
			0111: 	begin  
				loadA = 0;  
				loadB = 0;  
				loadC = 0;  
				loadIR = 0;  
				loadPC = 1;  
				incPC = 1;  
				mode = 1'b0;  
				we_DM = 1;  
				selA = 1'b1;  
				selB = 1'b1;  
				end   
			// Mode 1, ALU operation for opcode 000  
			1000: 	begin  
				loadA = 0;  
				loadB = 0;  
				loadC = 0;  
				loadIR = 0;  
				loadPC = 0;  
				incPC = 1;
				mode = 1'b1;  
				we_DM = 1;  
				#5 we_DM = 0;  
				selA = 1'b0;  
				selB = 1'b0;  
				end  
			// Mode 1, ALU operation for opcode 001  
			1001: 	begin  
				loadA = 0;  
				loadB = 0;  
				loadC = 0;  
				loadIR = 0;  
				loadPC = 0;  
				incPC = 1;  
				mode = 1'b1;  
				we_DM = 1;  
				selA = 1'b0;  
				selB = 1'b0;  
				end  
			// Mode 1, ALU operation for opcode 010   
			1010: 	begin  
				loadA = 0;   
				loadB = 0;  
				loadC = 0;  
				loadIR = 0;  
				loadPC = 0;  
				incPC = 1;  
				mode = 1'b1;   
				we_DM = 1;  
				#5 we_DM = 0;  
				selA = 1'b0;  
				selB = 1'b0;  
				end  
			// Mode 1, ALU operation for opcode 011  
			1011: 	begin  
				loadA = 0;  
				loadB = 0;  
				loadC = 0;  
				loadIR = 0;
				loadPC = 0;  
				incPC = 1;  
				mode = 1'b1;  
				we_DM = 1;  
				#5 we_DM = 0;  
				selA = 1'b0;  
				selB = 1'b0;   
				end   
			// Mode 1, ALU operation for opcode 100   
			1100: 	begin   
				loadA = 0;  
				loadB = 0;   
				loadC = 0;   
				loadIR = 0;  
				loadPC = 0;  
				incPC = 1;  
				mode = 1'b1;  
				we_DM = 1;  
				#5 we_DM = 0;  
				selA = 1'b0;  
				selB = 1'b0;  
				end  
			// Mode 1, ALU operation for opcode 101  
			1101: 	begin   
				loadA = 0;  
				loadB = 0;   
				loadC = 0;   
				loadIR = 0;   
				loadPC = 0;  
				incPC = 1;   
				mode = 1'b1;   
				we_DM = 1;   
				#5 we_DM = 0;   
				selA = 1'b0;  
				selB = 1'b0;  
				end   
			// Mode 1, ALU operation for opcode 110   
			1110: 	begin   
				loadA = 0;    
				loadB = 0;   
				loadC = 0;   
				loadIR = 0;   
				loadPC = 0;   
				incPC = 1;   
				mode = 1'b1;   
				we_DM = 1;   
				#5 we_DM = 0;   
				selA = 1'b0;    
				selB = 1'b0;   
				end   
			// Mode 1, ALU operation for opcode 111   
			1111:	begin   
				loadA = 0;   
				loadB = 0;   
				loadC = 0;   
				loadIR = 0;   
				loadPC = 0;   
				incPC = 1;   
				mode = 1'b1;   
				we_DM = 1;   
				selA = 1'b0;   
				selB = 1'b0;  
				end   
			default: begin  
				loadA = 0;  
				loadB = 0;  
				loadC = 0;  
				loadIR = 1;  
				mode = 1'bZ;   
				we_DM = 0;   
				selA = 1'b0;  
				selB = 1'b0;  
				end  
			endcase  
			next_state = load;  
			end  
			
		default: begin  
			loadA = 0;   
			loadB = 0;  
			loadC = 0;   
			loadIR = 1;  
			mode = 1'bZ;  
			we_DM = 0;  
			selA = 1'b0;  
			selB = 1'b0;  
			next_state = reset;  
			end  
		endcase  
	end  
end  
endmodule  

## CPU TOP
///////////////////////////////////////////////////////////////////////  
/////	Design of the 16-Bit CPU is shown below			//////  
/////////////////////////////////////////////////////////////////////  
module CPU(clk, en, we_IM, codein, immd, za, zb, eq, gt, lt);  
input clk;  
input en;  
input we_IM;  
input [15:0] codein;  
input [11:0] immd;  
output za;  
output zb;  
output eq;  
output gt;  
output lt;  

reg za;  
reg zb;  
reg eq;  
reg gt;  
reg lt;  

wire [11:0] curradd; wire [15:0] outIMd; wire [11:0] addressd; wire [3:0] opcodeD;  
wire loadIRd, loadAd, loadBd, loadCd, moded, we_DMd, selAd, selBd, loadPCd, incPCd;  
wire [11:0] execaddd; wire [15:0] dataAoutd; wire [15:0] dataBoutd; wire [31:0] outALUd;  
wire [31:0] currdat; wire [31:0] outDMd; wire [31:0] dataCoutd;  
wire zad, zbd, eqd, gtd, ltd;  
instmem 	a1 (.clk(clk), .we_IM(we_IM), .dataIM(codein), .addIM(curradd), .outIM(outIMd));   
insReg 		a2 (.clk(clk), .loadIR(loadIRd), .insin(outIMd), .address(addressd), .opcode(opcodeD));    
controller 	a3 (.clk(clk), .en(en), .opcode(opcodeD), .loadA(loadAd), .loadB(loadBd), .loadC(loadCd), .loadIR(loadIRd), .loadPC(loadPCd), .incPC(incPCd), .mode(moded), .we_DM(we_DMd), .selA(selAd), .selB(selBd));   
PC 		a4 (.clk(clk), .loadPC(loadPCd), .incPC(incPCd), .address(addressd), .execadd(execaddd));   
muxB		a5 (.clk(clk), .in1(execaddd), .in2(immd), .sel(selBd), .outB(curradd));  
regA 		a6 (.clk(clk), .loadA(loadAd), .dataAin(outDMd[15:0]), .dataAout(dataAoutd));  
regB 		a7 (.clk(clk), .loadB(loadBd), .dataBin(outDMd[31:16]), .dataBout(dataBoutd));  
regC		a8 (.clk(clk), .loadC(loadCd), .dataCin(currdat), .dataCout(dataCoutd));  
datamem 	a9 (.clk(clk), .we_DM(we_DMd), .dataDM(dataCoutd), .addDM(addressd), .outDM(outDMd));  
muxA		b1 (.clk(clk), .in1(outALUd), .in2({4'b0000,immd}), .sel(selAd), .outA(currdat));  
ALU 		b2 (.a(dataAoutd), .b(dataBoutd), .opcode(opcodeD[2:0]), .mode(moded), .outALU(outALUd), .za(zad), .zb(zbd), .eq(eqd), .gt(gtd), .lt(ltd));   

endmodule   

