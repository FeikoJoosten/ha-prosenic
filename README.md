[![License][license-shield]](LICENSE.md) ![Project Maintenance][maintenance-shield]

[![hacs][hacsbadge]][hacs]

# Prosenic Home Assistant component
A full featured Homeassistant component to control Prosenic vacuum cleaner locally without the cloud. 
This component is based on the underlying PyTuya library available [here][pytuya].

## Towards Homeassistant official integration
My personal goal is to make this component fully compliant with Homeassistant, so 
that it may be added as the official library to handle Prosenic vacuum cleaners. 
However, before pushing a PullRequest to the official Homeassistant repository, I would like to share it to some users.
In this way we can test it massively, check it for any bug and make it **robust enough** to be seamlessly integrated 
with Homeassistant.

For now, the component has been integrated as a custom component into [HACS][hacs].

## Installation
You can install this component in two ways: via HACS or manually.
HACS is a nice community-maintained components manager, which allows you to install git-hub hosted components in a few clicks.
If you have already HACS installed on your HomeAssistant, it's better to go with that.
On the other hand, if you don't have HACS installed or if you don't plan to install it, then you can use manual installation.

## Option A: Installing via HACS
Search for "Prosenic Vacuum Cleaner" and click on Install: when done, proceed with component setup.

## Option B: Classic installation (custom_component)
1. Download the latest zip release archive from [here][release-zip] (or clone the git master branch)
1. Unzip/copy the prosenic direcotry within the `custom_components` directory of your homeassistant installation.
The `custom_components` directory resides within your homeassistant configuration directory.
In other words, the configuration directory of homeassistant is where the configuration.yaml file is located.
After a correct installation, your configuration directory should look like the following.
    ```
    └── ...
    └── configuration.yaml
    └── secrects.yaml
    └── custom_components
        └── prosenic
            └── __init__.py
            └── const.py
            └── ...
    ```

    **Note**: if the custom_components directory does not exist, you need to create it.

## Component setup    
Once the component has been installed, you need to configure it in order to make it work.
First we need to find out the _device_id_, _ip address_ and _local key_ of your prosenic vacuum cleaner.
Todo that you have multiple options:
 * Use *mitmproxy* to intercept the connection between the prosenic app and the tuya cloud (Described below and I used this approach)
 * Read the following [link][tuyaapi-docs]
     
### Extracting the required data with mitmproxy.

1. First we need to setup mitmproxy. The easiest way is to setup a docker container. Other installation type can be found [here][mitmproxy-install]
    Below the docker-compose script, which will setup mitmproxy on port 8080 with the web interface on port 8081
    ```yaml
    version: "3.3"
    services:
      mitmweb:
        ports:
            - "8080:8080"
            - "127.0.0.1:8081:8081"
        image: mitmproxy/mitmproxy
        entrypoint: "mitmweb --web-iface 0.0.0.0"
    ```
1. 
    * If you have an iPhone or iPad, use please one of these device as the setup is easier there.
    Setup the proxy to point to your docker container. A tutorial can be found [here][ios-proxy].
    Afterwards go to _mitm.it_ and install the certificate. The tutorial for this step can be found [here][mitmproxy-certs]
    * If yoi have a rooted Android device, then you can install the certificate nearly the same way as iOS. Only be sure you install it as System Certificate.
    * If you have a "normal" (not rooted) Android device, please refer to this tutorial [here][mitmproxy-android]
    
1. Open the prosenic app and refresh all your devices
1. On the computer, where your mitmproxy docker container is running, open  the following link [http://localhost:8081](http://localhost:8081)
1. There you should see some request to the tuya cloud. Depending on your region, you should see request to tuyaeu.com or similar.
Click on the first of these requests and a detail view will be open on the right hand sight. Go to the response and search for _localKey_
You should get similar result as the follwoing:
```json
...
 "result": [{
            "virtual": false,
            "dpName": {},
            "lon": "0",
            "uuid": "REMOVED_FOR_PRIVACY",
            "mac": "REMOVED_FOR_PRIVACY",
            "iconUrl": "https://images.tuyaeu.com/smart/product_icon/sd.png",
            "lat": "0",
            "runtimeEnv": "prod",
            "devId": "YOUR_DEVICE_ID",
            "productId": "REMOVED_FOR_PRIVACY",
            "dps": {
...
            },
            "activeTime": 1580185647,
            "ip": "REMOVED_FOR_PRIVACY",
            "categoryCode": "wf_sd",
            "moduleMap": {
...
            },
            "devAttribute": 3,
            "name": "Cocosmart 820T",
            "timezoneId": "Europe/Rome",
            "category": "sd",
            "localKey": "YOUR_LOCAL_KEY"
        }]
...
```
You will need the _localKey_ and the _devId_ (device id).

### Find out the local ip address of your vacuum robot

There are multiple ways to find out the local ip address of your vacuum robot:
* Login into your router or dhcp server and check there for the assigned ip address
* Run nmap to find all devices in your network.
The Prosenic (Cocosmart) 820T has the open port 6668. 
So we can search for devices, which have open this port with the following script. Please adjust the command, that nmap is searching your network.
```shell script
nmap -p 6668 192.168.0.0/24 --open
```

### Configuration via editing configuration.yaml

1. Enable the component by editing the configuration.yaml file (within the config directory as well).
Edit it by adding the following lines:
    ```yaml
    vacuum:
      - platform: prosenic
        host: YOUR_IP
        device_id: YOUR_DEVICE_ID
        local_key: YOUR_KEY
        remember_fan_speed: false #Optional, default false
        enable_debug: false #Optional, default false
    ```
    **Note!** If you have already configured other vacuum robot, add your configuration there.

1. Reboot hassio
1. Congrats! You're all set!

## Additional Information

Currently this integration is only tested with a Prosenic (Cocosmart) 820T, because I only have this one.
@gibranZawahra confirmed that also the Model 830 is working.
Please give me feedback, if it works with other models too.

The integration is communicating locally only, so you can block the access of your vacuum robot to the internet.

If you find a problem/bug or you have a feature request, please open an issue.


## What's next?
- Better error handling
- Automated test

[hacs]: https://hacs.xyz
[hacsbadge]: https://img.shields.io/badge/HACS-CUSTOM-inactive?style=for-the-badge
[license-shield]: https://img.shields.io/github/license/FeikoJoosten/ha-prosenic?style=for-the-badge
[maintenance-shield]: https://img.shields.io/badge/Maintainer-Feiko%20Joosten-blue?style=for-the-badge
[pytuya]: https://github.com/clach04/python-tuya
[tuyaapi-docs]: https://github.com/codetheweb/tuyapi/blob/master/docs/SETUP.md
[release-zip]: https://github.com/FeikoJoosten/ha-prosenic/releases/latest/download/prosenic.zip

[mitmproxy-install]: https://docs.mitmproxy.org/stable/overview-installation/
[mitmproxy-certs]: https://docs.mitmproxy.org/stable/concepts-certificates/
[mitmproxy-android]: https://docs.mitmproxy.org/stable/howto-install-system-trusted-ca-android/
[ios-proxy]: http://www.iphonehacks.com/2017/02/how-to-configure-use-proxy-iphone-ipad.html
