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

<img width="751" height="490" alt="Image" src="https://github.com/user-attachments/assets/02c49e1d-bd41-471c-a894-96edd2976288" />

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


# Component design with spice netlist

### PTAT section


Using NGSPICE we will simulate the slope of V(qp1), resulting in -0.00173539 mV/deg C

<img width="1542" height="886" alt="image" src="https://github.com/user-attachments/assets/d0b3012b-e90e-46e4-97f6-a1e165d99322" />

#

### CTAT section


Now will simulate the slope of V(Ra)-(qp2), resulting in 0.0001875 mV/deg C

<img width="512" height="373" alt="image" src="https://github.com/user-attachments/assets/e78d7656-b9a4-4dcc-9610-6324ac043213" />

#

### Mirror Section and ref current

<img width="1058" height="770" alt="image" src="https://github.com/user-attachments/assets/70931215-cb9d-44f8-8718-4b898c00a05c" />

#

Now will observe our vdd compared to reference and net currents in transient analysis

<img width="1497" height="861" alt="image" src="https://github.com/user-attachments/assets/3dc810e6-d891-4ad4-b5a2-f77e9f182354" />

<img width="1499" height="870" alt="image" src="https://github.com/user-attachments/assets/fbd89476-fa54-4bfd-9b1c-0558921fcb98" />

<img width="1462" height="867" alt="image" src="https://github.com/user-attachments/assets/14168c36-c43a-419d-a864-0ab16c0bb858" />

# Layout

Using magic we will create the layout of our circuit

### Starter circuit
<img width="762" height="594" alt="image" src="https://github.com/user-attachments/assets/df3caf4b-776b-4bc1-b4ec-fbd35e735609" />

#
### Res bank
Created following common centroid matching technique 
<img width="1276" height="801" alt="image" src="https://github.com/user-attachments/assets/338e588e-59af-447f-b550-7dc141adcdd5" />

#
### The Pfet layer
As previously calculated i used 16 transistors + 1 dummy
<img width="1266" height="801" alt="image" src="https://github.com/user-attachments/assets/87efd72f-168d-46de-835d-c3f455944aa9" />
<img width="1264" height="771" alt="image" src="https://github.com/user-attachments/assets/f20f48a6-6457-427f-a63e-3e7fccb6d4c7" />

#
### The Nfet layer
16 + some dummies, common centroid matching
<img width="1277" height="796" alt="image" src="https://github.com/user-attachments/assets/f5305220-8584-4987-9310-1db4f9cac634" />
<img width="1276" height="773" alt="image" src="https://github.com/user-attachments/assets/475dcc3b-3801-42fd-9a6c-20d91135a20d" />

#
### BJT Layer
10 Bjt surrounded by dummies, 8 transistors from Q2,1 from Q1,  1 from Q3 
<img width="1270" height="775" alt="image" src="https://github.com/user-attachments/assets/cfec0593-d2b9-4af8-a52c-f8d9862e42a4" />

#
### Full circuit layout

<img width="1279" height="775" alt="image" src="https://github.com/user-attachments/assets/514d29fe-a886-40a1-82f5-900074248d5b" />
<img width="927" height="809" alt="image" src="https://github.com/user-attachments/assets/fe3cf08b-96de-44d8-8b4c-696e5be6bf24" />


