# Arduino-remote-control-and-OTA-via-ESP8266

![image](https://user-images.githubusercontent.com/19898602/135787990-ef0a1bd8-16ab-4f6d-a190-b4b63ce5c198.png)


Overview#
VCON is a framework for remote (OTA) firmware updates for a wide range of microcontrollers: STM32Fx, STM32Lx, SAMD21, SAMD51, Mega328. VCON framework consists of two parts:

A hardware OTA shield, wired to your microcontroller (Host MCU),


A device management cloud (public at https://dash.vcon.io or private).


![image](https://user-images.githubusercontent.com/19898602/135788029-252a2959-2f9b-4a00-aff8-2c22d66aaf3b.png)


VCON OTA shield is, in turn:

Any device or module based on an ESP32 microcontroller,
With a special pre-built firmware loaded on ESP32.
VCON shield is a "smart" connectivity module of a new type. Unlike traditional modules, it knows how to reprogram a Host MCU (a microcontroller that is wired to a VCON module). So essentially, a VCON module is a remote programmer with REST API that can re-program your device at any time.

Optionally, in addition to the OTA wiring, VCON module can be wired to the Host MCU's UART. In this case, VCON shield could be configured to act as a network bridge - either in transparent, or non-transparent mode:

In trasparent mode, everything that Host MCU prints to the UART, VCON forwards to the server of your choice, MQTT or Websocket. Everything that VCON receives from the server, VCON writes to the MCU's UART. This enables bi-directional communication, where Host MCU "thinks" it talks to the UART where in fact it talks to the remote server of your choice - e.g. AWS IoT, or Azure, or your private MQTT server.


In non-transparent mode, Host MCU can exchange JSON-RPC commands with the VCON shield and access a wide range of functionality - call external REST services, manage files, etc. Also, it can export RPC services itself, and VCON shield bridges those services to the REST API. This way, it is easy to create custom management interface to communicate with your MCU via familiar, secure REST API.


# Arduino Nano

![image](https://user-images.githubusercontent.com/19898602/135788071-75a2ad2e-e674-4f84-a88c-03f402445c9c.png)

![image](https://user-images.githubusercontent.com/19898602/135788546-556f670f-bae6-4c58-a52f-27252500243e.png)

Before moving fuurther I would like to tell you something about PCB

Yes PCB are the heart of the electronics based project usually we hesitate to try custom PCB and opt to homemade solutions

like breadboard or Zero PCB earlier I also was in the same boat, I hesitate to try custom PCB my belief was they are much expensive.

but then I came to know about [JLCPCB.COM](https://jlcpcb.com/IAT) and I was totally surprised how low price PCB's are they offering 

there PCB quality is best in market, now I always go with PCB for my project and [JLCPCB.COM](https://jlcpcb.com/IAT) is my trusted 

PCB manufacturer, you can also try there PCB service for more details you can visit their website [JLCPCB.COM](https://jlcpcb.com/IAT)
You can also try there new purple colour for PCB without any extra cost.
![image](https://user-images.githubusercontent.com/19898602/134336832-cb9953e9-02a6-4ff7-9d27-2caad10fe7c7.png)
![image](https://user-images.githubusercontent.com/19898602/130722577-c30b7b43-ea89-4847-9c6b-058f9fabeda3.png)![image](https://user-images.githubusercontent.com/19898602/130722585-b5268db1-5f17-428f-ba60-b823140f2a70.png)




# Shield Setup

Once wiring is complete, power the Arduino Nano board by plugging in USB cable. The VCON shield should start blinking blue LED, which indicates the network is unconfigured. Jump to the Network Setup section to configure network on your VCON shield, and return here when done. The blue LED should be lit solid, and a respective device on a dashboard should become online.

# Host MCU setup

Now we have to tell VCON shield, which microcontroller is attached, which pins are used, etcetera. Switch to the https://dash.vcon.io, select "action" → "Manage device":


![image](https://user-images.githubusercontent.com/19898602/135788092-9349f279-f3e9-431c-a0f3-a7ef95dfafef.png)


This loads device management console with several different sections. Now we're interested in "Host MCU" section. Change values to make it look like this and hit "save":

![image](https://user-images.githubusercontent.com/19898602/135788103-0fb90392-a9cc-48fe-a435-b8e307066527.png)


After a second or two, a device should momentarily go offline and then online again: that means it has updated configuration and rebooted. Now we are ready to update firmware over the air!

# OTA firmware update

Fire Arduino IDE, and create a new sketch. This sketch is a modification of the classic Blink sketch with some changes:

> LED status is printed on the serial console on each "blink"
> By entering numbers from "0" to "9" on a serial console, we could change blink interval
This sketch, therefore, implements some trivial remote control over UART. Choose one of the following:

> Skip firmware build step and download a pre-compiled firmware for this sketch mega328p.hex

> Build the firmware yourself. Copy/paste the code into a new Arduino sketch:

```javascript

int led = LED_BUILTIN, sleeptime = 1200, on = 0;

void setup() {
 Serial.begin(115200);
 pinMode(led, OUTPUT);
}

void loop() {
 on = !on;               // Invert LED status
 digitalWrite(led, on);  // Set LED
 delay(sleeptime);       // Sleep for `sleeptime` milliseconds
 Serial.println(on);     // Print LED status to serial

 // Read serial input. If we read a number from "0" to "9",
 // then change blink interval 
 if (Serial.available() > 0) {
   int ch = Serial.read();
   if (ch >= '0' && ch <= '9') sleeptime = (ch - '0' + 1) * 300;
 }
}

```

Next, let us compile (but not upload!) this sketch into a downloadable file. In Arduino IDE, perform the following steps:

Select "Tools" → "Board" → "Arduino AVR boards" → Arduino Uno
Select "Sketch" → "Export Compiled Binary"
Select "Sketch" → "Show Sketch Folder"
A folder with a compiled .hex file should popup. It should be named SKETCH_NAME.ino.standard.hex, which is a text file in Intel hex format that contains binary code of your compiled firmware.

Switch to the VCON dashboard, device management console. In the "Host MCU" section, click on the "firmware update" button, choose firmware .hex file. A button is going to show an OTA progress indicator, and finish in a second or two:

The dashboard grabs your input and calls a serial.write function on the VCON shield, which sends the input to the specified rx UART pin. Also, serial monitor prints your input on a console in an alternate color, making it easy to see input and output. Host MCU receives character "7", adjusts blinking interval to 800 milliseconds, and Arduino starts blinking slower.

Congratulations! Now your Arduino is remotely controllable.

The cool things is that Arduino is not even "aware" it is connected to the Internet. It "thinks" it communicates via UART - which is very easy to develop, debug and test. This way, a Host MCU can be nicely isolated to perform only required business tasks and not care about networking and management.

See Recipes section for some further insights on how can you enhance remote control and data reporting.


![Arduino remote control and OTA via ESP8266 and Vcon io](https://user-images.githubusercontent.com/19898602/135788660-5f32e4d1-7b24-418d-a9e8-3231ec0bbb01.gif)


