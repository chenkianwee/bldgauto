# Bacpypes3 

- If you need a deep debug messages you can add on this option
    ```
    python your_bacpypes3_script.py <arguments> --debug bacpypes3.app.Application
    ```

- check your ubuntu firewall, if inactive means its disabled if active you will need to enable port 47808 for communication. 
    ```
    sudo ufw status
    ```

- allow bacnet through firewall
    ```
    sudo ufw allow 47808/udp
    ```

- to check your 47808 bacnet port on your ubuntu
    ```
    sudo tcpdump -i any port 47808
    ```

- bacpypes 3 code for checking the received data from controller
```{dropdown} code snippet
    def parse_notification(self, apdu):
        for value in apdu.listOfValues:
            prop_id = value.propertyIdentifier
            raw_any = value.value  # This is the <Any object>
            
            try:
                if prop_id == PropertyIdentifier.presentValue:
                    # 1. Grab the pure data tag directly (ignoring the open/close tags) the data is always sitting at index 1
                    data_tag = raw_any.tagList[1]
        
                    # Force it to print the raw ASN.1 tags and byte length
                    if hasattr(raw_any, 'debug_contents'):
                        raw_any.debug_contents()
                    else:
                        print(f"Raw Object: {raw_any}")
```

- bacpypes3 code for parsing different types of value recieved from controller
```{dropdown} code snippet
    # def parse_property_identifiers():
        # print(f'THIS IS THE PROPERTY VALUE {property_value}')
        # pv_type_str = type(my_data).__name__
        # if pv_type_str == 'ArrayOfPriorityValue':
        #     for pv in property_value:
        #         print(f'THIS IS A PRIORITY VALUE ARRAY {pv.real}')
        #         # choice_name = pv._choice
        #         # actual_value = getattr(pv, choice_name)
        #         # print(f"The active type is: {choice_name}")
        #         # print(f"The actual value is: {actual_value}")
        # # elif pv_type_str == 'CharacterString':
        # #     print(f'this is the character string {property_value.value}')
        # elif pv_type_str == 'Real':
        #     print(f'this is the character string {property_value.value}')
        # # print(type(property_value))

        # elif pv_type_str == 'CharacterString':
        #     print(f'this is the character string {property_value.value}')
        #         pv_type_str = type(property_value).__name__
        #         print(pv_type_str)
```