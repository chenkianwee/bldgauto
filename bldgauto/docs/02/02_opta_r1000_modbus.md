# Arduino Opta and Seeed Studio reComputer r1000 Modbus TCP/IP

## Hardware and software
For this tutorial we will need the following hardware

- Arduino Opta Wifi (Can be bought <a href="https://store.arduino.cc/products/opta-wifi?queryID=undefined" target="_blank">here</a>)
- 24V power supply (Can be bought <a href="https://www.amazon.com/dp/B09281KTS8?ref=nb_sb_ss_w_as-reorder_k1_1_5&amp=&crid=2KKGJ6OCSOW31&amp=&sprefix=24+v+" target="_blank">here</a>)
- Seeed Studio reComputer R1000 series (Can be bought <a href="https://www.seeedstudio.com/reComputer-R1025-10-p-5895.html" target="_blank">here</a>)
- Power Adaptor for reComputer (Can be bought <a href="https://www.seeedstudio.com/Power-Adapter-12V-2A-EU-p-5732.html" target="_blank">here</a>)
- Ethernet cable (Can be bought <a href="https://www.adafruit.com/product/5441" target="_blank">here</a>)
- Monitor, keyboard and mouse for accessing the r1000 computer.
- Laptop to program Arduino Opta.

We will need the following software

- Arduino IDE (Download it <a href="https://www.arduino.cc/en/software/" target="_blank">here</a>)
- Install VScode on R1000. Follow instructions <a href="https://code.visualstudio.com/" target="_blank">here</a>.
- Install pymodbus Python library on R1000. Follow instructions <a href="https://github.com/pymodbus-dev/pymodbus/?tab=readme-ov-file#install" target="_blank">here</a>. 

## Instructions
1. Wire your Opta to the R1000 as shown.
```{image} ../../_static/opta_r1000_modbus/opta_r1000_modbus1.png
:width: 100%
:align: center
```
<br/><br/>

2. The connected devices shown below.
```{image} ../../_static/opta_r1000_modbus/opta_r1000_modbus2.jpg
:width: 100%
:align: center
```
<br/><br/>

3. Once wired we need to do some programming on the Arduino Opta. Open the Arduino IDE. Copy and paste the following script onto the IDE and upload it to Opta.
``` {dropdown} opta_r1000_modbus_ip
    /**
    Getting Started with Opta™ and Seeed Studio reComputer R1000
    Purpose: Communication between R1000 and Opta using Modbus TCP/IP, configure Opta as a Modbus Server.
    @author Kian Wee Chen
    */

    #include <SPI.h>
    #include <Ethernet.h>
    #include <ArduinoRS485.h>
    #include <ArduinoModbus.h>

    // Define the IP address for the Modbus TCP server
    IPAddress ip(10, 0, 0, 227);
    IPAddress dns(10, 0, 0, 228);
    IPAddress gateway(10, 0, 0, 229);
    IPAddress subnet(255, 255, 255, 0);

    // Server will listen on Modbus TCP standard port 502
    EthernetServer ethServer(502);

    // Create a Modbus TCP server instance
    ModbusTCPServer modbusTCPServer;

    int buttonState = 0;
    int onOff = 0;

    void setup() {
        // Initialize Relays outputs
        pinMode(D0, OUTPUT);

        // Initialize OPTA LEDs
        pinMode(LED_D0, OUTPUT);
        pinMode(BTN_USER, INPUT);

        // Initialize serial communication at 9600 bauds,
        // wait for the serial port to connect,
        // initialize Ethernet connection with the specified IP address
        Serial.begin(9600);
        while (!Serial);
        // Ethernet.begin(NULL, ip);
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
    }

    // The loop function runs over and over again while the device is on
    void loop() {
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
                updateRelayLED();
            }

            Serial.println("Client disconnected.");
        }

        // Switch on and off the relay and holding register value based on the user button
        buttonState = digitalRead(BTN_USER);
        if(buttonState == LOW){
            if(onOff == 0) {
                onOff = 1;
            }
            else {
                onOff = 0;
            }

            if (onOff == 0) {
                modbusTCPServer.holdingRegisterWrite(0, 0);
            }
            else {
                modbusTCPServer.holdingRegisterWrite(0, 1);
            }
        }
        updateRelayLED();
        delay(300);
    }

    void updateRelayLED() {
        // Read the current value of the coil at address 0
        int holdValue = modbusTCPServer.holdingRegisterRead(0);

        // Set the LED state; HIGH if hold value is 1, LOW if hold value is 0
        if (holdValue == 0) {
            digitalWrite(D0, LOW);
            digitalWrite(LED_D0, LOW);
        }

        else if (holdValue == 1) {
            digitalWrite(D0, HIGH);
            digitalWrite(LED_D0, HIGH);
        }
    }
```

4. In the Raspberry Pi OS environment configure the wired connection to read from the connected Opta. 
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

5. In the VSCode environment. Copy and paste the following script to read the Modbus holding register from the Opta. Run the script and you should be switching on and off the LED 1 and relay 1.
``` {dropdown} read_opta_modbus
    from enum import Enum
    from time import sleep
    # --------------------------------------------------------------------------- #
    # import the various client implementations
    # --------------------------------------------------------------------------- #
    from pymodbus.client import ModbusTcpClient
    from pymodbus.exceptions import ModbusException
    from pymodbus.pdu import ExceptionResponse

    HOST = "10.0.0.227" # change it to the modbus device of interest
    PORT = 502

    def main() -> None:
        """Run client setup."""
        print("### Client starting")
        client: ModbusTcpClient = ModbusTcpClient(
            host=HOST,
            port=PORT
        )
        client.connect()
        print("### Client connected")
        sleep(1)
        print("### Client starting")
        register_val = read_register(client, 0, 'H')
        print(f"### The value of the register is {register_val}")
        if register_val == 0:
            write_register(client, 0, 1)
        else:
            write_register(client, 0, 0)
        client.close()
        print("### End of Program")

    def write_register(client: ModbusTcpClient, addr: int, value: int) -> None:
        """Write a register."""
        try:
            # rr = client.write_registers(address=addr, values=[value], slave=1)
            rr = client.write_register(addr, value)
            # TODO: cannot figure out why can't I use write_register instead
            print(f"write is {rr}, Value = {value}")
        except ModbusException as exc:
            print(f"Modbus exception: {exc!s}")
        if rr.isError():
            print(f"Error")
        if isinstance(rr, ExceptionResponse):
            print(f"Response exception: {rr!s}")

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
        data_type = get_data_type(format)
        # print(data_type)
        count = data_type.value[1]
        var_type = data_type.name
        print(f"*** Reading register({var_type})")
        try:
            if reg_type == 'Holding Register':
                rr = client.read_holding_registers(address=addr, count=count, slave=1)
            elif reg_type == 'Input Register':
                rr = client.read_input_registers(address=addr, count=count, slave=1)
            # rr = client.read_coils(addr)
        except ModbusException as exc:
            print(f"Modbus exception: {exc!s}")
    
        if rr.isError():
            print(f"Error")

        if isinstance(rr, ExceptionResponse):
            print(f"Response exception: {rr!s}")

        # print(rr.registers)
        value = client.convert_from_registers(rr.registers, data_type)
        return value

    def get_data_type(format: str) -> Enum:
        """Return the ModbusTcpClient.DATATYPE according to the format"""
        for data_type in ModbusTcpClient.DATATYPE:
            if data_type.value[0] == format:
                return data_type

    if __name__ == "__main__":
        main()
```