# Data Logger (and using cool sensors!)

For this lab, we will be experimenting with a variety of sensors, sending the data to the Arduino serial monitor, writing data to the EEPROM of the Arduino, and then playing the data back.

## Overview

What's in this lab?

A. [Writing to the Serial Monitor](#part-a--writing-to-the-serial-monitor)

B. [RGB LED](#part-b-rgb-led)

C. [Resistance & Voltage Varying Sensors](#part-c-resistance--voltage-varying-sensors)

D. [I2C Sensors](https://github.com/FAR-Lab/Developing-and-Designing-Interactive-Devices/wiki/Lab-03#part-d-i2c-sensors)

E. [Logging Values to the EEPROM and Reading Them Back](#part-f-logging-values-to-the-eeprom-and-reading-them-back)

F. [Create your own Data logger!](#part-g-create-your-own-data-logger)

## Part A.  Writing to the Serial Monitor
 
![pot schematic](https://github.com/FAR-Lab/Developing-and-Designing-Interactive-Devices/wiki/images/Pot_schem.png)
 
**a. Based on the readings from the serial monitor, what is the range of the analog values being read?**

The serial monitor displays values from 8 to 1023, corresponding to readings of 0 to 5 volts. My readings don't quite reach 0 because my circuit also includes a small voltage divider resistor (10 ohms). 
 
**b. How many bits of resolution does the analog to digital converter (ADC) on the Arduino have** 

Since the readings max out at 1023, we can guess that the converter has 10 bits of resolution (2^10 is 1024). This is confirmed by simply looking up this question in the Arduino documentation. 
 
Wow!!! The serial plotter is an awesome new feature. I'll be using that to visualize sensor data for the lab. 

## Part B. RGB LED

**You should add the LEDs in the schematic below.**

![RGB LED schematic](https://github.com/FAR-Lab/Developing-and-Designing-Interactive-Devices/wiki/images/rgbled.png)

**How might you use this with only the parts in your kit? Show us your solution.**

Done! The provided sketch is working as intended. Since the kit doesn't include a 150 ohm resistor, I took two of the 330 ohm resistors and used them both in parallel to make a 165 ohm resistor to approximate the correct schematic. 

## Part C. Resistance & Voltage Varying Sensors 
 
### FSR

**a. What voltage values do you see from your force sensor?**

With a 10K resistor on the back half of the divider, I'm seeing values of 0 to 1000 from the sensor. 

**b. What kind of relationship does the voltage have as a function of the force applied? (e.g., linear?)**

The relationship seems to be distinctly non-linear. The sensor is very sensitive to small force inputs, so as a button or binary sensor it would be very effective. Higher force inputs give seriously diminishing returns - it takes pretty minimal force to get the reading from 0 to 750, then quite a lot of force to get from 750 to 1000. This is supported by the following figure from the datasheet:

[insert image]

**c. In `Examples->Basic->Fading` the RGB LED values range from 0-255. What do you have to do so that you get the full range of output voltages from the RGB LED when using your FSR to change the LED color?**

To make use of the full 0-255 range of the LED, we simply scale the value written to the pin proportional to where the sensor reading falls within its own possible range. For instance, if the sensor reads 500, this is halfway between minimum and maximum and we write 255 x (500/1000) (equal to 255 x 0.5) to the LED. 

## Flex Sensor, Photo cell, Softpot
Now experiment with the [flex sensor (Optional)](https://www.adafruit.com/product/1070), [photo cell](https://www.adafruit.com/product/161) and [softpot](https://www.adafruit.com/product/178).

<img src=https://cdn-shop.adafruit.com/1200x900/1070-01.jpg alt="flex sensor" width=250>
<img src=https://cdn-shop.adafruit.com/1200x900/161-00.jpg alt="photocell" width=250>
<img src=https://cdn-shop.adafruit.com/1200x900/178-00.jpg alt="softpot" width=250>

**a. What resistance do you need to have in series to get a reasonable range of voltages from each sensor?**

For the flex sensor, the resistance takes on a particular 'center' value when the sensor is straight, then it goes either up or down based on which direction the sensor is bent. With a 10K resistor on the back half of the divider, the center value is 263, and the bent values range from roughly 200 to 290. While this doesn't make use of the full range of the ADC converter, it is reasonable and usable. Using a 100 ohm resistor shrinks the input range to only swing of only about 3 on either side of 3, which is not usable. Bigger resistance (100K) maintains similar behavior to the 10K, but moves the range of inputs sensed up higher in the ADC's range to values on either side of 800. 

For the photocell, I followed the directions on the Adafruit page and used a 10K resistor in the divider. This gives a range of about 950 (flashlight) to 0 (dark). 

**b. What kind of relationship does the resistance have as a function of stimulus? (e.g., linear?)**

The flex sensor appears to have a roughly linear relationship to stimulus for small bend values, but then stops giving as much return for high bend values. The photocell has a roughly linear relationship to stimulus between dark and flashlight. 

**Control the colors of the LED using the above sensors ( including FSR )**

Done! See the picture below for the breadboard setup (shamefully overlapping, I know :D). Isolating each color separately takes some hand contortion, but I've successfully gotten it to turn each of red, green, and blue. The code for this section is sensorsToLED.ino. 

[insert pic]

## Part D. I2C Sensors 

### Accelerometer

**Set up the RGB LED so that each color is mapped to the X, Y and Z axes accelerations.**

Done! I've indexed red, green, and blue to X, Y, and Z position respectively. The relevant code is accelLED.ino.
 
**Get a feel for the data the accelerometer provides. Pick up the Arduino+accelerometer board and tilt it in various directions. Start by holding it so that the accelerometer board is parallel to the ground. Find in which direction the X reading increases and decreases; do the same for the Y reading.**

The accelerometer provides both position and acceleration on the X, Y, and Z axes. The X and Y position seem to have 14 bits of signed precision (between +/- 16384), while the Z position has 15 bits of signed precision (+/- 32768). The X and Y position take on sensible values based on what angle the sensor is tilted to, but since the Z position is just an integrated version of the Z acceleration data (and there isn't really a Z-tilt), the Z position number is pretty meaningless. 

The sensor primarily is actually measuring acceleration, and this is borne out in the fact that all three axes get meaningful acceleration readouts. At rest, the sensor reads what component of gravity (9.81 m/s^2) is felt by each axis, so when the sensor is flat the Z axis reads close to gravity while X and Y are close to zero. Tilting the sensor generates temporary acceleration which is registered by the various axes, but when held still at various tilts, the acceleration readings again just show which components of gravity are felt along which axis. This is the easiest way to determine which direction is which, and which direction is the positive direction along that axis. For instance, tilting the sensor on either side of the Y axis shows either positive or negative ~9.81, indicating whether the positive or negative side is pointing down. 
 
**a. Include your accelerometer read-out code in your write-up.**

As noted above, the relevant code is accelLED.ino.

## Part E. Logging values to the EEPROM and reading them back

**a. Does it matter what actions are assigned to which state? Why?**

It doesn't really matter which numbered state is which, just the method by which you choose each state needs to line up with the desired functionality. 

**b. Why is the code here all in the setup() functions and not in the loop() functions?**

We only want the state code to run once for each time the dial is turned to that setting. Putting it in the setup section means it will run once then wait for the next state instruction. 

**c. How many byte-sized data samples can you store on the Atmega328?**

Since the EEPROM is 1024 bytes in size, we can store 1024 byte samples. 

**d. How would you get analog data from the Arduino analog pins to be byte-sized? How about analog data from the I2C devices?**

We can either cast it to a datatype that is byte-sized directly, or translate the larger samples proportionally into an 8-bit format then read back out that encoding in reverse to use the values. 

**e. Alternately, how would we store the data if it were bigger than a byte? (hint: take a look at the [EEPROMPut](https://www.arduino.cc/en/Reference/EEPROMPut) example)**

We could split the sample into pieces and store it as back to back bytes. 

**Modify the code to take in analog values from your sensors and print them back out to the Arduino Serial Monitor.**

Done! The code is included in dataLogger.ino. 

### 2. Design your logger
 
**a. Turn in a copy of your final state diagram.**

I used the same state diagram as the original code, where the states move linearly from clear to write to read-out (then cycle back to clear). This is controlled with the potentiometer dial. 

## Part G. Create your own data logger!

I chose to make a system that will simply record a light pattern (from a flashlight or hand movements) from the photocell sensor and play it back on a simple LED. Thus the system is pretty much playing Simon Says against a flashlight. 
 
**a. Record and upload a short demo video of your logger in action.**

The video of the system is [here](https://youtu.be/F4sZAyLVwlQ).

