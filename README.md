# Welcome to the Open Source Physical Design wiki!

## **Introduction to IC Design Components & Open-Source EDA tools**

From RTL to GDS flow we can use following EDA tools:

* 1st step: Logic Synthesis: 
In this step RTL netlist and specification is converted to a synthesizable netlist consisting of logic gates and flip flops. 
**Yosys** Open Synthesis tool is open source tool for logic synthesis.

* 2nd step: Floorplanning:
After receiving the synthesized netlist you place the pre-placed cells, do power planning.

* 3rd step: Placement:
Do the placement as per decided floorplan.

* 4th step: CTS (Clock tree synthesis):
Clock is routed such that you get zero skew or skew defined by he designer.

* **Graywolf** is open source tool which does floorplanning, placement and CTS.

* 4th step: Routing:
All the components placed during placement are routed. Different styles of routings can be used such as maze routing.
**Qrouter** is the open source tool that does routing.

After each step mentioned about you have to do STA (Static timing analysis)
This analyzes signals after each steps so that we don't have any surprises at final netlist sign off.
**Opentimer** is open source tool for STA

**Magic Layout Viewer** : is open source layout editor. This can be used after CTS or placement or routing to view and do drc corrections to the layout.

**ngSpice**: Open source tool to do spice simulation on the netlist extracted from Magic tool. To do pre and post layout simulation on the netlist.

**eSim**: Open source schematic editor tool.

![Open Source EDA Tools](https://user-images.githubusercontent.com/40173944/99911433-b5e10f00-2cf4-11eb-813c-08193f6d6112.png)

## Steps to install and run on UBUNTU:
[source : Kunal Gosh [https://github.com/kunalg123/vsdflow](https://github.com/kunalg123/vsdflow)]
```
sudo apt-get install git
git clone https://github.com/kunalg123/vsdflow.git
cd vsdflow
chmod 777 opensource_eda_tool_install.sh
./opensource_eda_tool_install.sh
```
**NOTE for experienced UNIX users : It has lot of sudo apt-get and sudo remove commands, so you might want to review before running
```
./vsdflow spi_slave_design_details.csv
./vsdflow picorv32_design_details.csv
```
Steps to test 'vsdflow' on Ubuntu:
```
cd outdir_spi_slave
qflow display spi_slave
```
List of Tools installed:
1) Yosys - RTL Synthesis
2) blifFanout - High fanout net (HFN) synthesis
3) graywolf - Placement
4) qrouter - Detailed routing
5) magic - VLSI Layout tool
6) netgen - LVS
7) OpenTimer and OpenSTA - Static timing analysis tool

# Lab Day 1  Synthesis:

Type below command
```
git clone https://github.com/kunalg123/vsdflow.git
cd vsdflow
mkdir my_picorv32
cd my_picorv32
mkdir source synthesis layout
cp ~/vsdflow/verilog/picorv32.v source/.
qflow gui &
```

![Start Qflow GUI](https://user-images.githubusercontent.com/40173944/99911626-06a53780-2cf6-11eb-8a7d-75e85926f3a4.png)

Select below options in gui:

	Technology = osu018
	Verilog source file : picorv32.v
	Verilog module : picorv32
	Click on Set Stop

Then run labs till synthesis.
Confirm if the % ratio of flipflop/total logic is 12-13.99% in the synthesis log file.
![Flip Flop count](https://user-images.githubusercontent.com/40173944/99913725-69e49900-2cf9-11eb-98f8-7de980c413a7.png)

# Lab Day 2  Placement:

Type below commands:
```
git clone https://github.com/kunalg123/vsdflow.git
cd vsdflow/my_picorv32
qflow gui &
```

Placement setting:

	Initial density: 0.7
	Rest as default

![Placement Settings](https://user-images.githubusercontent.com/40173944/99912203-e5454b00-2cf7-11eb-9638-d69b44abd464.png)

Arrange pin:
1. Click auto group to group all vector pins togeather.
2. Then Arrange pins as in image below but don't specify too many constraints so as to have some relaxed placement:

![picorv32 SOC](https://user-images.githubusercontent.com/40173944/99913603-da3eea80-2cf8-11eb-8d3b-c68b3157387f.png)

![Qflow Pin arrangement window](https://user-images.githubusercontent.com/40173944/99913554-9fd54d80-2cf8-11eb-9397-8acb834efd29.png)

Run the placement step, this will start a graywolf placement window:

![Placement in progress](https://user-images.githubusercontent.com/40173944/99913714-6224f480-2cf9-11eb-980a-a19ac8a81c65.png)

You can close the qflow window and type following cmd in terminal:

`qflow display picorv32 &`

This will open layout and tkcon window In the layout window, select whole chip using below steps
Take cursor to bottom left
Left mouse click
Take cursor to top right
Right mouse click
Press Shift+i

This will select the whole layout Now in tkcon window, type below command

`box`

What is the area of you chip in microns? Ans: 812062.19

![Chip area after placement](https://user-images.githubusercontent.com/40173944/99915194-7d483200-2d02-11eb-8d57-808be4220f02.PNG)


# Lab Day 3 Spice Simulations:

Type below commands in terminal:
```
git clone https://github.com/kunalg123/ngspice_labs.git
cd ngspice_labs
cat inv.spice
```
![Read Inverter spice file](https://user-images.githubusercontent.com/40173944/99915320-4a526e00-2d03-11eb-923c-d081a3e6b3e5.PNG)

What is the width of PMOS transistor? Ans: 0.5u
What is Wp/Wn ratio? Ans: 1.3

Now type below commands:

`ngspice inv.spice`

There will be terminal like below

`ngspice 1 ->`

On the above ngspice terminal, type below commands
```
run
setplot dc1
plot out in
```
This will open a plot with CMOS VTC and Blue 45 degree line

Click on the intersection of Blue line and CMOS VTC.

Go to terminal
"x0" values denotes the switching threshold of a Inverter.
What does "x0" value lies between? Ans: 1.0v-1.1v
![Inverter ngspice sim](https://user-images.githubusercontent.com/40173944/99915589-d7e28d80-2d04-11eb-86d6-ccb110a20509.PNG)


Now Open file called "inv_tran.spice" using below command
```
leafpad inv_tran.spice
```
Change PMOS width to 0.75u, Save and Close

Type below commands for transient simulations
```
ngspice inv_tran.spice
ngspice 1 -> run
ngspice 1 -> setplot tran1
ngspice 1 -> plot out in
```
What is the rise delay? Ans: 76 ps
![Inverter transient simulation](https://user-images.githubusercontent.com/40173944/99915763-eaa99200-2d05-11eb-9f20-4c10ba27b0ed.PNG)


# Lab Day 3 Routing:

Type below commands
```
cd ngspice_labs
magic -T min2.tech
```
This will open magic layout window and tkcon window

Go to tkcon window and type below command
```
source draw_fn.tcl
```
This will generate layout for the tcl file provided.

![TCL generated layout](https://user-images.githubusercontent.com/40173944/99916191-f21e6a80-2d08-11eb-911f-e9e26d0b8b2d.PNG) 


To create labels on layout select the point on layout where label is placed and then in tkcon window type:
```
label labelName
```
Then on layout you can modify or create routing and then save the layout:
```
save layName.mag
```

Now we want to run a post layout simulation to verify if the output waveform from layout drawn matched with the pre layout sim results.

For that first we extract parasitics using “extract all” cmd. Then we extract the netlist from layout using “ext2spice” cmd on tkcon window:
```
extract all
ext2spice
```
![Extract post layout netlist from Magic tool](https://user-images.githubusercontent.com/40173944/99911810-196c3c00-2cf7-11eb-97eb-3daedce97380.png)

Open the extracted spice file from layout and add the simulation commands to it from the "fn_prelayout.spice":
Hint: We don't have Gnd, but we have 0. Please modify postlayout spice netlist accordingly

![Post layout netlist](https://user-images.githubusercontent.com/40173944/99916574-272bbc80-2d0b-11eb-852b-3765a8d0eae0.PNG)

Also if there are two grounds gnd and gnd! , so since they both are same we should connect the global ground gnd! with gnd with very small resistor so that they are connected together:
	`RX gnd! gnd 0.001`

Now you can run the spice simulation (ngspice) on this post layout spice file generated and verify the output waveform.
```
ngspice fn_postlayout.spice
ngspice 1 -> run
ngspice 1 -> setplot tran1
ngspice 1 -> plot out in
```
What is the value of X0 at intersection rising & falling waveform and intersection of horizontal blue line? Ans: around 1.58ns and 2.27ns respectively

![Qflow Routing](https://user-images.githubusercontent.com/40173944/99954989-e1f7a100-2d83-11eb-9838-09414e478ab2.png)


# Lab Day 4 STA (Static Timing Analysis):

Type below commands:
```
/usr/local/share/qflow/tech/osu018/osu018_stdcells.lib
```
![OSU018 Std Lib Models](https://user-images.githubusercontent.com/40173944/99917061-437d2880-2d0e-11eb-88a3-adc7afd25fa6.PNG)

Here you can find values for your STA from the lib file.

Type below command
```
cd vsdflow/my_picorv32
leafpad picorv32.sdc
```
Type below lines in the file picorv32.sdc file which you have just opened above
```
create_clock -name clk -period 2.5 -waveform {0 1.25} [get_ports clk]
```
Save and close the above file

Now type below command
```
leafpad prelayout_sta.conf
```
Type below lines in prelayout_sta.conf file which you have just opened above
```
read_liberty /usr/local/share/qflow/tech/osu018/osu018_stdcells.lib
read_verilog synthesis/picorv32.rtlnopwr.v
link_design picorv32
read_sdc picorv32.sdc
report_checks
```
Save and close the above file

Now type below command
```
sta prelayout_sta.conf
```
What is the SLACK value you see? Ans: around -0.56ns

![Prelayout STA](https://user-images.githubusercontent.com/40173944/99911834-3b65be80-2cf7-11eb-88a1-05fa61fb72f3.png)

To get the values upto 4 decimals use following cmd:
```
report_checks -digits 4
```
Type below command in terminal to get STA after clock propogation:
```
set_propagated_clock [all_clocks]
report_checks
```
What is the SLACK value after clock propagation ? Ans: around -0.68ns

![STA after clock propagation](https://user-images.githubusercontent.com/40173944/99912406-f4c49400-2cf7-11eb-9530-8c7accfa877c.png)

What is launch clock network delay? Ans: 0.58ns
Hint - Scroll a little up at the beginning of report after "report_checks" command. There is line after "clock clk (rise edge)"

What is launch clock network delay? Ans: 0.58ns
Hint - Scroll a little up at the beginning of report after "report_checks" command. There is line after "clock clk (rise edge)"

What is the capture clock network delay? Ans: around 0.56ns
Hint - Look in the same report for the line "clock clk (rise edge)"

Type below command:
```
report_checks -path_delay min -digits 4
```
What is library hold time and hold slack? Ans: -9.4ps and 222.5ps respectively

# Lab Day 5 Routing using qflow &  pre and post layout STA:

![Qflow Routing](https://user-images.githubusercontent.com/40173944/99913651-1c682c00-2cf9-11eb-8a17-db13ed606209.png)

Open terminal and type below commands
```
cd vsdflow/my_picorv32
qflow route picorv32
```
![Routing of picorv32](https://user-images.githubusercontent.com/40173944/99938337-deeeb780-2d67-11eb-843b-906707898230.png)

Now perform pre and post layout STA:
```
qflow sta picorv32
qflow backanno picorv32
leafpad log/sta.log
```
In STA log you will find the prelayout clock frequency: 315 MHz
![Pre Layout STA log](https://user-images.githubusercontent.com/40173944/99911751-c3979400-2cf6-11eb-979d-69feb2b3471c.png)

Now if you look in post_sta log file
```
leafpad log/post_sta.log
```
![Post layout STA log](https://user-images.githubusercontent.com/40173944/99911802-0ce7e380-2cf7-11eb-9184-389f60f85385.png)

You will get the post layout clock frequency: 297 MHz

So the post to pre layout clock freq difference is 18 MHz, this is because of the paracitics added after routing.



## Acknowledgement

Kunal Ghosh, Co-founder (VSD Corp. Pvt. Ltd)
