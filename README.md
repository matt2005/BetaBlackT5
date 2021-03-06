# BetaBlackT5
A repository for using the AI Thinker Beta Black T5 using the Arduino IDE

The intent is to create a library that allows access to the AI tThinker Beta Black T5 IOT board.
The board comes with an ESP8266 E13 and a second 8051 style microcontroller that is connected to the on-board peripherals.
The E13 and the 8051 communicate over serial links, very little of the ESP8266 IO pins are used and the 8051 controls the E13 via AT commands.

The board has an Android App available that is for controlling the board via AI Thinker cloud services. The App is almost exclusively in Chinese which puts this English only speaker at a distinct disadvantage. Luckily, a colleague can read Chinese. The documentation is [here](https://anxinke.gitbooks.io/balckboard/content/) and google translate does a pretty good job. The app, uController source and schematics are [here](http://t.cn/RLxuyfo).

## How to use the BBT5
Several people have used the BBT5 by cutting out the 8051 and wiring the peripherals directly to the E13. While possible, it didn't seem very practical to me. I decided to see if I could reverse engineer the protocol between the 2 microcontrollers.

The BBT5 has the serial links available at a header with, so the communication between the uC can be monitored. 
A couple of cheap USB to serial boards and wire jumpers to the header allowed viewing of the protocol in action.

Using a linux box and minicom with the RXD lines of the two USB adapters connected to the header.

The connection wiring and modes are detailed [here](SerialConnections.md)

The first observation is that the protocol is in English text at 115200 baud, 8N1 and no hardware handshaking.
The 8051 beeps until the ESP8266 successfully connects to the WiFi, or more specfically responds with a success message after the initialisation.

## The protocol
Since the ESP8266 is relatively dumb, the 8051 does all the AT commands.

### Wireless connect
I manually connected the ESP8266 using the manufacturers documentation for the AT commands. I did this before getting the Android App, I assume the App would find the device and connect it.

Use connection Mode A
```
AT+CWMODE=1
AT+CWJAP="SSID","Password"
```


Once the ESP8266 has connected over wireless and told the 8051, the incessant beeping is quietened :)
alternatively, sending the following commands to the 8051 makes it think it is connected and goes into normal mode (using connection Mode C )
```
WIFI CONNECTED
WIFI GOT IP
```
The 8051 responds with:
```
AT+CLDSTART                                                                   
AT+LANSTART 
``` 


### Temperature reading (use mode B or C)
The 8051 sends the current temperature in degrees x10 once per minute.
```
AT+CLDSEND=9
TEMP:300;
```
Doe not seem to give the humidity even though it has it built in.

### Controlling the LEDs (Use mode C)
Send the LED brightness commands to the 8051 (0=off 99=Full brightness)

   * Control the red LED brightness

```
LIGHTR:0
LIGHTR:99
```
   * Control the green LED brightness
```
LIGHTG:0
LIGHTG:99
```
   * Control the blue LED brightness
```
LIGHTG:0
LIGHTG:99
```

### Controlling the relay (use mode C)

   * Energise relay
```
LIGHT:1
```
The 8051 responds with
```
LIGHT:1;AT+CLDSEND=11                                                         
LIGHTR:99                                                                     
AT+CLDSEND=11                                                                 
LIGHTR:99                                                                     
AT+CLDSEND=11                                                                 
LIGHTG:99                                                                     
AT+CLDSEND=11                                                                 
LIGHTB:99          
```
   * De-energise the relay
```
LIGHT:0
```
The 8051 responds with:
```
LIGHT:0;AT+CLDSEND=10                                                         
LIGHTR:0                                                                      
AT+CLDSEND=10                                                                 
LIGHTR:0                                                                      
AT+CLDSEND=10                                                                 
LIGHTG:0                                                                      
AT+CLDSEND=10                                                                 
LIGHTB:0     
