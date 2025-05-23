# BACnet TCP/IP server with BACpypes

## Hardware and software
For this tutorial we will need the following hardware

- Seeed Studio reComputer R1000 series (Can be bought <a href="https://www.seeedstudio.com/reComputer-R1025-10-p-5895.html" target="_blank">here</a>)
- Power Adaptor for reComputer (Can be bought <a href="https://www.seeedstudio.com/Power-Adapter-12V-2A-EU-p-5732.html" target="_blank">here</a>)
- Monitor, keyboard and mouse for accessing the r1000 computer.
- Laptop as the BACnet client.  

We will need the following software

- Install VScode on R1000. Follow instructions <a href="https://code.visualstudio.com/" target="_blank">here</a>.
- Install BACpypes3 Python library on R1000 and your BACnet client laptop. Follow instructions <a href="https://github.com/pymodbus-dev/pymodbus/?tab=readme-ov-file#install" target="_blank">here</a>.

## Instructions
1. Connect your Seeed Studio R1000 with your client BACnet laptop to the same Wifi network.
```{image} ../../_static/bacnet_server/bacnet_server1.png
:width: 50%
:align: center
```
<br/><br/>

2. Copy and past this code into a file called bacnet_server.py on the R1000.
``` {dropdown} bacnet_server.py
    import asyncio
    import random
    import sys

    from bacpypes3.argparse import SimpleArgumentParser
    from bacpypes3.app import Application
    from bacpypes3.local.analog import AnalogValueObject
    from bacpypes3.local.binary import BinaryValueObject
    from bacpypes3.local.cmd import Commandable
    from bacpypes3.debugging import bacpypes_debugging, ModuleLogger

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

            self.read_only_av = AnalogValueObject(
                objectIdentifier=("analogValue", 1),
                objectName="read-only-av",
                presentValue=4.0,
                statusFlags=[0, 0, 0, 0],
                covIncrement=1.0,
                units="degreesFahrenheit",
                description="Simulated Read-Only Analog Value",
            )

            self.read_only_bv = BinaryValueObject(
                objectIdentifier=("binaryValue", 1),
                objectName="read-only-bv",
                presentValue="active",
                statusFlags=[0, 0, 0, 0],
                description="Simulated Read-Only Binary Value",
            )

            self.commandable_av = CommandableAnalogValueObject(
                objectIdentifier=("analogValue", 2),
                objectName="commandable-av",
                presentValue=0.0,
                statusFlags=[0, 0, 0, 0],
                covIncrement=1.0,
                units="degreesFahrenheit",
                description="Commandable Analog Value (Simulated)",
            )

            self.commandable_bv = CommandableBinaryValueObject(
                objectIdentifier=("binaryValue", 2),
                objectName="commandable-bv",
                presentValue="inactive",
                statusFlags=[0, 0, 0, 0],
                description="Commandable Binary Value (Simulated)",
            )

            for obj in [
                self.read_only_av,
                self.read_only_bv,
                self.commandable_av,
                self.commandable_bv
            ]:
                self.app.add_object(obj)

            _log.info("BACnet Objects initialized.")
            asyncio.create_task(self.update_values())

        async def update_values(self):
            test_values = [
                ("active", 1.0),
                ("inactive", 2.0),
                ("active", 3.0),
                ("inactive", 4.0),
            ]
            while True:
                await asyncio.sleep(INTERVAL)
                prob = random.random()
                if prob < 0.3:
                    next_value = test_values.pop(0)
                    test_values.append(next_value)

                    self.read_only_av.presentValue = next_value[1]
                    self.read_only_bv.presentValue = next_value[0]

                if _debug:
                    _log.debug(f"Read-Only AV: {self.read_only_av.presentValue}")
                    _log.debug(f"Read-Only BV: {self.read_only_bv.presentValue}")
                    _log.debug(f"Commandable AV: {self.commandable_av.presentValue}")
                    _log.debug(f"Commandable BV: {self.commandable_bv.presentValue}")

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

3. On the R1000 execute the bacnet_server.py code with the following command. Replacing the --address with your ip4 address.
```
python bacnet_server.py --address xx.xx.xx.xx/24 --name bacnet_device --instance 1 --debug
```
## Accessing the BACnet server from a BACnet Client
1. Make sure your BACnet client laptop is connected to the same Wifi network as the R1000. Copy and paste this code into a file called read_device_obj_props.py on your laptop which will be the BACnet client. Replace the parameters and execute the script as accordingly.
``` {dropdown} read_device_obj_props_simple.py
    import asyncio
    import socket

    from bacpypes3.app import Application
    from bacpypes3.primitivedata import CharacterString
    from bacpypes3.pdu import Address
    from bacpypes3.json.util import sequence_to_json
    from bacpypes3.basetypes import HostNPort
    from bacpypes3.primitivedata import ObjectIdentifier, PropertyIdentifier
    from bacpypes3.apdu import AbortReason, AbortPDU, ErrorRejectAbortNack
    from bacpypes3.vendor import get_vendor_info
    #=======================================================================================
    #PARAMETERS
    #=======================================================================================
    low_limit = 1
    high_limit = 3000

    localIPAddr = Address('xx.xx.xx.xxx/24')
    device_id: int = 2
    localObjName: str = 'BACapp'
    vendorId: int = 999
    vendorName: str = 'unknown'
    model_name: str = 'Scripting Tool'
    bbmdTTL: int = 0
    bdtable: list = None,
    bbmdAddress: str = None
    network_number: int = None
    #=======================================================================================
    #FUNCTIONS
    #=======================================================================================
    def charstring(val):
        return CharacterString(val) if isinstance(val, str) else val

    def validate_ip_address(ip: Address) -> bool:
        result = True
        s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        try:
            if not isinstance(ip, Address):
                raise ValueError("Provide Address as bacpypes.Address object")
            s.bind(ip.addrTuple)
        except OSError:
            result = False
        finally:
            s.close()
        return result

    async def object_identifiers( app: Application, device_address: Address, device_identifier: ObjectIdentifier) -> list[ObjectIdentifier]:
        """
        Read the entire object list from a device at once, or if that fails, read
        the object identifiers one at a time.
        """

        # try reading the whole thing at once, but it might be too big and
        # segmentation isn't supported
        try:
            object_list = await app.read_property(
                device_address, device_identifier, "object-list"
            )
            return object_list
        except AbortPDU as err:
            if err.apduAbortRejectReason != AbortReason.segmentationNotSupported:
                print(f"{device_identifier} object-list abort: {err}\n")
                return []
        except ErrorRejectAbortNack as err:
            print(f"{device_identifier} object-list error/reject: {err}\n")
            return []

        # fall back to reading the length and each element one at a time
        object_list = []
        try:
            # read the length
            object_list_length = await app.read_property(
                device_address,
                device_identifier,
                "object-list",
                array_index=0,
            )

            # read each element individually
            for i in range(object_list_length):
                object_identifier = await app.read_property(
                    device_address,
                    device_identifier,
                    "object-list",
                    array_index=i + 1,
                )
                object_list.append(object_identifier)
        except ErrorRejectAbortNack as err:
            print(f"{device_identifier} object-list length error/reject: {err}\n")

        return object_list

    #=======================================================================================
    #=======================================================================================
    if bbmdAddress is not None:
        mode = "foreign"
    elif bdtable:
        mode = "bbmd"
    else:
        mode = "normal"

    if not validate_ip_address(localIPAddr):
        raise ValueError("IP address not valid")

    base_cfg = {"application": 
        [
            {
                "active-cov-subscriptions": [],
                "apdu-segment-timeout": 1000,
                "apdu-timeout": 3000,
                "application-software-version": "1.0",
                "database-revision": 1,
                "device-address-binding": [],
                "firmware-revision": "N/A",
                "max-apdu-length-accepted": 1024,
                "max-segments-accepted": 16,
                "model-name": charstring(model_name),
                "number-of-apdu-retries": 3,
                "object-identifier": f"device,{device_id}",
                "object-list": [
                    f"device,{device_id}",
                    "network-port,1"
                ],
                "object-name": charstring(localObjName),
                "object-type": "device",
                "property-list": [
                    "object-identifier",
                    "object-name",
                    "object-type",
                    "property-list",
                    "system-status",
                    "vendor-name",
                    "vendor-identifier",
                    "model-name",
                    "firmware-revision",
                    "application-software-version",
                    "protocol-version",
                    "protocol-revision",
                    "protocol-services-supported",
                    "protocol-object-types-supported",
                    "object-list",
                    "max-apdu-length-accepted",
                    "segmentation-supported",
                    "max-segments-accepted",
                    "local-time",
                    "local-date",
                    "apdu-segment-timeout",
                    "apdu-timeout",
                    "number-of-apdu-retries",
                    "device-address-binding",
                    "database-revision",
                    "active-cov-subscriptions",
                    "status-flags"
                ],
                "protocol-object-types-supported": [],
                "protocol-revision": 22,
                "protocol-services-supported": [
                    "acknowledge-alarm",
                    "confirmed-cov-notification",
                    "confirmed-event-notification",
                    "subscribe-cov",
                    "atomic-write-file",
                    "add-list-element",
                    "read-property",
                    "read-property-multiple",
                    "write-property",
                    "write-property-multiple",
                    "read-range"
                ],
                "protocol-version": 1,
                "segmentation-supported": "segmented-both",
                "status-flags": [],
                "system-status": "operational",
                "vendor-identifier": vendorId,
                "vendor-name": charstring(vendorName)
            },
            {
                "bacnet-ip-mode": mode,
                "bacnet-ip-udp-port": localIPAddr.addrPort,
                "changes-pending": False,
                "ip-address": str(localIPAddr),
                "ip-subnet-mask": str(localIPAddr.netmask),
                "link-speed": 0.0,
                "network-number": network_number,
                "network-number-quality": "unknown",
                "network-type": "ipv4",
                "object-identifier": "network-port,1",
                "object-name": "NetworkPort-1",
                "object-type": "network-port",
                "out-of-service": False,
                "property-list": [
                    "object-identifier",
                    "object-name",
                    "object-type",
                    "property-list",
                    "status-flags",
                    "reliability",
                    "out-of-service",
                    "network-type",
                    "protocol-level",
                    "network-number",
                    "network-number-quality",
                    "changes-pending",
                    "mac-address",
                    "link-speed",
                    "bacnet-ip-mode",
                    "ip-address",
                    "bacnet-ip-udp-port",
                    "ip-subnet-mask"
                ],
                "protocol-level": "bacnet-application",
                "reliability": "no-fault-detected",
                "status-flags": [],
                "bbmd-broadcast-distribution-table": [],
                "bbmd-accept-f-d-registrations": False,
                "bbmd-foreign-device-table": [],
                "fd-bbmd-address": sequence_to_json(HostNPort(bbmdAddress)),
                "fd-subscription-lifetime": bbmdTTL,
            }
        ]
    }

    async def main() -> None:
        app = None
        try:
            # build an application
            app = Application.from_json(base_cfg["application"])
            print(app)
            # run the query
            i_ams = await app.who_is(low_limit, high_limit)
            for i_am in i_ams:
                device_address: Address = i_am.pduSource
                device_identifier: ObjectIdentifier = i_am.iAmDeviceIdentifier
                vendor_info = get_vendor_info(i_am.vendorID)
                print(f"{device_identifier} @ {device_address}, from vendor-{vendor_info.vendor_identifier}")
                object_list = await object_identifiers(app, device_address, device_identifier)
                try:
                    device_description: str = await app.read_property(
                        device_address, device_identifier, "description"
                    )
                    print(f"    description: {device_description}")
                except ErrorRejectAbortNack as err:
                    print(f"{device_identifier} description error: {err}\n")

                for object_identifier in object_list:
                    object_class = vendor_info.get_object_class(object_identifier[0])
                    if object_class is None:
                        print(f"unknown object type: {object_identifier}\n")
                        continue
                    print(f"    {object_identifier}:")
                
                    # read the property list
                    property_list = None
                    try:
                        property_list = await app.read_property(device_address, object_identifier, "property-list")
                    except ErrorRejectAbortNack as err:
                        print(f"{object_identifier} property-list error: {err}\n")

                    for property_name in (
                        "object-name",
                        "description",
                        "present-value",
                        "units",
                    ):
                        try:
                            # don't bother attempting to read the property if the object doesn't say it exists
                            property_identifier = PropertyIdentifier(property_name)
                            if property_identifier not in property_list:
                                continue

                            property_class = object_class.get_property_type(property_identifier)
                            if property_class is None:
                                print(f"{object_identifier} unknown property: {property_identifier}\n")
                                continue

                            property_value = await app.read_property(
                                device_address, object_identifier, property_identifier
                            )

                            print(f"        {property_name}: {property_value}")

                        except ErrorRejectAbortNack as err:
                            print(f"{object_identifier} {property_name} error: {err}\n")            
        finally:
            if app:
                app.close()

    if __name__ == "__main__":
        asyncio.run(main())
```

2. You should be able to get the following result.
```
device,1 @ xxx.xxx.xxx.xxx, from vendor-999
device,1 description error: property: unknown-property

    device,1:
        object-name: zulu
    network-port,1:
        object-name: NetworkPort-1
    analog-value,1:
        object-name: read-only-av
        description: Simulated Read-Only Analog Value
        present-value: 4.0
        units: degrees-fahrenheit
    binary-value,1:
        object-name: read-only-bv
        description: Simulated Read-Only Binary Value
        present-value: inactive
    analog-value,2:
        object-name: commandable-av
        description: Commandable Analog Value (Simulated)
        present-value: 0.0
        units: degrees-fahrenheit
    binary-value,2:
        object-name: commandable-bv
        description: Commandable Binary Value (Simulated)
        present-value: inactive
```

## Other Useful Scripts: Read and Write Multiple
1. You can also run the script below to read multiple properties.
``` {dropdown} read_multiple_properties_simple_script.py
    import asyncio
    import socket

    from bacpypes3.app import Application
    from bacpypes3.primitivedata import CharacterString
    from bacpypes3.pdu import Address
    from bacpypes3.json.util import sequence_to_json
    from bacpypes3.basetypes import HostNPort
    from bacpypes3.apdu import ErrorRejectAbortNack
    from bacpypes3.constructeddata import AnyAtomic
    #=======================================================================================
    #PARAMETERS
    #=======================================================================================
    # for single read
    object_identifier: str = 'analog-value,1'
    property_identifier: str = 'present-value'
    # for multiple read
    pmter_ls = ['analog-value,1',['present-value', 'object-name'],'analog-value,2',['present-value']]
    # parameters for creating a bacpypes app
    server_ipaddr = Address("10.42.0.1")
    localIPAddr: Address = Address("10.42.0.103/24")
    device_id: int = 2
    localObjName: str = 'BACapp'
    vendorId: int = 999
    vendorName: str = 'unknown'
    model_name: str = 'Scripting Tool'
    bbmdTTL: int = 0
    bdtable: list = None,
    bbmdAddress: str = None
    network_number: int = None
    #=======================================================================================
    #FUNCTIONS
    #=======================================================================================
    def charstring(val):
        return CharacterString(val) if isinstance(val, str) else val

    def validate_ip_address(ip: Address) -> bool:
        result = True
        s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        try:
            if not isinstance(ip, Address):
                raise ValueError("Provide Address as bacpypes.Address object")
            s.bind(ip.addrTuple)
        except OSError:
            result = False
        finally:
            s.close()
        return result

    #=======================================================================================
    #=======================================================================================
    if bbmdAddress is not None:
        mode = "foreign"
    elif bdtable:
        mode = "bbmd"
    else:
        mode = "normal"

    if not validate_ip_address(localIPAddr):
        raise ValueError("IP address not valid")

    base_cfg = {"application": 
        [
            {
                "active-cov-subscriptions": [],
                "apdu-segment-timeout": 1000,
                "apdu-timeout": 3000,
                "application-software-version": "1.0",
                "database-revision": 1,
                "device-address-binding": [],
                "firmware-revision": "N/A",
                "max-apdu-length-accepted": 1024,
                "max-segments-accepted": 16,
                "model-name": charstring(model_name),
                "number-of-apdu-retries": 3,
                "object-identifier": f"device,{device_id}",
                "object-list": [
                    f"device,{device_id}",
                    "network-port,1"
                ],
                "object-name": charstring(localObjName),
                "object-type": "device",
                "property-list": [
                    "object-identifier",
                    "object-name",
                    "object-type",
                    "property-list",
                    "system-status",
                    "vendor-name",
                    "vendor-identifier",
                    "model-name",
                    "firmware-revision",
                    "application-software-version",
                    "protocol-version",
                    "protocol-revision",
                    "protocol-services-supported",
                    "protocol-object-types-supported",
                    "object-list",
                    "max-apdu-length-accepted",
                    "segmentation-supported",
                    "max-segments-accepted",
                    "local-time",
                    "local-date",
                    "apdu-segment-timeout",
                    "apdu-timeout",
                    "number-of-apdu-retries",
                    "device-address-binding",
                    "database-revision",
                    "active-cov-subscriptions",
                    "status-flags"
                ],
                "protocol-object-types-supported": [],
                "protocol-revision": 22,
                "protocol-services-supported": [
                    "acknowledge-alarm",
                    "confirmed-cov-notification",
                    "confirmed-event-notification",
                    "subscribe-cov",
                    "atomic-write-file",
                    "add-list-element",
                    "read-property",
                    "read-property-multiple",
                    "write-property",
                    "write-property-multiple",
                    "read-range"
                ],
                "protocol-version": 1,
                "segmentation-supported": "segmented-both",
                "status-flags": [],
                "system-status": "operational",
                "vendor-identifier": vendorId,
                "vendor-name": charstring(vendorName)
            },
            {
                "bacnet-ip-mode": mode,
                "bacnet-ip-udp-port": localIPAddr.addrPort,
                "changes-pending": False,
                "ip-address": str(localIPAddr),
                "ip-subnet-mask": str(localIPAddr.netmask),
                "link-speed": 0.0,
                "network-number": network_number,
                "network-number-quality": "unknown",
                "network-type": "ipv4",
                "object-identifier": "network-port,1",
                "object-name": "NetworkPort-1",
                "object-type": "network-port",
                "out-of-service": False,
                "property-list": [
                    "object-identifier",
                    "object-name",
                    "object-type",
                    "property-list",
                    "status-flags",
                    "reliability",
                    "out-of-service",
                    "network-type",
                    "protocol-level",
                    "network-number",
                    "network-number-quality",
                    "changes-pending",
                    "mac-address",
                    "link-speed",
                    "bacnet-ip-mode",
                    "ip-address",
                    "bacnet-ip-udp-port",
                    "ip-subnet-mask"
                ],
                "protocol-level": "bacnet-application",
                "reliability": "no-fault-detected",
                "status-flags": [],
                "bbmd-broadcast-distribution-table": [],
                "bbmd-accept-f-d-registrations": False,
                "bbmd-foreign-device-table": [],
                "fd-bbmd-address": sequence_to_json(HostNPort(bbmdAddress)),
                "fd-subscription-lifetime": bbmdTTL,
            }
        ]
    }

    async def main() -> None:
        app = None
        try:
            # build an application
            app = Application.from_json(base_cfg["application"])
            try:
                # read a single property
                sgl_response = await app.read_property(
                        server_ipaddr,
                        object_identifier,
                        property_identifier,
                    )

                # read multiple property
                mltp_response = await app.read_property_multiple(server_ipaddr, pmter_ls)
                # print(f"    - response: {response}")
            except ErrorRejectAbortNack as err:
                print(f"    - exception: {err}")
                mltp_response = err

            if isinstance(mltp_response, AnyAtomic):
                print("    - schedule objects")
                mltp_response = mltp_response.get_value()

            print(f"this is the single response\n   {sgl_response}")
            print(f"this is the multiple read response\n    {mltp_response}")
    
        finally:
            if app:
                app.close()

    if __name__ == "__main__":
        asyncio.run(main())
```

2. You can also run the script below to write to multiple properties.
``` {dropdown} write_property_simple_script.py
    import re
    import socket
    import asyncio

    from bacpypes3.debugging import ModuleLogger
    from bacpypes3.app import Application
    from bacpypes3.primitivedata import CharacterString
    from bacpypes3.pdu import Address
    from bacpypes3.primitivedata import ObjectIdentifier
    from bacpypes3.apdu import ErrorRejectAbortNack
    from bacpypes3.json.util import sequence_to_json
    from bacpypes3.basetypes import HostNPort

    # some debugging
    _debug = 1
    _log = ModuleLogger(globals())
    # 'property[index]' matching
    property_index_re = re.compile(r"^([0-9A-Za-z-]+)(?:\[([0-9]+)\])?$")
    #=======================================================================================
    #PARAMETERS
    #=======================================================================================
    # for single write
    obj_identifier: str = 'analog-value,2'
    prop_identifier: str = 'present-value'
    val2write = 63.2
    write_priority: int = None
    # parameters for creating a bacpypes app
    server_ipaddr = Address("10.42.0.1")
    localIPAddr: Address = Address("10.42.0.103/24")
    device_id: int = 2
    localObjName: str = 'BACapp'
    vendorId: int = 999
    vendorName: str = 'unknown'
    model_name: str = 'Scripting Tool'
    bbmdTTL: int = 0
    bdtable: list = None,
    bbmdAddress: str = None
    network_number: int = None
    #=======================================================================================
    #FUNCTIONS
    #=======================================================================================
    def charstring(val):
        return CharacterString(val) if isinstance(val, str) else val

    def validate_ip_address(ip: Address) -> bool:
        result = True
        s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        try:
            if not isinstance(ip, Address):
                raise ValueError("Provide Address as bacpypes.Address object")
            s.bind(ip.addrTuple)
        except OSError:
            result = False
        finally:
            s.close()
        return result

    #=======================================================================================
    #=======================================================================================
    if bbmdAddress is not None:
        mode = "foreign"
    elif bdtable:
        mode = "bbmd"
    else:
        mode = "normal"

    if not validate_ip_address(localIPAddr):
        raise ValueError("IP address not valid")

    base_cfg = {"application": 
        [
            {
                "active-cov-subscriptions": [],
                "apdu-segment-timeout": 1000,
                "apdu-timeout": 3000,
                "application-software-version": "1.0",
                "database-revision": 1,
                "device-address-binding": [],
                "firmware-revision": "N/A",
                "max-apdu-length-accepted": 1024,
                "max-segments-accepted": 16,
                "model-name": charstring(model_name),
                "number-of-apdu-retries": 3,
                "object-identifier": f"device,{device_id}",
                "object-list": [
                    f"device,{device_id}",
                    "network-port,1"
                ],
                "object-name": charstring(localObjName),
                "object-type": "device",
                "property-list": [
                    "object-identifier",
                    "object-name",
                    "object-type",
                    "property-list",
                    "system-status",
                    "vendor-name",
                    "vendor-identifier",
                    "model-name",
                    "firmware-revision",
                    "application-software-version",
                    "protocol-version",
                    "protocol-revision",
                    "protocol-services-supported",
                    "protocol-object-types-supported",
                    "object-list",
                    "max-apdu-length-accepted",
                    "segmentation-supported",
                    "max-segments-accepted",
                    "local-time",
                    "local-date",
                    "apdu-segment-timeout",
                    "apdu-timeout",
                    "number-of-apdu-retries",
                    "device-address-binding",
                    "database-revision",
                    "active-cov-subscriptions",
                    "status-flags"
                ],
                "protocol-object-types-supported": [],
                "protocol-revision": 22,
                "protocol-services-supported": [
                    "acknowledge-alarm",
                    "confirmed-cov-notification",
                    "confirmed-event-notification",
                    "subscribe-cov",
                    "atomic-write-file",
                    "add-list-element",
                    "read-property",
                    "read-property-multiple",
                    "write-property",
                    "write-property-multiple",
                    "read-range"
                ],
                "protocol-version": 1,
                "segmentation-supported": "segmented-both",
                "status-flags": [],
                "system-status": "operational",
                "vendor-identifier": vendorId,
                "vendor-name": charstring(vendorName)
            },
            {
                "bacnet-ip-mode": mode,
                "bacnet-ip-udp-port": localIPAddr.addrPort,
                "changes-pending": False,
                "ip-address": str(localIPAddr),
                "ip-subnet-mask": str(localIPAddr.netmask),
                "link-speed": 0.0,
                "network-number": network_number,
                "network-number-quality": "unknown",
                "network-type": "ipv4",
                "object-identifier": "network-port,1",
                "object-name": "NetworkPort-1",
                "object-type": "network-port",
                "out-of-service": False,
                "property-list": [
                    "object-identifier",
                    "object-name",
                    "object-type",
                    "property-list",
                    "status-flags",
                    "reliability",
                    "out-of-service",
                    "network-type",
                    "protocol-level",
                    "network-number",
                    "network-number-quality",
                    "changes-pending",
                    "mac-address",
                    "link-speed",
                    "bacnet-ip-mode",
                    "ip-address",
                    "bacnet-ip-udp-port",
                    "ip-subnet-mask"
                ],
                "protocol-level": "bacnet-application",
                "reliability": "no-fault-detected",
                "status-flags": [],
                "bbmd-broadcast-distribution-table": [],
                "bbmd-accept-f-d-registrations": False,
                "bbmd-foreign-device-table": [],
                "fd-bbmd-address": sequence_to_json(HostNPort(bbmdAddress)),
                "fd-subscription-lifetime": bbmdTTL,
            }
        ]
    }

    async def main() -> None:
        app = None
        try:
            # interpret the address
            if _debug:
                _log.debug("device_address: %r", server_ipaddr)

            # interpret the object identifier
            object_identifier = ObjectIdentifier(obj_identifier)
            if _debug:
                _log.debug("object_identifier: %r", object_identifier)

            # split the property identifier and its index
            property_index_match = property_index_re.match(prop_identifier)
            if not property_index_match:
                raise ValueError("property specification incorrect")
            property_identifier, property_array_index = property_index_match.groups()
            if property_identifier.isdigit():
                property_identifier = int(property_identifier)
            if property_array_index is not None:
                property_array_index = int(property_array_index)
            # check the priority
            priority = None
            if write_priority:
                priority = write_priority
                if (priority < 1) or (priority > 16):
                    raise ValueError(f"priority: {priority}")
            if _debug:
                _log.debug("priority: %r", priority)

            # build an application
            app = Application.from_json(base_cfg["application"])
            if _debug:
                _log.debug("app: %r", app)

            try:
                response = await app.write_property(
                    server_ipaddr,
                    object_identifier,
                    property_identifier,
                    val2write,
                    property_array_index,
                    priority,
                )

                if _debug:
                    print("response: %r", response)

            except ErrorRejectAbortNack as err:
                print(str(err))

        finally:
            if app:
                app.close()


    if __name__ == "__main__":
        asyncio.run(main())

```

## Resources
- https://bac0.readthedocs.io/en/latest/read.html#read-examples
- BACnet hierachy 
    - Device
        - each device can contain multiple objects Objects (e.g. 'analog-value,1', 'network-port,1', 'device,1')
            - within each object there are multiple Properties (e.g. 'present-value', 'object-name')

- using the sample console scripts from the BACpypes sample folder, on the client side run this
    - python read-property.py xx.xx.xx.xx analog-value,1 present-value
    - python discover-devices.py --address xx.xx.xx.xx/24 1 3000 --vendoridentifier 999 --debug
    - python write-property.py xx.xx.xx.xx/24 analog-value,1 present-value 28.0

