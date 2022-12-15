# What is this
A quick how to install Seafile Professional within a docker stack (docker-compose) including anti-virus, elastic-search and DB behind nginx-proxy-manager
as reverse proxy and Let's Encrypt SSL certificate manager with a nice GUI.

This dockered Seafile supports seaf-fuse.sh filesystem inside the container and rsync to a second drive.

## Environment
- Host IP: 192.168.2.10
- Seafile Port: 8180/tcp for HTTP (Host) -> 80/tcp (Container)
- nginx-proxy-manager Port: 80/tcp (HTTP), 443/tcp (HTTPS) and 81/tcp (HTTP) (all Host)
- Seafile URL: wolke8.intern.myCustomDomain.de
- Firewall rules: Allow incoming 80/tcp and 443/tcp to HOST IP
- /dev/sda: Internal disk
- /dev/sdb: External disk for Seafile data and configs (/dev/sdb1 mounted on /mnt/sdb1-usb on the host)
- /dev/sdc: External disk for Seafile backup (/dev/sdc1 mounted on /mnt/sdc1-usb on the host)

# HowTo

## Preparations
- Add /mnt/sdb1-usb to /etc/fstab on the host
- Add /mnt/sdc1-usb to /etc/fstab on the host
- apt-get install rsync
- apt-get install rclone (recommended for encrypted cloud backup) and configure as needed

## Initial Seafile Pro setup
- Copy docker-compose.yml to destination (example: /opt/docker-seafile-pro)
- Adjust placeholder variables
- Replace DB root password twice (mysql container and seafile container)
- Initial start with docker-compose up and watch the logs (plenty of errors for elasticsearch)
- Stop with Control-C
- chmod -R 777 ./seafile-elasticsearch

## Install and fire up nginx-proxy-manager
- Login into nginx-proxy-manager
- Add new proxy host (wolke8.intern.myCustomDomain.de as specified in seafile-docker-compose.yml
- Use HTTP (not HTTPS) to HOST IP, Port 8180 (as specified in seafile-docker-compose.yml)
- Turn on SSL options and force SSL
- Add custom nginx configuration within Advanced

````
proxy_read_timeout 310s;
proxy_set_header Host $host;
proxy_set_header Forwarded "for=$remote_addr;proto=$scheme";
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header Connection "";
proxy_http_version 1.1;

client_max_body_size 0;
````

## Test connection
- Login to Seafile should work now
- Upload/Download will not work

## Adjust config files in Seafile container
- cd to seafile-data/seafile/conf/
- edit seahub_settings.py
- Change SERVICE_URL and FILE_SERVER_ROOT to use "https://" instead of "http://"
- docker-compose down
- docker-compose up -d

## Backup with FUSE and rsync
- Mount backup-drive to /mnt/sdc1-usb (as specified in the docker-compose.yml)
- docker exec -it seafile /bin/bash (will give you access to container)
- modprobe fuse (load kernel module)
- /opt/seafile-server-latest/seaf-fuse.sh start /shared/seafile/fuse-data
- rsync -avP /shared/seafile/fuse-data /mnt/seafile-backup

## Cloud backup
- Add rclone cronjob
