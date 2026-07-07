Entorno en 

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

```bash
agarti@docker-server:~$ groups agarti
agarti : agarti adm cdrom sudo dip plugdev users lxd docker

agarti@docker-server:~$ docker run -d --name vuln_server alpine tail -f /dev/null
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
55afa1ecc21d: Pull complete
56dceff11b33: Download complete
f5124fb579e2: Download complete
Digest: sha256:28bd5fe8b56d1bd048e5babf5b10710ebe0bae67db86916198a6eec434943f8b
Status: Downloaded newer image for alpine:latest
a9fcad33c38b4e081a952b637c1a0b32adfaa718d651f585970ccb1a05111ace

agarti@docker-server:~$ docker --version
Docker version 29.1.3, build 29.1.3-0ubuntu4.1

agarti@docker-server:~$ docker ps
CONTAINER ID   IMAGE     COMMAND               CREATED         STATUS         PORTS     NAMES
a9fcad33c38b   alpine    "tail -f /dev/null"   3 minutes ago   Up 3 minutes             vuln_server
```
