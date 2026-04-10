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