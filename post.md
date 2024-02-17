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

<img src="https://github.com/mlcourses/lab-3-blog-post-group3_cs281/blob/main/assets/7-segment.png" alt="alt text" width="200"/> 

For reference, following is the LED functionality table for testing:

<img src="https://github.com/mlcourses/lab-3-blog-post-group3_cs281/blob/main/assets/table.png" alt="alt text" width="550"/> 

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

Once again, from the truth table, we have the following K-Map and the resulting SOP expression to activate LED C:

<img src="https://github.com/mlcourses/lab-3-blog-post-group3_cs281/blob/main/assets/LED%20C.png" alt="alt text" width="300"/> 

We will reuse the chips from the previous step:
- Connect pin 12 on the Arduino (`B1`) to gate `2A` of the 7404 inverter.
- Connect gate `2Y` to gate `2A` of the 7432 OR chip.
- On the same row of gate `1A` of the 7404 inverter, connect a small wire on that row to another row. We will use this new row as the normal, un-inverted input `B0`. Connect any point on this row to gate `2B` of the 7432 OR chip.
- Connect the output gate `2Y` on the 7432 OR chip to the corresponding light like the diagram. **Remember to connect them through a transistor**.

### Step 4: Lighting up LED E

Once again, from the truth table, we have the following K-Map and the resulting SOP expression to activate LED E:

<img src="https://github.com/mlcourses/lab-3-blog-post-group3_cs281/blob/main/assets/LED%20E.png" alt="alt text" width="300"/> 

We will use a new chip for this light. We will be using a 7408 chip for the AND gate:
- Wire and ground the chip.
- Reuse the outputs from step 1. Connect gates `1Y` and `3Y` of the 7404 inverter to gates `1A` and `1B` respectively on the 7408 AND chip.
- Connect the output gate `1Y` on the 7408 AND chip to the corresponding light like the diagram. **Remember to connect them through a transistor**.

### Step 5: Lighting up LED F

Once again, from the truth table, we have the following K-Map and the resulting SOP expression to activate LED F:

<img src="https://github.com/mlcourses/lab-3-blog-post-group3_cs281/blob/main/assets/LED%20F.png" alt="alt text" width="300"/> 

We will reuse the chips and outputs from the previous steps:
- Connect gates `1Y` and `2Y` to gates `2A` and `2B` of the 7408 AND chip.
- Connect output gate `2Y` of the 7408 AND chip to gate `3A` of the 7432 OR chip.
- Similarly like in step 3, on the same row of gate `3A` of the 7404 inverter, connect a small wire on that row to another row. We will use this new row as the normal, un-inverted input `B2`. Connect any point on this row to gate `3B` of the 7432 OR chip.
- Connect the output gate `3Y` on the 7432 OR chip to the corresponding light like the diagram. **Remember to connect them through a transistor**.

### Step 6: Lighting up LED G

Once again, from the truth table, we have the following K-Map and the resulting SOP expression to activate LED G:

<img src="https://github.com/mlcourses/lab-3-blog-post-group3_cs281/blob/main/assets/LED%20G.png" alt="alt text" width="300"/> 

We will reuse the chips and outputs from the previous steps:
- Similarly like in steps 3 and 5, on the same row of gate `2A` of the 7404 inverter, connect a small wire on that row to another row. We will use this new row as the normal, un-inverted input `B1`. Connect any point on this row to gate `4A` of the 7432 OR chip.
- Connect any point on the row of the un-inverted `B2` input to gate `4B` of the 7432 OR chip.
- Connect the output gate `4Y` on the 7432 OR chip to the corresponding light like the diagram. **Remember to connect them through a transistor**.

### Step 7: Lighting up LEDs A and D

Finally, we will light up LEDs A and D. Once again, from the truth table, we have the following K-Maps and the resulting SOP expressions to activate LEDs A and D:

<img src="https://github.com/mlcourses/lab-3-blog-post-group3_cs281/blob/main/assets/LED%20A.png" alt="alt text" width="300"/> 

<img src="https://github.com/mlcourses/lab-3-blog-post-group3_cs281/blob/main/assets/LED%20D.png" alt="alt text" width="300"/> 

Notice that LEDs A and D share the same SOP expression, so we only need to build one circuit for this, and reuse the output twice for both LEDs.

For this step, we will need to use the 7486 XOR chip, a new 7432 OR chip, and reuse some chips and outputs from the previous steps:
- Wire and ground the chips.
- Connect any point on the rows of the un-inverted `B2` and `B0` inputs to gates `1B` and `1A`, respectively, of the 7486 XOR chip.
- Connect output gate `1Y` of the 7486 XOR chip to gate `4A` of the 7404 inverter.
- Connect output gate `4Y` of the 7404 inverter to gate `1A` of the second 7432 OR gate.
- Connect any point on the row of the un-inverted `B1` to gate `1B` of the second 7432 OR gate.
- Connect the output gate `1Y` on the 7432 chip to the corresponding light like the diagram. Remember to reuse this output twice for both LEDs A and D, and also remember to connect both of them through a transistor.

## Testing

- **For the first step:** After following the instructions described in step 1, we found that the analog input correctly converts the analog input into digital input using arduino. When using the analog input and sliding it to the right, we saw that the digital input switching accordingly in the computer that used the arduino code, as you can see in the video below.

    [Analog to Digital Test](https://drive.google.com/file/d/1BLrT0ejpcy9QNPotIVF0K96MkQ4J8IKp/view?usp=sharing)

- **For the second step:** After following the instructions described in step 1 to perform the B segment, we found out that the light of the B segment is correctly on when the analog input is between 0-4, and correctly off when the analog input is 5. This means that the steps we took were right and the K-Map we used to reduce the SOP expression helped us to get the right expression in order to perform the digital circuit for B. 

    [LED B](https://drive.google.com/file/d/1N9dXuz-IEJQ0ioLYqwcgS-USWrzfAkdV/view?usp=sharing)

- **For the third step:** After following the instructions described in step 1 to perform the C segment, we found out that the light of the C segment is correctly on when the analog input is 0,1,3,4 and 5, and correctly off when the analog input is 2. This means that the steps we took were right and the K-Map we used to reduce the SOP expression helped us to get the right expression in order to perform the digital circuit for C.

    [LED C](https://drive.google.com/file/d/15_XhrI5EPNUJ7y-NmflAwXvYtTtIIT_5/view?usp=sharing)

- **For the fourth step:** After following the instructions described in step 1 to perform the E segment, we found out that the light of the E segment is correctly on when the analog input is 0 and 2 and correctly off when the analog input is 1,3,4 and 5. This means that the steps we took were right and the K-Map we used to reduce the SOP expression helped us to get the right expression in order to perform the digital circuit for E.

    [LED E](https://drive.google.com/file/d/1GcQgfObOXNqkvy0AnheIQKijr65ZvTGY/view?usp=sharing)

- **For the fifth step:** After following the instructions described in step 1 to perform the F segment, we found out that the light of the F segment is correctly on when the analog input is 0,4 and 5 and correctly off when the analog input is 1,2, and 3. This means that the steps we took were right and the K-Map we used to reduce the SOP expression helped us to get the right expression in order to perform the digital circuit for F.

    [LED F](https://drive.google.com/file/d/14Jek7hG8vOtHKoT1yxMLghT8idrbiItP/view?usp=sharing)

- **For the sixth step:** After following the instructions described in step 1 to perform the G segment, we found out that the light of the G segment is correctly on when the analog input is 2,3,4 and 5 and correctly off when the analog input is 0 and 1. This means that the steps we took were right and the K-Map we used to reduce the SOP expression helped us to get the right expression in order to perform the digital circuit for G.

    [LED G](https://drive.google.com/file/d/1gq1jbODvl0pPVR5PBpXOyzxt-j5bNS3e/view?usp=sharing)

- **For the seventh step:** After following the instructions described in step 1 to perform the A and D segment, we found out that the light of the A and D segment is correctly on when the analog input is 0,2,3,5 and correctly off when the analog input is 1 and 4. This means that the steps we took were right and the the K-Map we used to reduce the SOP expression helped us to get the right expression in order to perform the digital circuits for A and D.

    [LEDs A and D](https://drive.google.com/file/d/12xSz-r9K1N-8Ukt-0Fw_llIYHg9soq4I/view?usp=sharing)

- **For the final step:** After testing the whole 7-segment number using the analog input we found that all the numbers are correctly performed as they should according to the current input of the analog input. You can view the final testing in the video below.

    [Final Test](https://drive.google.com/file/d/1KI66cSP4xl619u0_rlAEA7bCeUy9tb0C/view?usp=sharing)

## Conclusion




