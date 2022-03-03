# Roger-skyline

Usefull links and steps

Debian Installation
Install Debian 10.2 creating one partion 4.2gb
No need to install desktop

UPDATE
apt install sudo (to enable sudo)
sudo apt update/ sudo apt upgrade (to make package up to date)

NETWORK AND SECURITY PART

1/2**Create a non-root user >> sudo adduser johnny >> sudo adduser johnny sudo


3** class /30 submask > https://petri.com/how-30-and-32-bit-ip-subnet-masks-can-help-with-cisco-networking
setup static ip >> https://linuxconfig.org/how-to-setup-a-static-ip-address-on-debian-linux
                   https://www.codesandnotes.be/2018/10/16/network-of-virtualbox-instances-with-static-ip-addresses-and-internet-access/
set settings network to bridge
into ect/network/interface/ change primary network interface to auto enp0s3
then create interfaces.d
$ cd interfaces.d
$ sudo touch enp0s3
$ sudo vim enp0s3
set ip first 10.12.223.255 (something free)
netmask  255.255.255.252 (/30 or 30 * 1)
gateway  10.11.254.254

then restart sudo service networking restart
ifconfing to check result (quite a few can have gone bad a this point, in case of error try to reboot the machine. If further errors ifup -a -v is a friend)

4**  change default ssh port https://www.linuxlookup.com/howto/change_default_ssh_port
need to change the ssh file
sudo vim /etc/ssh/ssh_config
change #port 22 by a port in the correct range (remove the comment # and add as follow port 50000
restart ssh service : sudo service ssh restart // sudo /etc/init.d/ssh restart
then try to connect username@hostname -p 60000 ( I picked 60000)

5** connect with public keys https://www.linode.com/docs/guides/use-public-key-authentication-with-ssh/
https://www.cyberciti.biz/faq/ubuntu-18-04-setup-ssh-public-key-authentication/
Create a key ssh-keygen -t rsa, keep default configuration
ssh-copy-id -i ~/.ssh/id_rsa.pub johnny@10.11.223.255 -p 60000 from your terminal (not vm) to allow connection with public key
edit /etc/ssh/sshd_config to remove root login and pwd authentication (#PasswordAuthentication and #PermitRootLogin)

NEED TO TEST CONNECT REMOTELY (not sure i make it work -______-

6** Setup firevall on your server
https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-18-04
install ufw with sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 443
sudo ufw allow 80/tcp
sudo ufw allow 60000/tcp
port 80/tcp is HTTP (only for TCP not UDP), 443 is HTTPS, and 60000 is SSH.

sudo ufw enable sudo ufw status verbose

7** Denial Of Service Attack
First need to fail2ban and have apache2 installed
https://bobcares.com/blog/centos-ddos-protection/
https://www.tothenew.com/blog/fail2ban-port-80-to-protect-sites-from-dos-attacks/https://www.tothenew.com/blog/fail2ban-port-80-to-protect-sites-from-dos-attacks/
create a conf file into jail.d and set configuration to it on the port

  [DEFAULT]
  bantime  = 10m
  findtime  = 10m
  maxretry = 5

  [sshd]
  enabled = true
  port = 50113
  maxretry = 3
  findtime = 300
  bantime = 600
  logpath = %(sshd_log)s
  backend = %(sshd_backend)s

  [http-get-dos]
  enabled = true
  port = http,https
  filter = http-get-dos
  logpath = /var/log/apache2/access.log
  maxretry = 300
  findtime = 300
  bantime = 600
  action = iptables[name=HTTP, port=http, protocol=tcp]
  
 create a filter sudo vim /etc/fail2ban/filter.d/http-get-dos.conf and set configuration in
  
  [Definition]
  failregex = ^<HOST> -.*"(GET|POST).*
  ignoreregex =
  
  reload service
  sudo ufw reload
  sudo service fail2ban restart
  Activate fail2ban sudo service enable fail2ban
  sudo service status fail2ban  
  sudo iptables -L to check the rules
  check logs sudo tail -f /var/log/fail2ban.log / sudo fail2ban-client status sshd
  To test out re enable pwd authentication sudo vim /etc/ssh/sshd_config #PasswordAuthentication

  sudo fail2ban-client status sshd to check if ban was effective
  sudo cat /var/log/fail2ban.log shows also dos attacks
  
  8** Protection against port scan
  first install nmap tool used to launch port scan and check open and closed ports
  sudo apt install nmap
  you can scan the server with nmap -PN -sS 10.11.223.255
  
  ###config first iptables, enables pakcets with 
  #iptables -A INPUT -j LOG
  #iptables -A FORWARD -j LOG
  
  9** Stop Services not needed
  
  https://www.digitalocean.com/community/tutorials/how-to-use-systemctl-to-manage-systemd-services-and-units
  
  ls /etc/init.d
  systemctl list-units --type service --all
  sudo systemctl list-unit-files --type service | grep enabled
  
  sudo systemctl disable bluetooth.service
  sudo systemctl disable console-setup.service
  sudo systemctl disable keyboard-setup.service ect ect
  
  sudo service --status-all
  
  DOUBLE CHECK
  
  10** Update packages script
  
  Create script to execute sudo apt update and upgrade record it in /var/log/update.log
  
  sudo crontab -e
  0 4 * * 0 sudo ~scripts/update.sh
  @reboot sudo ~/scripts/update.sh
  
  to plan the update at 4am and on reboot
  
  11** Monitor Crontab changes
  
  Install mail 
  sudo apt install mailutils
  
  create a script for update and then to monitor changes
  
  add 0 0 * * * sudo ~/scripts/monitor_changes.sh to execute at midnight
  
EVAL ANSWERS
  
  PART 1
  VM VERSION
cat /etc/os-release
cat /etc/debian_version
  
  TRAEFIK DOCKER VAGRANT
  
  dpkg -l
  apt list --installed
  
  SIZE OF DISK/PARTITION
  To check partition fdisk -l into >> sudo parted unit GB >> print list
  
  CHECK IF UP TO DATE
  sudo apt update
  
  PART2
  
  CREATE SUDO and CONNECT SSH
  Create a non-root user >> sudo adduser johnny >> sudo adduser johnny sudo
  cat /etc/group
  sudo -l
  
  ssh rene@10.11.223.255
  Create a key ssh-keygen -t rsa, keep default configuration
  ssh-copy-id -i ~/.ssh/id_rsa.pub johnny@10.11.223.255 -p 60000 from your terminal (not vm) to allow connection with public key
  
  CHECK DHCP
  sudo service dhcp status
  ip r 
  Should both show no DHCP
  
  CHANGE NETMASK
  
  set settings network to bridge
into etc/network/interface/ change primary network interface to auto enp0s3
then create interfaces.d
$ cd interfaces.d
$ sudo touch enp0s3
$ sudo vim enp0s3
set ip first 10.12.223.255 (something free)
netmask  255.255.255.252 (/30 or 30 * 1)
gateway  10.11.254.254
  sudo service networking restart
  check on debian
  and reconnect
  check ifconfig
  
  CHECK SSH MODIFICATION WIHT PUBLICKEYS
  
  sudo cat /etc/ssh/sshd_config
	#Port 60000
	#PermitRootLogin no
	#PubkeyAuthentication yes
	#PasswordAuthentication no 
	
	#can try to connect 
	# ssh root@10.11.142.143 -p 60000
  
  LIST FIREWALL RULES
  
  sudo ufw status verbose
  
  
  
