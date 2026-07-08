Entorno en 
```bash

         ──────────────────────────────────────────────────────────────────────────────────────────
                                                ┌────────┐     ┌────────┐
                                                │        │     │ WEB    │
                                       ((WB-1)) │        │     │        │ ((WB-2))
                                                └───┬────┘     └───┬────┘ 
                                                    │              │
                                                ┌───▼────┐     ┌───▼────┐
                                       ((MD-2)) │        │     │ php    │ ((middleware 2))  
                                ┌─────────┐     └───┬────┘     └───┬────┘ 
                       ((MD-1)) │         │         │ pt. 80       │ pt. 50000
                                └────┬────┘         └──────┬───────┘
                                     │ pt. 22              │ 
                                ┌────▼────┐           ┌────▼────┐               
                      ((COM-1)) │         │           │  OS aLPINE   │ ((middleware))          
                                └─────────┘           └─────────┘      
                                     │                     │      
                                     └──────────┼──────────┘
                                                │
                                        ┌───────▼───────┐
        ────────────────────────────────│ Container     │─────────────────────(   )──────────
                                        └───────────────┘                       │
                                                                                │ VPN directo al puerto de ubuntu con conexión a 
                                                                                │ internet  
                                        ┌───────────────┐                       │
                         ((AC-1)) ──────│ Ubuntu server │ ((SO-1))     ──────────
                                        └───────────────┘
                                        
```
System information
```bash
docker-server login: agarti
Password:
Welcome to Ubuntu 26.04 LTS (GNU/Linux 7.0.0-27-generic x86_64)

 * Documentation: https://docs.ubuntu.com
 * Management: https://landscape.canonical.com
 * Support: https://ubuntu.com/pro

System information as of Tue Jul 7 08:41:48 AM UTC 2026

  System load:  0.54
  Usage of /:   48.7% of 11.21GB
  Memory usage: 12%
  Swap usage:   0%

  Processes:    256
  Users logged in: 0
  IPv4 address for ens33: 192.168.184.135

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

14 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

agarti@docker-server:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host preferred_loopback
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:eb:57:bf brd ff:ff:ff:ff:ff:ff
    altname enp0s1
    altname enx000c29eb57bf
    inet 192.168.184.135/24 metric 100 brd 192.168.184.255 scope global dynamic ens33
       valid_lft 1783sec preferred_lft 1783sec
    inet6 fe80::20c:29ff:feab:57bf/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 52:70:49:d0:31:1d brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
agarti@docker-server:~$

```
Mounting the honeypot
```bash
agarti@docker-server:~$ groups agarti
agarti : agarti adm cdrom sudo dip plugdev users lxd docker

agarti@docker-server:~$ mkdir -p honeypot

agarti@docker-server:~/honeypot$ nano docker-compose.yml
```

```bash
agarti@docker-server:/opt/honeypot$ cat docker-compose.yml
version: '3.8'

services:
  db_trampa:
    image: mysql:8.0
    container_name: honeypot_mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - honeypot_db_data:/var/lib/mysql
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

  wp_trampa:
    image: wordpress:5.6
    container_name: honeypot_wordpress
    restart: always
    ports:
      - "80:80"
    depends_on:
      - db_trampa
    environment:
      WORDPRESS_DB_HOST: db_trampa
      WORDPRESS_DB_USER: ${MYSQL_USER}
      WORDPRESS_DB_PASSWORD: ${MYSQL_PASSWORD}
      WORDPRESS_DB_NAME: ${MYSQL_DATABASE}
    volumes:
      - honeypot_wp_data:/var/www/html
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

volumes:
  honeypot_db_data:
  honeypot_wp_data:
```

```bash
agarti@docker-server:~/honeypot$ docker compose up -d
WARN[0000] /home/agarti/honeypot/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion
[+] Running 41/41
 ✔ db Pulled                                                                      104.1s
 ✔ wordpress Pulled                                                               109.3s

[+] Running 3/3
 ✔ Network honeypot_default   Created                                               0.6s
 ✔ Container honey_mysql      Started                                               1.2s
 ✔ Container honey_wordpress  Started                                               0.7s
 ```
Antes y después del firewall
```bash                                                                                                    
┌──(kali㉿kali)-[~]
└─$ nmap 192.168.184.135        
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-08 12:38 +0200
Nmap scan report for 192.168.184.135
Host is up (0.00011s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 00:0C:29:EB:57:BF (VMware)

Nmap done: 1 IP address (1 host up) scanned in 0.81 seconds
                                                                                                       
┌──(kali㉿kali)-[~]
└─$ nmap 192.168.184.135 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-08 12:39 +0200
Nmap scan report for 192.168.184.135
Host is up (0.00047s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 00:0C:29:EB:57:BF (VMware)

Nmap done: 1 IP address (1 host up) scanned in 5.37 seconds
 ```

