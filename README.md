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

# Day 3
## SPICE Deck creation for CMOS Inverter

SPICE deck contains the information of netlist such as:
- Connectivity Information
- Component values
- 'Nodes' identified
- 'Node' names

![image](https://github.com/yagnavivek/PES_OpenLane_PD/assets/93475824/fde8c66e-6547-49a2-bdad-478c812d5419)

## Fabrication Process for a CMOS Inverter

Fabrication of CMOS Inverter is a 16-Mask process

### 1. Selecting the substrate 

- P-Type substrate with resistivity around (5-50 ohm) doping level (10^15 cm^-3) and orientation (100).
- Note that substrate doping should be less than well doping (used to fabricate NMOS and PMOS)

### 2. Create active resistance

This step creates pockets for NMOS and PMOS
1. Grow SiO2(~40nm) on Psub
2. deposit ~80nm Si3N4 on SiO2
3. deposit 1um layer of photoresist(used to define regions)
4. photolithography
5. etch out Si3N4 and SiO2 using a suitable solvent
6. Place the obtained structure in oxidartion furnace due to which field oxide is grown.This process is called ```LOCOS``` that is ```Local oxidation of silicon```
7. Etch out Si3N4 using hot phosphoric acid

### 3.NWel and PWel formation

- Apply photoresist, apply mask that covers NMOS
- Expose to UV, Wash, remove mask, appl boron(p-type) using Ion Implantation at an energy of 200Kev(for diffusion)
- repeat it for the other half using phosphorous @400Kev because phosphorous is heavier
- Wells have been created but the depth is low. Therefore subject it to high temperature furnace which increases the well depth.

### 4. Formation of Gate

- We repeat the step 3 but at low energy with p-type implant as boron @60Kev and n-type implant as Arsenic.
- Due to this The SiO2 is damaged as the dopants penetrate through it.
- Therefore original SiO2 is etched out using dilute HF solution and regrown to give high quality oxide(~10 nm thin)
- Finally for the gate to form, apply N-type ion implants for low gate resistance.
- Now mask on small width of Nwell and PWell above SiO2  and perform photolithography
- Gate Formation is Done

### 5. Lighlt Doped Drain Formation(LDD Formation)

- On the surface of SiO2 corresponding to NWell, apply photoresist, mask it, put phosphorous to make N-Implant on p-well(N-)
- Similarly do it for the other side using boron that forms (p-) implant
- This LDD has to be protected from further process
- so, Deposit 0.1um thick SiO2 on full structure and etch out using plasma anisotropic etching that results in formation of side wall spacers..

### 6. Source and Drain Formation

- Mask Nwell structure, deposit arsenic @75KeV that forms an N+ implant on Pwell
- use boron for P+ implant formation on Nwell
- Subject it to high temperature furnace that results in required thickness of N+,P+,N-,P- implants.

### 7. Steps to form contacts and interconnects

- Etch thin SiO2 oxide in HF solution
- Deposit Titanium of wafer surface using sputtering all over the structure
- Wafer heated at 600-700 degree in ambient N2 environment for 60 sec that reults in low resistance TiSi2 where the gate of both MOS is present.
- At the other places, TiN is formed that's used for local communication
- Etch off TiN on and half around gate structure of both MOS using RCA Cleaning

### 8. Higher level metal formation

- On the resulted structure, deposit a thick layer of (1um) SiO2 doped with P/B known as phosphoborosilicate glass
- To make the added surface plain, use CMP (Chemical Metal Polishing)
- For the creation of contact pins, proper holes with contacts have to be made
- This can be done using Al, W and TiN layer depositions.
- Deposit a layer of Si3N4 that acts as dielectric to protect the chip.

### 9. Final STructure

 ![image](https://github.com/yagnavivek/PES_OpenLane_PD/assets/93475824/0e355a75-55ff-4723-96ae-4abd5845697c)

**A Git Clone and some other steps for future labs**

```
git clone https://github.com/nickson-jose/vsdstdcelldesign.git
```

- Now we need to copy the 'sky130A.tech' file into the directory we just cloned
- We can do this by using
```
cp sky130A.tech /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/vsdstdcelldesign
```
in the follwoing directory shown in the figure

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/c0cefbbc-dfd8-40b0-859e-3603e5589416)

### Lab work
## Inverter Layout using Magic

```
cd Desktop/work/tools/openlane_working_dir/openlane/vsdstdcelldesign
magic -T sky130A.tech sky130_inv.mag
```

## Exploring the Layout displayed by MAGIC

Select the specific layer/device by hovering over the object and pressing, s, iteratively, until you traverse the hierarchy to the specified object:

![Screenshot from 2023-09-12 18-15-54](https://github.com/yagnavivek/PES_OpenLane_PD/assets/93475824/1a918a4c-da78-4c9f-b553-e080ddd3e7e7)

- select a region from the layout, go to the console and type ```what``` to display the information of selected area
- To select a region, place ```cursor``` on that point and  press```s```. More the number of times you press ```s```, higher the abstraction selected.

![image](https://github.com/yagnavivek/PES_OpenLane_PD/assets/93475824/fdd5bf6b-3483-4471-9b68-d98fa0b80af3)

refer to [inverter](https://github.com/nickson-jose/vsdstdcelldesign) to create layout for CMOS Inverter

### DRC Check

To check for DRC Errors, select a region (left click for starting point, right click at end point) and see the DRC column at the top that shows how many DRC errors are present.The Details of DRC Errors will be printed on the console.

![image](https://github.com/yagnavivek/PES_OpenLane_PD/assets/93475824/eebc0109-4408-40fa-a18e-ead67419cfa7)

For more information on DRC errors plase refer to: [DRC_Erros](https://skywater-pdk--136.org.readthedocs.build/en/136/)
For more information on how to fix these DRC errors using Magic please refer to: [fix_DRC](http://opencircuitdesign.com/magic/)


## Extracting PEX to SPICE with MAGIC

Select Full inverter layout. Then

![image](https://github.com/yagnavivek/PES_OpenLane_PD/assets/93475824/36c93dc8-6c1e-4ac4-9eac-f2c7a001b82a)

![image](https://github.com/yagnavivek/PES_OpenLane_PD/assets/93475824/e58613be-86ee-4248-8298-ef002274429b)

The above file has details of inverter netlist but the sources and their values are not specified. So we have to modify the file.

- Grid size from the layout is 0.01u
- specify the library for MOS
- create VDD, VSS, Input pulse Va
- specify the type of analysis to be done

### Grid Size

- We can use 'g' on the keyboard to activate the grid and after selecting a grid by right clicking on the mouse, we type ```box``` in tkcon window to check the minimum value of the layout window.

![image](https://github.com/yagnavivek/PES_OpenLane_PD/assets/93475824/1fd94afe-3a40-4269-92ed-d7c46f248417)

## Modified Spice netlist

![Screenshot from 2023-09-14 17-25-22](https://github.com/yagnavivek/PES_OpenLane_PD/assets/93475824/21be1cbb-da31-4409-bf53-a3a80f11ac97)

- Next we need to open the spice file using the command
```
gedit sky130_inv.spice
```
- We need to configure it to the above specifications.

**Characterize Inverter using Sky130 Models**

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/548d6ace-9d74-47f4-906f-4614c1babaca)
- We now plot the graph for output vs input sweeping the time.
- We first use the command
```
ngspice sky130_inv.spice
```
- In the ngspice shell we use the command
```
plot y vs time a
```
![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/b810d9fe-11e4-44f7-863c-ed19593e0b3c)
- The following graph is displayed.

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/400506ec-61c3-458c-9130-7147ec496c6a)

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/55c43b65-5c02-451a-bb11-391de8d87571)
- Rise Time -> time taken to rise from 20% to 80% of the max value -> 2.25075e-09 - 2.184e-09 = 0.006675e-09 s.

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/93f95ac2-43e2-48de-93e1-07399e20b4b1)

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/fc3b2887-d910-4864-ae33-ef02fdb8035f)
- Propogation Delay/Cell Rise Delay -> 2.21379e-09 - 2.15e-09 = 0.06379e-09 s.

**Sky130 PDKS and Steps to Download Magic Tool**

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/9f99bcfc-e79c-4c8b-afcf-27f3502b477f)
- Enter the command
```
 wget http://opencircuitdesign.com/open_pdks/archive/drc_tests.tgz
```

- Move the file to desktop using
```
mv drc_tests.tgz Desktop/
```

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/dd984ca9-3692-43f1-b111-f90e5e7a4c08)
- Extract the file using
```
tar xfz drc_tests.tgz 
```
- Do ```ls``` to view all the files in it.

To open the software we type
```
magic -d XR
```

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/8571a471-4937-4b89-8197-99c2648dfa66)
- We click 'file' and open the 'met3.mag' file.

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/cbec69ce-fedb-41cd-af31-0dd9750c1ed9)
![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/b9551c50-a44b-472b-bf63-d4979c2f8caa)
- If we select an area and type ```drc why``` in the tkcon wndow, it will show us the DRC error.

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/53fdfab7-7d24-4638-9fb3-855589c89b83)
- To add contact cuts to metal3, first select an area using left and right click. Then hovering over the m3contact we click middle mouse button.

**Fixing DRC Errors**
- There is a DRC error in the poly.mag file in 'poly.9'.
- Open the sky130A.tech file in the editor and make the following changes

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/d786e946-95e0-42c8-a0bd-a83a57697e04)

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/4274217a-ba1b-499d-b81a-fa26b4262301)

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/efafb2d3-520d-4405-b1ae-731b0308ac99)
- Now open the tkcon window and type
```
load tech sky130A.tech
drc check
```

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/ebd7630c-128d-46ba-adb7-a30169ba7fe6)
- As we can see the error is fixed.

**DRC Error as Geometrical Construct**
- We open the nwell.mag file.

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/4c49e5c5-4738-449c-9df0-048d2034825d)
- We type the above commands

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/40097614-0dfe-4479-b85c-4418e7b43787)
- The following is displayed

**Find Missing or Incorrect Rules and Fix Them**

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/338b998a-9a6b-4485-8d2b-095fbbcf3d83)

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/cc00bc2f-31ed-45b3-96d8-a12b28697bd4)
- As we can see this is an incorrect implementation and the above rule is violated.

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/d689590d-b9a7-43b2-b1da-72e95a9f4dcb)

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/71d62451-245b-4010-8246-234727062c8b)
- We make the following changes

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/08900ffa-8f91-4550-be8c-375b4cb42866)
- Now we select the nwell.4 and type the following commands
```
tech load sky130A.tech
drc check
drc style drc(full)
drc check
```

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/dbb58cd3-4845-4d28-a22b-0228b8260cf6)
- As we can see the error still persists
- We can fix it by the following method.

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/4bc70446-9a20-47cd-95b9-d850e170887a)
- Select the existing nwell.4 and make a copy of it by selecting it and clicking 'c'.
- Now select a small area on the nwell.4 and add an 'nsubstratecontact' by hovering over it and clicking middle mouse button.

















![Screenshot from 2023-09-18 16-01-30](https://github.com/NandeeshaSwamy/pes_pd/assets/135755149/43ed9a97-2c00-48e0-8bfa-717d74a80dea)
