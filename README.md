![[Version 0.5.1~beta](https://github.com/trhacknon)](http://img.shields.io/badge/version-0.5.1~beta-orange.svg)
![[Python 3](https://github.com/trhacknon)](http://img.shields.io/badge/python-3-blue.svg)
![[GPL-3.0 License](https://github.com/trhacknon)](https://img.shields.io/badge/license-GPL%203.0-brightgreen.svg)
![[Date](https://github.com/r3nt0n)](http://img.shields.io/badge/date-2022-yellow.svg)


# zombiegang

<div align="center"><img src="https://github.com/trhacknon/zombiegang/blob/master/img/intro.gif" /></div>
<br>  

Zombiegang is a botnet written mostly in Python and PHP. It supports **asynchronous communication between cc and zombies**, **remote-shell** live sessions and **task scheduler**. It also has a **plugin manager**, which comes with some modules pre-included to perform most typical attacks (**DDoS, bruteforce** and **keylogger**). This modular approach allows anyone to **extend features by writing his own modules** (I will appreciate any contribution).  

<br>
<div align="center"><img src="https://github.com/trhacknon/zombiegang/blob/master/img/intro2.gif" /></div>  
<br>

The **Command and Control server** is a semi-CRUD API written in php, which manages database read/write operations and authentication. This schema also allows to separate the front-end, which resides entirely in the client used by masters. 

Several kind of clients could be used to admin the botnet, and several kind of "zombie-clients" could co-exists too.
+ **master clients:** cli and web-based. The webclient is a light flask app focused on browse db info and schedule tasks. The cli client is intended to run remote-shell live sessions with one or more zombies simultaneously. Both of them support proxy configuration to reach cc-server anonymously.
+ **zombie clients:** by now, we only have a python client. Take note that you can write a zombie in the programming language of your preference, you just need to write a simple http client to communicate with API and maybe add some "zombie routines" (you can take the python client as an example). Again, any contribution would be welcome.   


Having a centralized db makes it easier for masters and zombies to exchange information asynchronously, removing the requirement of both being online at the same time.

You can schedule tasks and the zombies will receive this info as soon as they go online and refresh his "assignments". If the task was scheduled to be executed in future, the zombie will save this homework and run the task when the start time comes. You also can schedule stop datetimes.

<br>
<div align="center"><img src="https://github.com/trhacknon/zombiegang/blob/master/img/attack_example.png" /></div>
<br>  
  
There are special fields in DB which are designed to be nested values, so you can create new fields inside without touching any config (e.g.: `Tasks.task_content`,`Zombies.sysinfo`)  
  
<br>
<div align="center"><img src="https://github.com/r3nt0n/zombiegang/blob/master/img/zombies_info.gif" /><p style="font-decoration: italic;">zombie reports</p></div>  
<br>

## Get started
**Note:** This is just a simple way to kickstart all the initial stuff. Obviously, in production environments you can use separate servers for DB and CC, and replace the http server for something like Apache or Nginx.

**IMPORTANT NOTE:** zombiegang is still on development phase, some features wasn't tested under all possible scenarios yet. Any bug reported could be helpful.

### Download zombiegang framework
```
git clone https://github.com/trhacknon/zombiegang
```
  
### Create database
 
```
sudo apt-get install mariadb-server, mariadb-client
sudo mysql_secure_installation
cd cc-server
# you should change db default password here:
nano api/config/data/init.sql
sudo ./initdb
```
**Create your master profile**
```
mariadb -u zgang -p
use zgang;
insert into Masters SET username = '<username>', public_key = '<public-key>';
exit
```
***Note:** By now, you shouldn't specify any `password`, we will create it later. As another note, the `public_key` can be an empty string, since PKI logic isn't implemented yet.*
  
*Optional*: if you want to dump some mocked zombies into db for testing purposes:
```
./dump-testdata
```

### Start cc-server

```
cd cc-server
# Edit this file to match your db config
nano api/config/database.php
# and create your own secret-key
nano api/config/core.php 

# For testing purposes, you can use the simple http server provided by php
sudo php -S 127.0.0.1:8080
```

Now you should have the cc-server listening on port 8080 and connected to the database created before.

***Note:** You can disable masters access logging in `core.php`*


### Zombie clients

*On the victim machine:*
```
# install zombieclient dependencies 
cd zombie-client
pip install -r requirements.txt 

# to run the zombieclient
python3 run.py
```

Zombies will send info about themselves on every boot and check regularly for new tasks, they just keep asking and serving to the cc-server forever (in fact, until you kill this process).

In real scenarios, you will also need persistence, obfuscation and probably compilation (since Python is not available by default on most systems). 

### Master clients

*On the attacker machine:*

  
  
#### web-client
```
# install masterclient (web-client) dependencies 
cd master-client
pip install -r requirements.txt 

# to run the masterclient (web-client)
./run.sh
```

Now you should have a Flask app running and listening on port 5000. Browse to http://localhost:5000 and check it.

Once inside, you will see something like a desktop. You can **enable/disable proxy configuration** and **login to the botnet** with the aproppiate software (`proxy.exe` and `zgang.exe`).

On this stage you are going to create your master password: with `zgang.exe`, create a user with the same name used in your master profile. Now you are logged in as master and can start to admin the botnet.

**Note**: Mozilla Firefox is the recommended browser, any other could work but won't be officially supported. Some visual features (e.g.: emojis, form elements...) could vary across different browsers.

<br>
<div align="center"><img src="https://github.com/trhacknon/zombiegang/blob/master/img/menu.png" /></div>
<br>

If you want to cover your trace, use the built-in proxy tool to connect to cc-server through the socks5 proxy of your choice:

<br>
<div align="center"><img src="https://github.com/trhacknon/zombiegang/blob/master/img/proxy_example.png" /><p style="font-decoration: italic;">proxy configuration example</p></div>
<br>

#### cli-client
Additionally, you have a cli client (keeping msfconsole style) to login to cc-server and run remote-shell live sessions with online zombies, you could also connect through a socks5 proxy (like in web-based client) setting `PXHOST` and `PXPORT` before `login`.
```
# to run the masterclient (cli)
python3 cli.py
```

<div align="center"><img src="https://github.com/trhacknon/zombiegang/blob/master/img/master-client_cli_live_session_example.png" /><p style="font-decoration: italic;">simple cli live session example</p></div>
<br>


## Modules and custom attacks

+ `dos`: ddos attacks, implemented and working, by now just one type (`dos/slowloris`)
+ `brt`: brute force attacks, to be implemented (by now, just an example template)
+ `rsh`: execute remote commands

By his nature, `keylogger module` is an special task and doesn't inherit from the base class `Task`. By now, logic to log keypresses and create logs into `cc-server` is implemented in `zombie-client` (windows and linux systems).  To be implemented:

  + logic to start/stop keylogger remotely
  + logic to show logs created by zombies on the master-client side

[Working on a comprehensive explanation about how to build custom attacks and contribute to app with new modules]


## TO-DO (NEXT)
+ PKI authentication not implemented.
+ Write docs about how to create custom modules/attacks.
+ Task schedulers stop points (manual and auto) not working yet.
+ Tasks details (master-client) not showing yet.
+ Keylogger on/off not working yet.
+ Keylogger logs not showing yet.


## Legal disclaimer
This is a personal project, and is created for the sole purpose of security awareness and education, it should not be used against systems that you do not have permission to test/attack. The author is not responsible for misuse or for any damage that you may cause. You agree that you use this software at your own risk. I don't own the rights of any image included, is just a funny tribute to some iconic legends (if you are the owner of any picture and want it to be removed, please contact me and I will do as soon as posible). You can't distribute this app with commercial purposes.


## References
+ CSS sheet is based on the work of <a href="https://github.com/trhacknon/">Trhackno </a> (<a href="https://github.com/jdan/98.css/blob/master/LICENSE">css win98 repo</a>)
+ Age of Empires icons found <a href="https://www.forgottenempires.net/age-of-empires-ii-definitive-edition/campaigns">here</a>
+ mIRC icon designed by <a href="https://www.flaticon.es/autores/pixel-perfect" title="Pixel perfect">Pixel perfect</a> from <a href="https://www.flaticon.es/" title="Flaticon"> www.flaticon.es  </a>
+ All pictures were found on the Internet


