---
title: Playing with arduinos, in Python & Java
date: 2014-02-06 18:29:33.000000000 +05:30
---
Arduinos are great for prototyping stuff, but for each firmware change you've to re-flash the Arduino. With RAAP, pyArduino and JArduino, you can quickly and easily automate a arduino connected with computer using programming language of your choice, currently I have written supporting packages for python and java.

## RAAP (Remote Arduino Automation Protocol)
For remote automation, we need a protocol to communicate to and from Arduino without need to re-flash, so I have drafted a communication protocol for this.

<a href="https://github.com/linuxexp/RAAP">Sources and firmware available on github</a>

RAAP can be used for remote communication, automation, inter-arduino communication to perform I/O among other things, with arduino board using a communication link. In this firmware release, "RAAP" communicates through Serial Communication. RAAP is a fast, reliable, binary protocol with low overhead.</p>

### RAAP Communication
<p>RAAP client and server exchanges binary control messages. The structure of a control message is as follows</p>

<pre>
|- 1 uint8 Operation opcode -|- 1 uint8 size of payload, after this header -|- arbitrary long payload -|
</pre>

<p>Operation opcode denotes what the RAAP server must perform, payload includes the required paramters for the requested operation.</p>
<p>Upon correct reception, and structural correctness of the control message, the "RAAP" server replies with "OK" control message otherwise "FAIL" control message. 

It is to be noted that "OK" control message doesn't denotes that the requested operation completed successfully. For example requesting pinMode operation will return "OK", but not necessarily denotes that pinMode is successful.</p>

#### Operation opcodes

<pre>
OK               0x00 (server only)
FAIL             0x01 (server only)
PIN_MODE         0x00 
DIGITAL_WRITE    0x01
DIGITAL_READ     0x02
ANALOG_REFERENCE 0x03
ANALOG_READ      0x04
ANALOG_WRITE     0x05
TONE             0x06
TONE_DURATION    0x07
NO_TONE          0x08
SHIFT_OUT        0x09
SHIFT_IN         0x10
PULSE_IN         0x11
</pre>

#### PORT opcodes

<pre>
PORT_OUTPUT	 0x00
PORT_INPUT       0x01
</pre>

These are used with pinMode control message

#### DIGITAL I/O opcodes

<pre>
PORT_HIGH        0x00
PORT_LOW         0x01
</pre>

These are used with Digital I/O control messages

#### ANALOG I/O opcodes

<pre>
AREF_DEFAULT      0x00
AREF_INTERNAL     0x01
AREF_INTERNAL1V1  0x02
AREF_INTERNAL2V56 0x03
AREF_EXTERNAL     0x04
</pre>

These are used with analogReference control message

#### Examples
OK Control Message<br />
0x00 0x0</p>
<p><strong>pinMode Control Message</strong></p>
<pre>0x00 0x01 &lt;pin_number&gt; &lt;0x00, 0x01&gt;</pre>
<p><strong>Initialized STATUS control message</strong><br />
There is a special OK control message which is only sent once, when the RAAP server initialises, the client must verify this control message before it sends any other control message to server. This OK control message also includes the version of the RAAP implemented by server.</p>
<p>e.g OK initialised message<br />
0x00 0x0 0x1</p>
<p>The third byte is the version information.</p>
<p><strong>Supported Arduino Functions</strong><br />
[x] Digital IO<br />
pinMode()<br />
digitalWrite()<br />
digitalRead()</p>
<p>[x] Analog I/O<br />
analogReference()<br />
analogRead()<br />
analogWrite() - PWM</p>
<p>[ ] Due only<br />
analogReadResolution()<br />
analogWriteResolution()</p>
<p>[x] Advanced I/O<br />
tone()<br />
noTone()<br />
shiftOut()<br />
shiftIn()<br />
pulseIn()</p>
<p>Other Arduino functions from</p>
<p>Time { millis(), micros(), delay(), delayMicroseconds() }<br />
Math { min(), max(), abs(), constrain(), map(), pow(), sqrt() }<br />
Trigonometry { sin(), cos(), tan() }<br />
Random Numbers { randomSeed(), random() }<br />
Bits and Bytes { lowByte(), highByte(), bitRead(), bitWrite(), bitSet(), bitClear(), bit() }</p>
<p>are either available in the wrapping language, or could be easily implemented using I/O functions, besides implementing these in "RAAP" would could cost device storage space. Currently there is no support for Interrupts handling (attachInterrupt(), detachInterrupt(), interrupts(), noInterrupts()), USB (Leonardo and Due only) Keyboard &amp; Mouse.</p>
