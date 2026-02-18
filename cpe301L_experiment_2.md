CPE 301L / CPE310L EMBEDDED SYSTEM DESIGN LABORATORY

LABORATORY 2 GETTING STARTED WITH AVR

DEPARTMENT OF ELECTRICAL AND COMPUTER ENGINEERING
UNIVERSITY OF NEVADA, LAS VEGAS

# GOAL

Learn how to install AVR Studio and WinAVR. Learn how to assemble, compile, debug, and program using Atmel Studio. Understand the basic functions for compiling and debugging in AVR Studio 7. Understand the basic functions of the AVRISP mkII. Understand the ports and pins in AVR.

# BACKGROUND

## MICROCHIP STUDIO INSTALLATION:

Follow the instruction manual in Webcampus

## AVR ISP MKII

Review AVR ISP mkII user guides:
http://people.ece.cornell.edu/land/courses/ece4760/AtmelStuff/avrispmkii_ug.pdf

## Connecting AVR ISP mkII

For the majority of these labs we will be using the AVRISP mkII. The figure below displays the required connections needed for the ISP.

![img-0.jpeg](assets/cpe301L_experiment_2_img-0.jpeg)
Fig. 3. mkII connector

Using the ATmega 328P, the same ports must be attached to the ISP. Failure to connect to the proper ports will result in an error displayed when the AVR ISP mkII attempts to read the attached device. You will have to refer to the datasheet of each IC to see the location of each port (SCK, MISO, MOSI) since they will be different for each chip.

UNIV DEPARTMENT OF ELECTRICAL AND COMPUTER ENGINEERING

---

CPE301L / CPE310L EMBEDDED SYSTEM DESIGN

# BOARD SETUP

The UNLV ECE development board uses the AVR ISP MKII programmer:

![img-1.jpeg](assets/cpe301L_experiment_2_img-1.jpeg)

Steps to set up the board:

1. connect the AVR MKII programmer to the ISP connector on board
2. connect the AVR MKII programmer to PC computer with USB cable
3. connect the board to the PC computer using another USB cable (this provides power to the board)
4. set the SELECTOR switch to USB (selector select the power source: USB or DC power supply).

Board components: (for the full documentation, see Board Manual in references).

UNLV DEPARTMENT OF ELECTRICAL AND COMPUTER ENGINEERING

---

CPE301L / CPE310L EMBEDDED SYSTEM DESIGN

![img-2.jpeg](assets/cpe301L_experiment_2_img-2.jpeg)

1. LCD Header.
2. CONTRAST: This potentiometer controls the contrast of the LCD Display.
3. DATA: Data for LCD. Control pins are at 21.
4. SW1-SW4: Switches which can be used as a general purpose input signal. Male header is the output for respective switches, when pressed is logic 1.
5. Keypad: Keypad array. C1, C2, C3, C4, R1, R2, R3, R4 are used to excite and detect the pressed key.
6. DIP Switch: 1-8 are the output pins for the respective switch.
7. ISP: This is an ISP connector to connect AVR ISP programmer.
8. BUZZER: This a piezoelectric buzzer. Signal to the buzzer is at 21.
9. SELECTOR: This is used to select the power source between USB and Adapter.
10. RESET: This switch resets microcontroller.

11. POWER: This is a power indicator.
12. DB9: This is a DB9 connector and pinout for respective pins of DB9 connector.
13. X2: This is used for USB power supply.
14. J2: DC power adapter jack.
15. BATT: This is a battery source. VBAT+ and VBAT- are positive and negative terminal of the battery.
16. RTC-CLK: This is a connection to 32.768KHz clock.
17. PORTB/PORTC-AREF/PORTD: This is a pin out for GPIOs and other peripherals of the microcontroller.
18. CLK-OUT / EXT OSC: EXT OSC is used to connect external clock and CLK-OUT is a pinout that can be connected back to the microcontroller.
19. LED: Pin out for LEDs
20. LED Array
21. Pinout for LCD, Buzzer and USB. RS, RW, EN are control pins for LCD, DATA pins are at 3. Buzzer is an input signal for a buzzer. D+, D- are data line for a USB.

UNIV DEPARTMENT OF ELECTRICAL AND COMPUTER ENGINEERING

---

CPE301L / CPE310L EMBEDDED SYSTEM DESIGN

# REGISTER BITS

- DDxn bits are accessed at the DDRx I/O address
- PORTxn bits at the PORTx I/O address
- PINxn bits at the PINx I/O address

Where n = pin bit number in the Port Register

# DDRxn

The DDxn bits in the DDRx Register select the direction of this pin. If DDxn is written to '1', Pin is configured as an output pin. If DDxn is written to '0', Pin is configured as an input pin.

# PORTxn

The PORTxn bits in the PORTx register have two functions. They can control the output state of an output pin and the setup of an input pin.

## As an Output:

If a '1' is written to the bit when the pin is configured as an output pin, the port pin is driven high. If a '0' is written to the bit when the pin is configured as an output pin, the port pin is driven low.

## As an Input:

If a '1' is written to the bit when the pin is configured as an input pin, the pull-up resistor is activated. If a '0' is written to the bit when the pin is configured as an input pin, the port pin is tri-stated.

# PINxn

The PINxn bits in the PINx register are used to read data from port pin. When the pin is configured as a digital input (in the DDRx register), and the pull-up is enabled (in the PORTx register) the bit will indicate the state of the signal at the pin (high or low).

**Note:** If a port is made an output, then reading the PINx register will give you data that has been written to the port pins.

## As a Tri-State Input:

When the PORTx register disables the pull-up resistor the input will be tri-stated, leaving the pin left floating. When left in this state, even a small static charge present on surrounding objects can change the logic state of the pin. If you try to read the corresponding bit in the pin register, its state cannot be predicted.

UNIV DEPARTMENT OF ELECTRICAL AND COMPUTER ENGINEERING

---

CPE301L / CPE310L EMBEDDED SYSTEM DESIGN

Examples:

All PORTA pins set as inputs with pull-ups enabled and then read data from PORTA:

DDRB = 0x00;  //make PORTB all inputs
PORTB = 0xFF;  //enable all pull-ups
int data = PINA;  //read PORTA pins into variable data

PORTB set to tri-state inputs:

DDRB = 0x00;  //make PORTB all inputs
PORTB = 0x00;  //disable pull-ups and make all pins tri-state, also a default state after reset

PORTA lower nibble set as outputs, higher nibble as inputs with pull-ups enabled:

DDRA = 0x0F;  //lower pins output, higher pins input
PORTA = 0xF0;  //output pins set to 0, input pins enable pull-ups

## PINOUT OF ATMEGA328P

The microcontroller ATmega328P includes multiple inputs and outputs. Note that most of the pins have multiple function. Refer to the diagram below:

![img-3.jpeg](assets/cpe301L_experiment_2_img-3.jpeg)

## Assembly code

Assembly is a low-level programming language that allows users to generate expressions that are equivalent to machine langue instructions. Because assembly is essentially machine language, it is often used instead of high-level programming languages (such as C, C++, Java, etch) when programming devices such as microcontrollers and microprocessors. Simple instructions such as c = a+b are much longer in assembly code. The benefit of this is that assembly language provides a programmer more control over the instructions. Below table shows basic instructions in AVR assembly. The notation of operation: Rd ← Rd + Rr means that the result of addition of Rd and Rr registers will be placed in the Rd register.

UNIV DEPARTMENT OF ELECTRICAL AND COMPUTER ENGINEERING

---

CPE301L / CPE310L EMBEDDED SYSTEM DESIGN

AVR Assembly instructions

|  Mnemonics | Operands | Description | Operation  |
| --- | --- | --- | --- |
|  ARITHMETIC AND LOGIC INSTRUCTIONS  |   |   |   |
|  ADD | Rd, Rr | Add two Registers | Rd ← Rd + Rr  |
|  SUB | Rd, Rr | Subtract two Registers | Rd ← Rd - Rr  |
|  AND | Rd, Rr | Logical AND Registers | Rd ← Rd • Rr  |
|  OR | Rd, Rr | Logical OR Registers | Rd ← Rd v Rr  |
|  NEG | Rd | Two's Complement | Rd ← 0x00 - Rd  |
|  INC | Rd | Increment | Rd ← Rd + 1  |
|  DEC | Rd | Decrement | Rd ← Rd - 1  |
|  BRANCH INSTRUCTIONS  |   |   |   |
|  RJMP | k | Relative Jump | PC ← PC + k + 1  |
|  RCALL | k | Relative Subroutine Call | PC ← PC + k + 1  |
|  RET |  | Subroutine Return | PC ← STACK  |
|  BREQ | k | Branch if Equal | if (Z = 1) then PC ← PC + k + 1  |
|  BRNE | k | Branch if Not Equal | if (Z = 0) then PC ← PC + k + 1  |
|   |  | Relative Jump | PC ← PC + k + 1  |
|  DATA TRANSFER INSTRUCTIONS  |   |   |   |
|  MOV | Rd, K | Move Between Registers | Rd ← Rr  |
|  LDI | Rd, K | Load Immediate | Rd ← K  |
|  ST | X, Rr | Store Indirect | (X) ← Rr  |
|  IN | Rd, P | In Port | Rd ← P  |
|  OUT | P, Rr | Out Port | P ← Rr  |
|  PUSH | Rr | Push Register on Stack | STACK ← Rr  |
|  POP | Rd | Pop Register from Stack | Rd ← STACK  |

Fig. 4. Basic ASM functions

## PRELAB

### 1. AVR Installation

Watch the videos explaining how to install and use Atmel Studio
https://www.microchip.com/content/dam/mchp/documents/atmel-start/Atmel-Studio-7-User-Guide.pdf

### 2. Watch videos about using Atmel Studio

- Create a New C Project for GCC in Atmel Studio 7
https://www.microchip.com/content/dam/mchp/documents/atmel-start/Atmel-Studio-7-User-Guide.pdf

- Build a Project in Atmel Studio 7
https://www.microchip.com/content/dam/mchp/documents/atmel-start/Atmel-Studio-7-User-Guide.pdf

- Using the Simulator in Atmel Studio 7
https://www.microchip.com/content/dam/mchp/documents/atmel-start/Atmel-Studio-7-User-Guide.pdf

UNIV DEPARTMENT OF ELECTRICAL AND COMPUTER ENGINEERING

---

CPE301L / CPE310L EMBEDDED SYSTEM DESIGN

3. What is bit masking? What can you achieve when masking by AND, OR, XOR? Show with example.
4. Refresh your knowledge about binary and hexadecimal number systems
5. Carefully read sections 4.1-4.3 of AVR Assembler User Guide: http://eelabs.faculty.unlv.edu/docs/guides/AVR_assembly.pdf
6. Overview the rest of AVR Assembler User Guide: http://eelabs.faculty.unlv.edu/docs/guides/AVR_assembly.pdf
7. Read the datasheet pg 116-124

## PRELAB DELIVERIES: [20 PTS]

1. Submit answer to question (3) from prelab [5 pts]
2. Identify the binary, hexadecimal and decimal numbers among 101, 0b101, 0x101. Convert each of these numbers in decimal, binary and hexadecimal. [5 pts]
3. Find the solution to the following operation: 1011 &gt;&gt; 2, 1011 ^ 1010, 1011 ^ (~1011), 1011 &amp; (~0 &lt;&lt; 2) [5 pts]
4. What is the status register in AVR? How many conditions are there which sets the status register? (Hint: Check Datasheet for answer) [5 pts]

## EXPERIMENTS

### 1. Create simple assembly project for Assembly and C

Program the following assembly code using Atmel Studio 7 into your ATmega328P. The code should light up an LED connected to pin (PB.4)

```
.org 0
SBI DDRB,4 ;set PB4 as an output
LDI R17,16 ;value is 16 to light up bit 4
OUT PORTB,R17 ;sends value of R17 to corresponding bit
```

Program the following C code using AVR studio with your ATmega328P.

```c
#include <avr io.h="">
int main ()
{
DDRB = 0xFF; //set all of portB to output
while (1)
PORTB = 0x88; //set PB7 and PB3 high
return 0;
}
```

UNLV DEPARTMENT OF ELECTRICAL AND COMPUTER ENGINEERING</avr>

---

CPE301L / CPE310L EMBEDDED SYSTEM DESIGN

2. Taking input (Connect LEDs to PORTB using FRC, Connect one of the pushbutton switch to PORTC pin 0 (PC0).

C:

```c
#include <avr io.h="">
int main(void)
{
DDRB = 0xFF;  //set PORTB as an output for LED
DDRC = 0x00;  //set PORTC as an input PORT
PORTC = 0xFF;  //activate the pull-up resistor for PORTC
while (1)
{
if ((PINC &amp; 0x01) == 0x01)  //check if pushbutton connected to PINC.0 is pressed
PORTB = 0xFF;  //if the button is pressed, turn ON LEDs
else
PORTB = 0x00;  //if the button is not pressed, turn OFF LEDs
}
}
```

ASM:

.org 0
ldi r16, 0xff  // load register with 0xff

out DDRB,r16  //set PORTB as an output for LED
// no need to set DDRC = 0x00, by default PORTs are set as input
out PORTC,r16  //activate the pull-up resistor for PORTC

loop:
in r16, PINC  //read the input status from PINC
andi r16, 0x01  //masking
cpi r16,0x01  //compare if PINC == 0x01, to check the button press
BREQ LED_ON  //if button is pressed, branch to label LED_ON
ldi r16, 0x00  //turn off the led, PORTB = 0x00
out PORTB, r16
rjmp loop

LED_ON:  //turn on the led, PORTB = 0xFF
ldi r16, 0xFF
out PORTB, r16

UNIV DEPARTMENT OF ELECTRICAL AND COMPUTER ENGINEERING</avr>

---

CPE301L / CPE310L EMBEDDED SYSTEM DESIGN

rjmp loop

//return back

## 3. Create a led sequence

Use 8 led to create two different fancy LED sequence using the idea that you learned so far. Only use masking and shifting technique to create the sequence.

Create a variable called “MODE_” that holds the input from a pushbutton switch.

Depending on the input of “MODE_” variable, “1” will display the first sequence and “0” will display the second sequence.

Write in both assembly and C.

## 4. Write a C and assembly code to read the status of the 2-dip switch connected at PB0,PB1 and based on the status of the DIP switch perform the following operation on the 4 LEDs (LED1,LED2,LED3,LED4) connected to PB2, PB3,PB4,PB5:

```
00 -&gt; 1111 (all LEDs ON)
01 -&gt; 1010 (alternate LED is on)
10 -&gt; 0000 (all LEDs OFF)
11 -&gt; 0011 (LED1 and LED2 OFF, LED3 and LED4 ON)
```

## POSTLAB [40 PTS]

Include the following elements in the report document:

|  Section | Element  |   |
| --- | --- | --- |
|  1 | Theory of operation [2 pts]
Include a brief description of every element and phenomenon that appears during the experiments.  |   |
|  2 | Prelab report  |   |
|  3 | Results of the experiments  |   |
|   |  Experiment | Experiment Results  |
|   |  1 | Picture of the board [1 pts]  |
|   |  2 | Picture of the board [1 pts]  |
|   |  3 | C and ASM code [5 + 5] pts
Picture of the board [2 + 2] pts  |

UNIV DEPARTMENT OF ELECTRICAL AND COMPUTER ENGINEERING

---

CPE301L / CPE310L EMBEDDED SYSTEM DESIGN

|   | 4 | C and ASM code [5+5] pts
Picture of the board [2 + 2] pts  |
| --- | --- | --- |
|  4 | Answer the questions  |   |
|   |  Question no. | Question  |
|   |  1 | Go to Atmel's website and find another microcontroller that is similar enough to replace the Atmega328P. Explain why it would be a suitable replacement and list some of the differences from the 328P. [1 pts]  |
|   |  2 | In experiment 2, do you think the code will work perfectly fine if the internal pull ups were not activated? Show the C code changes that would fix the problem. [1 pts]  |
|   |  3 | List and explain the meanings for the different LED combinations on the AVRISP mkII [1 pts]  |
|   |  4 | Explain what is happening on each line of the following code. If you were to execute this code what would be the final decimal value stored in R20? [3 pts]  |
|   |   |  LDI R20, 0x75  |
|   |   |  LDI R21, 0x05  |
|   |   |  LDI R22, 0x24  |
|   |   |  ADD R20, R22  |
|   |   |  SUB R22, R21  |
|   |   |  ADD R20, R21  |
|   |   |  SUB R22, R20  |
|   |   |  ADD R20, R22  |
|   |   |  MOV R20, R21  |
|   |   |  RJMP DONE  |
|  ADD R21, R20  |   |   |
|  SUB R21, R22  |   |   |
|  DONE: SUB R20, R21  |   |   |
|  END: RJMP END  |   |   |
|  5 | Conclusions [2 pts]
Write down your conclusions, things learned, problems encountered during the lab and how they were solved, etc.  |   |
|  6 | Attachments
Zip your projects. Upload in webcampus, or provide link to the zip file on Google Drive
List of attachments to deliver:
1. Project from experiment 3
2. Project from experiment 4  |   |

# REFERENCES

1. ATMega 328P Manual: http://eelabs.faculty.unlv.edu/docs/guides/ATmega328P_Datasheet.pdf
2. UNLV Board Manual: http://eelabs.faculty.unlv.edu/docs/guides/UNLV_328P_Board_Manual.pdf
3. AVR Assembly Intro: http://eelabs.faculty.unlv.edu/docs/guides/AVR_assembly.pdf
4. AVR Assembly Instruction Set: http://eelabs.faculty.unlv.edu/docs/guides/AVR_instruction_set_manual.pdf
5. AVR assembly overview:
6. http://web.csulb.edu/~hill/ee346/Lectures/03%20Load-Store%20Progr.pdf

UNLV

DEPARTMENT OF ELECTRICAL AND COMPUTER ENGINEERING