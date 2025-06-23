# Arduino Opta, Belimo Thermostat Modbus RTU and FROST-Server

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
1. Install the FROST-Server on a computer. Follow the instruction <a href="https://chenkianwee.github.io/yun2infinity/docs/040/0413deploy.html" target="_blank">here</a>. Make sure your FROST-Server computer and your Arduino is connected to the same wifi if you are doing a local network version.

2. Wire your Opta to the thermostat as shown below:
```{image} ../../_static/opta_tstat_frost/opta_tstat_frost1.png
:width: 100%
:align: center
```
<br/><br/>

3. The connected devices shown below.
```{image} ../../_static/opta_tstat/opta_tstat2.jpg
:width: 100%
:align: center
```
<br/><br/>

4. Once connected we need to configure settings on the Belimo thermostat for the connection. Download the 'Belimo Assistant 2' app on your NFC enabled phone. Select the NFC option to connect to a device. Follow the instruction to connect to your thermostat. Once connected go to Setup -> Communication. Change the parameters to the following and 'Write to Device' through NFC:
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

5. Once configured we need to do some programming on the Arduino Opta. Open the Arduino IDE. Copy and paste the following script onto the IDE and upload it to Opta.
``` {dropdown} read_belimo_tstat_post
    #include <SPI.h>
    #include <ArduinoRS485.h> // ArduinoModbus depends on the ArduinoRS485 library
    #include <ArduinoModbus.h>
    #include <WiFi.h>
    #include "secrets.h" 
    #include <NTPClient.h>
    #include <mbed_mktime.h>
    #include "Base64.h"
    #include <ArduinoJson.h>

    //------------------------------------------------------------------------
    // Wifi Settings
    char ssid[] = SECRET_SSID;        // your network SSID (name)
    char pass[] = SECRET_PASS;        // your network password (use for WPA, or use as key for WEP)
    int keyIndex = 0;                 // your network key Index number (needed only for WEP)

    const char* authUser = "admin";
    const char* authPass = "admin";

    int status = WL_IDLE_STATUS;
    WiFiClient client;
    //------------------------------------------------------------------------
    RTC_DS3231 rtc;
    // NTP client configuration and RTC update interval.
    WiFiUDP   ntpUDP;
    // NTPClient timeClient(ntpUDP, "pool.ntp.org", 9*3600, 0);
    NTPClient timeClient(ntpUDP, "pool.ntp.org", 0, 0); //UTC time
    // Display time every 5 seconds.
    unsigned long interval = 5000UL;
    unsigned long lastTime = 0;
    //------------------------------------------------------------------------
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
    int onOff_htg = 0; // indicator if the relays are on/off, 0=Off, 1=On
    int onOff_clg = 0; // indicator if the relays are on/off, 0=Off, 1=On
    double deadband = 0.5; //degC

    void setup() {
        Serial.begin(9600);
        // while (!Serial);
        // Serial.println("Modbus RTU Client");
        //------------------------------------------------------------------------
        // Wifi settings
        // check for the WiFi module:
        if (WiFi.status() == WL_NO_SHIELD) {
            Serial.println("Communication with WiFi module failed!");
            // don't continue
            while (true);
        }

        // attempt to connect to Wifi network:
        while (status != WL_CONNECTED) {
            Serial.print("Attempting to connect to SSID: ");
            Serial.println(ssid);
            // Connect to WPA/WPA2 network. Change this line if using open or WEP network:
            status = WiFi.begin(ssid, pass);
            // wait 3 seconds for connection:
            delay(3000);
        }
        
        Serial.println("Connected to wifi");
        printWifiStatus();
        //------------------------------------------------------------------------
        // NTP client object initialization and time update, display updated time on the Serial Monitor.
        timeClient.begin();
        timeClient.update();
        const unsigned long epoch = timeClient.getEpochTime();
        set_time(epoch);

        // Show the synchronized time.
        Serial.println();
        Serial.println("- TIME INFORMATION:");
        Serial.print("- RTC time: ");
        Serial.println(getLocalTime());
        //------------------------------------------------------------------------
        // Initialize Relays outputs
        // pinMode(D0, OUTPUT);
        // pinMode(D1, OUTPUT);
        // Initialize OPTA LEDs
        pinMode(LED_D0, OUTPUT);
        pinMode(LED_D1, OUTPUT);

        RS485.setDelays(preDelayBR, postDelayBR);
        // Start the Modbus RTU client
        if (!ModbusRTUClient.begin(baudrate, SERIAL_8N2)) {
            Serial.println("Failed to start Modbus RTU Client!");
            while (1);
            }
    }

    void loop() {
        //get the current time
        int dsId = 1;
        // String timeNow = Time.format(Time.now(), TIME_FORMAT_ISO8601_FULL);
        String timeNow = getLocalTime();

        int setpt_raw = ModbusRTUClient.holdingRegisterRead(server_id, setpt_addr);
        int airtemp_raw = ModbusRTUClient.inputRegisterRead(server_id, airtemp_addr);
        if (setpt_raw != -1 && setpt_raw != -1) {
            double setpt = setpt_raw * setpt_scale;
            double airtemp = airtemp_raw * airtemp_scale;
            double temp_diff = setpt - airtemp; // if positive heat, if negative cool
            Serial.println(setpt);
            Serial.println(airtemp);
            //-------------------------------------------------------------------------------------------------------------------------
            // post the data to FROST server
            JsonDocument doc1;
            doc1["@iot.id"] = dsId;
            JsonDocument doc2;
            doc2["phenomenonTime"] = timeNow;
            doc2["resultTime"] = timeNow;
            doc2["result"] = airtemp;
            doc2["Datastream"] = doc1;
            String jsonData;
            serializeJson(doc2, jsonData);
            Serial.println(jsonData);
            post2Frost(jsonData);
            //-------------------------------------------------------------------------------------------------------------------------
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
        
        //-------------------------------------------------------------
        delay(10000);
        // Serial.println();
    }

    void post2Frost(String jsonData) {
        //-------------------------------------------------------------
        // Wifi Settings
        // if you don't want to use DNS (and reduce your sketch size) use the numeric IP instead of the name for the server:
        //// PUT IN YOUR OWN IP ADDRESS OR WEBPAGE THAT IS HOSTING YOUR FROST SERVER.
        IPAddress server(127,0,0,1);  // IP address for example.com (no DNS)
        // char server[] = "databca.globalenvtech.com";       // host name for example.com (using DNS)
        //// SPECIFY YOUR OWN USERNAME AND PASSWORD 
        char authString[] = "USERNAME:PASSWORD"; //username + password
        int inputStringLength = strlen(authString);
        int encodedLength = Base64.encodedLength(inputStringLength);
        int encodedLength2 = encodedLength + 1;
        char encodedString[encodedLength2];
        Base64.encode(encodedString, authString, inputStringLength);

        // Construct HTTP POST request
        if (client.connect(server, 80)) {
            Serial.println("connected to server");
            // Make a HTTP request:
            client.println("POST /frost/v1.1/Observations HTTP/1.1");
            client.print("Host: ");
            client.println(server);
            client.println("Content-Type: application/json");
            client.println("Authorization: Basic " + String(encodedString));
            client.print("Content-Length: ");
            client.println(jsonData.length());
            client.println(); // End of headers
            client.println(jsonData); // Body
        }
        // Waiting for the response is very slow and will take up to a minute. Just keep posting as we do not need to know.
        // while (client.available()) {
        //     char c = client.read();
        //     Serial.write(c);
        // }
        // // Wait for response
        // while (client.connected() || client.available()) {
        //     if (client.available()) {
        //     String line = client.readStringUntil('\n');
        //     Serial.println(line);
        //     }
        // }
        // client.stop();
    }

    void turnOnClgRelay() {
    if (onOff_clg == 0) {
            // turn on the relay
            // digitalWrite(D1, HIGH);
            digitalWrite(LED_D1, HIGH);
            onOff_clg = 1;
        }
    }

    void turnOffClgRelay() {
    if (onOff_clg == 1) {
            // turn off the relay
            // digitalWrite(D1, LOW);
            digitalWrite(LED_D1, LOW);
            onOff_clg = 0;
        }
    }

    void turnOnHtgRelay() {
    if (onOff_htg == 0) {
            // turn on the relay
            // digitalWrite(D0, HIGH);
            digitalWrite(LED_D0, HIGH);
            onOff_htg = 1;
        }
    }

    void turnOffHtgRelay() {
    if (onOff_htg == 1) {
            // turn on the relay
            // digitalWrite(D0, LOW);
            digitalWrite(LED_D0, LOW);
            onOff_htg = 0;
        }
    }

    void printWifiStatus() {
    // print the SSID of the network you're attached to:
    Serial.print("SSID: ");
    Serial.println(WiFi.SSID());

    // print your board's IP address:
    IPAddress ip = WiFi.localIP();
    Serial.print("IP Address: ");
    Serial.println(ip);

    // print the received signal strength:
    long rssi = WiFi.RSSI();
    Serial.print("signal strength (RSSI):");
    Serial.print(rssi);
    Serial.println(" dBm");
    }

    String getLocalTime() {
    char buffer[32];
    tm t;
    _rtc_localtime(time(NULL), &t, RTC_FULL_LEAP_YEAR_SUPPORT);
    strftime(buffer, 32, "%Y-%m-%dT%H:%M:%S+00:00", &t);
    return String(buffer);
    }
```