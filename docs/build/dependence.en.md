## Setting up the Raspberry and Raspian Buster

Download a [Raspian Buster Lite](https://www.raspberrypi.org/downloads/raspbian/) system image and [flash](https://www.balena.io/etcher/) it on a microSD card (I'm using a fast 32 GB one, smaller is fine too).

Assuming that access via SSH & the local wifi network is required:

* Create an empty SSH file to enable [shell access](https://www.raspberrypi.org/documentation/remote-access/ssh/) (or hook up the Pi to a keyboard, mouse & display directly). 
* Create an [wpa_supplicant.conf](https://www.raspberrypi.org/documentation/configuration/wireless/headless.md) file with the correct credentials for the local wifi (or hook the Pi up to a LA).
* Check your router for the IP address of the Pi, and use this to log in using your favourite SSH client (Putty, etc)

Raspian standard username // password is: pi // raspberry

Update the system and install necessary system packages, PostgreSQL and the optional memcached package). Installing npm and nodejs here is probably redundant, but it looks like this works. Drop them at your own risk. 

```
sudo apt-get update -y
sudo apt-get full-upgrade -y
sudo apt-get install -y zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libgdbm-dev libncurses5-dev automake libtool bison libffi-dev git curl poppler-utils unrtf tesseract-ocr catdoc libxml2 libxml2-dev libxslt1-dev memcached postgresql postgresql-contrib libpq-dev libsass1 libsass-dev sassc npm nodejs
```

## Expand filesystem
Expand the filesystem to take full advantage of the size of SD card chosen;
Start **raspi-config** and go to **advanced options**, **expand filesystem**.  
Quit and reboot.


## Increase swap space to 4 GB
Not actually necessary, but possibly useful for the 2 GB board version (not recommended). Make sure to reverse the setting after the installation to avoid burning out the memory card by excessive read/write cycles.

```
sudo dphys-swapfile swapoff
sudo nano /etc/dphys-swapfile 
***find the CONF_SWAPSIZE and CONF_MAXSWAP line and change to 4096 or similar***
sudo dphys-swapfile swapon
```

## Set up user accounts

Create the openproject group/user. For the standard installation I set 'openproject' as the password.

```
sudo groupadd openproject
sudo useradd --create-home --gid openproject openproject
sudo passwd openproject #(***pick a password***)
```

Switch to the PostgreSQL system user and create the database user. The official documentation creates a user without privileges, the -sd flags will create a superuser with CREATEDB privileges.

```
sudo su - postgres
createuser -drsW openproject
```
Check PostgreSQL users and their privileges. If the CREATE DB privilege is missing, the installation will fail at a later point.
```
psql
\du
exit
```
The output should look like this:

```
postgres=# \du
                                    List of roles
  Role name  |                         Attributes                         | Member of
-------------+------------------------------------------------------------+-----------
 openproject | Superuser, Create role, Create DB                          | {}
 postgres    | Superuser, Create role, Creaboglte DB, Replication, Bypass RLS | {}

```

 
 Create the database and revert to the original user account:
 
 ```
 createdb -O openproject openproject
 exit
 ```
