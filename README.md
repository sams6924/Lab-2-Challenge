# Lab-2-Challenge
Solution to challenge work for DECA Lab 2

## Sections
1. [The Task](#task)
2. [Plan](#plan)
3. [Hardware and Implementation](#hardware)
4. [Instruction](#instruction)
5. [Assembly Code](#assembly)
6. [Testing](#testing)
7. [Evaluation](#evaluation)

<a id="task"></a>
### The Task :
The goal of this challenge task is to create and implement new hardware and instructions into the eep1 model cpu to improve the efficiency of the binary multiplication process in the CPU.
The current multiplication algorithm designed in the eep1 cpu is extremely slow and inefficient, as it is done via repeated addition. This is slow as each addition process requires a seperate fetch and store which each seperately take 1 cycle to do.
Therefore, to speed up this task, specific proprietary hardware can be implemented which only requires 1 fetch and store operation for when the process ends and finishes.

<a id="plan"></a>
### Plan :
The plan for this solution was to create a specific block in the ALU of the eep1 that handles binary multiplication using a different algorithm that requires both is hardware and clock cycle efficient.
A common and fairly simple solution to this task would be to use the "Shift and Add" algorithm which shifts the multiplicand left after each partial addition to align each bit in the multiplier with each partial product then being accumulated in each step.
However, this solution requires a multiplicand which is constantly growing in bit width as it is being left shifted which would require different size adders which wouldn't fit within the limitations of the task.
<br> <br>
Therefore, a different but similar approach can be use called the "Right Shift and Add" algorithm. This process involves right shifting the accumulated total between each addition with preserves the bit width of the operation as an extra 0 can be 
joined to the accumulator to maintain an 8 bit width. <br>
This lack of need of dynamic bit widths also helps deal with hardware efficiency as different sized registers and adders can be avoided, simplifying the design.

<a id="hardware"></a>
### Hardware and Implementation:
The hardware for this solution was fairly simple to build and fit within the 64 bit adder constraint as only 1 adder was needed per bit of the input, so 8 bit inputs can be handled for the multiplier which is good enough bandwidth for most simple operations.
Larger 16 bit operations can also be handled using this hardware by using seperate registers to store seperate words of the total- which will only increases the number of clock cycles required for each 8 bit increase so it is still much faster than the original approach.

![image](https://github.com/user-attachments/assets/24ad2c79-57e6-419b-ad23-c3d366ea149c)

<br>

Above is an image of the multiply block that I made in order to use the right shift and add algorithm which stays in the limits of the task guidelines. <br>
The following images is of the edited ALU, datapath, and DPDECODE blocks in order to accomodate the use of the new hardware: <br> <br>
![image](https://github.com/user-attachments/assets/7b617a5a-197b-4bdb-98a4-cd75641be170)
<br> <br>
![image](https://github.com/user-attachments/assets/cef608e0-bf06-4a13-84f3-d78cf0800a26)
<br> <br>
![image](https://github.com/user-attachments/assets/43967d20-977b-491f-9db2-898ebf3ed51e)
<br> <br>

In the DPDECODE block, the unused bits 4:2 are repurposed to control the selection of the multiplier block by raising the outout MSEL when the instruction input bits 4:2 are either 001 or 010.
This is due to two new instructions that I have coded into the logic of the eep1 CPU : <br> <br>
- The first instruction, MUL, is a basic multiply which takes the value stored in Ra and multiplies it against the value stored in Rb and stores the result in Ra. <br> <br>
- The second instruction builds off of this and is the SQR function which takes the value in Rb, squares it and stores the result in Ra. <br> <br>

The logic that controls this can be seen in the image above, with the 8th bit of the instruction word needing to be 1 for either of these functions to raise MUL or SQR to 1.
<br><br>

These outputs are then passed to the ALU through the datapath and control two MUXs in the ALU block.
The first MUX is used to select if Rb is used as both the multiplier and the multiplicand which is selected by the SQR input, if SQR is 1, the value in Rb is multiplied by itself and is stored in Ra. <br>
The second MUX is used to select if the output of the ALU comes from the multiplier block and is controlled by MSEL OR SQR.

<a id="instruction"></a>
### Instruction :
The INS word can be modified to allow extra instructions such as MUL and SQR due to unused bits in the MOV instruction, MOV can only be used with two register inputs and therefore the values of bits 4:2 which are usually used to select the Rc address is unused. <br>
These 3 bits can then be used to define 8 new instructions, therefore, this design can be further built upon, but for the purposes of this challenge is unnecessary unless the new instruction works with the multiply block (which SQR does). <br>

<a id="assembly"></a>
### Assembly Code :
Usually when defining new instructions, new assembly code also has to be developed for the correct machine code to be input into the ROM of the eep1 otherwise these new functions are inaccessible. <br>
Luckily, the eep1 assembler already has been coded to allow for the bits 4:2 to be used for seperate instructions other than MOV and to access these functions, the command MOVCn is used instead, where n is the value of the bits 4:2. <br>
Therefore, to use these new functions, the following definitions can be used: <br><br>
##### - MOVC1 Ra, Rb = MUL Ra, Rb <br>
   Multiplies the value in Rb with the value of Ra, storing the result in Ra. <br> <br>
##### - MOVC2 Ra, Rb = SQR Ra, Rb <br>
   Squares the value stored in Rb and stores the result in Ra. <br> 

<a id="testing"></a>
### Testing :
Below are a few test cases, using the new functions and showcasing that they have correct outputs and comparing them to the number of clock cycles the eep1 originally took: <br> <br>
![image](https://github.com/user-attachments/assets/3a35cbdb-eb22-4213-a480-5cc1f9d911b4) <br>

8 X 4 took 35 clock cycles using the software approach, so 32 clock cycles were saved.

#### Loaded values (Clock Cycle 2): <br>
![image](https://github.com/user-attachments/assets/4dce7e0d-9776-4cb0-833e-7d2b1d1bcc47)

#### MOVC1 R1, R2 (Clock Cycle 3): <br>
![image](https://github.com/user-attachments/assets/ba9a3d74-9868-419d-912b-458a3d27b6bc) 

#### MOVC2 R1, R2 (Clock Cycle 4): <br>
![image](https://github.com/user-attachments/assets/57920f1f-9b8c-45d5-900b-36a2c0333e6e) <br> <br>

![image](https://github.com/user-attachments/assets/9fe5c817-ffe6-45bf-b3d7-aba662813c55) <br><br>

10 X 20 took 36 clock cycles using the software approach, so 33 clock cycles were saved.

#### Loaded values (Clock Cycle 2): <br>
![image](https://github.com/user-attachments/assets/959f7f69-ba71-41bb-b520-3ea688c4f285)

#### MOVC1 R1, R2 (Clock Cycle 3): <br>
![image](https://github.com/user-attachments/assets/7ccaf953-a6d1-48d1-a0b8-9d1577d25b4c)


#### MOVC2 R1, R2 (Clock Cycle 4): <br>
![image](https://github.com/user-attachments/assets/319dce24-9e89-484d-ada8-2a305493989a) <br> <br>

![image](https://github.com/user-attachments/assets/aecb7e05-f662-4e66-947a-67a7830f2741) <br>

This process couldn't be completed by the original software solution.

#### Loaded values (Clock Cycle 2) : <br>
![image](https://github.com/user-attachments/assets/8cdacd15-8821-4fe8-a620-0f77072d9844) <br>
Due to the EXTEND block, the value of #255 or FF gets extended to 16 bits <br>


#### MOVC1 R1, R2 (Clock Cycle 3): <br>
![image](https://github.com/user-attachments/assets/3f2981ac-b9ed-401f-a2a3-8fe4748d2f59) <br>
However, since the multiply block only uses the first 8 bits of the input, this doesnt effect the output of the multiply function and gives the correct output. <br>

#### MOVC2 R1, R2 (Clock Cycle 4): <br>
![image](https://github.com/user-attachments/assets/2068b583-d644-4d79-b095-915bddc17cb5) <br> <br>


![image](https://github.com/user-attachments/assets/0330e12e-afc7-433c-b3a2-cb1da82ff9bf) <br>

#### Loaded values (Clock Cycle 2): <br>
![image](https://github.com/user-attachments/assets/eb9072aa-0ee9-4239-bab2-79e099194af2)

#### MOVC1 R1, R2 (Clock Cycle 3): <br>
![image](https://github.com/user-attachments/assets/8bc5b309-8cb2-4193-b338-754c81792030)

<a id="testing"></a>
### Evaluation:
The design as showcaesd above is a much more efficient method of 8 bit multiplication with a 16 bit output and a range of 0-255 on the inputs as compared to the repeated addition method already in the hardware. <br>
However, each design has pros and cons which can allow some use cases where my design falls short while repeated addition does not: <br> <br>

#### Pros :
- Extremely fast, each multiplication requires 3 cycles in total and doesnt vary due to input.
- Allows input of 8 bit numbers in range of 0-255 without issues of forced extension
- Can be used for 16 bit multiplication by using multiple registers per 8 bit word, but still at a much faster speed than the original process
- Simplifies a large amount of assembly, including removing jumps, to achieve the result
- Fairly simple and low cost hardware as it falls below 64 bit adders
- Allows useful instructions to be built upon the design (e.g SQR) without drastically increasing processing time


#### Cons :
- In the current iteration cannot handle signed integers or preserve the sign bit which is an extremely useful feature
- Could be simplified in design further and make it more readable by simplifying repeated logic into blocks
- Even though it falls within the guidelines of the task, this approach requires 7 more 8 bit adders than the purely software solution.

The design could be further tweaked for two's compliment addition by converting the negative number to its positive representation and storing the sign bit so the product can be converted back to the negative of itself. <br>
Also the design could potentially include pipelining for cases of continuous multiplication operations happening in a row which would also increase the efficiency of the design.








