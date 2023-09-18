# Day 1

## How to Talk to Computers
- First we look at the introduction to the RISC-V ISA(Instructiion Set Architecture). Supposing we need to execute a C program on a particular hardware. First the C-program is converted into Assembly Code( here for RISC-V processor). Then the assembly code is converted into binary. An RTL implements this code for the particular layout of the RISC-V processor and the output is visible.
- An application running on a system is usually written with the help of a high level language such as C,C++,Python etc. The code of these applications are compiled with the help of compilers running on a system software(OS). The compiler converts the high level code into assembly intructions for the particular processor. The assembler then converts the instructions into binary which is fed into the layout of the chip that processes every pattern of bits and the program is hence run.

## SoC Design and OpenLANE

**What is a PDK?**
- PDK stands for Process Design Kit.
- It is a collection of files used to model a fabrication process for the EDA tools used to design an IC
  - Process Design Rules.
  - Device Models
  - Digital Standard Cell Libraries
  - I/O Libraries
 
A simplified RTL to GDSII Flow is :
- Synthesis -> Floor/Power Planning -> Placement -> Clock Tree Synthesis -> Routing -> Signoff

- Synthesis - Converts RTL to a ciruit, out of compomments from the standard cell library.
- Floor and Power Planning - Obejctive here is to plan the silicon area and create robust power distribution network to power the chip.
  - Chip Floor Planning - Partition the chip die between different system building blocks and place the I/O pads.
  - Macro Floor Planning - We define the macro dimensions, pin locations and rows are defined.
  - Power Planning - The power distribution network is contructed.
- Placement - Placing the cells on the floorplan rows, aligned with the sites. There are 2 steps: Global and Detailed.
- Clock Tree Synthesis - To deliver the clock to all sequential elements.
- Routing - Implement the interconnect using the available metal layers.
- Sign Off - Perform physical verification such as DRC(Design Rule Check) and LVS(Layout vs Synthesis). Also perform STA(Static Timiing Analysis).

**OpenLANE ASIC Flow**

![Screenshot from 2023-09-10 23-56-09](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/b1bbef29-0748-4fd7-acf8-8c421d599aca)

## Getting Familiar with the Open Source EDA Tools

### Design Preparation Step

- Let us first go the the working directory using the following commands
```
cd Desktop/work/tools/
```

```
cd openlane_working_dir/openlane/
```
- We now type the command ```docker```.
- This will open the shell as shown in the figure below
- 
![Screenshot from 2023-09-13 20-49-28](https://github.com/NandeeshaSwamy/pes_pd/assets/135755149/57850463-57c1-43eb-9d04-0122add76552)

- Now we type
```
./flow.tcl -interactive
```
- If the 'interactive' keyword is not present, then the entire flow of the tool is run.

![Screenshot from 2023-09-13 20-50-56](https://github.com/NandeeshaSwamy/pes_pd/assets/135755149/b0b3176c-51c3-4cdc-809f-de2dd3734da0)


- Now we must import all the packages required to run the flow, we use the command:
```
package require openlane 0.9
```
![Screenshot from 2023-09-13 20-52-10](https://github.com/NandeeshaSwamy/pes_pd/assets/135755149/02b2652b-fae4-4576-a726-190bfbfbf0ef)

- We will be working with the 'picorv32a' design.
- The src folder has the verilog file and the sdc file of the design
- Now we do the design setup stage using the command:
```
prep -design picorv32a
```
![Screenshot from 2023-09-13 21-09-18](https://github.com/NandeeshaSwamy/pes_pd/assets/135755149/da10c62a-2342-4b0a-912b-c825399c219c)

- After preparing the design, we can see that a new 'runs' folder is created.

- To synthesize the design we type
```
run_synthesis
```
- This command invokes yosys, runs the synthesis and the abc commands.
- A long process is observed after typing this command, which for a little over two mintues.

![Screenshot from 2023-09-13 21-23-41](https://github.com/NandeeshaSwamy/pes_pd/assets/135755149/11ed643c-84d0-42e3-9cd0-52ff30146a06)

- A synthesis successful message must be displayed.

![Screenshot from 2023-09-13 21-36-30](https://github.com/NandeeshaSwamy/pes_pd/assets/135755149/5be4b9a0-8d52-49e6-8a2a-e065f6f988a2)

- The flop ratio can be calculated by using:
```
No. of flops/No. of cells = 1613/14876 = 0.108
```
- In percentage there is 10.8% of the total number of cells are Flops.

- Under the runs folder we can check out the netlist file generated after synthesis.

![Screenshot from 2023-09-13 21-39-26](https://github.com/NandeeshaSwamy/pes_pd/assets/135755149/60fe79cf-a698-4342-a330-f7ae4a8d4b69)























![Screenshot from 2023-09-18 16-01-30](https://github.com/NandeeshaSwamy/pes_pd/assets/135755149/43ed9a97-2c00-48e0-8bfa-717d74a80dea)
