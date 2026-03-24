# Enocean Motion Detector with Illumination Sensor

## Using EMDCU with Contemporary Controls Enocean Gateway
### Hardware and software
For this tutorial we will need the following hardware

- Contemporary Controls Enocean BACnet gateway (Can be bought <a href="https://www.ccontrols.com/basautomation/enocean.php" target="_blank">here</a>)
- Enocean Occupancy Sensor EMDCU (If you are in US can be bought here <a href="https://enocean.cdistore.com/products/detail/emdcu-enocean/685908/?pid=" target="_blank">here</a>)
- 24V power supply x 1 (Can be bought <a href="https://www.amazon.com/dp/B09281KTS8?ref=nb_sb_ss_w_as-reorder_k1_1_5&amp=&crid=2KKGJ6OCSOW31&amp=&sprefix=24+v+" target="_blank">here</a>)
- Phone with NFC enabled.
- Laptop.

### Instructions
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

## Using EMDCB DB-9 with Python Bleak Library.
### Hardware and software
For this tutorial we will need the following hardware

- Enocean Occupancy Sensor EMDCB (If you are in US can be bought here <a href="https://enocean.cdistore.com/products/detail/emdcb-enocean/671180/" target="_blank">here</a>)
- Phone with NFC enabled.
- Laptop.

### Instructions
1. Install Python and install bleak, and pycryptodome library.

2. Run the following script to communicate with the sensor.
    ```{dropdown} read_emdcb.py
    import asyncio
    from typing import Any

    from bleak import BleakScanner
    from Crypto.Cipher import AES  # requires pycryptodome
    # -----------------------------------------------------------------------------
    # ---- EMDCB payload parsing (per manual’s Appendix “Parsing telegrams”) ------
    # Layout inside Manufacturer payload:
    #   [0:4]  Sequence counter (LE)
    #   [4:-4] Sensor Data (10 bytes in canonical example)
    #   [-4:]  MIC (4 bytes)
    #
    # Sensor Data uses "descriptor/value" pairs (example set shown in the manual):
    #   0x02 -> Energy level (1B; percent = value/2)
    #   0x44 -> Solar-cell illuminance (2B LE; lux)
    #   0x45 -> Light-sensor illuminance (2B LE; lux)
    #   0x20 -> Occupancy flag (0x02 => occupied, 0x01 => not occupied)
    # See: EMDCB User Manual (DB rev), “Parsing EMDCB telegrams” example.
    # --- USER SETTINGS -----------------------------------------------------------
    enocean_ble_addr_hex = ''
    security_key_hex = ''
    # 2) EnOcean company identifier 0x03DA (986) (default for EMDCB unless changed via NFC) You can change MANUFACTURER_ID via NFC if needed.
    enocean_id = 0x03DA
    prev_seq_cnt = None

    def sort_data(mfg_payload: bytes) -> dict[str, bytes]:
        """
        sort mfg payload into a dictionary
        
        Parameters
        ----------
        mfg_payload: bytes
            bluetooth payload from python bleak library.
            
        Returns
        -------
        dict
            dictionary with the following
            - sequence
            - signature
            - sensor_data
            - authentication
        """
        if len(mfg_payload) != 18:
            return {"error": "payload length is not right", "raw_hex": mfg_payload.hex().upper()}
        
        seq = mfg_payload[0:4]
        sensor_data = mfg_payload[4:-4]
        sig = mfg_payload[-4:]

        result = {
            'sequence': seq,
            'signature': sig,
            'sensor_data': sensor_data
            }
        return result

    def authenticate(ble_addr: bytes,  mfg_payload: bytes) -> dict[str, bytes]:
        """
        authenticate the telegram from enocean
        
        Parameters
        ----------
        ble_addr: bytes
            bluetooth addr
        
        mfg_payload: bytes
            bluetooth payload from python bleak library.
            
        Returns
        -------
        dict
            dictionary with the following:
            - sequence
            - signature
            - sensor_data
            - 
        """
        sorted_data = sort_data(mfg_payload)

        seq_cter = sorted_data['sequence']
        snsr_data = sorted_data['sensor_data']
        signature = sorted_data['signature']
        
        # build the nonce
        ble_addr_le = ble_addr[::-1]
        padding = b'\x00\x00\x00'
        nonce = ble_addr_le + seq_cter + padding
        
        # build the input data 
        # length of payload
        ble_length = 1 + 2 + len(mfg_payload)
        # build the authenticated payload
        ble_len_bytes = ble_length.to_bytes(1, 'little')
        type_bytes = 0xFF.to_bytes(1, 'little')
        enocean_id_bytes = enocean_id.to_bytes(2, 'little')
        auth_payload = ble_len_bytes + type_bytes + enocean_id_bytes + seq_cter + snsr_data
        # auth_payload = bytes([ble_length, 0xFF, 0xDA, 0x03]) + seq_cter_le + snsr_data
        
        # Authenticate using AES-128 CCM
        key = bytes.fromhex(security_key_hex)
        cipher = AES.new(key, AES.MODE_CCM, nonce=nonce, mac_len=4)
        cipher.update(auth_payload)
        calc_sig = cipher.digest()
        if calc_sig == signature:
            # print('Authentication SUCCESS!')
            sorted_data['authentication'] = True
        else:
            # print('Authentication FAILED!')
            sorted_data['authentication'] = False
        # cipher.verify(signature)

        return sorted_data

    def parse_emdcb_payload(sensor_data: bytes) -> dict[str, Any]:
        i = 0
        result = {}
        while i < len(sensor_data):
            descriptor = sensor_data[i] # data descriptor
            i += 1

            # Extract Size (top 2 bits) and Type ID (bottom 6 bits)
            size_bits = (descriptor >> 6) & 0x03
            type_id = descriptor & 0x3F

            # Determine data length in bytes
            if size_bits == 0:
                data_len = 1
            elif size_bits == 1:
                data_len = 2
            elif size_bits == 2:
                data_len = 4

            # take into account commissioning info
            if type_id == 0x3E:
                data_len = 22

            # 2. Extract the Data Bytes
            data = sensor_data[i : i + data_len]
            i += data_len

            if type_id == 0x02:  # Energy level (8-bit), percent = val/2
                val = data[0] * 0.5
                result["energy_percent"] = val

            elif type_id == 0x04:  # Solar-cell illuminance (16-bit LE)
                lux = int.from_bytes(data, "little")
                result["lux_solar_cell"] = lux

            elif type_id == 0x05:  # Light-sensor illuminance (16-bit LE)
                lux = int.from_bytes(data, "little")
                result["lux_sensor"] = lux

            elif type_id == 0x20:  # Occupancy present => occupied
                occ = data[0]
                if occ == 0x01:
                    result["occupied"] = False
                elif occ == 0x02:
                    result["occupied"] = True
            else:
                # Unknown/aux descriptor – record it and continue
                result.setdefault("unknown_descriptors", []).append(descriptor)

        return result

    def mac_str_to_bytes(mac: str) -> bytes:
        """Convert 'AA:BB:CC:DD:EE:FF' to b'\\xAA\\xBB\\xCC\\xDD\\xEE\\xFF'."""
        return bytes(int(x, 16) for x in mac.split(":"))

    # ------------------------------ BLE scanner ----------------------------------
    def on_adv(device, adv_data):
        global prev_seq_cnt
        ble_addr_hex = device.address.replace(':', '')
        if ble_addr_hex.upper() == enocean_ble_addr_hex:
            mfg_data = adv_data.manufacturer_data
            mfg_id = list(mfg_data.keys())[0]
            if mfg_id == enocean_id:
                payload = mfg_data[mfg_id]
                ble_addr = bytes.fromhex(ble_addr_hex)
                authenticated_sorted_data = authenticate(ble_addr, payload)
                if authenticated_sorted_data['authentication'] == True:
                    seq_ct_int = int.from_bytes(authenticated_sorted_data['sequence'], 'little')
                    print(f'Current seq cnt is {seq_ct_int}, prev cnt is {prev_seq_cnt}')
                    if prev_seq_cnt == None or seq_ct_int > prev_seq_cnt:
                        sensor_data = authenticated_sorted_data['sensor_data']
                        parsed = parse_emdcb_payload(sensor_data)
                        prev_seq_cnt = seq_ct_int
                        print(f'Telegram count {seq_ct_int}')
                        print(parsed)
                else:
                    print('Authentication FAILED')

    async def main():
        scanner = BleakScanner(on_adv)
        await scanner.start()
        print("Listening for EMDCB advertisements… Press Ctrl+C to stop.")
        try:
            while True:
                await asyncio.sleep(1)
        except KeyboardInterrupt:
            pass
        finally:
            await scanner.stop()

    if __name__ == "__main__":
        asyncio.run(main())

    ```