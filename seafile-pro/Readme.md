# What is this
A quick how to install Seafile Professional within a docker stack (docker-compose) including anti-virus, elastic-search and DB behind nginx-proxy-manager
as reverse proxy and Let's Encrypt SSL certificate manager with a nice GUI.

## Environment
- Host IP: 192.168.2.10
- Seafile Port: 8180/tcp for HTTP -> 80/tcp 
- nginx-proxy-manager Port: 80/tcp (HTTP), 443/tcp (HTTPS) and 81/tcp (HTTP)
- Seafile URL: wolke8.intern.myCustomDomain.de
- Firewall rules: Allow incoming 80/tcp and 443/tcp to HOST IP

# HowTo

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

Done
