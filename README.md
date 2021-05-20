Zabbix Freebox Monitoring
=========================

Install
-------

```
cd /usr/local/src
git clone git@github.com:GuillaumeHullin/freebox-monitoring.git
cd freebox-monitoring
cp fbx_monitor.py /usr/lib/zabbix/externalscripts/fbx_monitor.py
chmod +x /usr/lib/zabbix/externalscripts/fbx_monitor.py
```

You might need to overide the variable **api_url** by replacing *mafreebox.freebox.fr* with your Freebox IP address, in case of a double WAN setup.

Requirements
------------

You need Python 3.6 or above. All required packages is listed on requirements.txt
please type the following command to install all requirements:

```
apt install python3-pip
pip3 install -r requirements.txt
```


Setup
-----

The script must be authorized on the Livebox. Type the following command:

```
python3 ./fbx_monitor.py authorize
```

You will have to confirm the application access on Freebox front panel.


Usage
-----
You can use the Zabbix Template in this repository or you can make your own template.
The `fbx_monitor.py` script let you get a json answer of Freebox API calls.
