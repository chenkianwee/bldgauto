# Tabletop Digital Twin

1. Once you have setup the tabletop digital twin from the previous tutorial. Download the Y2I software stack to develop a digital twin of the tabletop BAS <a href="https://store.arduino.cc/products/opta-wifi?queryID=undefined" target="_blank">here</a>.

2. Unzip the file. Go to the yun2infinity-0.0.13_tabletop_dt/django/yun2inf_project folder. <a href="https://docs.docker.com/engine/install/ubuntu/#installation-methods" target="_blank">Install Docker</a>. Build the the container with the following command.
    ```
    sudo docker build . -t y2i/yun2inf:0.0.9a
    ```

3. Install the Y2I stack by going to the yun2infinity-0.0.13_tabletop_dt/shellscript folder and executing the following command. Once installed go to localhost/csviewer to see the 3D model of the tabletop prototype.
    ```
    sudo sh setup_yun2inf.sh
    ```

4. Now we are going to configure the FROST-server to properly receive data from the bacnet server.

5. First run this Python script to register the observation properties and sensors. You should get 4 201 response after running the script.
    ```{dropdown} sensorthings_reg_sensor_prop.py
        import requests
        from requests.auth import HTTPBasicAuth

        url = 'http://localhost/frost/v1.1/'
        auth_user = 'admin'
        auth_pass = 'admin'

        #========================================================================
        #observed property
        #========================================================================
        observed_prop1 = {"name": "temperature",
                        "description": "temperature",
                        "definition": ""
                        }

        r1 = requests.post(url+"ObservedProperties", 
                        auth=HTTPBasicAuth(auth_user, auth_pass), 
                        json=observed_prop1)

        print(r1)

        observed_prop1 = {"name": "on_off",
                        "description": "on_off",
                        "definition": ""
                        }

        r1 = requests.post(url+"ObservedProperties", 
                        auth=HTTPBasicAuth(auth_user, auth_pass), 
                        json=observed_prop1)

        print(r1)

        #========================================================================
        #sensor
        #========================================================================
        sensor_data1 = {"name": "thermostat",
                        "description": "thermostat",
                        "encodingType": "application/pdf",
                        "metadata": ""
                        }

        r1 = requests.post(url+"Sensors", 
                        auth=HTTPBasicAuth(auth_user, auth_pass), 
                        json=sensor_data1)
        print(r1)

        sensor_data1 = {"name": "arduino control",
                        "description": "PLC controller",
                        "encodingType": "application/pdf",
                        "metadata": ""
                        }

        r1 = requests.post(url+"Sensors", 
                        auth=HTTPBasicAuth(auth_user, auth_pass), 
                        json=sensor_data1)
        print(r1)
    ```

6. Run the following Python script to register the necessary datastreams for monitoring.
    ``` {dropdown} sensorthings_reg_thing_ds.py
        import requests
        from requests.auth import HTTPBasicAuth

        url = 'http://localhost/frost/v1.1/'
        auth_user = 'admin'
        auth_pass = 'admin'
        #==========================================================================================================================================================
        # region: Fill in the parameters, #The sensorthings API data model is used (https://developers.sensorup.com/docs/#introduction)
        #==========================================================================================================================================================
        #THING PARAMETERS
        thing_name = 'supervisory controller'
        thingDesc = 'setup to test temp sensor'#describe the deployment
        #==========================================================================================================================================================
        #LOCATION PARAMETERS
        locName = 'xyz place'
        locDesc = 'xyz waiting area'
        loc_encoding_type = 'application/vnd.geo+json'
        #Define the geometry of the location. Geometry types from geojson is accepted. Refer to https://tools.ietf.org/html/rfc7946 for geometry types.
        loc_type = 'Point'
        coordinates = [-74.6448927740998, 40.344893511213165, 0] #0,0,0 lon/kat/alt is the coordinates of null island, this is just a place holder
        #==========================================================================================================================================================
        #DATASTREAM PARAMETERS
        ds_names = ['setpt', 'airtemp', 'htg_on_off']
        ds_descs = ['setpt of thermostat', 'airtemp of thermostat', 'is heater on or off']
        obs_types = ['http://www.opengis.net/def/observationType/OGC-OM/2.0/OM_Measurement', 'http://www.opengis.net/def/observationType/OGC-OM/2.0/OM_Measurement', 
                    'http://www.opengis.net/def/observationType/OGC-OM/2.0/OM_Measurement']

        ds_uoms = [{"name": "Temperature","symbol": "degC","definition": ""}, 
                {"name": "Temperature","symbol": "degC","definition": ""}, 
                {"name": "OnOff","symbol": "unitless","definition": ""}]
        #visit "http://www.qudt.org/qudt/owl/1.0.0/unit/Instances.html" to get the definition

        #id of the observed property, go to  https://localhost/frost/v1.1/ObservedProperties to check the properties availabe and specify the @iot.id
        #if properties are not available you will have to register a new property
        ds_obs_prop_ids = [1, 1, 2]
        #id of the sensor, go to https://localhost/frost/v1.1/Sensors to check the sensors availabe and specify the @iot.id
        #if sensors are not available you will have to register a new property 
        ds_sensor_ids = [1, 1, 2]

        # endregion
        #==========================================================================================================================================================
        # region:FUNCTIONS
        #==========================================================================================================================================================
        def get_id_from_header(location_str):
            split = location_str.split('/')
            thing = split[-1]
            idx1 = thing.index('(')
            idx2 = thing.index(')')
            thing_id = int(thing[idx1+1:idx2])
            return thing_id

        def postds(thing_id, dsname, dsdesc, dsobs, dsuom, dsobsprop_id, dssensor_id):
            #Parameters that defines the datastream
            datastream_data = {'name': dsname,
                            'description': dsdesc,
                            'observationType': dsobs,
                            'unitOfMeasurement': dsuom,
                            'Thing':{"@iot.id":thing_id},
                            'ObservedProperty':{'@iot.id':dsobsprop_id},
                            'Sensor':{'@iot.id':dssensor_id}}
            dr = requests.post(url+"Datastreams", auth=HTTPBasicAuth(auth_user,auth_pass), json=datastream_data)
            ds_loc_str = dr.headers['Location'] 
            ds_id = get_id_from_header(ds_loc_str )
            return ds_id

        # endregion
        #==========================================================================================================================================================
        # region: THE JSON OBJECTS TO BE POST
        #==========================================================================================================================================================
        #Parameters that defines the location of the thing    
        loc_data = {'name': locName,
                    'description': locDesc,
                    'encodingType': loc_encoding_type,
                    'location': {'type': loc_type,
                                'coordinates': coordinates}}

        #Parameters that defines the thing
        thingloc_data = {'name': thing_name,
                        'description': thingDesc,
                        'Locations': [loc_data]}
        #==========================================================================================================================================================
        #==========================================================================================================================================================
        is_thing_reg = False
        #check if this thing already exist on the database
        filter_thing = "Things?$filter=name eq '" + thing_name + "'&$expand=Locations($select=name),Datastreams($select=id)&$select=id"
        find_thing = requests.get(url+filter_thing)
        thing_content = find_thing.json()
        thing_ls = thing_content['value']

        nthings = len(thing_ls)
        if nthings > 0:
            loc_name_val = thing_ls[0]['Locations'][0]['name']
            if loc_name_val == locName:
                is_thing_reg = True
                
        if is_thing_reg == False:
            r = requests.post(url+"Things",auth=HTTPBasicAuth(auth_user,auth_pass),
                            json=thingloc_data)
            print(r.text)
            loc_str = r.headers['Location']
            thing_id = get_id_from_header(loc_str)
            # post datastreams to the thing
            ds_ids = []
            nds = len(ds_names)
            for i in range(nds):
                dsname = ds_names[i]
                dsdesc = ds_descs[i]
                dsobs = obs_types[i]
                dsuom = ds_uoms[i]
                dsobsprop_id = ds_obs_prop_ids[i]
                dssensor_id = ds_sensor_ids[i]
                ds_id = postds(thing_id, dsname, dsdesc, dsobs, dsuom, dsobsprop_id, dssensor_id)
                ds_ids.append(ds_id)
        else:
            thing_id = thing_ls[0]['@iot.id']
            dss = thing_ls[0]['Datastreams']
            ds_ids = []
            for ds in dss:
                ds_id = ds['@iot.id']
                ds_ids.append(ds_id)

        #==========================================================================================================================================================
        print('===============================================================================')
        print('Thing ID:', thing_id)
        print('Visit this page to look at the registered Device: \n',url+'Things(' + str(thing_id) + ')')
        print('Datastream ID:', ds_ids)
        print('Use this information to post observation in sensorthing_post_observation.py')
        print('===============================================================================')
        # endregion
    ```

7. Run the following Python script to register the necessary datastreams for controls.
    ``` {dropdown} sensorthings_reg_ctrl_ds.py
        import requests
        from requests.auth import HTTPBasicAuth

        url = 'http://localhost/frost/v1.1/'
        auth_user = 'admin'
        auth_pass = 'admin'
        #==========================================================================================================================================================
        # region: Fill in the parameters, #The sensorthings API data model is used (https://developers.sensorup.com/docs/#introduction)
        #==========================================================================================================================================================
        #THING PARAMETERS
        thing_name = 'supervisory controller setpt'
        thingDesc = 'change setpt'#describe the deployment
        #==========================================================================================================================================================
        #LOCATION PARAMETERS
        locName = 'xyz place'
        locDesc = 'xyz waiting area'
        loc_encoding_type = 'application/vnd.geo+json'
        #Define the geometry of the location. Geometry types from geojson is accepted. Refer to https://tools.ietf.org/html/rfc7946 for geometry types.
        loc_type = 'Point'
        coordinates = [-74.6448927740998, 40.344893511213165, 0] #0,0,0 lon/kat/alt is the coordinates of null island, this is just a place holder
        #==========================================================================================================================================================
        #DATASTREAM PARAMETERS
        ds_names = ['setpt_change']
        ds_descs = ['setpt_change']
        obs_types = ['http://www.opengis.net/def/observationType/OGC-OM/2.0/OM_Measurement']

        ds_uoms = [{"name": "Temperature","symbol": "degC","definition": ""}]
        #visit "http://www.qudt.org/qudt/owl/1.0.0/unit/Instances.html" to get the definition

        #id of the observed property, go to  https://localhost/frost/v1.1/ObservedProperties to check the properties availabe and specify the @iot.id
        #if properties are not available you will have to register a new property
        ds_obs_prop_ids = [1]
        #id of the sensor, go to https://localhost/frost/v1.1/Sensors to check the sensors availabe and specify the @iot.id
        #if sensors are not available you will have to register a new property 
        ds_sensor_ids = [1]

        # endregion
        #==========================================================================================================================================================
        # region:FUNCTIONS
        #==========================================================================================================================================================
        def get_id_from_header(location_str):
            split = location_str.split('/')
            thing = split[-1]
            idx1 = thing.index('(')
            idx2 = thing.index(')')
            thing_id = int(thing[idx1+1:idx2])
            return thing_id

        def postds(thing_id, dsname, dsdesc, dsobs, dsuom, dsobsprop_id, dssensor_id):
            #Parameters that defines the datastream
            datastream_data = {'name': dsname,
                            'description': dsdesc,
                            'observationType': dsobs,
                            'unitOfMeasurement': dsuom,
                            'Thing':{"@iot.id":thing_id},
                            'ObservedProperty':{'@iot.id':dsobsprop_id},
                            'Sensor':{'@iot.id':dssensor_id}}
            dr = requests.post(url+"Datastreams", auth=HTTPBasicAuth(auth_user,auth_pass), json=datastream_data)
            ds_loc_str = dr.headers['Location'] 
            ds_id = get_id_from_header(ds_loc_str )
            return ds_id

        # endregion
        #==========================================================================================================================================================
        # region: THE JSON OBJECTS TO BE POST
        #==========================================================================================================================================================
        #Parameters that defines the location of the thing    
        loc_data = {'name': locName,
                    'description': locDesc,
                    'encodingType': loc_encoding_type,
                    'location': {'type': loc_type,
                                'coordinates': coordinates}}

        #Parameters that defines the thing
        thingloc_data = {'name': thing_name,
                        'description': thingDesc,
                        'Locations': [loc_data]}
        #==========================================================================================================================================================
        #==========================================================================================================================================================
        is_thing_reg = False
        #check if this thing already exist on the database
        filter_thing = "Things?$filter=name eq '" + thing_name + "'&$expand=Locations($select=name),Datastreams($select=id)&$select=id"
        find_thing = requests.get(url+filter_thing)
        thing_content = find_thing.json()
        thing_ls = thing_content['value']

        nthings = len(thing_ls)
        if nthings > 0:
            loc_name_val = thing_ls[0]['Locations'][0]['name']
            if loc_name_val == locName:
                is_thing_reg = True
                
        if is_thing_reg == False:
            r = requests.post(url+"Things",auth=HTTPBasicAuth(auth_user,auth_pass),
                            json=thingloc_data)
            print(r.text)
            loc_str = r.headers['Location']
            thing_id = get_id_from_header(loc_str)
            # post datastreams to the thing
            ds_ids = []
            nds = len(ds_names)
            for i in range(nds):
                dsname = ds_names[i]
                dsdesc = ds_descs[i]
                dsobs = obs_types[i]
                dsuom = ds_uoms[i]
                dsobsprop_id = ds_obs_prop_ids[i]
                dssensor_id = ds_sensor_ids[i]
                ds_id = postds(thing_id, dsname, dsdesc, dsobs, dsuom, dsobsprop_id, dssensor_id)
                ds_ids.append(ds_id)
        else:
            thing_id = thing_ls[0]['@iot.id']
            dss = thing_ls[0]['Datastreams']
            ds_ids = []
            for ds in dss:
                ds_id = ds['@iot.id']
                ds_ids.append(ds_id)

        #==========================================================================================================================================================
        print('===============================================================================')
        print('Thing ID:', thing_id)
        print('Visit this page to look at the registered Device: \n',url+'Things(' + str(thing_id) + ')')
        print('Datastream ID:', ds_ids)
        print('Use this information to post observation in sensorthing_post_observation.py')
        print('===============================================================================')
        # endregion
    ```

8. Make sure you are connected to the same wifi as the BAS prototype. Make sure the BACnet server is running on the supervisory controller as described [here](05_2tabletop_bas.md#configure-seeed-studio-r1000).

9. Run the following Python script on your command line. Remember to change your local ipaddress in the get_bacnet_points() function. You can check your IP address by running the command 'ifconfig' in the Ubuntu OS.
``` {dropdown} get_bacnet_points.py
    import time
    import asyncio
    import datetime
    import socket
    import requests
    from requests.auth import HTTPBasicAuth

    from bacpypes3.app import Application
    from bacpypes3.primitivedata import CharacterString
    from bacpypes3.pdu import Address
    from bacpypes3.json.util import sequence_to_json
    from bacpypes3.basetypes import HostNPort
    from bacpypes3.primitivedata import ObjectIdentifier, PropertyIdentifier
    from bacpypes3.apdu import AbortReason, AbortPDU, ErrorRejectAbortNack
    from bacpypes3.vendor import get_vendor_info

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

    async def get_bacnet_points() -> None:
        low_limit = 1
        high_limit = 3000

        localIPAddr = Address('xx.xx.xx.xx/24')
        device_id: int = 1
        localObjName: str = 'BACapp'
        vendorId: int = 999
        vendorName: str = 'unknown'
        model_name: str = 'Scripting Tool'
        bbmdTTL: int = 0
        bdtable: list = None,
        bbmdAddress: str = None
        network_number: int = None

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

        app = None
        try:
            # build an application
            app = Application.from_json(base_cfg["application"])
            # run the query
            i_ams = await app.who_is(low_limit, high_limit)
            points = []
            for i_am in i_ams:
                device_address: Address = i_am.pduSource
                device_identifier: ObjectIdentifier = i_am.iAmDeviceIdentifier
                vendor_info = get_vendor_info(i_am.vendorID)
                # print(f"{device_identifier} @ {device_address}, from vendor-{vendor_info.vendor_identifier}")
                object_list = await object_identifiers(app, device_address, device_identifier)
                try:
                    device_description: str = await app.read_property(
                        device_address, device_identifier, "description"
                    )
                    # print(f"    description: {device_description}")
                except ErrorRejectAbortNack as err:
                    print(f"{device_identifier} description error: {err}\n")

                for object_identifier in object_list:
                    object_class = vendor_info.get_object_class(object_identifier[0])
                    if object_class is None:
                        # print(f"unknown object type: {object_identifier}\n")
                        continue
                    print(f"    {object_identifier}:")
                
                    # read the property list
                    property_list = None
                    try:
                        property_list = await app.read_property(device_address, object_identifier, "property-list")
                    except ErrorRejectAbortNack as err:
                        print(f"{object_identifier} property-list error: {err}\n")
                    
                    point_dict = {}
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
                            point_dict[str(property_name)] = property_value
                            print(f"        {property_name}: {property_value}")

                        except ErrorRejectAbortNack as err:
                            print(f"{object_identifier} {property_name} error: {err}\n")

                    points.append(point_dict)

            return points
        finally:
            if app:
                app.close()

    def get_bas_data() -> dict:
        point_list = asyncio.run(get_bacnet_points())
        val_dict = {}
        for point in point_list:
            if point['object-name'] == 'tstat-setpt':
                setpt_val = point['present-value']
                val_dict['setpt'] = setpt_val
            elif point['object-name'] == 'read-only-tstat-airtemp':
                airtemp_val = point['present-value']
                val_dict['airtemp'] = airtemp_val
            elif point['object-name'] == 'read-only-tstat-ht':
                htg_onoff = point['present-value']
                val_dict['htg_onoff'] = htg_onoff
                
        return val_dict

    def post2stapi():
        url = 'http://localhost/frost/v1.1/'
        auth_user = 'admin'
        auth_pass = 'admin'

        ds_ids = [1,2,3]
        point_list = get_bas_data()
        utc_tz = datetime.timezone(datetime.timedelta(hours=0))
        tz = datetime.timezone(datetime.timedelta(hours=-4)) #singapore timing
        pydt = datetime.datetime.now(tz)
        pydt = pydt.astimezone(utc_tz)
        dstr = pydt.isoformat(timespec='seconds')
        data_array = []
        for ds_id in ds_ids:
            if ds_id == 1:
                res = round(point_list['setpt'],2)
            elif ds_id == 2:
                res = round(point_list['airtemp'], 2)
            elif ds_id == 3:
                res = point_list['htg_onoff']

            obs = {"Datastream": {"@iot.id": ds_id}, "components": ["phenomenonTime", "resultTime", "result"], 
                    "dataArray": [[dstr,dstr,res]]}
            data_array.append(obs)

        r1 = requests.post(url+"CreateObservations", 
                        auth=HTTPBasicAuth(auth_user, auth_pass), 
                        json=data_array)
        
        print(r1)

    if __name__ == "__main__":
        while True:
            try:
                post2stapi()
            except Exception as e:
                print(f"Error reading BACnet: {e}")
            time.sleep(10)

```

10. Once this script is run successfully you will be able to see the live update on the 3d dashboard (localhost/csviewer).

11. Run the following Python script on your command line. This will allow you to control the setpoint of the thermostat. Make sure to change the ipaddress of the BACnet server and your local machine in the write2bacnet_setpt() function.
``` {dropdown} write_bacnet_setpt.py
    import re
    import socket
    import asyncio
    import pytz
    import datetime
    import time
    from dateutil import parser

    from bacpypes3.debugging import ModuleLogger
    from bacpypes3.app import Application
    from bacpypes3.primitivedata import CharacterString
    from bacpypes3.pdu import Address
    from bacpypes3.primitivedata import ObjectIdentifier
    from bacpypes3.apdu import ErrorRejectAbortNack
    from bacpypes3.json.util import sequence_to_json
    from bacpypes3.basetypes import HostNPort

    import requests
    from requests.auth import HTTPBasicAuth

    # some debugging
    _debug = 1
    _log = ModuleLogger(globals())
    # 'property[index]' matching
    property_index_re = re.compile(r"^([0-9A-Za-z-]+)(?:\[([0-9]+)\])?$")
    #=======================================================================================
    #PARAMETERS
    #=======================================================================================

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

    def process_obs(obs_json):
        res = None
        obss = obs_json['Observations']
        res_dict = {}
        for obs in obss:
            res = obs['result']
            res_time = obs['phenomenonTime']
            res_dt = parser.parse(res_time)
            res_dt_nj = res_dt.astimezone(tz=pytz.timezone('US/Eastern'))
            res_dict[res_dt_nj] = res

        res_dict = dict(sorted(res_dict.items(), reverse=True))
        dkeys = list(res_dict.keys())
        if len(dkeys) != 0: 
            res = res_dict[dkeys[0]]

        return res

    async def write2bacnet_setpt(val2write: float | int) -> None:
        # for single write
        obj_identifier: str = 'analog-value,1'
        prop_identifier: str = 'present-value'
        write_priority: int = None
        # parameters for creating a bacpypes app
        server_ipaddr = Address("xx.xx.xx.xx")
        localIPAddr: Address = Address("xx.xx.xx.xx/24")
        device_id: int = 2
        localObjName: str = 'BACapp'
        vendorId: int = 999
        vendorName: str = 'unknown'
        model_name: str = 'Scripting Tool'
        bbmdTTL: int = 0
        bdtable: list = None,
        bbmdAddress: str = None
        network_number: int = None

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

    def get_stapi_setpt_change():
        auth_user = 'admin'
        auth_pass = 'admin'
        dt_now = datetime.datetime.now()
        dt_now = dt_now.astimezone(tz=pytz.timezone('US/Eastern'))
        start_time = dt_now - datetime.timedelta(seconds=15)
        start_time_str = start_time.isoformat(timespec='seconds')
        now_time_str = dt_now.isoformat(timespec='seconds')

        ds_id = 4
        dstream_url = 'http://localhost/frost/v1.1/Datastreams'
        filter_id = f"?$filter=id eq {ds_id}"
        filter_time = "&$expand=Observations($filter=phenomenonTime ge %s and phenomenonTime le %s;$orderby=phenomenonTime desc;$top=300;$select=result,phenomenonTime)&$select=description,id"% (start_time_str, now_time_str)
        url = dstream_url + filter_id + filter_time
        
        r = requests.get(url, auth=HTTPBasicAuth(auth_user, auth_pass))
        json_data = r.json()
        val = json_data['value'][0]
        setpt2use = process_obs(val)
        return setpt2use

        
    #=======================================================================================
    #=======================================================================================
    if __name__ == "__main__":
        while True:
            try:
                val2write = get_stapi_setpt_change()
                print(val2write)
                if val2write != None:
                    asyncio.run(write2bacnet_setpt(val2write))
            except Exception as e:
                print(f"Error reading BACnet: {e}")
            time.sleep(10)

```

12. Congrats you have successfully setup the digital twin dashboard.