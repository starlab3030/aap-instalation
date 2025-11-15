# 외부에 AAP 데이터베이스 설치

## 1. 시스템 준비

### 1.1 데이터베이스 시스템 환경

#### 1.1.1 기본 정보

실행 명령어
```bash
uname -a
hostnamectl 
```

실행 결과
```
[root@aap-db ~]# uname -a
Linux aap-db.thinkmore.net 5.14.0-570.12.1.el9_6.x86_64 #1 SMP PREEMPT_DYNAMIC Fri Apr 4 10:41:31 EDT 2025 x86_64 x86_64 x86_64 GNU/Linux

[root@aap-db ~]# hostnamectl 
 Static hostname: aap-db.thinkmore.net
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: e2e0ece2a51e4fe6a23b219e6b9d3983
         Boot ID: 61b10b2f245946f4920eb4caaf3f88b4
  Virtualization: kvm
Operating System: Red Hat Enterprise Linux 9.6 (Plow)     
     CPE OS Name: cpe:/o:redhat:enterprise_linux:9::baseos
          Kernel: Linux 5.14.0-570.12.1.el9_6.x86_64
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
Firmware Version: 1.16.2-1.fc37

[root@aap-db ~]# 
```

#### 1.1.2 네트워크 정보

실행 명령어
```bash
ip --br a s
ip route
```

실행 결과
```
[root@aap-db ~]# ip --br a s
lo               UNKNOWN        127.0.0.1/8 ::1/128 
ens3             UP             192.168.0.47/24 fe80::5054:ff:fe4a:b7da/64 
ens8             UP             192.168.253.47/24 fe80::5054:ff:fe0a:23b2/64 

[root@aap-db ~]# ip route
192.168.0.0/24 dev ens3 proto kernel scope link src 192.168.0.47 metric 100 
192.168.253.0/24 dev ens8 proto kernel scope link src 192.168.253.47 metric 101 

[root@aap-db ~]# 
```

#### 1.1.3 스토리지 정보

실행 명령어
```bash
df -h
```

실행 결과
```
[root@aap-db ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               4.0M     0  4.0M   0% /dev
tmpfs                  1.8G     0  1.8G   0% /dev/shm
tmpfs                  732M  9.3M  722M   2% /run
/dev/mapper/rhel-root  112G  5.2G  107G   5% /
/dev/sda1              4.0G  386M  3.6G  10% /boot
tmpfs                  366M   36K  366M   1% /run/user/0
tmpfs                  366M  100K  366M   1% /run/user/1000

[root@aap-db ~]# 
```
<br>

### 1.2 패키지 설치 및 업데이트

#### 1.2.1 시스템 등록

실행 명령어
```bash

```

실행 결과
```
[root@aap-db ~]# subscription-manager register 
...<snip>...

[root@aap-db ~]#
```

#### 1.2.2 릴리즈 설정

실행 명령어
```bash
subscription-manager release --show
subscription-manager release --list
subscription-manager release --set 9.6
subscription-manager release --show
```

실행 결과
```
[root@aap-db ~]# subscription-manager release --show
출시가 설정되지 않음

[root@aap-db ~]# subscription-manager release --list
+-------------------------------------------+
          사용 가능한 릴리즈       
+-------------------------------------------+
9
9.0
9.1
9.2
9.3
9.4
9.5
9.6
9.7

[root@aap-db ~]# subscription-manager release --set 9.6
릴리즈가 설정됨: 9.6

[root@aap-db ~]# subscription-manager release --show
릴리즈: 9.6

[root@aap-db ~]# 
```

#### 1.2.3 패키지 설치

실행 명령어
```bash
dnf install -y postgresql python3-pip skopeo
```

실행 결과
```
[root@aap-db ~]# dnf install -y nfs-utils postgresql python3-pip skopeo
...<snip>...

[root@aap-db ~]#
```
* *nfs-utils*: NFS 클라이언트 패키지 설치
* *postgresql*: PostgreSQL 연결 테스트를 위한 클라이언트 패키지 설치
<br>

### 1.3 사용자 생성

#### 1.3.1 사용자 생성 및 암호 설정

실행 명령어
```bash
useradd shadowman
passwd shadowman 
```

실행 결과
```
[root@aap-db ~]# useradd shadowman

[root@aap-db ~]# passwd shadowman 
shadowman 사용자의 비밀 번호 변경 중
새 암호: ******
새 암호 재입력: ******
passwd: 모든 인증 토큰이 성공적으로 업데이트 되었습니다.

[root@aap-db ~]# 
```

#### 1.3.2 생성된 사용자 확인

실행 명령어
```bash
grep shadowman /etc/passwd
grep shadowman /etc/group
```

실행 결과
```
[root@aap-db ~]# grep shadowman /etc/passwd
shadowman:x:1001:1001::/home/shadowman:/bin/bash

[root@aap-db ~]# grep shadowman /etc/group
shadowman:x:1001:

[root@aap-db ~]# 
```

#### 1.3.3 사용자의 linger 활성화

실행 명령어
```bash
loginctl enable-linger shadowman
loginctl list-users
loginctl show-user shadowman
```

실행 결과
```
[root@aap-db ~]# loginctl enable-linger shadowman

[shadowman@aap-db ~]$ loginctl list-users
 UID USER      LINGER STATE    
   0 root      no     active
  42 gdm       no     active
1001 shadowman yes    lingering

[shadowman@aap-db ~]$ loginctl show-user shadowman
UID=1001
GID=1001
Name=shadowman
Timestamp=Fri 2025-11-14 10:50:09 KST
TimestampMonotonic=5365727515
RuntimePath=/run/user/1001
Service=user@1001.service
Slice=user-1001.slice
State=lingering
Sessions=
IdleHint=yes
IdleSinceHint=0
IdleSinceHintMonotonic=0
Linger=yes

[root@aap-db ~]#
```
* *podman* 사용을 위해서 linger 활성화가 필요
<br>

### 1.4 데이터 베이스 스토리지 준비

#### 1.4.1 디렉터리 생성

```bash
mkdir -pv /home/shadowman/nfs/aap/db/pgsql-data
chmod 777 /home/shadowman/nfs/aap/db/pgsql-data/
ls -ld /home/shadowman/nfs/aap/db/pgsql-data/
```

실행 결과
```
[root@aap-db ~]# mkdir -pv /home/shadowman/nfs/aap/db/pgsql-data
mkdir: created directory '/home/shadowman/nfs'
mkdir: created directory '/home/shadowman/nfs/aap'
mkdir: created directory '/home/shadowman/nfs/aap/db'
mkdir: created directory '/home/shadowman/nfs/aap/db/pgsql-data'

[root@aap-db ~]# chmod 777 /home/shadowman/nfs/aap/db/pgsql-data/

[root@aap-db ~]# ls -lh /home/shadowman/nfs/aap/db/pgsql-data/
합계 0

[root@aap-db ~]#
```

#### 1.4.2 소유권 설정

실행 명령어
```bash
chown -R shadowman:shadowman /home/shadowman/nfs
ls -ld /home/shadowman/nfs/
```

실행 결과
```
[root@aap-db log]# chown -R shadowman:shadowman /home/shadowman/nfs

[root@aap-db log]# ls -ld /home/shadowman/nfs/
drwxr-xr-x. 3 shadowman shadowman 17 11월 14 09:53 /home/shadowman/nfs/

[root@aap-db log]# 
```

#### 1.4.3 파일시스템 테이블 설정

실행 명령어
```bash
echo "192.168.0.3:/starlab/nfs/aap/db/pgsql-data /home/shadowman/nfs/aap/db/pgsql-data/ nfs noauto,rw,user 0 0" >> /etc/fstab
egrep -v "^#|^$" /etc/fstab 
```

실행 결과
```
[root@aap-db ~]# echo "192.168.0.3:/starlab/nfs/aap/db/pgsql-data /home/shadowman/nfs/aap/db/pgsql-data/ nfs noauto,rw,user 0 0" >> /etc/fstab

[root@aap-db ~]# egrep -v "^#|^$" /etc/fstab 
...<snip>...

192.168.0.3:/starlab/nfs/aap/db/pgsql-data /home/shadowman/nfs/aap/db/pgsql-data/ nfs noauto,rw,user 0 0

[root@aap-db ~]# 
```
<br>
<br>

## 2. PostgreSQL 데이터베이스 설치

### 2.1 데이티베이스 저장소 마운트

#### 2.1.1 사용자 전환

실행 명령어
```bash
su - shadowman
```

실행 결과
```
[root@aap-db ~]# su - shadowman

[shadowman@aap-db ~]$ 
```

#### 2.1.2 데이터 저장소 마운트

실행 명령어
```bash
su - shadowman
```

실행 결과
```
[shadowman@aap-db ~]$ mount ~/nfs/aap/db/pgsql-data/

[shadowman@aap-db ~]$ df -h | grep pgsql-data
192.168.0.3:/starlab/nfs/aap/db/pgsql-data  223G   27G  197G  12% /home/shadowman/nfs/aap/db/pgsql-data

[shadowman@aap-db ~]$ 
```
<br>

### 2.2 AAP를 위한 DB 이미지 가져오기 

#### 2.2.1 옵션1 - AAP 번들 패키지에서 가져 오기

이미지를 로컬 파일시스템에서 가져오기
```bash
podman load -i ~/temp/ansible-setup/bundle/images/postgresql-15.tar.gz 
podman images
```
* 글로벌 레지스트리 혹은 로컬 레지스트리 등에서 가져올 수 있음

실행 결과
```
[shadowman@aap-db ~]$ podman load -i ~/temp/ansible-setup/bundle/images/postgresql-15.tar.gz 
Getting image source signatures
Copying blob 64d064da291c done   | 
Copying blob 4493ee5cf8cf done   | 
Copying blob 8a15e950eb63 done   | 
Copying config 2392b56e6b done   | 
Writing manifest to image destination
Loaded image: registry.redhat.io/rhel8/postgresql-15:latest

[shadowman@aap-db ~]$ podman images
REPOSITORY                              TAG         IMAGE ID      CREATED      SIZE
registry.redhat.io/rhel8/postgresql-15  latest      2392b56e6b5e  11 days ago  525 MB

[shadowman@aap-db ~]$ 
```

#### 2.2.2 옵션2 - 레지스트리에서 컨테이너 이미지 다운로드

실행 명령어
```bash
podman login registry.redhat.io -u='<USER_ID>' -p='<USER_PW>'
podman pull registry.redhat.io/rhel9/postgresql-15:9.7
```

실행 결과
```
[shadowman@aap-db ~]$ podman login registry.redhat.io -u='<USER_ID>' -p='<USER_PW>'
Login Succeeded!

[shadowman@aap-db ~]$ podman pull registry.redhat.io/rhel9/postgresql-15:9.7
Trying to pull registry.redhat.io/rhel9/postgresql-15:9.7...
Getting image source signatures
Checking if image destination supports signatures
Copying blob f9924b1a7733 done   | 
Copying blob 3022abd78a96 done   | 
Copying blob 44bcdab67def done   | 
Copying config 2738b002e0 done   | 
Writing manifest to image destination
Storing signatures
2738b002e00c3ad1690f6d9df0a2df6395387900888f0681b5c80254c5118e38

[shadowman@aap-db ~]$ podman images
REPOSITORY                              TAG         IMAGE ID      CREATED       SIZE
registry.redhat.io/rhel9/postgresql-15  9.7         2738b002e00c  16 hours ago  405 MB

[shadowman@aap-db ~]$ 
```
<br>

### 2.3 PostgreSQL 컨테이너 생성

#### 2.3.1 포드맨으로 데이터베이스 생성 및 실행

~/create-pgsql-for-aap.sh 파일
```bash
# 
# Create PostgreSQL container for AAP-DB
#

# Global Env
#
DB_USER=postgres
DB_PASSWD=redhat
ADMIN_PASSWD=redhat
DB_IMG_URL=registry.redhat.io/rhel9/postgresql-15
DB_IMG_TAG=9.7

# Create & Start
#
podman run -d --name postgresql \
   -e POSTGRESQL_ADMIN_PASSWORD=$ADMIN_PASSWD \
   -p 5432:5432 \
   -v ~/nfs/aap/db/pgsql-data:/var/lib/pgsql/data \
   ${DB_IMG_URL}:${DB_IMG_TAG}
```
* **-v** 옵션에 NFS를 사용하는 경우 *:Z*를 빼고 입력
* **-u** 옵션으로 사용자를 지정하지 않는 경우 컨테이너 상의 사용자와 그룹은 postgres:postgres(26:26) 임 

다음 명령어를 실행
```bash
~/create-pgsql-for-aap.sh 
podman ps
```

실행 결과
```
[shadowman@aap-db ~]$ ./create-pgsql-for-aap.sh 
6d7bb204de2904ddc703b39f1124a9a63690190b6b4ad2cd00cd15cc81cd9f57

[shadowman@aap-db ~]$ podman ps
CONTAINER ID  IMAGE                                       COMMAND         CREATED        STATUS        PORTS                   NAMES
6d7bb204de29  registry.redhat.io/rhel9/postgresql-15:9.7  run-postgresql  4 seconds ago  Up 4 seconds  0.0.0.0:5432->5432/tcp  postgresql

[shadowman@aap-db ~]$
```

#### 2.3.2 실행 중인 로그 확인

실행 명령어
```bash
podman logs postgresql
```

실행 결과
```
[shadowman@aap-db ~]$ podman logs postgresql
Warning: Can't detect cpu quota from cgroups
Warning: Can't detect cpuset size from cgroups, will use nproc
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.utf8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

fixing permissions on existing directory /var/lib/pgsql/data/userdata ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... UTC
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok


Success. You can now start the database server using:

    pg_ctl -D /var/lib/pgsql/data/userdata -l logfile start

initdb: warning: enabling "trust" authentication for local connections
initdb: hint: You can change this by editing pg_hba.conf or using the option -A, or --auth-local and --auth-host, the next time you run initdb.
waiting for server to start....2025-11-15 01:05:13.358 UTC [30] LOG:  redirecting log output to logging collector process
2025-11-15 01:05:13.358 UTC [30] HINT:  Future log output will appear in directory "log".
 done
server started
/var/run/postgresql:5432 - accepting connections
=> sourcing /usr/share/container-scripts/postgresql/start/set_passwords.sh ...
ALTER ROLE
waiting for server to shut down.... done
server stopped
Starting server...
2025-11-15 01:05:13.982 UTC [1] LOG:  redirecting log output to logging collector process
2025-11-15 01:05:13.982 UTC [1] HINT:  Future log output will appear in directory "log".

[shadowman@aap-db ~]$
```

#### 2.3.4 데이터베이스 접속 확인

실행 명령어 
```bash
podman exec -it postgresql /bin/bash --
psql
\l
\q
exit
```

실행 결과
```
[shadowman@aap-db ~]$ podman exec -it postgresql /bin/bash --

bash-5.1$ psql
psql (15.14)
Type "help" for help.

postgres=# \l
                                                List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    | ICU Locale | Locale Provider |   Access privileges   
-----------+----------+----------+------------+------------+------------+-----------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/postgres          +
           |          |          |            |            |            |                 | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/postgres          +
           |          |          |            |            |            |                 | postgres=CTc/postgres
(3 rows)

postgres=# \q

bash-5.1$ exit
exit

[shadowman@aap-db ~]$ 
```
<br>

### 2.4 PostreSQL 컨테이너 생성 에러

유형: lsetxattr ... operation not supported
```
[shadowman@aap-db ~]$ ./create-pgsql-for-aap.sh
Error: lsetxattr /home/shadowman/nfs/aap/db/pgsql-data: operation not supported

[shadowman@aap-db ~]$
```
* 컨테이너 생성 시 -v 옵션으로 지정한 파일시스템인 NFS
* NFS가 v4 이전이면 SELinux를 지원하지 않아 발생
* -v 옵션 끝은 SELinux 설정을 지정하는 *:Z*를 지움

유형: mkdir ... Permission denied
```
[shadowman@aap-db ~]$ ./create-pgsql-for-aap.sh
mkdir: cannot create directory '/var/lib/pgsql/data/userdata': Permission denied

[shadowman@aap-db ~]$
```
* 컨테이너 생성 시 -v 옵션으로 지정한 파일시스템 소유자와 컨테이너 상의 소유자가 다른 경우
* PostgreSQL 컨테이너의 예에서는 내부 사용자가 postgres:postgres (26:26) 임
* 해당 디렉터리에 쓰기 권한이 없는 경우 발생
* setfacl 옵션으로 설정할 수 있음 (단, NFS의 경우 지원하지 않을 수 있음)
<br>
<br>

## 3. PostgreSQL 컨테이너를 사용자 서비스로 구성

### 3.1 AAP 사용자 서비스로 구성을 위한 파일 생성

#### 3.1.1 서비스 파일 생성

실행 명령어
```bash
podman ps -a
podman generate systemd --new --files --name postgresql
ls -lh /home/shadowman/container-postgresql.service 
```

실행 결과
```
[shadowman@aap-db ~]$ podman ps -a
CONTAINER ID  IMAGE                                          COMMAND         CREATED         STATUS        PORTS                   NAMES
d1df45b76621  registry.redhat.io/rhel8/postgresql-15:latest  run-postgresql  25 minutes ago  Up 7 minutes  0.0.0.0:5432->5432/tcp  postgresql

[shadowman@aap-db ~]$ podman generate systemd --new --files --name postgresql

DEPRECATED command:
It is recommended to use Quadlets for running containers and pods under systemd.

Please refer to podman-systemd.unit(5) for details.
/home/shadowman/container-postgresql.service

[shadowman@aap-db ~]$ ls -lh ~/container-postgresql.service 
-rw-r--r--. 1 shadowman shadowman 833 Oct 16 13:44 /home/shadowman/container-postgresql.service

[shadowman@aap-db ~]$
```

#### 3.1.2 서비스 파일 확인

~/container-postgresql.service 파일
```bash
# container-postgresql.service
# autogenerated by Podman 5.4.0
# Sat Nov 15 10:25:57 KST 2025

[Unit]
Description=Podman container-postgresql.service
Documentation=man:podman-generate-systemd(1)
Wants=network-online.target
After=network-online.target
RequiresMountsFor=%t/containers

[Service]
Environment=PODMAN_SYSTEMD_UNIT=%n
Restart=on-failure
TimeoutStopSec=70
ExecStart=/usr/bin/podman run \
        --cidfile=%t/%n.ctr-id \
        --cgroups=no-conmon \
        --rm \
        --sdnotify=conmon \
        --replace \
        -d \
        --name postgresql \
        -e POSTGRESQL_ADMIN_PASSWORD=redhat \
        -p 5432:5432 \
        -v /home/shadowman/nfs/aap/db/pgsql-data:/var/lib/pgsql/data registry.redhat.io/rhel9/postgresql-15:9.7
ExecStop=/usr/bin/podman stop \
        --ignore -t 10 \
        --cidfile=%t/%n.ctr-id
ExecStopPost=/usr/bin/podman rm \
        -f \
        --ignore -t 10 \
        --cidfile=%t/%n.ctr-id
Type=notify
NotifyAccess=all

[Install]
WantedBy=default.target
```
<br>

### 3.2 PostgreSQL 컨테이너 서비스 구성

#### 3.2.1 서비스 파일 생성

AAP 사용자의 서비스로 구성
```bash
mkdir -pv ~/.config/systemd/user/
cp -rp container-postgresql.service ~/.config/systemd/user/
ls -lh ~/.config/systemd/user/container-postgresql.service 
```

실행 결과
```
[shadowman@aap-db ~]$ mkdir -pv ~/.config/systemd/user/
mkdir: created directory '/home/shadowman/.config/systemd'
mkdir: created directory '/home/shadowman/.config/systemd/user/'

[shadowman@aap-db ~]$ cp -rp container-postgresql.service ~/.config/systemd/user/

[shadowman@aap-db ~]$ ls -lh ~/.config/systemd/user/container-postgresql.service 
-rw-r--r--. 1 shadowman shadowman 896 11월 15 10:25 /home/shadowman/.config/systemd/user/container-postgresql.service

[shadowman@aap-db ~]$
```

#### 3.2.2 해당 파일로 systemd 리로드

실행 명령어
```bash
systemctl --user daemon-reload
```

실행 결과
```
[shadowman@aap-db ~]$ systemctl --user daemon-reload

[shadowman@aap-db ~]$
```
<br>

### 3.3 systemd의 사용자 서비스 구성 테스트

#### 3.3.1 서비스 시작 

```bash
systemctl --user start container-postgresql.service
podman ps
systemctl  --user status container-postgresql.service 
```

실행 결과
```
[shadowman@aap-db ~]$ systemctl --user start container-postgresql.service

[shadowman@aap-db ~]$ podman ps
CONTAINER ID  IMAGE                                       COMMAND         CREATED        STATUS        PORTS                   NAMES
412cbfc07f4f  registry.redhat.io/rhel9/postgresql-15:9.7  run-postgresql  3 seconds ago  Up 4 seconds  0.0.0.0:5432->5432/tcp  postgresql

[shadowman@aap-db ~]$ systemctl --user status container-postgresql.service
● container-postgresql.service - Podman container-postgresql.service
     Loaded: loaded (/home/shadowman/.config/systemd/user/container-postgresql.service; disabled; preset: disabled)
     Active: active (running) since Sat 2025-11-15 10:46:51 KST; 10s ago
       Docs: man:podman-generate-systemd(1)
   Main PID: 6739 (conmon)
      Tasks: 2 (limit: 22975)
     Memory: 21.2M
        CPU: 207ms
     CGroup: /user.slice/user-1001.slice/user@1001.service/app.slice/container-postgresql.service
             ├─6737 /usr/bin/pasta --config-net -t 5432-5432:5432-5432 --dns-forward 169.254.1.1 -u none -T none -U none -->
             └─6739 /usr/bin/conmon --api-version 1 -c 412cbfc07f4f4a7ae9e722559cb131e7ace390220836b23c5ec46eb45f1916ff -u >

[shadowman@aap-db ~]$ 
```

#### 3.3.2 서비스 종료

```bash
systemctl --user stop container-postgresql.service
podman ps
systemctl  --user status container-postgresql.service 
```

실행 결과
```
[shadowman@aap-db ~]$ systemctl --user stop container-postgresql.service

[shadowman@aap-db ~]$ podman ps
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES

[shadowman@aap-db ~]$ systemctl  --user status container-postgresql.service 
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
○ container-postgresql.service - Podman container-postgresql.service
     Loaded: loaded (/home/shadowman/.config/systemd/user/container-postgresql.service; disabled; preset: disabled)
     Active: inactive (dead)
       Docs: man:podman-generate-systemd(1)

[shadowman@aap-db ~]$ 
```
<br>

### 3.4 사용자 세션 로그인 없이 서비스 실행

#### 3.4.1 서비스 시작 및 로그아웃

실행 명령어
```bash
systemctl --user start container-postgresql.service
logout
```

실행 결과
```
[shadowman@aap-db ~]$ systemctl --user start container-postgresql.service

[shadowman@aap-db ~]$ exit
로그아웃
Connection to aap-db closed.
```

#### 3.4.2 다른 사용자로 로그인 및 서비스 확인

AAP 사용자 세션에서 로그아웃하여, 다른 사용자로 로그인해서 확인
```bash
systemd-cgls --user-unit --cgroup-id=true
```

실행 결과
```
[root@aap-db ~]# systemd-cgls --user-unit --cgroup-id=true
Control group /:
-.slice
├─user.slice (#1308)
│ → user.invocation_id: 653366fc75514213b7d7aedc2eadf512
│ → trusted.invocation_id: 653366fc75514213b7d7aedc2eadf512
│ ├─user-1001.slice (#4084)
│ │ → user.invocation_id: 41d78141fd2f489090b2c1e09981dbc1
│ │ → trusted.invocation_id: 41d78141fd2f489090b2c1e09981dbc1
│ │ └─user@1001.service … (#4946)
│ │   → user.invocation_id: 197fa86c70864dfb917cdb5fe68491d4
│ │   → user.delegate: 1
│ │   → trusted.invocation_id: 197fa86c70864dfb917cdb5fe68491d4
│ │   → trusted.delegate: 1
│ │   ├─user.slice (#7671)
│ │   │ ├─podman-pause-009b433e.scope (#7749)
│ │   │ │ └─5926 catatonit -P
│ │   │ └─libpod-3a72a9c203ad7566fb74128b59c958cab4f688f013e7a6196d4310221fd2bc5f.scope (#9076)
│ │   │   └─container (#9115)
│ │   │     ├─6902 postgres
│ │   │     ├─6938 postgres: logger
│ │   │     ├─6939 postgres: checkpointer
│ │   │     ├─6940 postgres: background writer
│ │   │     ├─6942 postgres: walwriter
│ │   │     ├─6943 postgres: autovacuum launcher
│ │   │     └─6944 postgres: logical replication launcher
│ │   ├─app.slice (#5219)
│ │   │ ├─dbus-broker.service (#7632)
│ │   │ │ ├─5937 /usr/bin/dbus-broker-launch --scope user
│ │   │ │ └─5938 dbus-broker --log 4 --controller 9 --machine-id e2e0ece2a51e4fe6a23b219e6b9d3983 --max-bytes 1000000000000>
│ │   │ └─container-postgresql.service (#9037)
│ │   │   ├─6897 /usr/bin/pasta --config-net -t 5432-5432:5432-5432 --dns-forward 169.254.1.1 -u none -T none -U none --no->
│ │   │   └─6900 /usr/bin/conmon --api-version 1 -c 3a72a9c203ad7566fb74128b59c958cab4f688f013e7a6196d4310221fd2bc5f -u 3a7>
│ │   └─init.scope (#4985)
│ │     ├─974 /usr/lib/systemd/systemd --user
│ │     └─981 (sd-pam)

...<snip>...

[root@aap-db ~]#
```
<br>
<br>

## 4. AAP 서비스 확인

### 4.1 방화벽 구성 및 사용자 전환

#### 4.1.1 방화벽 구성

```bash
firewall-cmd --list-all
firewall-cmd --permanent --zone=public --add-service=postgresql
firewall-cmd --reload
firewall-cmd --list-all
```

실행 결과
```
[root@aap-db ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens3 ens8
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 

[root@aap-db ~]# firewall-cmd --permanent --zone=public --add-service=postgresql
success

[root@aap-db ~]# firewall-cmd --reload
success

[root@aap-db ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens3 ens8
  sources: 
  services: cockpit dhcpv6-client postgresql ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 

[root@aap-db ~]#
```

#### 4.1.2 로그아웃 및 DB 계정으로 로그인

실행 명령어
```bash
logout
ssh shadowman@aap-db
```
<br>

### 4.2 PostgreSQL 연결 테스트

#### 4.2.1 컨테이너 내부에서 테스트

```bash
podman exec -it postgresql /bin/bash --
psql
\q
exit
```

실행 결과
```
[shadowman@aap-db ~]$ podman exec -it postgresql /bin/bash --

bash-4.4$ psql
psql (15.8)
Type "help" for help.

postgres=# \l
                                                List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    | ICU Locale | Locale Provider |   Access privileges
-----------+----------+----------+------------+------------+------------+-----------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/postgres          +
           |          |          |            |            |            |                 | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/postgres          +
           |          |          |            |            |            |                 | postgres=CTc/postgres
(3 rows)

postgres=# \q

bash-4.4$ exit
exit

[shadowman@aap-db ~]$
```

#### 3.2.2 로컬 호스트에서 테스트

```bash
podman port postgresql
psql --username=postgres --host=aap-db
\q
```

실행 결과
```
[shadowman@aap-db ~]$ podman port postgresql
5432/tcp -> 0.0.0.0:5432

[shadowman@aap-db ~]$ psql --username=postgres --host=aap-db
Password for user postgres: ******
psql (13.16, server 15.8)
WARNING: psql major version 13, server major version 15.
         Some psql features might not work.
Type "help" for help.

postgres=# \q

[shadowman@aap-db ~]$
```
* 해당 컨테이너의 포트 확인
* 기본 포트로 psql 접속

#### 3.2.3 원격 호스트에서 테스트

실행 결과
```
[shadowman@aap-c ~]$ psql --username=postgres --host=aap-db
Password for user postgres: ******
psql (13.16, server 15.8)
WARNING: psql major version 13, server major version 15.
         Some psql features might not work.
Type "help" for help.

postgres=# \q

[shadowman@aap-c1 ~]$ 
```
<br>
<br>

------
[차례](../README.md)