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
- Light buld stand that can be wired into the relay of Arduino Opta

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

    void setup() {
        // Initialize Relays outputs
        pinMode(D0, OUTPUT);
        pinMode(D1, OUTPUT);
        // Initialize OPTA LEDs
        pinMode(LED_D0, OUTPUT);
        pinMode(LED_D1, OUTPUT);

        Serial.begin(9600);
        // while (!Serial);
        // Serial.println("Modbus RTU Client");

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
        int success = modbusTCPServer.configureHoldingRegisters(0, 1);
        Serial.println(success);
        //-------------------------------------------------------
    }

    void loop() {
        int setpt_raw = ModbusRTUClient.holdingRegisterRead(server_id, setpt_addr);
        int airtemp_raw = ModbusRTUClient.inputRegisterRead(server_id, airtemp_addr);
        if (setpt_raw != -1 && setpt_raw != -1) {
            writeSetptOptaReg(setpt_raw);
            double setpt = setpt_raw * setpt_scale;
            double airtemp = airtemp_raw * airtemp_scale;
            double temp_diff = setpt - airtemp; // if positive heat, if negative cool
            // Serial.println(setpt);
            // Serial.println(airtemp);
            if (airtemp < setpt) {
                //need to heat
                if (temp_diff > deadband) {
                    turnOffClgRelay();
                    turnOnHtgRelay();
                } else {
                    turnOffClgRelay();
                    turnOffHtgRelay();
                }
            
            } else if (airtemp > setpt) {
                //need to cool
                if (temp_diff*-1 > deadband) {
                    turnOffHtgRelay();
                    turnOnClgRelay();
                } else {
                    turnOffClgRelay();
                    turnOffHtgRelay();
                }
            } else {
                // air temp is equal to setpoint no heating or cooling is required
                // switch off all the relay
                turnOffClgRelay();
                turnOffHtgRelay();
            }

        } else {
            Serial.print("failed! ");
            Serial.println(ModbusRTUClient.lastError());
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
        //-------------------------------------------------------
        delay(10000);
        // Serial.println();
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

    void writeSetptOptaReg(int setptVal) {
        int regwrite = modbusTCPServer.holdingRegisterWrite(0, setptVal);
        if (regwrite == 0) {
            Serial.println("Writing to Opta Holding Register Failed!");    
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
                rr = client.read_holding_registers(address=addr, count=count, slave=1)
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
    setpt_addr = 0
    setpt_scalg = 0.01

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

            self.commandable_av = CommandableAnalogValueObject(
                objectIdentifier=("analogValue", 1),
                objectName="tstat-setpt",
                presentValue=0.0,
                statusFlags=[0, 0, 0, 0],
                covIncrement=1.0,
                units="degreesCelsius",
                description="Setpoint Temperature of the Belimo Tstat",
            )

            for obj in [
                self.commandable_av
            ]:
                self.app.add_object(obj)

            _log.info("BACnet Objects initialized.")
            asyncio.create_task(self.update_values())

        async def update_values(self):
            prev_setpt = 0
            while True:
                await asyncio.sleep(INTERVAL)
                # get the present setpt value of bacnet
                setpt_val = self.commandable_av.presentValue
                # get the tstat val
                client: ModbusTcpClient = ModbusTcpClient(
                    host=HOST,
                    port=PORT
                )
                client.connect()
                sleep(1)
                modbus_setpt_val = read_register(client, setpt_addr, 'H')
                if modbus_setpt_val != None:
                    modbus_setpt_val_flt = modbus_setpt_val*setpt_scalg
                    if setpt_val == 0.0:
                        # it means this is the 1st loop
                        self.commandable_av.presentValue = modbus_setpt_val_flt
                        prev_setpt = modbus_setpt_val_flt
                    else:
                        # this is not the 1st loop prev_setpt cannt be 0
                        if setpt_val != prev_setpt: # new write to the bacnet setpt
                            if modbus_setpt_val_flt != prev_setpt: # new write from the thermostat
                                # thermostat always take priority, overwrite bacnet
                                self.commandable_av.presentValue = modbus_setpt_val_flt
                                prev_setpt = modbus_setpt_val_flt
                            elif modbus_setpt_val_flt == prev_setpt: # no new write from thermostat
                                write_register(client, setpt_addr, int(setpt_val/setpt_scalg))
                                prev_setpt = setpt_val
                        else: # no write from bacnet
                            if modbus_setpt_val_flt != prev_setpt: # new write from the thermostat
                                # expose the thermostat setpt
                                self.commandable_av.presentValue = modbus_setpt_val_flt
                                prev_setpt = modbus_setpt_val_flt

                client.close()

                if _debug:
                    _log.debug(f"Commandable AV: {self.commandable_av.presentValue}")

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