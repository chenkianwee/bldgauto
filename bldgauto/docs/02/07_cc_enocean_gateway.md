# Contemporary Controls Enocean Gateway

## Hardware and software
For this tutorial we will need the following hardware

- Contemporary Controls Enocean BACnet gateway (Can be bought <a href="https://www.ccontrols.com/basautomation/enocean.php" target="_blank">here</a>)
- Enocean Occupancy Sensor (If you are in US can be bought here <a href="https://enocean.cdistore.com/products/detail/emdcu-enocean/685908/?pid=" target="_blank">here</a>)
- 24V power supply x 1 (Can be bought <a href="https://www.amazon.com/dp/B09281KTS8?ref=nb_sb_ss_w_as-reorder_k1_1_5&amp=&crid=2KKGJ6OCSOW31&amp=&sprefix=24+v+" target="_blank">here</a>)
- Phone with NFC enabled.
- Laptop.

## Instructions
1. Setup the Enocean gateway by plugging in the power supply and ethernet cable. Connect the ethernet cable to your computer. For Linux (Ubuntu) machine go to Settings -> Network ->  Wired -> Network Options (cog button) -> IPv4. In the IPv4 method choose the 'Manual' option. You can leave the gateway blank or use '192.168.92.68'.
    ```
    Address: 192.168.92.5
    Netmask: 255.255.255.0
    Gateway: 
    ```
2. Once connected, go to 192.168.92.68 and configure the gateway. Use the default password if you are logging in for the first time.

3. Go to 'Add Devices' tab and click on 'Start Discovery' button and follow the instruction. It will discover the sensor and show you its unique id. The EEP profile of the occupancy sensor can be found on the installation guide of the sensor. I used the default 'A5-07-03'. Click register to register the device. 

4. Go to the 'Upload/Download' tab -> 'Backup/Restore Configuration' and click on the 'Update' button for the device to work.

5. Once the device is rebooted go to 'Mapping Status' tab and see your device. Click on 'READ' to get the values from the device. 