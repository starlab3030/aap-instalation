# 설치 준비

## 1. 사용자 구성

### 1.1 앤서블 사용자 추가 및 구성

```bash
useradd shadowman
passwd shadowman #(또는) echo '사용자_암호' | passwd --stdin shadowman 
grep shadowman /etc/passwd
grep shadowman /etc/group
```

실행 환경
```
[root@aap-c ~]# useradd shadowman

root@aap-c ~]# echo '사용자_암호' | passwd --stdin shadowman
Changing password for user shadowman.
passwd: all authentication tokens updated successfully.

[root@aap-c ~]# grep shadowman /etc/passwd
shadowman:x:1001:1001::/home/shadowman:/bin/bash

[root@aap-c ~]# grep shadowman /etc/group
shadowman:x:1001:

[root@aap-c ~]# 
```
<br>

### 1.2 앤서블 사용자의 sudo 구성

```bash
echo "shadowman ALL=(ALL)       NOPASSWD:ALL" > /etc/sudoers.d/shadowman
ls -lh /etc/sudoers.d/shadowman
su - shadowman
id
sudo -i
exit
exit
```

실행 환경
```
[root@aap-c ~]# echo "shadowman ALL=(ALL)       NOPASSWD:ALL" > /etc/sudoers.d/shadowman

[root@aap-c ~]# ls -lh /etc/sudoers.d/shadowman 
-rw-r--r--. 1 root root 30 Oct 14 10:40 /etc/sudoers.d/shadowman

[root@aap-c ~]# su - shadowman

[shadowman@aap-c ~]$ id
uid=1001(shadowman) gid=1001(shadowman) groups=1001(shadowman) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

[shadowman@aap-c ~]$ sudo -i

[root@aap-c ~]# exit
logout

[shadowman@aap-c ~]$ exit
logout

[root@aap-c ~]#
```
<br>
<br>

## 2. SSH 구성

### 2.1 앤서블 사용자의 SSH 키를 생성합니다.
```bash
su - shadowman
ssh-keygen -t rsa
ls -lh .ssh/
```

실행 환경
```
[root@aap-c ~]# su - shadowman

[shadowman@aap-c ~]$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/shadowman/.ssh/id_rsa): 
Created directory '/home/shadowman/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/shadowman/.ssh/id_rsa
Your public key has been saved in /home/shadowman/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:wOaFaA0W04S9hvru+JRuaVtOS3JaF9EHXj2DvQ7nMVY shadowman@aap-c.thinkmore.net
The key's randomart image is:
+---[RSA 3072]----+
|    =*.   . .+   |
|   ..*o. o o. = E|
|    o.*.o o .  = |
|   ..ooo . .. *  |
|   . .. S    * o |
|  .  .   .    o  |
|   .+.* .        |
|   +=X o         |
|  .*B.o          |
+----[SHA256]-----+

[shadowman@aap-c ~]$ ls -lh .ssh/
total 8.0K
-rw-------. 1 shadowman shadowman 2.6K Oct 14 10:56 id_rsa
-rw-r--r--. 1 shadowman shadowman  583 Oct 14 10:56 id_rsa.pub

[shadowman@aap-c ~]$ 
```
<br>

### 2.2 SSH 키 복사

앤서블 사용자의 SSH 키를 설치할 호스트에 복사합니다.
```bash
ssh-copy-id aap-c.thinkmore.net
ssh aap-c.thinkmore.net
exit
ssh aap-c
exit
```

실행 결과
```
[shadowman@aap-c ~]$ ssh-copy-id aap-c.thinkmore.net
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/shadowman/.ssh/id_rsa.pub"
The authenticity of host 'aap-c.thinkmore.net (192.168.0.43)' can't be established.
ED25519 key fingerprint is SHA256:핑거_프린트.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
shadowman@aap-c.thinkmore.net's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'aap-c.thinkmore.net'"
and check to make sure that only the key(s) you wanted were added.

[shadowman@aap-c ~]$ ssh aap-c.thinkmore.net
Register this system with Red Hat Insights: insights-client --register
Create an account or view all your systems at https://red.ht/insights-dashboard
Last login: Mon Oct 14 10:56:43 2024
[shadowman@aap-c ~]$ exit
logout
Connection to aap-c.thinkmore.net closed.

[shadowman@aap-c ~]$ ssh aap-c
The authenticity of host 'aap-c (192.168.0.43)' can't be established.
ED25519 key fingerprint is SHA256:핑거_프린트.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:1: aap-c.thinkmore.net
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'aap-c' (ED25519) to the list of known hosts.
Register this system with Red Hat Insights: insights-client --register
Create an account or view all your systems at https://red.ht/insights-dashboard
Last login: Mon Oct 14 11:00:29 2024 from 192.168.0.43

[shadowman@aap-c ~]$ exit
logout
Connection to aap-c closed.

[shadowman@aap-c ~]$ 
```
<br>
<br>

## 3. 컨테이너화 된 AAP 다운로드

access.redhat.com에서 패키지 선택 후 다운로드
<img align="left" src="./images/download_aap_from_portal.png" title="100px" alt="포탈에서 AAP 다운로드"></img>

* 리스트 중에 *컨테이너 기반 설치*를 다운로드
* 다운로드할 패키지의 링크를 복사 후 CLI에서 설치 가능
  ```
  [shadowman@aap-c ~]$ mkdir -pv ~/temp
  mkdir: created directory '/home/shadowman/temp'
  
  [shadowman@aap-c ~]$ wget 'https://access.cdn.redhat.com/...<snip>...' -O ansible-automation-platform-containerized-setup-2.6-2.tar.gz
  --2024-10-14 11:16:25--  https://access.cdn.redhat.com/...<snip>...
  Resolving access.cdn.redhat.com (access.cdn.redhat.com)... 184.26.91.232, 184.26.91.185, 2600:140e:6::b833:663b, ...
  Connecting to access.cdn.redhat.com (access.cdn.redhat.com)|184.26.91.232|:443... connected.
  HTTP request sent, awaiting response... 200 OK
  Length: 3544100 (3.4M) [application/octet-stream]
  Saving to: ‘/home/shadowman/temp/ansible-automation-platform-containerized-setup-bundle-2.6-2.tar.gz’
  
  ansible-automation-platform-cont 100%[=========================================================>]   3.38M  6.51MB/s    in 0.5s    
  
  2024-10-14 11:16:26 (6.51 MB/s) - ‘/home/shadowman/temp/ansible-automation-platform-containerized-setup-bundle-2.6-2.tar.gz’ saved [3544100/3544100]
  
  [shadowman@aap-c ~]$ ls -lh ~/temp/ansible-automation-platform-containerized-setup-bundle-2.6-2.tar.gz 
  -rw-r--r--. 1 shadowman shadowman 2.4G Oct  8 23:00 /home/shadowman/temp/ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64.tar.gz
  
  [shadowman@aap-c ~]$  
  ```
<br>
<br>

## 4. 컨테이너화 된 AAP 패키지 확인

### 4.1 패키지 압축 해제
```bash
cd ~/temp
tar xvzf ansible-automation-platform-containerized-setup-bundle-2.6-2.tar.gz 
```

실행 환경
```
[shadowman@aap-c ~]$ cd ~/temp

[shadowman@aap-c temp]$ tar xvzf ansible-automation-platform-containerized-setup-bundle-2.6-2.tar.gz 
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/collections/
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/collections/ansible_collections/

...<snip>...

ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/README.md
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/ansible.cfg
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/inventory
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/inventory-growth
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/bundle/
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/bundle/images/
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/bundle/images/controller-rhel8.tar.gz
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/bundle/images/de-supported-rhel8.tar.gz
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/bundle/images/eda-controller-rhel8.tar.gz
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/bundle/images/eda-controller-ui-rhel8.tar.gz
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/bundle/images/ee-minimal-rhel8.tar.gz
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/bundle/images/ee-supported-rhel8.tar.gz
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/bundle/images/gateway-rhel8.tar.gz
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/bundle/images/gateway-proxy-rhel8.tar.gz
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/bundle/images/hub-rhel8.tar.gz
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/bundle/images/hub-web-rhel8.tar.gz
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/bundle/images/receptor-rhel8.tar.gz
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/bundle/images/postgresql-15.tar.gz
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/bundle/images/redis-6.tar.gz
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/bundle/images/pcp.tar.gz

[shadowman@aap-c temp]$ 
```
<br>

### 4.2 패키지 구조 확인

```bash
tree -F -L 3 ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64
```

실행 결과
```
[shadowman@aap-c temp]$ tree -F -L 3 ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64
ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64
[  116]  ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64/
├── [  53K]  README.md
├── [  146]  ansible.cfg
├── [   39]  bundle/
│   ├── [   40]  collections/
│   │   ├── [ 4.0K]  certified/
│   │   └── [ 4.0K]  validated/
│   └── [ 4.0K]  images/
│       ├── [ 271M]  controller-rhel9.tar.gz
│       ├── [ 192M]  de-supported-rhel9.tar.gz
│       ├── [ 137M]  eda-controller-rhel9.tar.gz
│       ├── [  98M]  eda-controller-ui-rhel9.tar.gz
│       ├── [ 116M]  ee-minimal-rhel9.tar.gz
│       ├── [ 512M]  ee-supported-rhel9.tar.gz
│       ├── [  62M]  gateway-proxy-rhel9.tar.gz
│       ├── [ 146M]  gateway-rhel9.tar.gz
│       ├── [ 162M]  hub-rhel9.tar.gz
│       ├── [  98M]  hub-web-rhel9.tar.gz
│       ├── [ 1.4G]  lightspeed-chatbot-rhel9.tar.gz
│       ├── [ 246M]  lightspeed-rhel9.tar.gz
│       ├── [  71M]  mcp-tools-rhel9.tar.gz
│       ├── [  81M]  pcp.tar.gz
│       ├── [ 129M]  postgresql-15.tar.gz
│       ├── [ 102M]  receptor-rhel9.tar.gz
│       └── [  94M]  redis-6.tar.gz
├── [   33]  collections/
│   └── [ 4.0K]  ansible_collections/
│       ├── [   84]  ansible/
│       ├── [   24]  ansible.posix-1.5.4.info/
│       ├── [   38]  community/
│       ├── [   24]  community.crypto-2.26.5.info/
│       ├── [   24]  community.postgresql-3.4.0.info/
│       ├── [   20]  containers/
│       ├── [   24]  containers.podman-1.17.0.info/
│       ├── [   62]  infra/
│       ├── [   24]  infra.ah_configuration-2.0.6.info/
│       └── [   24]  infra.controller_configuration-2.9.0.info/
├── [ 4.2K]  inventory
└── [ 4.1K]  inventory-growth

18 directories, 21 files

[shadowman@aap-c temp]$ 
```
<br>
<br>

## 5. 컨테이너화 된 AAP 설치 환경

### 5.1 ansible.cfg 확인

```bash
ln -s ./ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64 ./ansible-setup
ls -lh
cd ansible-setup/
cat ansible.cfg 
```

실행 환경
```
[shadowman@aap-c temp]$ ln -s ./ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64 ./ansible-setup

[shadowman@aap-c temp]$ ls -lh
total 2.4G
drwxr-xr-x. 4 shadowman shadowman  116 Oct  8 03:56 ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64
-rw-r--r--. 1 shadowman shadowman 2.4G Oct  8 23:00 ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64.tar.gz
lrwxrwxrwx. 1 shadowman shadowman   69 Oct 14 11:45 ansible-setup -> ./ansible-automation-platform-containerized-setup-bundle-2.6-2-x86_64

[shadowman@aap-c temp]$ cd ansible-setup/

[shadowman@aap-c ansible-setup]$ cat ansible.cfg 
[defaults]
collections_on_ansible_version_mismatch = ignore
collections_path = ./collections
inventory = ./inventory
log_path = ./aap_install.log

[shadowman@aap-c ansible-setup]$ 
```
<br>

### 5.2 기본 인벤토리 파일

```bash
ls -lh inventory*
```

실행 환경
```
[shadowman@aap-c ansible-setup]$ ls -lh inventory*
-rw-r--r--. 1 root root 4.3K 10월 29일  20:22 inventory
-rw-r--r--. 1 root root 4.2K 10월 29일  20:22 inventory-growth

[shadowman@aap-c ansible-setup]$ 
```
* *inventory-growth* 파일은 stand-alone으로 구성 후, 스케일-아웃으로 확장할 때 사용
<br>
<br>

------
[차례](../README.md)