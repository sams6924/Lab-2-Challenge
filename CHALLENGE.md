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

![image](https://github.com/user-attachments/assets/42b14758-392f-4a8e-b335-404337631b58)
<br>

Above is an image of the multiply block that I made in order to use the right shift and add algorithm which stays in the limits of the task guidelines. <br>
The following images is of the edited ALU, datapath, and DPDECODE blocks in order to accomodate the use of the new hardware: <br> <br>
![image](https://github.com/user-attachments/assets/7b617a5a-197b-4bdb-98a4-cd75641be170)
<br> <br>
![image](https://github.com/user-attachments/assets/cef608e0-bf06-4a13-84f3-d78cf0800a26)
<br> <br>
![image](https://github.com/user-attachments/assets/43967d20-977b-491f-9db2-898ebf3ed51e)



<a id="instruction"></a>
### Instruction :

<a id="assembly"></a>
### Assembly Code :
