# Wifi for Opta 

- Post and Get request with Opta Arduino IDE
```{dropdown} Wifi.h
    #include <WiFi.h>
    #include "arduino_secrets.h" 
    #include "Base64.h"

    char ssid[] = SECRET_SSID;        // your network SSID (name)
    char pass[] = SECRET_PASS;        // your network password (use for WPA, or use as key for WEP)
    int keyIndex = 0;                 // your network key Index number (needed only for WEP)

    const char* authUser = "admin";
    const char* authPass = "admin";

    int status = WL_IDLE_STATUS;
    // if you don't want to use DNS (and reduce your sketch size)
    // use the numeric IP instead of the name for the server:
    IPAddress server(127.0.0.1);  // IP address for example.com (no DNS)
    // char server[] = "something.com";       // host name for example.com (using DNS)

    WiFiClient client;

    void setup() {
    //Initialize serial and wait for port to open:
    Serial.begin(9600);
    while (!Serial) {
        ; // wait for serial port to connect. Needed for native USB port only
    }

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

    char authString[] = "admin:admin"; //username + password
    int inputStringLength = strlen(authString);
    int encodedLength = Base64.encodedLength(inputStringLength);
    int encodedLength2 = encodedLength + 1;
    char encodedString[encodedLength2];
    Base64.encode(encodedString, authString, inputStringLength);
    // Serial.println(String(encodedString));

    Serial.println("\nStarting connection to server...");
    // if you get a connection, report back via serial:
    // Create JSON string
    String jsonData = "{\"name\":\"temperature\",\"description\":\"temperature\",\"definition\":\"\"}";
    String path = "/frost/v1.1/ObservedProperties";
    
    // Construct HTTP POST request
    if (client.connect(server, 80)) {
        Serial.println("connected to server");
        // Make a HTTP request:
        client.println("POST /frost/v1.1/ObservedProperties HTTP/1.1");
        client.print("Host: ");
        client.println(server);
        client.println("Content-Type: application/json");
        client.println("Authorization: Basic " + String(encodedString));
        client.print("Content-Length: ");
        client.println(jsonData.length());
        client.println(); // End of headers
        client.println(jsonData); // Body
    }

    // if (client.connect(server, 80)) {
    //   Serial.println("connected to server");
    //   // Make a HTTP request:
    //   client.println("GET /frost/");
    //   client.print("Host: ");
    //   client.println(server);
    //   client.println("Connection: close");
    //   client.println();
    // }
    }

    void loop() {
    // if there are incoming bytes available
    // from the server, read them and print them:
    while (client.available()) {
        char c = client.read();
        Serial.write(c);
    }

    // Wait for response
    while (client.connected() || client.available()) {
        if (client.available()) {
        String line = client.readStringUntil('\n');
        Serial.println(line);
        }
    }
    client.stop();

    // if the server's disconnected, stop the client:
    if (!client.connected()) {
        Serial.println();
        Serial.println("disconnecting from server.");
        client.stop();

        // do nothing forevermore:
        while (true);
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
```