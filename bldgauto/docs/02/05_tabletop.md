# Tabletop Prototype

## Hardware and software
For this tutorial we will need the following hardware

- Arduino Opta Wifi (Can be bought <a href="https://store.arduino.cc/products/opta-wifi?queryID=undefined" target="_blank">here</a>)
- 24V power supply (Can be bought <a href="https://www.amazon.com/dp/B09281KTS8?ref=nb_sb_ss_w_as-reorder_k1_1_5&amp=&crid=2KKGJ6OCSOW31&amp=&sprefix=24+v+" target="_blank">here</a>)
- Seeed Studio reComputer R1000 series (Can be bought <a href="https://www.seeedstudio.com/reComputer-R1025-10-p-5895.html" target="_blank">here</a>)
- Power Adaptor for reComputer (Can be bought <a href="https://www.seeedstudio.com/Power-Adapter-12V-2A-EU-p-5732.html" target="_blank">here</a>)
- Ethernet cable (Can be bought <a href="https://www.adafruit.com/product/5441" target="_blank">here</a>)
- Monitor, keyboard and mouse for accessing the r1000 computer.
- Laptop to program Arduino Opta.
- Belimo Thermostat 22RTH 5900RD (Can be bought <a href="https://blackhawksupply.com/products/belimo-22rth-5900d?variant=39943328530528" target="_blank">here</a>)
- Phone with NFC enabled.
- RS485 cable (Can be bought <a href="https://www.amazon.com/Custom-Cable-Connection-Conductor-Stranded/dp/B0DN8LBHVX/ref=sr_1_9?crid=2EF2MFSKE9GUN&dib=eyJ2IjoiMSJ9.ZAoXBHFAV3jQ649WESWLq7Fblh5kXmKAn1_op-ndrzo37jN83uGhdVz5CY4HlBjQX9DCS-Ay3BZ7cFtt-JEgYeZLjRaJMMP26yjvnQ8zoEum73c-2KSAm87qCudWMXyR7YtPswt5J-ssUUhkjlNT7TtXm6oFdM9Ort6cTMzfElhz7hfXuk8E5avDdm2bNu0bUcXjIR2dtJ0tv0KNl6ZxajMCDLxkTX1-6YbI9sAGEAY.wfyvcxwkvMGA_17ZK3BV3J_2LgEdFHk3WgHqKTdz2SA&dib_tag=se&keywords=rs485%2Bcable%2B3%2Bwire&qid=1746563660&sprefix=rs485%2Bcable%2B3%2Bwire%2Caps%2C71&sr=8-9&th=1" target="_blank">here</a>)
- USB-C Data Cable (Can be bought <a href="https://www.amazon.com/dp/B087JMJSVC?ref=nb_sb_ss_w_as-reorder_k0_1_16&amp=&crid=3V52YX7OEX7W9&sprefix=usb%2Bc%2Bdata%2Bcable&th=1" target="_blank">here</a>)
- 100 watts incandescent light bulb (Can be bought <a href="https://www.amazon.com/Medium-Frosted-Rough-Service-Light/dp/B072HCF51T/ref=sr_1_6?crid=JWWFS9QIGMAE&dib=eyJ2IjoiMSJ9.DCVVqqZFBw9GHZAe4mP1Vyqt67QJG7muqyUmw0JMFEisHiZgONYP9r5KoZgu_RX9RSCotv1X16RHWDHw8eEbn5hY6iSNn520Es4hNKP1EjpK3cBs4BEvm58vn8c6VfGsToa51mkfallntn3ICFIVQApG2Mn_4mtSWT_-YhCF2sUETkcQIWcyo0my3l3Zw6bkyupQZ94GScfZp2P7hTbz5-mb3lsGgP19FMGVOobn68dvrXFqnT_DmVnglpNDDA9KAGvK2ZrZD18RIs-_tFIb594LPKmPGHxbnQaBksJwZdQ.7H20C11vvhJf8bQ3FkGP6s6hZkbZa1FETN8Jsc3mCq4&dib_tag=se&keywords=100+watt+incandescent+light+bulbs&qid=1748018291&sprefix=100+watt+incandes+light+bulbs,aps,90&sr=8-6" target="_blank">here</a>)
- Light buld stand that can be wired into the relay of Arduino Opta (Can be bought <a href="https://www.amazon.com/dp/B0D1G3FNN9?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1" target="_blank">here</a>)
- Crimps for the light bulb stand (Can be bought <a href="https://www.amazon.com/SOMELINE-Crimping-Terminals-Quadrilateral-Connectors/dp/B0BZPS2T8Q/ref=sr_1_10?crid=KIEK4HXYXMLT&dib=eyJ2IjoiMSJ9.OWt6gCTBsA8-1E4RE3QLaPZESEAecgvT17XncS_n7XmgxMU30VjDHYJ8ijAnRTCZm7u7tvXwcyZVQ9dvM414oHdLwuGA-GF_M0eVMCOZ1nxBNi_tGzGqFIgrU7KupNSNQXLRJ2295JJ-kXJ4KaaY-XxReFRzbPXhnH-LfZHW8STzvGx1iwFz4WEgK7haG2EeRZ6be3K_xSGWEDWFDX73oSMp4dHjCMQrpMHtA53nplfpD2x9Ad_LzJRTqmgjcZ9wuEFelzkRDqwgykhS0kYYKiwqHfRtPeaDflpc-wgc_Ec.qnJaqGkujOGzfYLlEZ5og0fP4fHA3KxiLYDhHKeJWaE&dib_tag=se&keywords=crimp&qid=1756916691&sprefix=crimp%2Caps%2C123&sr=8-10&th=1" target="_blank">here</a>)
- Wire stripper (Can be bought <a href="https://www.amazon.com/haisstronica-Stripper-Automatic-Crimping-Universal/dp/B0B2NWK1QX/ref=sr_1_5?crid=1B8J5AVY1EWQE&dib=eyJ2IjoiMSJ9.dT18DHHPZqHkCRv5AXKjD4O93fZ6qnw1Qoaw-AGuu387ius88VW7fjEjY0OdhOuvEJd9b-GdvYlZ10b12nwE2vYcnll61t4qOPlFLHpB0tGMTWyydpv0tlr3-gz-evLU6CcRASjnxJCRW6kjjnH6BI3howB5H454TV5bXG8zxsM0Zpg-aWNUvbMVytL1x28FNPabwhc837vZ4pC0x83KdgPilpMxCE0FRfZbAatlhx8ybvv5AgxeKL6Y2TFnqjLp3TP1QrvO99Kyt0W14JZkqaRFtXF629qDMipxqFs8hu0.FlVU67eutO-N_LXI-ilRYIWA8Xya_n2WBw0P4fkDT80&dib_tag=se&keywords=wire%2Bstripper&qid=1756917243&s=industrial&sprefix=wire%2Bstr%2Cindustrial%2C71&sr=1-5&th=1" target="_blank">here</a>)
- Wire solid core hookup wires (Can be bought <a href="https://www.amazon.com/TUOFENG-Hookup-Wires-6-Different-Colored/dp/B07TX6BX47/ref=sr_1_6?crid=30319TX3N67HS&dib=eyJ2IjoiMSJ9.4q0mLzQlp-ARBkkUfmY0UAqaja_KFY4N4DNpxS2f-X_e4Lpu_qkjcLzFB9DeodN0aI2HPEwwVost7W8Z7Qpeoz8rb5ObHlcYTsvB5a43v8RbO7f-sz_vEOqKhnw6bqJGyLwYWlwFAedXdL2x7saxPDt8tid5idnk_fFQ4vH1xLPA8_y8fZMNFuGmyRepa7qJl5k_D7oI7G_J3L_SXpy5Fq8YWe-571Kp1FBuy5UiUSRbx1iDiLDPzSzuBZheHde2a_mgBeuW5ZklNGbH1y91ibvcInICirnjj94Ttcd8h9o.3sfRWHRvngA9XzSPKlbsX_7Wjf2-LoMTAtk3zrCx6gY&dib_tag=se&keywords=wire%2Broll&qid=1756923200&s=industrial&sprefix=wire%2Broll%2Cindustrial%2C117&sr=1-6&th=1
" target="_blank">here</a>)

We will need the following software
- Arduino IDE (Download it <a href="https://www.arduino.cc/en/software/" target="_blank">here</a>)
- Install VScode on R1000. Follow instructions <a href="https://code.visualstudio.com/" target="_blank">here</a>.
- Install pymodbus Python library on R1000. Follow instructions <a href="https://github.com/pymodbus-dev/pymodbus/?tab=readme-ov-file#install" target="_blank">here</a>. 
- Install BACpypes3 Python library on R1000 and your BACnet client laptop. Follow instructions <a href="https://github.com/pymodbus-dev/pymodbus/?tab=readme-ov-file#install" target="_blank">here</a>.

## Instructions
1. Wire and connect your equipment as shown.
```{image} ../../_static/tabletop/tabletop1.png
:width: 100%
:align: center
```
<br/><br/>

2. The connected devices shown below.
```{image} ../../_static/tabletop/tabletop2.jpg
:width: 100%
:align: center
```
<br/><br/>

## Configure Thermostat and Opta
1. Once connected we need to configure settings on the Belimo thermostat for the connection. Download the 'Belimo Assistant 2' app on your NFC enabled phone. Select the NFC option to connect to a device. Follow the instruction to connect to your thermostat. Once connected go to Setup -> Communication. Change the parameters to the following and 'Write to Device' through NFC:
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

2. Once configured we need to do some programming on the Arduino Opta, connect the Opta to your laptop installed with Arduino IDE using the USB-C. Open the Arduino IDE. Copy and paste the following script onto the IDE and upload it to Opta.
``` {dropdown} ctrl_belimo_tstat
    /**
        Getting started with Opta and Belimo
        Purpose: Read and write registers from Belimo 22rth 5900ud
        @author Kian Wee Chen
    */
    #include <SPI.h>
    #include <Ethernet.h>
    #include <ArduinoRS485.h> // ArduinoModbus depends on the ArduinoRS485 library
    #include <ArduinoModbus.h>

    constexpr auto baudrate { 38400 };
    // Calculate preDelay and postDelay in microseconds as per Modbus RTU Specification
    // MODBUS over serial line specification and implementation guide V1.02
    // Paragraph 2.5.1.1 MODBUS Message RTU Framing
    // https://modbus.org/docs/Modbus_over_serial_line_V1_02.pdf
    constexpr auto bitduration { 1.f / baudrate };
    constexpr auto preDelayBR { bitduration * 9.6f * 3.5f * 1e6 };
    constexpr auto postDelayBR { bitduration * 9.6f * 3.5f * 1e6 };

    //-------------------------------------------------------
    //Modbus tcp/ip
    //-------------------------------------------------------
    // Define the IP address for the Modbus TCP server
    IPAddress ip(10, 0, 0, 227);
    IPAddress dns(10, 0, 0, 228);
    IPAddress gateway(10, 0, 0, 229);
    IPAddress subnet(255, 255, 255, 0);
    // Server will listen on Modbus TCP standard port 502
    EthernetServer ethServer(502);
    // Create a Modbus TCP server instance
    ModbusTCPServer modbusTCPServer;
    //-------------------------------------------------------

    int server_id = 1;
    int setpt_addr = 21; //21 is the setpoint temp
    double setpt_scale = 0.01; // according to the belimo thermostat documentation
    int airtemp_addr=0; // 0 is the air temp
    double airtemp_scale = 0.01; // according to the belimo thermostat documentation
    int onOff_htg = 0; // indicator if the relays are on/off, 0=Off, 1=On
    int onOff_clg = 0; // indicator if the relays are on/off, 0=Off, 1=On
    double deadband = 0.5; //degC

    unsigned long previousMillis = 0;  
    const long interval = 10000; // 10 seconds in milliseconds

    void setup() {
        // Initialize Relays outputs
        pinMode(D0, OUTPUT);
        pinMode(D1, OUTPUT);
        // Initialize OPTA LEDs
        pinMode(LED_D0, OUTPUT);
        pinMode(LED_D1, OUTPUT);

        Serial.begin(9600);
        // while (!Serial);
        Serial.println("Modbus RTU Client");

        RS485.setDelays(preDelayBR, postDelayBR);
        // Start the Modbus RTU client
        if (!ModbusRTUClient.begin(baudrate, SERIAL_8N2)) {
            Serial.println("Failed to start Modbus RTU Client!");
            while (1);
            }
        //-------------------------------------------------------
        //Modbus tcp/ip
        //-------------------------------------------------------
        Ethernet.begin(NULL, ip, dns, gateway, subnet);
        // Check Ethernet hardware and cable connections
        if (Ethernet.hardwareStatus() == EthernetNoHardware) {
            Serial.println("- Ethernet interface not found!");
            while (true);
        }
        if (Ethernet.linkStatus() == LinkOFF) {
            Serial.println("- Ethernet cable not connected!");
        }
        // Start the Modbus TCP server
        ethServer.begin();
        if (!modbusTCPServer.begin()) {
            Serial.println("- Failed to start Modbus TCP Server!");
            while (1);
        }
        // Configure a holding register at address 0 for Modbus communication
        int success1 = modbusTCPServer.configureHoldingRegisters(0, 1);
        Serial.println(success1);
        int success2 = modbusTCPServer.configureInputRegisters(0,3);
        Serial.println(success2);
        //-------------------------------------------------------
    }

    void loop() { 
        unsigned long currentMillis = millis();
        if (currentMillis - previousMillis >= interval) {
            previousMillis = currentMillis;  // reset timer
            ctrlTstat();                     // call your function
        }
        //-------------------------------------------------------
        //Modbus tcp/ip
        //-------------------------------------------------------
        // Handle incoming client connections and process Modbus requests
        EthernetClient client = ethServer.available();
        if (client) {
            Serial.println("- Client connected!");
            // Accept and handle the client connection for Modbus communication
            modbusTCPServer.accept(client);
            // Update the LED state based on Modbus holding register value
            while (client.connected()) {
                // Process Modbus requests
                modbusTCPServer.poll(); 
                updateSetptReg();
            }
            Serial.println("Client disconnected.");
        }
    }

    void updateSetptReg() {
        // Read the current value at address 0, this is the setpt from the thermostat exposed by the arduino opta modbus interface.
        int setptVal = modbusTCPServer.holdingRegisterRead(0);
        // write the new value to the thermostat
        writeSetptTstatReg(setptVal);
    }

    void writeSetptTstatReg(int setptVal) {
        int regwrite = ModbusRTUClient.holdingRegisterWrite(server_id, setpt_addr, setptVal);
        if (regwrite == 0) {
            Serial.println("Writing to Tstat Holding Register Failed");    
        }
    }

    void ctrlTstat() {
        Serial.println("Function executed!");
        int setpt_raw = ModbusRTUClient.holdingRegisterRead(server_id, setpt_addr);
        int airtemp_raw = ModbusRTUClient.inputRegisterRead(server_id, airtemp_addr);
        Serial.println(setpt_raw);
        Serial.println(airtemp_raw);
        if (setpt_raw != -1 && airtemp_raw != -1) {
            writeSetptOptaReg(setpt_raw);
            writeAirTempOptaReg(airtemp_raw);
            double setpt = setpt_raw * setpt_scale;
            double airtemp = airtemp_raw * airtemp_scale;
            float heat_on_threshold = setpt - deadband / 2.0;
            float cool_on_threshold = setpt + deadband / 2.0;
            if (onOff_htg == 0 && onOff_clg == 0) {
                if (airtemp <= heat_on_threshold) {
                    turnOffClgRelay();  // cooler OFF
                    turnOnHtgRelay();   // heater ON
                } else if (airtemp >= cool_on_threshold) {
                    turnOffHtgRelay();  // heater OFF
                    turnOnClgRelay();   // cooler ON
                }
                else {
                    turnOffClgRelay();
                    turnOffHtgRelay();
                }
            } else if (onOff_htg == 1 && onOff_clg == 0) {
                if (airtemp < setpt) {
                    turnOffClgRelay();  // cooler OFF
                    turnOnHtgRelay();   // heater ON
                }
                else {
                    turnOffClgRelay();
                    turnOffHtgRelay();
                }
            } else if (onOff_htg == 0 && onOff_clg == 1) {
                if (airtemp > setpt) {
                    turnOnClgRelay();  // cooler OFF
                    turnOffHtgRelay();   // heater ON
                }
                else {
                    turnOffClgRelay();
                    turnOffHtgRelay();
                }
            }

            writeHtRelayOptaReg(onOff_htg);
            writeClgRelayOptaReg(onOff_clg);

        } else {
            Serial.print("failed! ");
            Serial.println(ModbusRTUClient.lastError());
        }
    }

    void writeSetptOptaReg(int setptVal) {
        int regwrite = modbusTCPServer.holdingRegisterWrite(0, setptVal);
        if (regwrite == 0) {
            Serial.println("setpt: Writing to Opta Holding Register Failed!");    
        }
    }

    void writeAirTempOptaReg(int airTempVal) {
        int regwrite = modbusTCPServer.inputRegisterWrite(0, airTempVal);
        if (regwrite == 0) {
            Serial.println("airTemp: Writing to Opta Holding Register Failed!");    
        }
    }

    void writeHtRelayOptaReg(int htRelay) {
        int regwrite = modbusTCPServer.inputRegisterWrite(1, htRelay);
        if (regwrite == 0) {
            Serial.println("htRelay: Writing to Opta Holding Register Failed!");    
        }
    }

    void writeClgRelayOptaReg(int clgRelay) {
        int regwrite = modbusTCPServer.inputRegisterWrite(2, clgRelay);
        if (regwrite == 0) {
            Serial.println("clgRelay: Writing to Opta Holding Register Failed!");    
        }
    }

    void turnOnClgRelay() {
    if (onOff_clg == 0) {
            // turn on the relay
            digitalWrite(D1, HIGH);
            digitalWrite(LED_D1, HIGH);
            onOff_clg = 1;
        }
    }

    void turnOffClgRelay() {
    if (onOff_clg == 1) {
            // turn off the relay
            digitalWrite(D1, LOW);
            digitalWrite(LED_D1, LOW);
            onOff_clg = 0;
        }
    }

    void turnOnHtgRelay() {
    if (onOff_htg == 0) {
            // turn on the relay
            digitalWrite(D0, HIGH);
            digitalWrite(LED_D0, HIGH);
            onOff_htg = 1;
        }
    }

    void turnOffHtgRelay() {
    if (onOff_htg == 1) {
            // turn on the relay
            digitalWrite(D0, LOW);
            digitalWrite(LED_D0, LOW);
            onOff_htg = 0;
        }
    }
```

## Configure Seeed Studio R1000 
1. In the Raspberry Pi OS environment configure the wired connection to read from the connected Opta. 
- Go to the network settings -> Advanced Options -> Edit Connections ...
- In the network connections window select Wired Connection 1 -> setting symbol
- In the Editing Wired Connection 1 window go to the IPv4 Settings tab. Change the method to 'Manual'. Click on the Add button and add in the following parameters and save the setting:
```
Address: 10.0.0.230
Netmask: 255.255.255.0
Gateway: 10.0.0.229
```
```{image} ../../_static/opta_r1000_modbus/opta_r1000_modbus3.png
:width: 100%
:align: center
```
<br/><br/>

2. Copy and past this code into a file called bacnet_server4opta.py on the R1000.
``` {dropdown} bacnet_server4opta.py
    """
    R1000 BACnet Device Example
    ==========================

    This script initializes a minimal BACnet server device using BACpypes3.
    It is ideal for rapid prototyping and testing with BACnet client tools
    or supervisory platforms. You can easily add or remove objects to fit your use case.

    Included Objects:
    -----------------
    - 1 Commandable Analog Value (AV)
    - 1 Commandable Binary Value (BV)

    Commandable Points:
    -------------------
    Commandable AV and BV points support writes via the BACnet priority array.
    They emulate real-world control points, such as thermostat setpoints, damper commands, etc.

    Usage:
    ------
    Run the script with the device name, instance ID, and optional debug flag:

        python mini-device-revisited.py --name BensServerTest --instance 3456789 --debug

    Arguments:
    ----------
    - --name       : The BACnet device name (e.g., "BensServerTest")
    - --instance   : The BACnet device instance ID (e.g., 3456789)
    - --address    : Optional — override the automatically detected IP address and port.
                    Requires ifaddr package for auto-detection.
                    See: https://bacpypes3.readthedocs.io/en/latest/gettingstarted/addresses.html#bacpypes3-addresses
    - --debug      : Enables verbose debug logging (built-in to BACpypes3)

    """
    import asyncio
    import sys
    from enum import Enum
    from time import sleep
    from datetime import date

    from bacpypes3.argparse import SimpleArgumentParser
    from bacpypes3.app import Application
    from bacpypes3.local.analog import AnalogValueObject
    from bacpypes3.local.binary import BinaryValueObject
    from bacpypes3.local.cmd import Commandable
    from bacpypes3.debugging import bacpypes_debugging, ModuleLogger

    from pymodbus.client import ModbusTcpClient
    from pymodbus.exceptions import ModbusException

    #------------------------------------------------------------------------------------------------------------
    # FUNCTIONS
    #------------------------------------------------------------------------------------------------------------
    def write_register(client: ModbusTcpClient, addr: int, value: int) -> None:
        """Write a register."""
        try:
            rr = client.write_register(addr, value)
            print(f"write is {rr}, Value = {value}")
        except ModbusException as exc:
            print(f"Modbus exception: {exc!s}")

    def read_register(client: ModbusTcpClient, addr: int, format: str, reg_type: str = 'Holding Register') -> int:
        """
        read a register.
    
        Parameters
        ----------
        client : ModbusTcpClient
            modbus client object
        
        addr: int
            address of the register

        format: str
            h=int16, H=uint16, i=int32 ,I=uint32, q=int64, Q=uint64, f=float32, d=float64, s=string, bits
        
        reg_type: str, optional
            default is Holding Register. Options are Input Register
        
        Returns
        -------
        vertices : np.ndarray
            np.ndarray(shape(number of edges, number of points in an edge, 3)) vertices of the line
        """
        value = None
        data_type = get_data_type(format)
        # print(data_type)
        count = data_type.value[1]
        var_type = data_type.name
        # print(f"*** Reading register({var_type})")
        try:
            if reg_type == 'Holding Register':
                rr = client.read_holding_registers(address=addr, count=1, slave=1)
            elif reg_type == 'Input Register':
                rr = client.read_input_registers(address=addr, count=count, slave=1)
            value = client.convert_from_registers(rr.registers, data_type)
        except ModbusException as exc:
            print(f"Modbus exception: {exc!s}")
        return value

    def get_data_type(format: str) -> Enum:
        """Return the ModbusTcpClient.DATATYPE according to the format"""
        for data_type in ModbusTcpClient.DATATYPE:
            if data_type.value[0] == format:
                return data_type
    #------------------------------------------------------------------------------------------------------------
    #------------------------------------------------------------------------------------------------------------
    # Modbus TCP/IP parameters
    HOST = "10.0.0.227" # change it to the modbus device of interest
    PORT = 502
    setpt_addr = 0 #holding registers
    airtemp_addr = 0 #input registers
    htrelay_addr = 1 #input registers
    clgrelay_addr = 2 #input registers
    setpt_scalg = 0.01
    airtemp_scalg = 0.01

    # Debug logging setup
    _debug = 0
    _log = ModuleLogger(globals())

    # Interval for updating values
    INTERVAL = 5.0

    @bacpypes_debugging
    class CommandableAnalogValueObject(Commandable, AnalogValueObject):
        """Commandable Analog Value Object"""

    @bacpypes_debugging
    class CommandableBinaryValueObject(Commandable, BinaryValueObject):
        """Commandable Binary Value Object"""

    @bacpypes_debugging
    class SampleApplication:
        def __init__(self, args):
            if _debug:
                _log.debug("Initializing SampleApplication")

            self.app = Application.from_args(args)

            self.setpt = CommandableAnalogValueObject(
                objectIdentifier=("analogValue", 1),
                objectName="tstat-setpt",
                presentValue=0.0,
                statusFlags=[0, 0, 0, 0],
                covIncrement=1.0,
                units="degreesCelsius",
                description="Setpoint Temperature of the Belimo Tstat",
            )

            self.airtemp = AnalogValueObject(
                objectIdentifier=("analogValue", 2),
                objectName="read-only-tstat-airtemp",
                presentValue=0.0,
                statusFlags=[0, 0, 0, 0],
                covIncrement=1.0,
                units="degreesCelsius",
                description="Air Temperature of the Belimo Tstat",
            )

            self.htrelay = AnalogValueObject(
                objectIdentifier=("analogValue", 3),
                objectName="read-only-tstat-ht",
                presentValue=0.0,
                statusFlags=[0, 0, 0, 0],
                covIncrement=1.0,
                description="htg device On/Off status",
            )

            self.clgrelay = AnalogValueObject(
                objectIdentifier=("analogValue", 4),
                objectName="read-only-tstat-clg",
                presentValue=0.0,
                statusFlags=[0, 0, 0, 0],
                covIncrement=1.0,
                description="clg device On/Off status",
            )

            for obj in [
                self.setpt,
                self.airtemp,
                self.htrelay,
                self.clgrelay
            ]:
                self.app.add_object(obj)

            _log.info("BACnet Objects initialized.")
            asyncio.create_task(self.update_values())

        async def update_values(self):
            prev_setpt = 0
            while True:
                await asyncio.sleep(INTERVAL)
                # get the present setpt value of bacnet
                setpt_val = self.setpt.presentValue
                airtemp_val = self.airtemp.presentValue
                htg_present_val = self.htrelay.presentValue
                clg_present_val = self.clgrelay.presentValue
                # get the tstat val
                client: ModbusTcpClient = ModbusTcpClient(
                    host=HOST,
                    port=PORT
                )
                client.connect()
                sleep(1)
                modbus_setpt_val = read_register(client, 0, 'H')
                modbus_airtemp_val = read_register(client, 0, 'H', reg_type='Input Register')
                mb_ht_relay_val = read_register(client, 1, 'H', reg_type='Input Register')
                mb_clg_relay_val = read_register(client, 2, 'H', reg_type='Input Register')
                print(modbus_setpt_val)
                print(modbus_airtemp_val)
                print(mb_ht_relay_val)
                print(mb_clg_relay_val)
                if modbus_setpt_val != None:
                    modbus_setpt_val_flt = modbus_setpt_val*setpt_scalg
                    if setpt_val == 0.0:
                        # it means this is the 1st loop
                        self.setpt.presentValue = modbus_setpt_val_flt
                        prev_setpt = modbus_setpt_val_flt
                    else:
                        # this is not the 1st loop prev_setpt cannt be 0
                        if setpt_val != prev_setpt: # new write to the bacnet setpt
                            if modbus_setpt_val_flt != prev_setpt: # new write from the thermostat
                                # thermostat always take priority, overwrite bacnet
                                self.setpt.presentValue = modbus_setpt_val_flt
                                prev_setpt = modbus_setpt_val_flt
                            elif modbus_setpt_val_flt == prev_setpt: # no new write from thermostat
                                write_register(client, setpt_addr, int(setpt_val/setpt_scalg))
                                prev_setpt = setpt_val
                        else: # no write from bacnet
                            if modbus_setpt_val_flt != prev_setpt: # new write from the thermostat
                                # expose the thermostat setpt
                                self.setpt.presentValue = modbus_setpt_val_flt
                                prev_setpt = modbus_setpt_val_flt
                
                if modbus_airtemp_val != None:
                    modbus_airtemp_val_flt = modbus_airtemp_val*airtemp_scalg
                    if modbus_airtemp_val_flt != airtemp_val:
                        # expose the thermostat air temp
                        self.airtemp.presentValue = modbus_airtemp_val_flt

                if mb_ht_relay_val != None:
                    if mb_ht_relay_val != htg_present_val: # new update from arduino
                        self.htrelay.presentValue = mb_ht_relay_val

                if mb_clg_relay_val != None:
                    if mb_clg_relay_val != clg_present_val: # new update from arduino
                        self.clgrelay.presentValue = mb_clg_relay_val

                client.close()

                if _debug:
                    _log.debug(f"Setpoint: {self.setpt.presentValue}")
                    _log.debug(f"Air Temp: {self.airtemp.presentValue}")
                    _log.debug(f"Htg Relay: {self.htrelay.presentValue}")
                    _log.debug(f"Clg Relay: {self.clgrelay.presentValue}")

    async def main():
        global _debug

        parser = SimpleArgumentParser()
        args = parser.parse_args()

        if args.debug:
            _debug = 1
            _log.set_level("DEBUG")
            _log.debug("Debug mode enabled")

        if _debug:
            _log.debug(f"Parsed arguments: {args}")

        app = SampleApplication(args)
        await asyncio.Future()


    if __name__ == "__main__":
        try:
            asyncio.run(main())
        except KeyboardInterrupt:
            _log.info("Keyboard interrupt received, shutting down.")
            sys.exit(0)

```

3. Assuming you are connected to a Wifi network. You can essentially create a wifi hotspot from your BACnet client laptop. On the R1000 execute the bacnet_server.py code with the following command. Replacing the --address with your ip4 address. You can now access your setup from another bacnet client laptop.
```
python bacnet_server4opta.py --address xx.xx.xx.xx/24 --name bacnet_device --instance 1 --debug
```

## Accessing R1000 BACnet Server from you BACnet client laptop
1. Make sure your BACnet client laptop is connected to the same Wifi network as the R1000. You can discover the R1000 BACnet server using [the 'read_device_obj_props_simple.py' script from step 1](04_bacnet_server.md#accessing-the-bacnet-server-from-a-bacnet-client).

2. You should be able to get the following result.
```
device,1 @ 10.42.0.159, from vendor-999
device,1 description error: property: unknown-property

    device,1:
        object-name: rm_device
    network-port,1:
        object-name: NetworkPort-1
    analog-value,1:
        object-name: tstat-setpt
        description: Setpoint Temperature of the Belimo Tstat
        present-value: 24.0
        units: degrees-celsius
```

3. You can use these [scripts](04_bacnet_server.md#other-useful-scripts-read-and-write-multiple) to read and write from the BACnet server.