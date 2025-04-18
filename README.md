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
Back to the Zabbix WebUI, things are still not yet working.

From the hosts list, I selected the freebox and then its items list.
There, I opened the "Template Freebox: Freebox connection" entry.
From this page I use the **Test** button at the bottom; Make sure to "get value from host" (1st tick box)

1st, for some unknown reasons, the script returns an unknown Python3 error _(it is working in the shell!)_.

`... env: python3: No such file or directory`

I fixed it by changing the shebang in fbx_monitor.py header to hardcoded absolute path to python3

`#!/usr/local/bin/python3`

2nd, another error is thrown stating the app is not authorized _(even though it was from the shell...)_
I worked it around by running the _authorize_ step from the UI, using again the Test feature:

- I replaced the _connection_ parameter by _authorize_ then test.
- The Freebox asks to validate the app but even though I pressed YES "instantly", the UI throws a timeout exception
-- This was fixed by increasing the timeout entry in /usr/local/etc/zabbix7/zabbix-server.conf to a greater value (arbitrarily 24 instead of 4)
-- next issue is a permission denied to create a /.cache folder
-- Fixed by manually creating /.cache/fbx-Zabbox folders (yes from / !!!)
`mkdir -p /.cache/fbx-Zabbox`

Finally the **script is running**, _hooray_, I have data from my Freebox into Zabbix.

### Now comes the remaining issues:

Why does it need to create the .cache under / directly? Why doesn't it use the /home/zabbix/.cache folder? i.e. the user running the zabbix processes
Why can't it use the env setup for python3 in the shebang?
All in all it seems the zabbix user in the shell is not behaving as the zabbix user from the WebUI. WHY?

Thanks a lot for your lights

### Original README

```bash
cd /usr/local/src
git clone git@github.com:Futur-Tech/futur-tech-zabbix-freebox.git
cd futur-tech-zabbix-freebox

# If you have only one WAN:
cp fbx_monitor.py /usr/lib/zabbix/externalscripts/fbx_monitor.py

# If you have some kind of double-WAN network balancing:
sed 's/mafreebox.freebox.fr/192.168.1.1/g' fbx_monitor.py > /usr/lib/zabbix/externalscripts/fbx_monitor.py

apt install python3-pip python3-requests python3-appdirs
chmod +x /usr/lib/zabbix/externalscripts/fbx_monitor.py

# Create the token
python3 /usr/lib/zabbix/externalscripts/fbx_monitor.py authorize
# You will have to confirm the application access on Freebox front panel.

# Give permission to Zabbix to check the token
chown zabbix:zabbix /etc/xdg/freebox-monitoring/config.ini

```
You can use the Zabbix Template in this repository or you can make your own template.
The `fbx_monitor.py` script let you get a json answer of Freebox API calls.

