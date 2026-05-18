# Bandgap-IP-Design-using-SKY130-technology-node-VSD
Designed a bandgap reference circuit and implemented it with the SKY130 technology.
#
<img width="803" height="932" alt="Image" src="https://github.com/user-attachments/assets/aff38f2b-5529-41e3-a39e-a4d040d582ad" />

#
EDA and technology files are provided from these 2 sources:

```bash
https://github.com/silicon-vlsi-org/eda-technology
```
```bash
https://github.com/google/skywater-pdk-libs-sky130_fd_pr
```
# Design specifications
- Supply voltage = 1.8V
- Temperature: -40 to 125 C
- Power Consumption < 60uW
- Off current < 2uA
- Start-up time < 2us
- Tempco. Of Vref < 50 ppm

# Device Datasheet
### 1. MOSFET Specifications
| Parameter | NFET (MOSFET) | PFET (MOSFET) |
| :--- | :--- | :--- |
| **Type** | LVT | LVT |
| **Operating Voltage** | 1.8V | 1.8V |
| **Threshold Voltage ($V_{th}$)** | ~ 0.4V | ~ -0.6V |
| **Model Name** | `Sky130_fd_pr__nfet_01v8_lvt` | `Sky130_fd_pr__pfet_01v8_lvt` |
### 2. BJT Specifications
| Parameter | PNP (BJT) |
| :--- | :--- |
| **Current Rating** | 1 µA - 10 µA/µm² |
| **Beta ($\beta$)** | ~ 12 |
| **Emitter Area** | 11.56 µm² |
| **Model Name** | `Sky130_fd_pr__pnp_05v5_W3p40I3p40` |

##  Design Methodology & Calculations


### Step-1: Calculation of Current
* Max. power Consumption < 60uW
* Max Total Current = 60 uW/1.8V=33.33uA
* So, we have chosen 10uA/branch, (3*10=30uA)
* Start-up current 1-2 uA

### Step-2: Choosing Number of BJT in parallel in Branch2
* Less number of BJT: require less resistance value but matching hampers
* More number of BJT: requires higher resistance value but gives good matching
* So a moderate number have chosen (8 BJT) for better layout matching and moderate resistance value.

### Step-3: Calculation of R1
* R1= Vt* ln (8)/I =26 mv *ln(8)/10.7uA=5 KOhm
* R1 size: W=1.41um, L=7.8um, Unit rer value: 2k Ohm
* Number of resistance needed: 2 in series and 2 in parallel (2+2+ (2||2))

### Step-4: Calculation of R2
* Current through ref branch:I3=I2=Vt*ln(8)/R1
* Voltage acress R2: R2*I3=R2/R1(Vt*ln(8))
* Slope of VR2= R2/R1 (ln(8)*115uv)/Deg C.
* Slope of VQ3=-1.6mV/Deg C
* Adding both and equating to zero, R2 will be around 33k Ohm
* Number of resistance needed: 16 in series and 2 in parallel (2+2...+2+ (2||2))

### Step-5: PMOS design in SBCM
* Make both the MP1 and MP2 well in Saturation
* To reduce channel length modulation used L=2um
* Finally the size is L=2u, W=5u and M=4

### Step-5: NMOS design in SBCM
* Make both the MN1 and MN2 either in Saturation or in deep sub-threshold
* We have made it in deep sub-threshold
* To reduce channel length modulation used L=1um
* Finally the size is L=1u, W=5u and M=8
<img width="1627" height="1002" alt="Image" src="https://github.com/user-attachments/assets/ff5dfebe-3cbf-406b-86bc-535a7c9e530f" />

# Component design with spice netlist

### PTAT section
We will start by writing our spice file for our PTAT circuit
ptat_circuit.sp
```bash
*** ptat voltage generation
.lib "/home/Desktop/cad/eda-technology/sky130/models/spice/models/sky130.lib.spice"
.include "/home/Desktop/cad/eda-technology/sky130/models/spice/models/sky130_fd_pr__model__pnp.model.spice"
.global vdd gnd
.temp 27
*** vcvs definition
e1 ra1 qp1 net2 gnd gain=1000
** mosfet definition
xmp1 q1 net2 vdd vdd sky130_fd_pr__pfet_01v8_lvt l=2 w=5 m=4
xmp2 q2 net2 vdd vdd sky130_fd_pr__pfet_01v8_lvt l=2 w=5 m=4
** resistor definition
xra ra1 qp2 gnd sky130_fd_pr__res_high_po_1p41 l=30
** bjt definition
xqp1 gnd gnd qp1 gnd sky130_fd_pr__pnp_05v5_W3p40L3p40 m=1
xqp2 gnd gnd qp2 gnd sky130_fd_pr__pnp_05v5_W3p40L3p40 m=8                                                                          
~
```
Then our PTAT's circuit voltage gen
ptat_voltage_gen.sp
```bash
**** ptat voltage generation circuit *****
.lib "/home/Desktop/cad/eda-technology/sky130/models/spice/models/sky130.lib.spice"
.include "/home/Desktop/cad/eda-technology/sky130/models/spice/models/sky130_fd_pr__model__pnp.model.spice"
.global vdd gnd
.temp 27
*** vcvs definition
e1      out      gnd      ra1      qp1      gain=1000
xmp1    q1       net2     vdd      vdd      sky130_fd_pr__pfet_01v8_lvt     l=2      w=5      m=4
xmp2    q2       net2     vdd      vdd      sky130_fd_pr__pfet_01v8_lvt     l=2      w=5      m=4
*** bjt definition
xqp1    gnd      gnd      qp1      vdd      sky130_fd_pr__pnp_05v5_W3p40L3p40        m=1
xqp2    gnd      gnd      qp2      vdd      sky130_fd_pr__pnp_05v5_W3p40L3p40        m=8
*** high-poly resistance definition
xra1    ra1      na1      vdd      sky130_fd_pr__res_high_po_1p41     w=1.41      l=7.8
xra2    na1      na2      vdd      sky130_fd_pr__res_high_po_1p41     w=1.41      l=7.8
xra3    na2      qp2      vdd      sky130_fd_pr__res_high_po_1p41     w=1.41      l=7.8
xra4    na2      qp2      vdd      sky130_fd_pr__res_high_po_1p41     w=1.41      l=7.8
*** voltage sources for current measurement
vid1    q1       qp1      dc       0
vid2    q2       ra1      dc       0
*** supply voltage
vsup    vdd      gnd      dc       2
.dc     temp     -40      125      1
*** control statement
.control
run
plot v(qp1) v(ra1) v(qp2) v(out)
plot vid1#branch vid2#branch
.endc
.end
```
# 
### CTAT section

```bash
**** ctat voltage generation circuit *****
.lib "/home/Desktop/cad/eda-technology/sky130/models/spice/models/sky130.lib.spice tt"
.include "/home/Desktop/cad/eda-technology/sky130/models/spice/models/sky130_fd_pr__model__pnp.model.spice"
.global vdd gnd
.temp 27
*** bjt definition
xqp1    gnd      gnd      qp1      gnd      sky130_fd_pr__pnp_05v5_W3p40L3p40        m=1
*** supply voltage and current
vsup    vdd      gnd      dc       2
isup    vdd      qp1      dc       10u
.dc     temp     -40      125      5
*** control statement
.control
run
plot v(qp1)
.endc
.end
~                                                                               
```
#
Using NGSPICE we will simulate V(qp1), resulting in -0.00173539 mV/deg C

<img width="1542" height="886" alt="image" src="https://github.com/user-attachments/assets/d0b3012b-e90e-46e4-97f6-a1e165d99322" />

