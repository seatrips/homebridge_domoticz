Homekit Siri
Jump to navigationJump to search
This wiki page describes the installation steps required to install Homebridge & Systemd (Raspbian, Ubuntu, Debian) with Siri support. For Docker or Synology DSM install click here. This enables you to talk to your apple iPhone or iPad to control your domoticz installation.

To manage Homebridge(Homekit) go to for example, http://192.168.1.20:9898. From here you can install, remove and update plugins, modify the Homebridge config.json and restart Homebridge.

Homebridge UserInterface



Contents
1	Update your system
2	Install Homebridge & Systemd
2.1	Step 1: Assume Root
2.2	Step 2: Install Node.js and Homebridge
2.3	Step 3: Create Homebridge Service User
2.4	Step 4: Create Homebridge Storage Directory
2.5	Step 5: Create Default config.json
2.6	Step 6: Create Systemd Service
2.7	Step 7: Fix Permissions
2.8	Step 8: Reload Systemd and Start Homebridge
2.9	Step 9: Manage and Configure Homebridge
2.10	Step 10: Adding Homebridge to iOS
3	Troubleshooting
4	Add Homebridge-Edomoticz plugin
5	Links
Update your system
Make sure your system is up-to-date:
```
 sudo apt-get update
 sudo apt-get upgrade
 ```
Install Homebridge & Systemd
Step 1: Assume Root
All the steps in this guide assume you are running as the root user.
```
 sudo su
 ```
Step 2: Install Node.js and Homebridge
Install the LTS version of Node.js from the official repository, as well as additional dependencies:
```
curl -sL https://deb.nodesource.com/setup_10.x | bash -
apt-get install -y nodejs gcc g++ make python
Test if node is working. This should display the version number of the installed node.js
```
```
 node -v
```
Informatie.jpgRaspberry Pi 1 and Zero Users, click here for alternative Node.js installation instructions.

Now install Homebridge and Homebridge Config UI X:
```
 npm install -g --unsafe-perm homebridge@latest homebridge-config-ui-x@latest
```
Step 3: Create Homebridge Service User
This is the user Homebridge will run under.
```
 useradd -m --system homebridge
```
This user will require password-less sudo permissions, to safely update your /etc/sudoers file run this command:
```
 echo 'homebridge    ALL=(ALL) NOPASSWD: ALL' | sudo EDITOR='tee -a' visudo
```
Step 4: Create Homebridge Storage Directory
This is where Homebridge will store all it's config and cache. Other plugins may also use this directory to store persistent data.
```
 mkdir -p /var/lib/homebridge
```
Step 5: Create Default config.json
Use the following command to create the default config.json file:

Copy and paste this entire block into the terminal as one command!
```
cat >/var/lib/homebridge/config.json <<EOL
{
    "bridge": {
        "name": "Homebridge",
        "username": "CB:22:3D:E2:CE:31",
        "port": 51826,
        "pin": "033-44-254"
    },
    "accessories": [],
    "platforms": [
        {
            "name": "Config",
            "port": 9898,
            "auth": "form",
            "theme": "teal",
            "restart": "sudo -n systemctl restart homebridge",
            "tempUnits": "c",
            "sudo": true,
            "log": {
                "method": "systemd"
            },
            "platform": "config"
        }
    ]
}
EOL
```
Step 6: Create Systemd Service
Use the command below to create the required /etc/systemd/system/homebridge.service file:

Copy and paste this entire block into the terminal as one command! Do not use a text editor. This template includes variables that get populated by bash as the command is executed.
```
cat >/etc/systemd/system/homebridge.service <<EOL
[Unit]
Description=Homebridge
After=syslog.target network-online.target

[Service]
Type=simple
User=homebridge
EnvironmentFile=/etc/default/homebridge
ExecStart=$(which homebridge) \$HOMEBRIDGE_OPTS
Restart=on-failure
RestartSec=3
KillMode=process
CapabilityBoundingSet=CAP_IPC_LOCK CAP_NET_ADMIN CAP_NET_BIND_SERVICE CAP_NET_RAW CAP_SETGID CAP_SETUID CAP_SYS_CHROOT CAP_CHOWN CAP_FOWNER CAP_DAC_OVERRIDE CAP_AUDIT_WRITE CAP_SYS_ADMIN
AmbientCapabilities=CAP_NET_RAW

[Install]
WantedBy=multi-user.target
EOL
```
Use the command below to create the required /etc/default/homebridge file:

Copy and paste this entire block into the terminal as one command! Do not use a text editor. This template includes variables that get populated by bash as the command is executed.
```
cat >/etc/default/homebridge <<EOL
```
# Defaults / Configuration options for homebridge
# The following settings tells homebridge where to find the config.json file and where to persist the data (i.e. pairing and others)

HOMEBRIDGE_OPTS=-U /var/lib/homebridge -I

# If you uncomment the following line, homebridge will log more
# You can display this via systemd's journalctl: journalctl -f -u homebridge
# DEBUG=*

# To enable web terminals via homebridge-config-ui-x uncomment the following line
# HOMEBRIDGE_CONFIG_UI_TERMINAL=1
EOL
Step 7: Fix Permissions
Ensure the homebridge user can access the storage directory:
```
 chown -R homebridge: /var/lib/homebridge
```
Step 8: Reload Systemd and Start Homebridge
This will ensure Homebridge starts on boot:
```
 systemctl daemon-reload
 systemctl enable homebridge
 systemctl start homebridge
```
Step 9: Manage and Configure Homebridge
To manage Homebridge go to http://<ip of server>:9898 in your browser. For example, http://192.168.1.20:9898. From here you can install, remove and update plugins, modify the Homebridge config.json and restart Homebridge.

The default username is admin with password admin. Remember you will need to restart Homebridge to apply any changes you make to the config.json.

Step 10: Adding Homebridge to iOS
Use the Home app (or most other HomeKit apps), you should now be able to add the single accessory "Homebridge", assuming that you're still running Homebridge and you're on the same Wifi network. When you attempt to add Homebridge, scan the QR code in Homebridge Status Screen or add with PIN code. The default code is 031-45-154 (but this can be changed in config). Adding this accessory will automatically add all accessories and platforms defined in config.

Troubleshooting
The Homebridge logs can be viewed using the web interface, but if cant open config ui or you need to view them via the terminal run:
```
 sudo journalctl -f -u homebridge
```
If you get this error in log: Error: Cannot add a bridged Accessory with the same UUID as another bridged Accessory:. Try delete cached accessories folder.
```
 rm -r /var/lib/homebridge/accessories
```
Add Homebridge-Edomoticz plugin
Go to http://<ip of server>:9898 in your browser. For example, http://192.168.1.20:9898.

Click on plugins
Search for edomoticz
Click Install in Homebridge Edomoticz container
Click on Config
Edit config to match your settings:
```
{
    "bridge": {
        "name": "Homebridge",
        "username": "CB:22:3D:E2:CE:31",
        "port": 51826,
        "pin": "031-45-154"
    },
    "accessories": [],
    "platforms": [
        {
            "name": "Config",
            "port": 9898,
            "auth": "form",
            "theme": "teal",
            "restart": "sudo -n systemctl restart homebridge",
            "tempUnits": "c",
            "sudo": true,
            "log": {
                "method": "systemd"
            },
            "platform": "config"
        },
        {
            "platform": "eDomoticz",
            "name": "eDomoticz",
            "server": "127.0.0.1",
            "port": "8080",
            "ssl": false,
            "roomid": 2,
            "mqtt": true,
            "excludedDevices": []
        }
    ]
}
```
Click Save then Restart Homebridge
For more additional settings or help visit eDomoticz or forum thread
