# Arduino Opta and Belimo Thermostat Modbus RTU

## Hardware and software
For this tutorial we will need the following hardware

- Arduino Opta Wifi (Can be bought <a href="https://store.arduino.cc/products/opta-wifi?queryID=undefined" target="_blank">here</a>)
- Belimo Thermostat 22RTH 5900RD (Can be bought <a href="https://blackhawksupply.com/products/belimo-22rth-5900d?variant=39943328530528" target="_blank">here</a>)
- 24V power supply x 2 (Can be bought <a href="https://www.amazon.com/dp/B09281KTS8?ref=nb_sb_ss_w_as-reorder_k1_1_5&amp=&crid=2KKGJ6OCSOW31&amp=&sprefix=24+v+" target="_blank">here</a>)
- RS485 cable (Can be bought <a href="https://www.amazon.com/Custom-Cable-Connection-Conductor-Stranded/dp/B0DN8LBHVX/ref=sr_1_9?crid=2EF2MFSKE9GUN&dib=eyJ2IjoiMSJ9.ZAoXBHFAV3jQ649WESWLq7Fblh5kXmKAn1_op-ndrzo37jN83uGhdVz5CY4HlBjQX9DCS-Ay3BZ7cFtt-JEgYeZLjRaJMMP26yjvnQ8zoEum73c-2KSAm87qCudWMXyR7YtPswt5J-ssUUhkjlNT7TtXm6oFdM9Ort6cTMzfElhz7hfXuk8E5avDdm2bNu0bUcXjIR2dtJ0tv0KNl6ZxajMCDLxkTX1-6YbI9sAGEAY.wfyvcxwkvMGA_17ZK3BV3J_2LgEdFHk3WgHqKTdz2SA&dib_tag=se&keywords=rs485%2Bcable%2B3%2Bwire&qid=1746563660&sprefix=rs485%2Bcable%2B3%2Bwire%2Caps%2C71&sr=8-9&th=1" target="_blank">here</a>)
- USB-C Data Cable (Can be bought <a href="https://www.amazon.com/dp/B087JMJSVC?ref=nb_sb_ss_w_as-reorder_k0_1_16&amp=&crid=3V52YX7OEX7W9&sprefix=usb%2Bc%2Bdata%2Bcable&th=1" target="_blank">here</a>)
- Phone with NFC enabled.
- Laptop to program Arduino Opta.

We will need the following software

- Arduino IDE (Download it <a href="https://www.arduino.cc/en/software/" target="_blank">here</a>)

## Instructions
1. Wire your Opta to the thermostat as shown below:
```{image} ../../_static/opta_tstat/opta_tstat1.png
:width: 100%
:align: center
```
<br/><br/>

2. The connected devices shown below.
```{image} ../../_static/opta_tstat/opta_tstat2.jpg
:width: 100%
:align: center
```
<br/><br/>

3. Once connected we need to configure settings on the Belimo thermostat for the connection. Download the 'Belimo Assistant 2' app on your NFC enabled phone. Select the NFC option to connect to a device. Follow the instruction to connect to your thermostat. Once connected go to Setup -> Communication. Change the parameters to the following and 'Write to Device' through NFC:
```
Bus Protocol: Modbus RTU
Address: 1
Baudrate: 38400
Terminating Resistor: Off
Parity: 1-8-N-2
```
```{image} ../../_static/opta_tstat/opta_tstat3.png
:width: 30%
:align: center
```
<br/><br/>

4. Once configured we need to do some programming on the Arduino Opta. Open the Arduino IDE. Copy and paste the following script onto the IDE and upload it to Opta.
``` {dropdown} read_belimo_tstat
    /**
    Getting started with Opta and Belimo
    Purpose: Read and write registers from Belimo 22rth 5900ud
    @author Kian Wee Chen
    */

    #include <ArduinoModbus.h>
    #include <ArduinoRS485.h> // ArduinoModbus depends on the ArduinoRS485 library

    constexpr auto baudrate { 38400 };

    // Calculate preDelay and postDelay in microseconds as per Modbus RTU Specification
    // MODBUS over serial line specification and implementation guide V1.02
    // Paragraph 2.5.1.1 MODBUS Message RTU Framing
    // https://modbus.org/docs/Modbus_over_serial_line_V1_02.pdf
    constexpr auto bitduration { 1.f / baudrate };
    constexpr auto preDelayBR { bitduration * 9.6f * 3.5f * 1e6 };
    constexpr auto postDelayBR { bitduration * 9.6f * 3.5f * 1e6 };

    int server_id = 1;
    int setpt_addr = 21; //21 is the setpoint temp
    double setpt_scale = 0.01; // according to the belimo thermostat documentation
    int airtemp_addr=0; // 0 is the air temp
    double airtemp_scale = 0.01; // according to the belimo thermostat documentation

    void setup() {
        Serial.begin(9600);
        while (!Serial);

        Serial.println("Modbus RTU Client");

        RS485.setDelays(preDelayBR, postDelayBR);

        // Start the Modbus RTU client
        if (!ModbusRTUClient.begin(baudrate, SERIAL_8N2)) {
            Serial.println("Failed to start Modbus RTU Client!");
            while (1);
        }
    }

    void loop() {
        int setpt_raw = ModbusRTUClient.holdingRegisterRead(server_id, setpt_addr);
        if (setpt_raw != -1) {
        double setpt = setpt_raw * setpt_scale;
        Serial.println(setpt);
        if (setpt != 28.0) {
            // set it to 28
            int regwrite = ModbusRTUClient.holdingRegisterWrite(server_id, setpt_addr, 2800);
            if (regwrite == 1) {
                Serial.println("Success!");    
            } else {
                Serial.println("Failed!");
            }
        } else {
            // if it is equal to 28
            int regwrite = ModbusRTUClient.holdingRegisterWrite(server_id, setpt_addr, 1800);
            if (regwrite == 1) {
                Serial.println("Success!");    
            } else {
                Serial.println("Failed!");
            }
        }

        } else {
            Serial.print("failed! ");
            Serial.println(ModbusRTUClient.lastError());
        }

        int airtemp_raw = ModbusRTUClient.inputRegisterRead(server_id, airtemp_addr);
        if (airtemp_raw != -1) {
            double airtemp = airtemp_raw * airtemp_scale;
            Serial.println(airtemp);
        } else {
            Serial.print("failed! ");
            Serial.println(ModbusRTUClient.lastError());
        }

        delay(10000);
        Serial.println();
    }
```

5. In the Arduino IDE you will see the script reading the air temperature and set point temperature. It will also be changing the setpoint on the thermostat from 18 degC to 28 degC every 10 seconds.

## Resource
- <a href="https://docs.arduino.cc/tutorials/opta/getting-started-with-modbus-rtu/" target="_blank">Arduino tutorial on using Modbus RTU</a>
