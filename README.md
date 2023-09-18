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

# Day 2
## Chip Floorplanning Considerations

### 1. Define Width and height of core and die

- ```Die``` : Structure that consists of core which is a small semiconductor material on which the fundamental circuit is fabricated.
- ```core``` : Structire that contains primary logic and functional components.

Whenever we come across the concepts of core and die, ```Utilisation factor``` plays an important role.
UTILISATION FACTOR = Area Occupied by the Netlist / Area of the core (usually 50%-70%)
ASPECT RATIO = Height / Width (1 = square, others = rectangle)

### 2. Define Location of Pre-Placed cells

```pre-placed cells``` : memories, clock gating cells, comparator, mux etc

- The arrangement of these IPs on chip is called FLOORPLANNING
- These IPs have user defined locations and hence are placed in chip before automated placement and routing. Therefore called pre-placed cells.
- Automated PnR tool places the remaining logical cells in design onto chip.

### 3. De-coupling capacitors

_____Problem_____
We know that all the combinational blocks are connected to Vdd and Vss for their operation. But when there is a large circuit with many resistors, then The capacitors in the logic might not get fully charged as there occurs voltage deop due to wire metal and the resistors present along the path. So after voltage drop, if the voltage obtained by the logic is within noise margin, then it works well but what if it doesn't? 

_____Solution_____
We use De-Coupling capacitors (A huge capacitance with voltage equal to that of supply voltage) that is placed close to the combinational logic. When the switching activity takes place, it detatches the circuit from main supply and this capacitor acts as power supply.

The local communication has been successfully eshtablished with the solution mentioned above. The global communication is taken care by power planning.

### 4. Power Planning

- Power planning during the Floorplanning phase is essential to lower noise in digital circuits attributed to voltage droop and ground bounce. Coupling capacitance is formed between interconnect wires and the substrate which needs to be charged or discharged to represent either logic 1 or logic 0.
- When a transition occurs on a net, charge associated with coupling capacitors may be dumped to ground. If there are not enough ground taps charge will accumulate at the tap and the ground line will act like a large resistor, raising the ground voltage and lowering our noise margin. To bypass this problem a robust PDN with many power strap taps are needed to lower the resistance associated with the PDN.

### 5. Pin Placement

- ```Pin placement``` is an essential part of floorplanning to minimize buffering and improve power consumption and timing delays.
- We usually place input pins on the left and output pins on the right
- for primary inputs and outputs, pin size may be small and for clock, the pin size would be large because clock should drive many cells so we need to make sure that the resistance is less.
- larger the area, lesser the resistance.
- ```Placement blockage``` is done inorder to makesure that no logic is placed along the area where the pin placement is carried out.

**Steps to run FLoorplan using OpenLANE**
- To view floorplan we type
```
run_floorplan
```
or
```
init_floorplan
```
in the OpenLANE shell.

- To open the Floorplan we go to the required directory that is
```
vsduser@vsdsquadron:~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/11-09_15-36/results/floorplan
```
using the ```cd``` command.

- Then we type the command:
```
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &
```

- The following layout is displayed

![Screenshot from 2023-09-17 00-57-49](https://github.com/NandeeshaSwamy/pes_pd/assets/135755149/3468b19d-f5c6-47c7-b1bd-75e185cb6a30)

- We can press 's' and then 'v' to align the design to the center of the screen.

- We can right click on the mouse and pess 'z' to zoom into a desired part.

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/3d251ffe-c125-41be-a96e-00f2391b89fb)
- We can see here that the I/O ports are equidistant

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/2b79ae12-8c15-44e1-b800-48981f3fc442)
- We can check the details of the ports as follows
  - Hover over a port with your crosshair and press 's' on your keyboard
  - Now open the tkcon command window and type ```what```.
  - This will show you the details of the selected port.

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/2ee6d6b3-a7e9-415b-b2dd-1271d16dac2c)
- If we zoom in a little more, we can see the tap cells.
- They are present to prevent latch up conditions which occur in the CMOS devices

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/7e726389-8a96-4572-b6a7-59d5eb0d821a)
- These are the standard cells that are used in the design

## Library Binding and Placement

### 1. Bind the netlist with physical cells

- ```Library``` consists of cells, sizes of cells, various flavours and shapes of the cells, Timing, Power and delay information.
- Now, we have the floorplan, netlist and representation of components of netlist in library
- place all the components such that the timing is not disturbed and distribute them properly. 


### 2. Optimize Placement

- Some components may be located very far to their inputs which can disturb signal integrity (as wire length increases, RC value increases). Therefore we use repeaters(may be series of buffers) inorder to avoid signal loss but area loss comes into picture.
- Assuming that all the clock signals are working at ideal rate, we do the timing analysis if the current placement works good.

### 3. Placement
- To view the placement we type
```
run_placement
```
in the OpenLANE shell.

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/56832e08-c84e-4c78-8a24-73e1b9bdb05f)
- This is the result displayed. As we can see the '/picorv32a.placement.def' file is read.

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/0adbe3c9-fe26-4770-9eb0-1ed9ca581402)
- We move one directory up from the 'floorplan' folder using
```
cd ../placement/
```
- To view the placement design we use the command
```
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def
```

![Screenshot from 2023-09-17 01-50-53](https://github.com/NandeeshaSwamy/pes_pd/assets/135755149/428f7c45-efc9-41e2-8374-f5f805fc0657)

- The above is displayed.
- All these standard cells were present at the initial layout of the floorplan.

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/2a7cce9b-9550-4eaf-922e-8a64306f05ac)
- If we zoom in we can see the placement of the standard cells in the standard cell rows.

## Cell Design Flow

Cell design is done in 3 parts:

1. **Inputs** - PDKs (Process design kits), DRC & LVS rules, SPICE models, library & user-defined specs.
2. **Design Steps** - Design steps of cell design involves Circuit Design, Layout Design, Characterization. The software GUNA used for characterization. The characterization can be classified as Timing characterization, Power characterization and Noise characterization.
3. **Outputs** - Outputs of the Design are CDL (Circuit Description Language), GDSII, LEF, extracted Spice netlist (.cir), timing, noise, power.libs, function.

### Standard cell Charachterization Flow

Standard Cell Libraries consist of cells with different functionality/drive strengths. These cells need to be characterized by liberty files to be used by synthesis tools to determine optimal circuit arrangement. The open-source software GUNA is used for characterization.
Characterization is a well-defined flow consisting of the following steps:

- Link Model File of CMOS containing property definitions
- Specify process corner(s) for the cell to be characterized
- Specify cell delay and slew thresholds percentages
- Specify timing and power tables
- Read the parasitic extracted netlist
- Apply input or stimulus
- Provide necessary simulation commands

### General Timing characterization parameters

#### Timing threshold definitions

- ```slew_low_rise_thr``` - 20% from bottom power supply when the signal is rising
- ```slew_high_rise_thr``` - 20% from top power supply when the signal is rising
- ```slew_low_fall_thr``` - 20% from bottom power supply when the signal is falling
- ```slew_high_fall_thr``` - 20% from top power supply when the signal is falling
- ```in_rise_thr``` - 50% point on the rising edge of input
- ```in_fall_thr``` - 50% point on the falling edge of input
- ```out_rise_thr``` - 50% point on the rising edge of ouput
- ```out_fall_thr``` - 50% point on the falling edge of ouput

These are the main parameters that we use to calculate factors such as propogation delay and transition time

- ```propogation delay ``` - time(out_*_thr) - time(in_*_thr)
- ```Transition time``` - time(slew_high_rise_thr) - time(slew_low_rise_thr)





















![Screenshot from 2023-09-18 16-01-30](https://github.com/NandeeshaSwamy/pes_pd/assets/135755149/43ed9a97-2c00-48e0-8bfa-717d74a80dea)
