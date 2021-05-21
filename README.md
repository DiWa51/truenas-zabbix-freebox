Zabbix Freebox Monitoring
=========================

Requirements
------------

You need Python 3.6 or above. All required packages is listed on requirements.txt
please type the following command to install all requirements:

```
apt install python3-pip
pip3 install -r requirements.txt
```

Install
-------

```bash
cd /usr/local/src
git clone git@github.com:GuillaumeHullin/freebox-monitoring.git
cd freebox-monitoring

# If you have only one WAN:
    cp fbx_monitor.py /usr/lib/zabbix/externalscripts/fbx_monitor.py

# If you have some kind of double-WAN network balancing:
    sed 's/mafreebox.freebox.fr/192.168.1.1/g' fbx_monitor.py > /usr/lib/zabbix/externalscripts/fbx_monitor.py

chmod +x /usr/lib/zabbix/externalscripts/fbx_monitor.py

# Create the token
python3 /usr/lib/zabbix/externalscripts/fbx_monitor.py authorize
# You will have to confirm the application access on Freebox front panel.

# Give permission to Zabbix to check the token
chown zabbix:zabbix /etc/xdg/freebox-monitoring/config.ini

```

Usage
-----
You can use the Zabbix Template in this repository or you can make your own template.
The `fbx_monitor.py` script let you get a json answer of Freebox API calls.
