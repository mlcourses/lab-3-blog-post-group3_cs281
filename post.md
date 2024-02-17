# Lab 3: Crafting a 7-Segment Ramp Circuit


## Overview and Motivation
In this week's lab, we will use an analog input to display a number on a 7-segment display. A 7-segment display contains 7 leds that are arranged to form a number. We will design and build a more complex combinational circuit to display a number corresponding to an analog input. Given the complexity of this circuit, we will use K-maps in order to minimize our circuit expressions so it will be simpler for us to build the circuits that will help us perform the 7-segment using analog input.

**Main goal**: Build a combinational circuit to perform the 7-segment. The digital signal will be a decimal value between 0 and 5 inclusive.

This is the circuit overview

<img src="https://github.com/mlcourses/lab-3-blog-post-group3_cs281/blob/main/assets/circuit%20overview.png" alt="alt text" width="550"/> 

To achieve this goal, we will use the voltage divider.

## Materials

- PB-503 breadboard
- 7-segment display
- Wires
- Arduino kit
- Arduino controller with the USB adapter
- Computer for the Arduino IDE
- 7404 chip (NOT gate)
- 7408 chip (AND gate)
- 7432 chip (OR gate)
- 7486 chip (XOR gate)


## Project Steps

### Step 1: Converting the potentiometer analog input to digital output using an Arduino

A voltage divider is an arrangement of resistors configured to create a target voltage.

<img src="https://github.com/mlcourses/lab-3-blog-post-group3_cs281/blob/main/assets/Voltage%20divider.png" alt="alt text" width="550"/> 

By choosing our two resistance values, we can pick any voltage V that we desire. Here, we will be using V = 5 volts, so with a voltage divider, we can pick between 0 and 5 volts.

Using the concept of a voltage divider, a potentiometer (“pot”) is a variable resistor that allows us to fine tune our voltage divider. Like the image below, the potentiometer allows us to vary the values of `R1` and `R2` so we can achieve any voltage V between 0 and 5 volts.

<img src="https://github.com/mlcourses/lab-3-blog-post-group3_cs281/blob/main/assets/potentio.png" alt="alt text" width="550"/> 

Our breadboards have two potentiometers, and we will be using the 10k potentiometer.

To wire a potentiometer to the arduino:
- The leftmost connection column of the pot will be wired to GND
- The rightmost column of the pot will be wired to +5V
- Wire one of the middle connections to *<ins>analog input pin 0 (**not digital pin 0**)* on the Arduino. This is how the output signal V from the pot will be fed to the Arduino.
- Remember to **CONNECT YOUR ARDUINO TO THE BREADBOARD VIA GND**.

Your wires on the pot should look like this:

<img src="https://github.com/mlcourses/lab-3-blog-post-group3_cs281/blob/main/assets/wiring.png" alt="alt text" width="550"/> 

Now, connect the Arduino to a laptop, type the following Arduino program, compile it, and write it to the board.

```c
const int potpin = 0;
const int WAIT = 1000; // 1 second delay
void setup () {
Serial.begin(9600);
pinMode(11,OUTPUT);
pinMode(12,OUTPUT);
pinMode(13,OUTPUT);
pinMode(potpin,INPUT);
}
void loop () {
int val;
int dval;
int bitval;
val = analogRead(potpin);
dval = val/171; // normalizing factor-->adjust this to get the range you want
Serial.print("From Pot: ");
Serial.println(val);
Serial.print("Decimal Value Conversion: ");
Serial.println(dval);
//use bit ops to get each bit!
bitval = dval & 1;
digitalWrite(11,bitval); // signal C
dval = dval >> 1;
bitval = dval & 1;
digitalWrite(12,bitval); // signal B
dval = dval >> 1;
bitval = dval & 1;
digitalWrite(13,bitval); // signal A
delay(WAIT);
}
```

The above code:

- Connects a potentiometer to Arduino, linking its middle pin to analog input `A0` and its ground to the breadboard's GND.
- Initiates serial communication at 9600 bps and configures digital pins 11, 12, and 13 as outputs, with the potentiometer pin as input.
- Reads analog value from the potentiometer, normalizes it to a decimal value between 0 and 5 by dividing with a factor of 171.
- Outputs the raw potentiometer value and the converted decimal value to the serial monitor.
- Uses bitwise operations to isolate and output each bit of the decimal value to digital pins 11, 12, and 13, representing signals `C`, `B`, and `A` respectively.
- Incorporates a one-second delay between readings for clear observation of changes.

Load the program and test your circuit.

### Step 2: Wiring the 7-segment display - LED B

We are now ready to wire our 7-segment display. We will start by wire the **LED B** first, and we will wire each LED as we go along the way through each step.

<img src="https://github.com/mlcourses/lab-3-blog-post-group3_cs281/blob/main/assets/7-segment.png" alt="alt text" width="250"/> 

Using this truth table:

<img src="https://github.com/mlcourses/lab-3-blog-post-group3_cs281/blob/main/assets/IMG_7724.jpg" alt="alt text" width="550"/> 

We were able to build the following circuit:

<img src="https://github.com/mlcourses/lab-3-blog-post-group3_cs281/blob/main/assets/D15A342E-E1D0-4C78-B1C1-B332B5DAC9EA.JPG" alt="alt text" width="550"/> 

To determine the SOP (Sum of Products) expression necessary for lighting up LED B, we utilized K-Maps for each LED. Below, you will find the K-Map and the resulting SOP expression to activate LED B:

<img src="https://github.com/mlcourses/lab-3-blog-post-group3_cs281/blob/main/assets/LED%20B.png" alt="alt text" width="300"/> 

As such, we will need a 7404 inverter for the NOT gate and one 7432 chip for the OR gate.

And now we are ready to wire the first LED light:
- Put the chips accordingly like the circuit on the breadboard.
- Wire and ground the chips.
- Input for `B2` will be from output pin 13 of the Arduino, `B1` will be from pin 12, and `B0` will be from pin 11.
- We would want to reuse the chips without overusing too many of them, so we would optimize and utilize **ALL GATES** of all of the chips.
- Connect pin 13 on the Arduino (`B2`) to gate `3A` of the 7404 inverter, and connect pin 11 (`B0`) to gate `1A` of the inverter. 
- From here, connect gate `1Y` and `3Y` (the two outputs) to gates `1A` and `1B`, respectively, of the 7432 OR chip.
- Finally, connect the output from gate `1Y` of the chip to the 7-segment display. Before we do this, **PUT A RESISTOR BETWEEN EACH SEGMENT AND THE INPUT**. We have to do this to make sure that we don’t overload the display. An easy way to do this is to tie each pin to its own row through a resistor.
- Make sure to **GROUND YOUR 7-SEGMENT DISPLAY**.

Follow the diagram below to connect the output to its corresponding lights we want: 

<img src="https://github.com/mlcourses/lab-3-blog-post-group3_cs281/blob/main/assets/design.png" alt="alt text" width="300"/> 

### Step 3: Lighting up LED C

Next, we will light up LED C. Once again, from the truth table, we have the following K-Map and the resulting SOP expression to activate LED C:

<img src="https://github.com/mlcourses/lab-3-blog-post-group3_cs281/blob/main/assets/LED%20C.png" alt="alt text" width="300"/> 

We will reuse the chips from the previous step:
- Connect pin 12 on the Arduino (`B1`) to gate `2A` of the 7404 inverter.
- Connect gate `2Y` to gate `2A` of the 7432 OR chip.
- On the same row of gate `1A` of the 7404 inverter, connect a small wire on that row to another row. We will use this new row as the normal, un-inverted input `B0`. Connect any point on this row to gate `2B` of the 7432 OR chip.
- Connect the output gate `2Y` on the 7432 OR chip to the corresponding light like the diagram. **Remember to connect them through a transistor**.



## Testing

## Conclusion




