# 사전 준비

## 1. 시스템 요구 사항

### 1.1 최소 시스템 리소스

Ansible Automation Platform(이하 AAP) 설치를 위해 다음과 같은 최소 시스템 요구 사항이 필요합니다.
|$\color{lime}{\texttt{리소스 타입}}$|$\color{lime}{\texttt{최소 크기}}$|
|:---|:---|
|`메모리`|16GiB|
|`CPU`|4-core|
|`디스크 공간`|60GiB|
|`디스크 IOPS`|3000|
<br>

### 1.2 운영체제 요구 사항

* RHEL 9.2 이상
* *root*가 아닌 앤서블 사용자
  - sudo 혹은 앤서블 권한 상승을 가진 사용자
  - *root* 사용자로 작업 시, 다음 메시지가 표시됨
    ```
    "the remote user should be a non root user"
    ```
* SSH 공개 키 인증 필요
  - 로컬 설치의 경우, *ansible_connection:local* 사용 가능
* 시스템 접속 시 해당 사용자로 SSH 로그인 필요
  - 다른 사용자로 로그인 후, su 혹은 sudo로 해당 사용자로 변경하면 설치 시 에러남
    ```
    ‘systemctl --user’ : “Failed to connect to bus: No such file or directory”
    ```
<br>

### 1.3 설치에 필요한 채널 및 패키지

채널 리스트
* rhel-9-for-x86_64-appstream-rpms
* rhel-9-for-x86_64-baseos-rpms

패키지 리스트
* ansible-core
* git
* rsync
* wget
<br>
<br>

## 2. 호스트 확인

### 2.1 기본 정보

호스트 기본 정보를 확인합니다.

```bash
hostname
uname -a
cat /etc/redhat-release
hostnamectl
```

실행 결과
```
[root@aap-c ~]# hostname
aap-c.thinkmore.net

[root@aap-c ~]# uname -a
Linux aap-c.thinkmore.net 5.14.0-427.13.1.el9_4.x86_64 #1 SMP PREEMPT_DYNAMIC Wed Apr 10 10:29:16 EDT 2024 x86_64 x86_64 x86_64 GNU/Linux

[root@aap-c ~]# cat /etc/redhat-release 
Red Hat Enterprise Linux release 9.4 (Plow)

[root@aap-c ~]# hostnamectl 
 Static hostname: aap-c.thinkmore.net
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 2c053fbe208d4448b45ba9c5ac3d1ce8
         Boot ID: d35fed8d67aa44f0aeb568fa77bd6688
  Virtualization: kvm
Operating System: Red Hat Enterprise Linux 9.4 (Plow)     
     CPE OS Name: cpe:/o:redhat:enterprise_linux:9::baseos
          Kernel: Linux 5.14.0-427.13.1.el9_4.x86_64
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
Firmware Version: 1.16.2-1.fc37

[root@aap-c ~]# 
```
* 정규화된 호스트 이름(FQDN)을 설정해야 함
<br>

### 2.2 네트워크 정보

호스트의 네트워크 정보를 확인합니다.

```bash
ip a s
ip route
```

실행 결과
```
[root@aap-c ~]# ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:78:4f:f9 brd ff:ff:ff:ff:ff:ff
    altname enp0s3
    inet 192.168.0.43/24 brd 192.168.0.255 scope global noprefixroute ens3
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe78:4ff9/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
3: ens8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:d9:cf:cd brd ff:ff:ff:ff:ff:ff
    altname enp0s8
    inet 192.168.253.43/24 brd 192.168.253.255 scope global noprefixroute ens8
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fed9:cfcd/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever

[root@aap-c ~]# ip route
default via 192.168.0.1 dev ens3 proto static metric 100 
192.168.0.0/24 dev ens3 proto kernel scope link src 192.168.0.43 metric 100 
192.168.253.0/24 dev ens8 proto kernel scope link src 192.168.253.43 metric 101 

[root@aap-c ~]# 
```
<br>

### 2.3 스토리지 정보

호스트의 스토리지 정보를 확인합니다.

```bash
df -h
```

실행 결과
```
[root@aap-c ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               4.0M     0  4.0M   0% /dev
tmpfs                  1.8G     0  1.8G   0% /dev/shm
tmpfs                  732M  9.2M  723M   2% /run
/dev/mapper/rhel-root  114G  5.0G  110G   5% /
/dev/sda1              2.0G  311M  1.7G  16% /boot
tmpfs                  366M   52K  366M   1% /run/user/42
tmpfs                  366M   36K  366M   1% /run/user/0

[root@aap-c ~]# 
```

### 2.4 DNS 정보

#### 2.4.1 기본 이름 풀이 확인

실행 명령어
```bash
ping -c 1 aap-c.thinkmore.net
```

실행 결과
```
[root@aap-c ~]# ping -c 1 aap-c.thinkmore.net
PING aap-c.thinkmore.net (192.168.0.43) 56(84) bytes of data.
64 bytes from aap-c.thinkmore.net (192.168.0.43): icmp_seq=1 ttl=64 time=0.038 ms

--- aap-c.thinkmore.net ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.038/0.038/0.038/0.000 ms

[root@aap-c ~]# 
```

#### 2.4.2 DNS 레코드 확인

실행 명려어
```bash
dig aap-c.thinkmore.net +noall +answer @192.168.0.3
```
* 대상 노드* *aap-c.thinkmore.net*
* DNS 노드: *@192.168.0.3*
* 응답만 표시: *+noall +answer*

실행 결과
```
[root@aap-c tasks]# dig aap-c.thinkmore.net +noall +answer @192.168.0.3
aap-c.thinkmore.net.    604800  IN      A       192.168.0.43

[root@aap-c tasks]# 
```
<br>
<br>

## 3. 채널 설정 및 패키지 설치

### 3.1 서브스크립션 등록

#### 3.1.1 시스템 등록

```bash 
sudo subscription-manager register --username <$INSERT_USERNAME_HERE> --password <$INSERT_PASSWORD_HERE>
```

#### 3.1.2 

```bash 
sudo subscription-manager refresh
```

#### 3.1.3 

```bash 
sudo subscription-manager identity
```

### 3.2 시스템 릴리즈 설정

시스템의 릴리즈를 맞게 설정합니다.
```bash
subscription-manager release --show
subscription-manager release --list
subscription-manager release --set 9.6
subscription-manager release --show   
```

실행 결과
```
[root@aap-c ~]# subscription-manager release --show
Release not set

[root@aap-c ~]# subscription-manager release --list
+-------------------------------------------+
          Available Releases       
+-------------------------------------------+
9
9.0
9.1
9.2
9.3
9.4
9.5
9.6

[root@aap-c ~]# subscription-manager release --set 9.6
Release set to: 9.6

[root@aap-c ~]# subscription-manager release --show   
Release: 9.6

[root@aap-c ~]#
```
<br>

### 3.3 REPO 확인

```bash
dnf repolist
```

실행 결과
```
[root@aap-c ~]# dnf repolist
Updating Subscription Management repositories.
repo id                                               repo name
rhel-9-for-x86_64-appstream-rpms                      Red Hat Enterprise Linux 9 for x86_64 - AppStream (RPMs)
rhel-9-for-x86_64-baseos-rpms                         Red Hat Enterprise Linux 9 for x86_64 - BaseOS (RPMs)

[root@aap-c ~]# 
```
<br>

### 3.4 패키지 설치

```bash
dnf install -y ansible-core git postgresql.x86_64 python3-pip rsync tcpdump wget
```

실행 결과
```
[root@aap-c ~]# dnf install -y ansible-core git postgresql.x86_64 python3-pip rsync tcpdump tmux wget
...<snip>...

[root@aap-c ~]# pip install yq
...<snip>...

[root@aap-c ~]#
```
<br>

### 3.5 시스템 업데이트

```bash
dnf update
sync; sync; sync
systemctl reboot
```

실행 결과
```
[root@aap-c ~]# dnf update
...<snip>...

[root@aap-c ~]# sync; sync; sync

[root@aap-c ~]# systemctl reboot
...<snip>...

```
<br>
<br>

## 4. 레지스트리 설정

### 4.1 레지스트리 로그인 정보 준비

#### 4.1.1 다음 URL로 이동

```
https://access.redhat.com/terms-based-registry/accounts
```

#### 4.1.2 레지스트리 서비스 계정 페이지에서 새 서비스 계정을 클릭

* 허용된 문자만 사용하여 계정 이름을 입력
  + 필요한 경우 계정에 대한 설명을 입력
  + 생성을 클릭

#### 4.1.3 검색 필드에서 이름을 검색하여 목록에서 생성된 계정 확인

* 생성한 계정의 이름을 클릭
* 또는 토큰 이름을 알고 있는 경우 URL을 입력하여 페이지로 직접 이동
  ```
  https://access.redhat.com/terms-based-registry/token/<name-of-your-token>
  ```
<br>

### 4.2 사용자 이름과 암호를 변수에 저장

#### 4.2.1 생성된 사용자 이름(계정 이름)과 토큰을 표시하는 토큰 페이지 확인

* 사용자 이름과 변수를 저장
  + 사용자 이름 (예: "1234567|shadow")을 *registry_username*으로 설정
  + 사용자 암호로써 토큰을 복사하여 *registry_password*로 설정
<br>
<br>

------
[차례](../README.md)