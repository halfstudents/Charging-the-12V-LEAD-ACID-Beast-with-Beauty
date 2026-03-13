# Charging-the-12V-LEAD-ACID-Beast-with-Beauty
A small 1A constant current charger for motobike lead acid batteries. Capable to charge from dead state comes with minimal form factor.

If you own a motorcycle, you already know the pain. I was little out of station for a couple of weeks and when I came home and gave a push to my bike, there was nothing. The battery is dead. Lead-acid batteries self-discharge over time, and if you don't ride often enough, the battery just slowly dies on you. Buying a new battery every few months is not an option. So, I used to charge manually on a workstation which costs me time and money. Even though most of them are using just a transformer with a rectifier, no regulation, no protection, just brute force current being dumped into the battery. But I am an electronics engineer and that’s how I decided to give a try to lead acid battery charging.

![Image](https://github.com/user-attachments/assets/97013eae-f58f-4bb6-9903-9815d4ed7259)

The one that actually understands how to charge a battery properly. With trickle charge for deeply discharged batteries, constant current for bulk charging, over-charge for topping it off, and float charge to keep it maintained without overcharging. The whole charging profile, done right. That's where the CN3767 comes in. It's a dedicated 12V lead-acid battery charger controller IC from Consonance Electronics that does everything I just described. I have designed a PCB in EasyEDA and fabricated it from [JLCPCB](https://jlcpcb.com/?from=audrey3) and tested out the final prototype. 

**Why CN3767?**

There are a lot of battery charger ICs in the market, but for 12V lead-acid specifically, the options narrow down quickly. Most charger ICs are designed for Li-ion and Li-Po. The CN3767 is purpose-built for 12V lead-acid batteries, and that makes the design so much cleaner.

![Image](https://github.com/user-attachments/assets/d5e3d7c1-8f76-4752-a3d1-dc41bae15b6a)


Features:

Complete 4-stage charging profile

Wide input range: 6.6V to 30V

Up to 4A charge current

Solar panel MPPT

Fixed regulation voltages

Status indication

High switching frequency: 300kHz

Automatic recharge

Sleep mode

The IC uses a PWM buck (step-down) topology with an external P-channel MOSFET as the switch. So it is clear from the description that this is a controller IC, not a standalone charger. Choose your own MOSFET, diodes and inductor based on your power requirements.

**Lead Acid Battery Charging Profile:**

Lead-acid batteries have a very specific charging requirement, and getting it wrong either undercharges the battery or overcharges it. Which causes capacity reduction or damaging the battery. The CN3767 handles all four stages automatically.

![Image](https://github.com/user-attachments/assets/ee27cf4d-ed43-4a32-b37b-2a0616f57c90)

Stage 1: Trickle Charge

When the battery is deeply discharged and its voltage is below 75% of the over-charge voltage (that's 75% of 14.8V = 11.1V), the CN3767 enters trickle charge mode. In this mode, the charge current is only 17.5% of the programmed constant current. Because dumping full current into a deeply discharged lead-acid battery can damage it. The low trickle current gently brings the battery up to a safe voltage level before the real charging begins.

Stage 2: Constant Current (CC)

Once the battery voltage rises above 11.1V, the charger switches to constant current mode. This is the main charging phase where the battery receives the full programmed current. The charge current is set by the sense resistor using: ICH = 0.12V / RCS.

For my design, I'm using a 0.12 ohm sense resistor, which gives me exactly 1A of charge current. That's a safe and steady rate for a typical motorcycle battery (usually 7-12Ah capacity). During CC mode, the battery voltage steadily rises while the current remains constant. The CHRG LED stays on to indicate active charging.

<img width="1174" height="654" alt="Image" src="https://github.com/user-attachments/assets/d8c84545-2ebd-46cd-9153-a33666277143" />

Stage 3: Over-Charge (CV) - Constant Voltage

When the battery voltage approaches 14.8V (the over-charge regulation voltage), the charger transitions to over-charge mode. Now the voltage is held constant at 14.8V and the current starts to taper down. This is the classic CV (constant voltage) phase. The over-charge continues until the current drops to 38% of the constant charge current. For my 1A charger, that means the over-charge ends when the current falls below 0.38A. At this point, the battery is essentially full.

Stage 4: Float Charge - Maintenance

After over-charge terminates, the charger enters float mode. The battery voltage is now regulated to 13.55V (which is 91.57% of the over-charge voltage). This is the maintenance voltage that keeps the battery topped up without overcharging. Float mode compensates for self-discharge and small loads. The DONE LED turns on to indicate the battery is fully charged and being maintained

**Components Required:**

Lead Acid Battery

CN3767

P-Channel MOSFET AO3401A

Schottky Diode SS34F-AT

Power Inductor 22uH

Current Sense Resistor100m ohm (0.1 ohm)

MPPT Divider (Upper) 120K ohm

MPPT Divider (Lower) 10K ohm

CHRG LED

DONE LED

Input Electrolytic Cap 100uF / 35V

Input Ceramic Cap 1uF

Output Electrolytic Cap 47uF / 25V

Output Ceramic Cap 1uF

Power source

PCB from JLCPCB

**Circuit Design & Schematic:**

<img width="1053" height="701" alt="Image" src="https://github.com/user-attachments/assets/05606be7-f393-499a-99e6-5fcecd5ad59b" />

When the DRV pin pulls the gate of Q2 low, the MOSFET turns on and current flows through the inductor into the battery. When the MOSFET turns off, the inductor current freewheels through the catch diode D2 back to ground. The AO3401A is a logic-level P-channel MOSFET in SOT-23 package. It has low Rds(on) and its gate threshold is well within the CN3767's drive voltage. It can handle the current requirements for a 1-5A charger easily.

<img width="1812" height="818" alt="Image" src="https://github.com/user-attachments/assets/6e2cd3bb-c36e-4bd3-b4f7-c73f1a49e7fc" />


The input diode D1 serves as a blocking diode to prevent battery current from flowing back to the input when the supply is removed. Without it, the CN3767 would draw about 52uA from the battery in sleep mode. The SS34F-AT is a 3A/40V Schottky diode with low forward voltage drop, keeping power losses minimal. C10 (100uF), C4 (1uF) and C3 (100nF) these three-capacitors suppress the high-frequency oscillation that can occur during MOSFET switching. The current sense resistor R1 sits between the inductor output and the battery. The CN3767 measures the voltage drop across this resistor via the CSP and BAT pins. With R1 = 0.1 ohm:

ICH = 0.12V / 0.10 ohm = 1.2A

If you want exactly 1A, use a 0.12 ohm resistor instead. The schematic includes a handy reference table for different current settings. LED1 (Red) lights up during active charging (trickle, CC and over-charge modes). LED2 (Green) lights up when the battery enters float mode (fully charged). The resistor R4 (10 ohm) is placed in the power path near the catch diode D2. This helps suppress high-frequency oscillation (>10MHz) that can occur during MOSFET switching transitions. And for output filtering C2 (47uF) and C8 (1uF) are there. The COM pin requires an RC network for loop stability. I used R3 (120 ohm) in series with compensation capacitors C5 and C6 connected to GND. This keeps both the current loop and voltage loop stable across all operating conditions.

**PCB Design:**

<img width="1629" height="579" alt="Image" src="https://github.com/user-attachments/assets/d656e15a-75d9-4cc7-85e2-02cb2222d5d8" />

I have tried to make the PCB as small as possible, because mine one is made to handle a max of 1A for now the design is not so critical. But we have to keep some rules in mind when using a switching convertor IC like this one. I have kept the switching loops tight; the traces connecting D1, D2, Q2 (MOSFET), L1 (inductor) and input bypass capacitors should be as short as possible. These carry high-frequency switching currents and long traces act as antennas. I have poured the GND copper pour on both sides, keeping the minimal via’s in the path of trace. Kept the DRV to MOSFET gate connection short to minimize ringing and gate drive losses. Place the decaps on the right position, input and output caps at the end of IO pads.

![Image](https://github.com/user-attachments/assets/20780f9d-56ea-486f-8cf5-f77bc1f6041c)

You can download the Gerber files along with BOM and CPL from here. I have used JLCPCB for manufacturing because their services are available in a wide domain with reasonable prices. And because I have designed the PCB in easyEDA online which has an integration with [JLCPCB](https://jlcpcb.com/?from=audrey3). At least this can give me a peace of mind over the files.

**Assembling the PCB:**

![Image](https://github.com/user-attachments/assets/5bd9d164-cd0f-4c15-8684-5b15bbec9e7b)

I had ordered the PCB and soldered it manually which takes me 2-3 hours of component sorting and got it done. Although I am able to make it working in the end. But I would rather use assembling services next time because the prices are quite affordable. If you are also soldering with hand, better to use a stencil. First solder all the SMD components in one go, so you can debug the PCB easily. After the SMDs are done go for TH soldering.

![Image](https://github.com/user-attachments/assets/b1d2b5a4-ed28-46c0-94c7-57cf49cb6633)


**Testing & Results:**

I set up a DC bench power supply set to 18V and took a 12V/7Ah lead-acid motorcycle battery. For monitoring put the multimeter on battery terminals, current clamp on charge line.

Constant Current (CC) Mode Test:

![Image](https://github.com/user-attachments/assets/4fb6b7b7-9d80-419f-aef4-f3b7b436bf83)

With the battery at around 12V, I connected the charger. The red CHRG LED immediately lit up, confirming the charger entered the charging state. The current stayed rock-steady at approximately 1A throughout the CC phase. That's the CN3767 doing its job, regulating the current via the sense resistor feedback loop.

Constant Voltage (CV) Mode Test:

![Image](https://github.com/user-attachments/assets/78044008-cabc-4fb2-b986-a47094077285)

As the battery voltage approached 14.8V, I observed the transition to over-charge mode. The voltage locked at 14.8V and current started tapering down from 1A. This is the critical phase where the charger is topping off the battery. The voltage holds steady while the current gradually decreases as the battery reaches full capacity.

**MPPT Solar Tracking Feature**

The most underrated feature is this MPPT one of this one, yet it is quite powerful in this package and price. The MPPT pin (Pin 6) connects to a voltage divider formed by R2 (120K) and R5 (10K). The CN3767 regulates this pin to 1.205V, which in turn sets the solar panel operating voltage:

VMPPT = 1.205 x (1 + R2/R5)

VMPPT = 15.665V

This is the sweet spot for a standard 18V solar panel, whose maximum power point typically falls around 15-16V. If you're not using a solar panel and just powering from a DC adapter, the MPPT divider still works fine and it just won't actively track anything.

**Outro:**

![Image](https://github.com/user-attachments/assets/7a8d7a97-89eb-4115-919b-be4b418baeac)

Building a proper battery charger is not that hard when you have the right IC. The CN3767 takes care of the entire charging algorithm. Even though I am going to sell some pieces to my motorcycle repair shop, so everyone who needs a proper solution can get this design. I will increase the rating to maybe 3A in CC mode for faster charging. Motorcycle batteries can easily handle up to 5A. We have seen that the transition from CC to CV mode is smooth, the regulation voltages are accurate, and the LED indicators give clear feedback on the charging state. The best part? That MPPT input means I can slap a small solar panel on the bike someday and have a self-maintaining battery system. Charging the beast with beauty, indeed.
