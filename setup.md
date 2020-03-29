# Setup Raspian
## Rasberry Imager
https://www.raspberrypi.org/downloads/

## Enable ssh at boot
```bash
touch /Volumes/boot/ssh
```

## Upgrade
```bash
sudo apt-get update
sudo apt-get dist-upgrade
```

## Set static ip
https://pimylifeup.com/raspberry-pi-static-ip-address/

```bash
sudo vi /etc/resolv.conf
```

```bash
sudo vi /etc/dhcpcd.conf

interface <NETWORK>
static ip_address=<STATICIP>/24
static routers=<ROUTERIP>
static domain_name_servers=<DNSIP>
```

## Check temperature
https://medium.com/@kevalpatel2106/monitor-the-core-temperature-of-your-raspberry-pi-3ddfdf82989f

```bash
/opt/vc/bin/vcgencmd measure_temp
```

# Ansible
http://unixetc.co.uk/2017/11/25/automatic-nextcloud-installation-on-raspberry-pi/

```bash
sudo apt-get -y install ansible
wget https://raw.githubusercontent.com/webtaster/Nextcloud/master/build_nextcloud.yml
sudo date ; ansible-playbook --become -c local -i "localhost," build_nextcloud.yml
```

```bash
sudo mv /var/nextcloud/data /media/ncdata
sudo ln -s /media/ncdata/data /var/nextcloud/data
```

## Increase memory
```bash
sudo cp /etc/php/7.3/apache2/php.ini /etc/php/7.3/apache2/php.ini.bak
change the memory to 512M (originally 128M)
sudo systemctl restart apache2
```

## Setup self signed SSL
https://pimylifeup.com/raspberry-pi-nextcloud-server/

### Create certificates
```bash
sudo mkdir -p /etc/apache2/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:4096 -keyout /etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt
sudo a2enmod ssl
```

### Update configuration
```bash
sudo cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/default-ssl.conf.bak
sudo vi /etc/apache2/sites-available/default-ssl.conf

# set the following
# SSLCertificateFile /etc/apache2/ssl/apache.crt
# SSLCertificateKeyFile /etc/apache2/ssl/apache.key
```

### Restart Apache
```bash
sudo a2ensite default-ssl.conf
sudo service apache2 restart
```

## Fix permission error
https://help.nextcloud.com/t/nextcloud-warnung-some-app-directories-are-owned-by-a-different-user-than-the-web-server-one/65590/3

```bash
sudo chown -R www-data:www-data /var/www/html/nextcloud
```

## Update to 18.0.3
Simply go to settings from within Nextcloud

# External drive
## Format
https://tecadmin.net/format-usb-in-linux/
https://superuser.com/questions/662614/raspberry-pi-how-to-format-hdd

```bash
sudo mkfs.ext4 /dev/s$DISK
ie sudo mkfs.ext4 /dev/sda1
```

## Mount
https://www.raspberrypi-spy.co.uk/2014/05/how-to-mount-a-usb-flash-disk-on-the-raspberry-pi/

```bash
ls -l /dev/disk/by-uuid/
sudo mkdir /media/ncdata
sudo chown -R pi:pi /media/ncdata
sudo mount /dev/sda1 /media/ncdata -o uid=pi,gid=pi
umount /media/ncdata
```

### Mount ext4 drive with osx
https://github.com/gerard/ext4fuse

### Install Fuse
```bash
brew cask install osxfuse
brew install ext4fuse
```

### List disks
```bash
diskutil list
```

### Mount
```bash
sudo ext4fuse /dev/disk$DISKs$SECTION /$MOUNT_POINT -o allow_other
# ie sudo ext4fuse /dev/disk0s5 ~tmp/my-linux-mount -o allow_other

diskutil umount /$MOUNT_POINT
diskutil unmountDisk /dev/disk$DISKs$SECTION
```

# Docker (WIP)
https://howchoo.com/g/nmrlzmq1ymn/how-to-install-docker-on-your-raspberry-pi

```bash
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker pi
groups pi
sudo reboot
```

## NextCloudPi
https://docs.nextcloudpi.com/en/how-to-get-started-with-ncp-docker/

```bash
docker run -d -p 4443:4443 -p 443:443 -p 80:80 -v /media/ncdata:/data --name nextcloudpi ownyourbits/nextcloudpi-armhf 
docker run -d -p 4443:4443 -p 443:443 -p 80:80 -v ncdata:/data -v /media/ncdata:/data/app/data  --name nextcloudpi ownyourbits/nextcloudpi-armhf 
```

## Nextcloud
https://hub.docker.com/_/nextcloud/