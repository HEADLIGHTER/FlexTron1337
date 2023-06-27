# FlexTron1337 Debian Server (!WIP!)
**Ultimate self-hosted server solution for maximum flexibility in small-team workflow and development.**
![IMG_5165.png](imgs%2FIMG_5165.png)
## Hardware
>   * Intel(R) Xeon(R) CPU E5-2650 v2 @ 2.60GHz
>   * 16 GB DDR3 1600 MT/s RAM
>   * 2 x 1.0 TB 2.5"HDD 5400 RPM
>   * GeForce 8500 GT 256 MB VRAM
>   * 650W Power Supply 
>   * 650VA/390W(!) AVR UPS for server and fiber optic router rated for an hour of battery life

**! It's ok that UPS wattage is lower than power supply itself. Server consumes about 200 watts at full load.**

## Software
>   * OS: Debian
>   * FireWall: UFW
>   * VCS: Gogs
>   * VCS DB: PostgreSQL
>   * File Server: SFTPGo
>   * VPN: WireGuard

**I've also created second local 192.168.2.0/24 subnet on router to avoid conflicts with other users local networks.**

## Table of Contents
* [STEP 0 : BIOS / VM](#0)  
* [STEP 1 : Setup Debian](#1)
* [STEP 2 : Configuring server](#2)
* [STEP 3 : Install and configure SFTPGo](#3)
* [STEP 4 : Install and configure Gogs VCS](#4)
* [STEP 5 : Install and configure WireGuard](#5)
<a name="headers"/>

### STEP <a name=0>0</a> : BIOS / VM
 !Your BIOS should support EFI! Set AHCI in bios if you turn RAID instead, I'm using program RAID1 by Debian. Also set Enable Efi in system settings if you are running on VM.
### STEP <a name=1>1</a> : Setup Debian
* Install (not graphical)
* Language - English
* Territorial settings of your choice -> American English keyboard layout
* Hostname -> Domain empty -> Root password twice -> Full username -> Login name -> Password -> Time Zone
* Partitioning method: Manual. 
* **Pre-partition setup** (You can set your own sizes for partitions):
>   * Required 2 similar at speed and capacity HDDs for RAID1 configuration, I was lucky to have two absolutely identical 1.0 TB disks. Next steps should be repeated on both HDD's 
>   * Create new empty partition table on both HDD (SCSIX or something...)
>   * Select created FREE SPACE (SCFS next)-> Create a new partition (CRANP next) -> 500m -> Primary -> Beginning -> Mount Point  (MP) -> EFI System partition -> 
>   * -> Done setting up the partition (DSUP)
>   * SCFS -> CRANP -> 225G -> Beginning -> ext4 or btrfs file system (or RAID device, but it's safer to make full pre-partition before RAID setup) -> 
>   * -> MP -> / root -> DSUP
>   * SCFS -> CRANP -> 225G -> Beginning -> ext4/btrfs/raid -> MP -> /home -> DSUP
>   * SCFS -> CRANP -> 90G -> Beginning -> ext4/btrfs/raid -> MP -> /tmp -> DSUP
>   * SCFS -> CRANP -> 225G -> Beginning -> ext4/btrfs/raid -> MP -> /var -> DSUP
>   * SCFS -> CRANP -> 225G -> Beginning -> ext4/btrfs/raid -> MP -> /srv -> DSUP
>   * SCFS -> CRANP -> 9.7G (or all remaining disk space) -> SWAP area -> DSUP
>   * Check order and capacity of partitions on both HDD's before next steps, they must be exactly the same.
>   * VM Partitions example (may differ): 
>   * ![Screenshot_20230521_170433.png](imgs%2FScreenshot_20230521_170433.png)
* **-> Configure software RAID ->** 
>   * Write the changes to the sorage devices and configure RAID? -> Yes
>   * Create MD device (CMDD) -> RAID1 -> Number of active devices... (NOAD) -> 2 -> Number of spare devices... (NOSD) -> 0 -> Active devices... (AD) -> 
>   * -> Set two /dev/sdaN and /dev/sdbN with same number, but don't add the ESP(Efi partition) in raid, only your ext4/btrfs and swap partitions. ->
>   * -> /dev/sda2 and /dev/sdb2 for this one. -> Write the changes... (WTC) -> Yes
>   * CMDD -> RAID1 -> NOAD -> 2 NOSD -> 0 -> AD -> /dev/sda3 and /dev/sdb3 -> WTC -> Yes
>   * CMDD -> RAID1 -> NOAD -> 2 NOSD -> 0 -> AD -> /dev/sda4 and /dev/sdb4 -> WTC -> Yes
>   * CMDD -> RAID1 -> NOAD -> 2 NOSD -> 0 -> AD -> /dev/sda5 and /dev/sdb5 -> WTC -> Yes
>   * CMDD -> RAID1 -> NOAD -> 2 NOSD -> 0 -> AD -> /dev/sda6 and /dev/sdb6 -> WTC -> Yes
>   * CMDD -> RAID1 -> NOAD -> 2 NOSD -> 0 -> AD -> /dev/sda6 and /dev/sdb6 -> WTC -> Yes
>   * CMDD -> RAID1 -> NOAD -> 2 NOSD -> 0 -> AD -> /dev/sda7 and /dev/sdb7 -> WTC -> Yes
>   * -> Finish. You should have 6 RAID1 devices now from #0 to #5 under "Configure" section 
>   * Example:
>   * ![Screenshot_20230521_180433.png](imgs%2FScreenshot_20230521_180433.png)
* **Create partition on RAID devices:** 
>   * RAID1 device #0 (RAID1_N)-> Use as (UA) -> Ext4 -> MP -> / -> DSUP
>   * RAID1_1 -> UA -> ext4/btrfs -> MP -> /home -> DSUP
>   * RAID1_2 -> UA -> ext4/btrfs -> MP -> /tmp -> DSUP
>   * RAID1_3 -> UA -> ext4/btrfs -> MP -> /var -> DSUP
>   * RAID1_4 -> UA -> ext4/btrfs -> MP -> /srv -> DSUP
>   * RAID1_3 -> UA -> SWAP area -> DSUP 
>   * Example:
>   * ![Screenshot_20230521_181401.png](imgs%2FScreenshot_20230521_181401.png)
* -> Finish partitioning and write changes to disk -> Yes -> Wait for installing the base system.
*  Scanning your installation media... -> No -> Debian archive mirror country -> Your country -> deb.debian.org -> Proxy empty -> 
* -> Wait for confuguring APT and software installing.
* Participate in the package usage survey -> NO
* Select only SSH server and standard server utilities in Software selection -> Wait for software  and GRUB install.
* Remove installation media and -> Continue
### STEP <a name=2>2</a> : Configuring server
* Enter login and password
* user@hostname:~$ ```ip addr```
* Find your local ip address, then connect via SSH
* pcuser@pcname:~$ ```ssh user@server.local.ip.address``` 
* From your PC terminal/cmd (not server, yes). If using VM, look for port forwarding in your VM settings.
* root@hostname:~$ ```su -``` 
* root@hostname:~$ ```apt install sudo```
* root@hostname:~$ ```sudo adduser <user> sudo```
* root@hostname:~$ ```reboot``` 
* Repeat ssh connection and login procedure -> ```su -```
* **Confiure EFI partition on second HDD:** 
>   * root@hostname:~$ ```lsblk -o +uuid```
>   * Output example in VM:
>   * ![Screenshot_20230521_194054.png](imgs%2FScreenshot_20230521_194054.png)
>   * as you see, we have same UUID's for pairs of sdaN/sdbN and mdN partition except sda1 and sdb1. We need to set them same too to prevent 
>   * emptiness on boot screen if HDD1 fail. GRUB currently only on sda1 and needed to be installed on sdb1 too:
>   * root@hostname:~$ ```apt install dosfstools```
>   * root@hostname:~$ ```mkdosfs -i F1563802 /dev/sdb1``` (yep, set sda1 uuid without '-' on sdb1)
>   * root@hostname:~$ ```efibootmgr -c -d /dev/sdb -p 1 -L "debian 2" -l "\EFI\debian\shimx64.efi"```
>   * root@hostname:~$ ```efibootmgr -v```
>   * and you will see something like this:
>   * ![Screenshot_20230521_203442.png](imgs%2FScreenshot_20230521_203442.png)
>   * if not, remove it with 
>   * root@hostname:~$ ```efibootmgr -b 6 -B```
>   * where 6 is Boot000N number of your created device
>   * root@hostname:~$ ```mount /dev/sdb1 /mnt```
>   * root@hostname:~$ ```cp -R /boot/efi/EFI/ /mnt```
>   * root@hostname:~$ ```umount /dev/sdb1```
>   * root@hostname:~$ ```reboot```
>   * root@hostname:~$ ```lsblk -o +uuid```
>   * VM output:
>   * ![Screenshot_20230521_204705.png](imgs%2FScreenshot_20230521_204705.png)
>   * My physical server output:
>   * ![Screenshot_20230531_013814.png](imgs%2FScreenshot_20230531_013814.png)
* You can check if RAID really working by unplugging one of your disks from server/VM
* **Configure static IP address for server:**
>   * ```root@hostname:~$ nano /etc/network/interfaces```
* see:
>   * ![Screenshot_20230529_002155.png](imgs%2FScreenshot_20230529_002155.png)
* change it to:
```console
# The primary network interface
allow-hotplug enp0s3    # change enp0s3 to your primary newtwork interface name
iface enp0s3 inet static    # same here
address new.static.ip.address   # 192.168.2.137 for me
netmask 255.255.255.0   # your netmask 
gateway your.router.ip.address  # 192.168.2.254 for me
dns-nameservers router.dns.ip.address 8.8.8.8 1.1.1.1   # 192.168.2.254 for me
```
### **__!Your SSH session will disconnect after next step, make sure you sat everything right or have physical access to server if something wrong!__**
>   * root@hostname:~$ ```systemctl restart networking.service```
>   * Reconnect SSH with your new static ip address
* __You can also reserve static address for your server in router admin panel. I did it on both server and router to avoid any conflicts in the future__
* **Firewall setup:**
>   * user@hostname:~$ ```sudo apt install ufw```
>   * user@hostname:~$ ```sudo ufw allow 22```       for SSH
>   * user@hostname:~$ ```sudo ufw allow 80```       for HTTP
>   * user@hostname:~$ ```sudo ufw allow 443```      for HTTPS
>   * user@hostname:~$ ```sudo ufw allow 53```       for DNS
>   * user@hostname:~$ ```sudo ufw allow 8080```     for alternate HTTP on SFTPGo
>   * user@hostname:~$ ```sudo ufw allow 8090```     for SFTPGo WebDav
>   * user@hostname:~$ ```sudo ufw allow 2022```     for SFTP
>   * user@hostname:~$ ```sudo ufw allow 3000```     for Gogs
>   * user@hostname:~$ ```sudo ufw allow 13370```    for WireGuard
>   * user@hostname:~$ ```sudo ufw enable```
>   * user@hostname:~$ ```sudo ufw status```
* **Install other necessary software:**
>   * user@hostname:~$ ```sudo apt install gnupg```
>   * user@hostname:~$ ```sudo apt install software-properties-common```
>   * user@hostname:~$ ```sudo apt install curl```
>   * user@hostname:~$ ```sudo apt install git```
>   * user@hostname:~$ ```sudo apt install hddtemp```
>   * user@hostname:~$ ```sudo apt install net-tools```
* 
__All initial preparations done, time to set up services__

### STEP <a name=3>3</a> : Install and configure SFTPGo
* user@hostname:~$ ```curl -sS https://ftp.osuosl.org/pub/sftpgo/apt/gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/sftpgo-archive-keyring.gpg```
* user@hostname:~$ ```CODENAME=`lsb_release -c -s````
* user@hostname:~$ ``` 
echo "deb [signed-by=/usr/share/keyrings/sftpgo-archive-keyring.gpg] https://ftp.osuosl.org/pub/sftpgo/apt ${CODENAME} main" | sudo tee /etc/apt/sources.list.d/sftpgo.list
                   ```
* user@hostname:~$ ```sudo apt update```
* user@hostname:~$ ```sudo apt install sftpgo```
* user@hostname:~$ ```sudo systemctl enable --now sftpgo```
* user@hostname:~$ ```systemctl status sftpgo``` - should be active and running
* for SFTP and WebDav over https we need some SSL certificates. Use your certificates or generate self-signed:
* user@hostname:~$ ```sudo openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout /etc/sftpgo.pem -out /etc/sftpgo/sftpgo.pem```
* provide required details for certificate and proceed
* user@hostname:~$ ```sudo chmod 544 /etc/sftpgo/sftpgo.pem```
* user@hostname:~$ ```sudo chmod 544 /etc/sftpgo.pem```
* user@hostname:~$ ```sudo nano /etc/sftpgo/sftpgo.json```
* ^+W and find "httpd"
* Replace "enable_https": false to true
* ![Screenshot_20230602_232119.png](imgs%2FScreenshot_20230602_232119.png)
* Ignore httpd "certificate_file" and "certificate_key_file", go to ~330 line and set your
* "certificate_file" as "/etc/sftpgo/sftpgo.pem" and "certificate_key_file" as "/etc/sftpgo.pem"
* ![Screenshot_20230602_232822.png](imgs%2FScreenshot_20230602_232822.png)
* __Optional:__  
>   * If you want to use WebDav go to "webdavd" in /etc/sftpgo/sftpgo.json and set 8090 port for it, 
>   * enable https and set your previously created certificates as in httpd
* user@hostname:~$ ```sudo systemctl restart sftpgo```
* user@hostname:~$ ```sudo mkdir /srv/sftpgo/<files>``` I will use "files" directory, you can create your own
* From your browser on PC go too https://your.server.ip.address:8080
* See next message if using self-signed certificates:
* ![Screenshot_20230603_002904.png](imgs%2FScreenshot_20230603_002904.png)
* Click on Advanced -> Proceed:
* ![Screenshot_20230603_003009.png](imgs%2FScreenshot_20230603_003009.png)
* In admin panel create new Folder with path to ```/srv/sftpgo/<files>``` :
* ![Screenshot_20230603_030134.png](imgs%2FScreenshot_20230603_030134.png)
* Create new Group with access to created folder: 
* ![Screenshot_20230603_004245.png](imgs%2FScreenshot_20230603_004245.png)
* And set permissions to this folder in ACLs section below:
* ![Screenshot_20230603_005432.png](imgs%2FScreenshot_20230603_005432.png)
* I've set \*. If you can't trust your users, set permissions that fits their needs
* Create user with your group as primary group and path to users home directory for personal files:
* ![Screenshot_20230603_004949.png](imgs%2FScreenshot_20230603_004949.png)
* Done! Open https://your.server.ip.address:8080 again and login into created user. Press WebAdmin if you want to log in with admin account.
* ![Screenshot_20230603_031524.png](imgs%2FScreenshot_20230603_031524.png)
### STEP <a name=4>4</a> : Install and configure Gogs VCS
* Gogs need go to compile and some of MySQL/PostgreSQL DB. I'm going to use postgres.
* **PostgreSQl setup:**
>   * user@hostname:~$ <code>sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'</code>
>   * user@hostname:~$ ```wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -```
>   * user@hostname:~$ ```sudo apt update```
>   * user@hostname:~$ ```sudo apt install postgresql```
* **GO setup:**
>   * user@hostname:~$ ```curl -O https://dl.google.com/go/go1.20.4.linux-amd64.tar.gz``` in ~ directory
>   * user@hostname:~$ ```sudo tar -C /usr/local/ -xzf go1.20.4.linux-amd64.tar.gz```
>   * user@hostname:~$ ```sudo nano $HOME/.profile```
>   * Add ```export PATH=$PATH:/usr/local/go/bin``` in the end of .profile
>   * user@hostname:~$ ```source $HOME/.profile```
>   * user@hostname:~$ ```go version``` to check if it works
* user@hostname:~$ <code>sudo adduser --disabled-login --gecos 'Gogs' git</code>
* user@hostname:~$ ```git clone --depth 1 https://github.com/gogs/gogs.git gogs```
* user@hostname:~$ ```cd gogs```
* user@hostname:~$ ```go build -o gogs```
* user@hostname:~$ ```sudo mkdir /srv/flextron-repos```
* user@hostname:~$ ```sudo mkdir /var/log/gogs```
* user@hostname:~$ ```sudo chown git /var/log/gogs```
* user@hostname:~$ ```sudo chown git /srv/flextron-repos```
* user@hostname:~$ ```cd ..```
* user@hostname:~$ ```sudo mv gogs /home/git/```
* user@hostname:~$ ```sudo chown -R git:git /home/git/gogs```
* Now we need new PSQL DB, DB user and set ownership:
* user@hostname:~$ ```sudo -u postgres psql -d template1```
* template1=# ```CREATE USER gogs1337 CREATEDB;```
* template1=# ```\password gogs1337```
* template1=# ```CREATE DATABASE gogs1337 OWNER gogs1337;```
* template1=# ```ctrl + z``` to exit
* user@hostname:~$ ```sudo su git```
* git@hostname:~$ ```cd ~/gogs/```
* git@hostname:~$ ```mkdir -p custom/conf/```
* git@hostname:~$ ```nano custom/conf/app.ini```
* Paste:

```console
[security]
INSTALL_LOCK = true
```

* save and exit
* git@hostname:~$ ```./gogs web```
* from browser http://your.server.ip.address:3000/install

* |Database Settings| Options       |
  |---------------|--------------:|
  | Database Type | PostgreSQL    |
  | Host | leave default |
  | User | gogs1337      |
  | Database Name | gogs1337      |
  | Schema | public        |
  |SSL Mode| Disable       |

* |Application General Settings| Options                             |
  |-------------------------------------|--------------:|
  | Application Name | FlexTron                            |
  | Repository Root Path | /srv/flextron-repos                 |
  | Run User | git                                 |
  | Domain | your.server.ip.address              |
  |SSH Port| 22                                  |
  |Use Builtin SSH Server| No                                  |
  |HTTP Port| 3000                                |
  |Application URL| http://your.server.ip.address:3000/ |
  |Log Path| /var/log/gogs                       |
  |Enable Console Mode| No                                  |
  |Default Branch| master                              |

* |Optional Settings| Options                                                                     |
  |-----------------------------------------------------------------------------|--------------:|
  | Skip Email Service | -                                                                           |
  | Server and Other Services Settings | Disable Self-registration and Enable Require SignIn to View Pages |
  | Admin Account Setup | :                                                                           |
  | Username | root1337                                                                    |
  | Password | ****                                                                        |
  |Skip email| -                                                                           |
*  Install Gogs
* Now you have access to Gogs via http://your.server.ip.address:3000/ 
* Add ssh key to your admin account for access to repositories
* To make Gogs running as service:
* git@hostname:~$ ```exit```
* user@hostname:~$ ```sudo nano /etc/systemd/system/gogs```
* Paste: [gogs](files%2Fgogs)
* user@hostname:~$ ```sudo cp /home/git/gogs/scripts/systemd/gogs.service /etc/systemd/system```
* git@hostname:~$ ```sudo systemctl daemon-reload```
* git@hostname:~$ ```sudo systemctl start gogs```
* git@hostname:~$ ```sudo systemctl enable gogs```
* git@hostname:~$ ```sudo reboot```
* user@hostname:~$ ```sudo systemctl status gogs```
* It should be active and running
* [if you want dark theme in gogs](https://github.com/Kos-M/GogsThemes)
* __Optional:__
>   * create new favicon.png 256x256 file
>   * Upload image to server with SFTPGo and move it or upload it directly with ssh to:
>   * /home/git/gogs/custom/public/img/
>   * user@hostname:~$ ```sudo systemctl restart gogs.service```
* __Example of Discord notifications on repositories activities in organization:__
>   * Go too http://your.server.ip.address:3000/ in your browser
>   * Big '+' button on top of the screen -> New Organization -> OrganizationName -> View OrganizationName at top right -> 
>   * -> Gear near organization name -> Webhooks -> Add a new webhook: -> Discord
>   * In Discord app select the Server, under Text Channels select Edit Channel (gear icon) Select Integrations ->
>   * -> View Webhooks and click New Webhook. Copy the Webhook URL and paste it to Payload URL in Gogs.
>   * username - name for "bot" in discord
>   * Set URL icon from server or internet
>   * Hex color of message
>   * When should this webhook be triggered? -> Let me choose what I need
>   * Select needed events

### STEP <a name=5>5</a> : Install and configure WireGuard
* user@hostname:~$ ```sudo apt install wireguard```
* user@hostname:~$ ```wg genkey | sudo tee /etc/wireguard/private.key```
* You will get your private key in output. 
* user@hostname:~$ ```sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key```
* You will get public key in output
* Save your keys somewhere 
* user@hostname:~$ ```sudo chmod go= /etc/wireguard/private.key```
* user@hostname:~$ ```sudo chmod go= /etc/wireguard/public.key```
* Select your IP range from [private network ip ranges](https://en.wikipedia.org/wiki/Private_network) 
* I've set 10.13.37.0/24 so 10.13.37.1 - 10.13.37.255 may be used
* Now we need to allow forward packets between interfaces:
>   * user@hostname:~$ ```sudo nano /etc/sysctl.conf```
>   * user@hostname:~$ ```net.ipv4.ip_forward=1```
>   * user@hostname:~$ ```sudo sysctl -p to check if its working```
* user@hostname:~$ ```sudo nano /etc/wireguard/wg0.conf```
* Paste:

```console
[Interface]
# Wirguard IP address and it's CIDR mask
Address = 10.13.37.37/24
# Port for VPN
ListenPort = 13370
# Private key of server
PrivateKey = yOuRSerVER1337pRiVatEKey=
```

* user@hostname:~$ ```ip addr``` if you forget your enpNs0 network interface
* For access to server internal services only and prevent using it for all users traffic. Even if they delete your server local IP from allowed IPs in wg0 configuration file:
* user@hostname:~$ ```sudo iptables -t nat -A POSTROUTING -s 10.13.37.0/24 -o enp7s0 -j MASQUERADE```
* user@hostname:~$ ```sudo iptables -A INPUT -p udp -m udp --dport 13370 -j ACCEPT```
* user@hostname:~$ ```sudo iptables -A FORWARD -i wg0 -j ACCEPT```
* user@hostname:~$ ```sudo iptables -A FORWARD -o wg0 -j ACCEPT```
* user@hostname:~$ ```sudo wg-quick up wg0```
* **On PC**
* pcuser@pcname:~$ ```wg genkey | sudo tee /etc/wireguard/private.key```
* You will get your private key in output.
* pcuser@pcname:~$ ```sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key```
* You will get public key in output
* Save your keys somewhere
* pcuser@pcname:~$ ```sudo chmod go= /etc/wireguard/private.key```
* pcuser@pcname:~$ ```sudo chmod go= /etc/wireguard/public.key```
* pcuser@pcname:~$ ```sudo nano /etc/wireguard/wg0.conf```
* Paste:

```
[Interface]
PrivateKey = yOurClIeNt1337PrIVAteKEy=
# VPN ip address for VPN
Address = 10.13.37.1/24
ListenPort = 13370
PostUp = ping -c1 10.13.37.37
[Peer]
PublicKey = yOuRsERVeR1337pUBliCKkEY=
# If you want to just test connection in local set your server's local ip
# Otherwise set your public IP and don't forget about port forwarding on router
Endpoint = server.or.public.ip:13370
# Allowed ip's for VPN access
AllowedIPs = 10.13.37.0/24, your.server.local.ip/24
# For keeping stable connection through firewalls
PersistentKeepalive = 25
```


* **On Server**
* user@hostname:~$ ```sudo nano /etc/wireguard/wg0.conf```
* Paste:

```console
[Peer]
PublicKey = yOurClIEnT1337pUbLICkKey=
AllowedIPs = 10.13.37.1/32
```

* user@hostname:~$ ```sudo wg-quick down wg0```
* user@hostname:~$ ```sudo systemctl enable wg-quick@wg0.service```
* user@hostname:~$ ```sudo systemctl start wg-quick@wg0.service```
* user@hostname:~$ ```sudo systemctl status wg-quick@wg0.service```
* Should be active (exited)
* user@hostname:~$ ```sudo wg show wg0```
* See allowed users ip
* **On PC**
* Open your router admin panel and port forward your server ip and port, change Endpoint ip in wg0 config to your public ip address
* Example:
* ![Screenshot_20230607_125547.png](imgs%2FScreenshot_20230607_125547.png)
* pcuser@pcname:~$ ```sudo wg-quick up wg0```

```console
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.027/0.027/0.027/0.000 ms 
```

* user@hostname:~$ ```sudo wg show``` on both machines, and you will see the latest handshake if your user is connected

### STEP <a name=6>6</a> : Monitoring Shell Script
* Create [monitoring.sh](srcs%2Fmonitoring.sh) file in /srv/
* user@hostname:~$ ```sudo chmod +x /srv/monitoring.sh```
* Now you can use it by calling user@hostname:~$ ```sudo /srv/monitoring.sh```
* Or create schedule in Cron:
>   *  user@hostname:~$ ```sudo crontab -e```
>   * Paste this for running script every 12 hours or adjust it to your needs:

```commandline
@reboot /srv/monitoring.sh
0 */12 * * * /srv/monitoring.sh
```