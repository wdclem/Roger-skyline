# Roger-skyline

Usefull links and steps

12**Create a non-root user >> sudo adduser johnny sudo


3** class /30 submask > https://petri.com/how-30-and-32-bit-ip-subnet-masks-can-help-with-cisco-networking
setup static ip >> https://linuxconfig.org/how-to-setup-a-static-ip-address-on-debian-linux
                   https://www.codesandnotes.be/2018/10/16/network-of-virtualbox-instances-with-static-ip-addresses-and-internet-access/
set settings network to bridge
into ect/network/interface/ change primary network interface to auto enp0s3
then create interfaces.d
$ cd interfaces.d
$ sudo touch enp0s3
$ sudo vim enp0s3
set ip first 10.12.202.255 (something free)
netmask  255.255.255.252 (/30 or 30 * 1)
gateway  10.11.254.254

then restart sudo service networking restart
ifconfing to check result (quite a few can have gone bad a this point, in case of error try to reboot the machine. If further errors ifup -a -v is a friend)

4**  change defautl ssh port https://www.linuxlookup.com/howto/change_default_ssh_port
need to change the ssh file
sudo vim /etc/ssh/ssh_config
change #port 22 by a port in the correct range (remove the comment # and add as follow port 50000
restart ssh service : sudo service ssh restart // sudo /etc/init.d/ssh restart
then try to connect username@hostname -p 60000 ( I picked 60000)

5** connect with public keys https://www.linode.com/docs/guides/use-public-key-authentication-with-ssh/
https://www.cyberciti.biz/faq/ubuntu-18-04-setup-ssh-public-key-authentication/

NEED TO TEST CONNECT REMOTELY (not sure i make it work -______-

6** Setup firevall on your server
https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-18-04
install ufw with sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 443
sudo ufw allow 80/tcp
port 80/tcp is HTTP (only for TCP not UDP), 443 is HTTPS, and 60000 is SSH.

sudo ufw enable sudo ufw status verbose
