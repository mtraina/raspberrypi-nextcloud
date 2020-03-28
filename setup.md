# install Raspian via Raspberr PI Imager
https://www.raspberrypi.org/downloads/

# add an empty file names ssh at in boot
touch /Volumes/boot/ssh

# update - upgrade
sudo apt-get update
sudo apt-get dist-upgrade

# set static ip
https://pimylifeup.com/raspberry-pi-static-ip-address/

sudo vi /etc/resolv.conf

sudo vi /etc/dhcpcd.conf
interface <NETWORK>
static ip_address=<STATICIP>/24
static routers=<ROUTERIP>
static domain_name_servers=<DNSIP>

# Ansible
http://unixetc.co.uk/2017/11/25/automatic-nextcloud-installation-on-raspberry-pi/

sudo apt-get -y install ansible
wget https://raw.githubusercontent.com/webtaster/Nextcloud/master/build_nextcloud.yml
sudo date ; ansible-playbook --become -c local -i "localhost," build_nextcloud.yml

sudo mv /var/nextcloud/data /media/ncdata
sudo ln -s /media/ncdata/data /var/nextcloud/data

## Increase memory
sudo cp /etc/php/7.3/apache2/php.ini /etc/php/7.3/apache2/php.ini.bak
change the memory to 512
sudo systemctl restart apache2

## Setup self signed SSL
https://pimylifeup.com/raspberry-pi-nextcloud-server/

sudo mkdir -p /etc/apache2/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:4096 -keyout /etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt
sudo a2enmod ssl
sudo cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/default-ssl.conf.bak
sudo vi /etc/apache2/sites-available/default-ssl.conf

SSLCertificateFile /etc/apache2/ssl/apache.crt
SSLCertificateKeyFile /etc/apache2/ssl/apache.key

sudo a2ensite default-ssl.conf
sudo service apache2 restart

# External drive
## Format
https://tecadmin.net/format-usb-in-linux/
https://superuser.com/questions/662614/raspberry-pi-how-to-format-hdd

sudo mkfs.ext4 /dev/sda1

## Mount
https://www.raspberrypi-spy.co.uk/2014/05/how-to-mount-a-usb-flash-disk-on-the-raspberry-pi/

ls -l /dev/disk/by-uuid/
sudo mkdir /media/ncdata
sudo chown -R pi:pi /media/ncdata
sudo mount /dev/sda1 /media/ncdata -o uid=pi,gid=pi
umount /media/ncdata

# Docker (WIP)
https://howchoo.com/g/nmrlzmq1ymn/how-to-install-docker-on-your-raspberry-pi

curl -sSL https://get.docker.com | sh
sudo usermod -aG docker pi
groups pi
sudo reboot

## NextCloudPi
https://docs.nextcloudpi.com/en/how-to-get-started-with-ncp-docker/

docker run -d -p 4443:4443 -p 443:443 -p 80:80 -v /media/ncdata:/data --name nextcloudpi ownyourbits/nextcloudpi-armhf 
docker run -d -p 4443:4443 -p 443:443 -p 80:80 -v ncdata:/data -v /media/ncdata:/data/app/data  --name nextcloudpi ownyourbits/nextcloudpi-armhf 