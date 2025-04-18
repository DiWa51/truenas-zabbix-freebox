Zabbix Freebox Monitoring from a Truenas CORE jail
==================================================

Background
----------

Originally my project started with [FanaticalHelp script](https://github.com/FanaticalHelp/freebox-monitoring). I had hard time to run it into a Zabbix jail running in TrueNas. So this is an **How To install Zabbix freebox-monitoring in a Truenas Jail** to make it work _(for future generations, myself included)_.

_Disclaimer: I'm quite new to Zabbix and a little bit more experienced with Truenas._

Install
-------

### Requirements

As a **prerequisite**, prepare a TrueNas Core jail with **Zabbix Server** installed (v7.0.11 as of today).  I'm running **Truenas** v13.3. This is not the purpose of this script given there are multiple existing How-Tos over internet for this.

### Prepare the ground
In the jail change zabbix user (the user meant to run Zabbix) to use a /home/zabbix home folder:

```
root@zabbix% mkdir -p /home/zabbix
root@zabbix% chown zabbix:zabbix /home/zabbix
root@zabbix% pw user mod zabbix -d /home/zabbix
```

1st from root user in the zabbix jail, installed python3, pip3 & required packages as explained in the README.md, you need Python 3.6 or above.
```
root@zabbix% pkg install python
```
Login as zabbix user
```
root@zabbix% su - zabbix
zabbix@zabbix% python -m ensurepip --upgrade
```

### The setup
#### in the jail
All required packages are listed in requirements.txt. Type the following command to install all requirements:
```
zabbix@zabbix% git clone https://github.com/DiWa51/truenas-zabbix-freebox.git
zabbix@zabbix% cd truenas-zabbix-freebox
zabbix@zabbix% pip3 install -r requirements.txt
```

place the script fbx_monitor.py under your Zabbix externalscripts folder

```
root@zabbix% ln -s /home/zabbix/truenas-zabbix-freebox/fbx_monitor.py /usr/local/etc/zabbix7/zabbix/externalscripts/
root@zabbix% chown -h zabbix:zabbix /usr/local/etc/zabbix7/zabbix/externalscripts/fbx_monitor.py
root@zabbix% chmod -h 700 /usr/local/etc/zabbix7/zabbix/externalscripts/fbx_monitor.py
```
[-h, --no-dereference affect each symbolic link instead of any referenced file](https://unix.stackexchange.com/a/218559)

#### with the Freebox
Enable new application request on the Freebox webUI itself, then as zabbix user, run the script from the shell to authorize the app
```
root@zabbix% su zabbix
zabbix@zabbix% /usr/local/etc/zabbix7/zabbix/externalscripts/fbx_monitor.py authorize
```
You need to press **YES** on the box

File has finally been created on the filesystem

```
zabbix@zabbix% ls /home/zabbix/.config/freebox-monitoring
config.ini
 ```

Test things are now working:

```
zabbix@zabbix% fbx_monitor.py connection
{"type"= [...]}
```
#### Zabbix
Back to the Zabbix WebUI, things should now be running.

From the hosts list, I selected the freebox and then its items list.
There, I opened the "Freebox: Get connection data" entry.
From this page I use the **Test** button at the bottom; Make sure to "get value from host" (1st tick box), then press "Get values" you should get data in the Value field.

### Known issue

For some reasons, the items get
```
Preprocessing failed for: Require authentication..{"temp_sw": 39, "user_main_storage": "", "temp_cpu_cp_slave": 72, "mac": ...
1. Failed: cannot extract value from json by path "$.uptime_val": invalid object format, expected opening character '{' or '[' at: 'Require authentication.
{"temp_sw": 39, "user_main_storage": "", "temp_cpu_cp_slave": 72, "temp_cpu_cp_master": 72, ...
```
The "Require authentication" shouldn't be displayed as authentication has been done, and despite this the data is returned.