# 운영 환경에서 Full-HA를 고려한 고성

## 1. AAP를 Full-HA로 설치

### 1.1 설치 인벤토리 파일

```ini
# This section is for your AAP Gateway host(s)
# -----------------------------------------------------
[automationgateway]
aap-gw1.thinkmore.net
aap-gw2.thinkmore.net

# This section is for your AAP Controller host(s)
# -----------------------------------------------------
[automationcontroller]
aap-c1.thinkmore.net
aap-c2.thinkmore.net

# This section is for your AAP Automation Hub host(s)
# -----------------------------------------------------
[automationhub]
aap-hub1.thinkmore.net
aap-hub2.thinkmore.net

# This section is for your AAP EDA Controller host(s)
# -----------------------------------------------------
[automationeda]
aap-eda1.thinkmore.net
aap-eda2.thinkmore.net

[redis]
aap-gw1.thinkmore.net
aap-gw2.thinkmore.net
aap-hub1.thinkmore.net
aap-hub2.thinkmore.net
aap-eda1.thinkmore.net
aap-eda2.thinkmore.net

[all:vars]

# Common variables
# -----------------------------------------------------
postgresql_admin_username=postgres
postgresql_admin_password=redhat

bundle_install=true
bundle_dir=/home/shadowman/temp/ansible-setup/bundle

# AAP Gateway
# -----------------------------------------------------
gateway_admin_password=redhat
gateway_pg_host=aap-db.thinkmore.net
gateway_pg_password=redhat

# AAP Controller
# -----------------------------------------------------
controller_admin_password=redhat
controller_pg_host=aap-db.thinkmore.net
controller_pg_password=redhat

# AAP Automation Hub
# -----------------------------------------------------
hub_admin_password=redhat
hub_pg_host=aap-db.thinkmore.net
hub_pg_password=redhat
hub_shared_data_path="192.168.0.3:/ext/nfs/aap/hub"

# AAP EDA Controller
# -----------------------------------------------------
eda_admin_password=redhat
eda_pg_host=aap-db.thinkmore.net
eda_pg_password=redhat
```
<br>

### 1.2 AAP 설치

```bash
cd ~/temp/ansible-setup/
ansible-playbook -i inventory ansible.containerized_installer.install
```

실행 결과
```
[shadowman@aap-c1 ~]$ cd ~/temp/ansible-setup/

[shadowman@aap-c1 ansible-setup]$ ansible-playbook -i inventory ansible.containerized_installer.install

...<snip>...


PLAY [Delete the ansible base shared secret] ***************************************************************************************

TASK [Delete the ansible base shared secret] ***************************************************************************************
ok: [aap-c2.thinkmore.net]
ok: [aap-c1.thinkmore.net]
ok: [aap-eda2.thinkmore.net]
ok: [aap-eda1.thinkmore.net]
ok: [aap-gw1.thinkmore.net]
ok: [aap-hub1.thinkmore.net]
ok: [aap-hub2.thinkmore.net]
ok: [aap-gw2.thinkmore.net]

PLAY RECAP *************************************************************************************************************************
aap-c1.thinkmore.net       : ok=209  changed=65   unreachable=0    failed=0    skipped=74   rescued=0    ignored=0
aap-c2.thinkmore.net       : ok=183  changed=52   unreachable=0    failed=0    skipped=60   rescued=0    ignored=0
aap-eda1.thinkmore.net     : ok=200  changed=53   unreachable=0    failed=0    skipped=55   rescued=0    ignored=0
aap-eda2.thinkmore.net     : ok=185  changed=46   unreachable=0    failed=0    skipped=52   rescued=0    ignored=0
aap-gw1.thinkmore.net      : ok=210  changed=35   unreachable=0    failed=0    skipped=62   rescued=0    ignored=0
aap-gw2.thinkmore.net      : ok=159  changed=19   unreachable=0    failed=0    skipped=61   rescued=0    ignored=0
aap-hub1.thinkmore.net     : ok=184  changed=44   unreachable=0    failed=0    skipped=85   rescued=0    ignored=0
aap-hub2.thinkmore.net     : ok=169  changed=38   unreachable=0    failed=0    skipped=79   rescued=0    ignored=0
localhost                  : ok=25   changed=0    unreachable=0    failed=0    skipped=38   rescued=0    ignored=0

[shadowman@aap-c1 ansible-setup]$
```
<br>
<br>

## 2. Automation Controller 확인

### 2.1 aap-c1.thinkmore.net 노드 확인

#### 2.1.1 이미지 리스트

```bash
podman image ls
podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
```

실행 결과
```
[shadowman@aap-c1 ~]$ podman image ls
REPOSITORY                                                          TAG         IMAGE ID      CREATED      SIZE
registry.redhat.io/rhel8/redis-6                                    latest      c99cedb5e4c1  13 days ago  329 MB
registry.redhat.io/ansible-automation-platform-25/receptor-rhel8    latest      ae9c0820d61d  2 weeks ago  598 MB
registry.redhat.io/ansible-automation-platform-25/controller-rhel8  latest      3d0cb3204030  2 weeks ago  1.74 GB

[shadowman@aap-c1 ~]$ podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
Names: registry.redhat.io/rhel8/redis-6:latest
Names: registry.redhat.io/ansible-automation-platform-25/receptor-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest

[shadowman@aap-c1 ~]$
```

#### 2.1.2 실행 중인 컨테이너 리스트

```bash
podman ps
podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
podman ps --format json | jq '[.[]|{"Image": .Image, "Names": .Names[0]}]'
```

실행 결과
```
[shadowman@aap-c1 ~]$ podman ps
CONTAINER ID  IMAGE                                                                      COMMAND               CREATED         STATUS         PORTS       NAMES
53886cc1808d  registry.redhat.io/rhel8/redis-6:latest                                    run-redis             52 minutes ago  Up 52 minutes              redis-unix
829f518ea5ea  registry.redhat.io/ansible-automation-platform-25/receptor-rhel8:latest    /usr/bin/receptor...  28 minutes ago  Up 28 minutes              receptor
52108e57d17a  registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest  /usr/bin/launch_a...  27 minutes ago  Up 23 minutes              automation-controller-rsyslog
09999e865e35  registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest  /usr/bin/launch_a...  27 minutes ago  Up 23 minutes              automation-controller-task
e2459e3d4a01  registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest  /usr/bin/launch_a...  27 minutes ago  Up 23 minutes              automation-controller-web

[shadowman@aap-c1 ~]$ podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
53886cc1808d    run-redis       redis-unix
829f518ea5ea    /usr/bin/receptor...    receptor
52108e57d17a    /usr/bin/launch_a...    automation-controller-rsyslog
09999e865e35    /usr/bin/launch_a...    automation-controller-task
e2459e3d4a01    /usr/bin/launch_a...    automation-controller-web

[shadowman@aap-c1 ~]$ podman ps --format json | jq '[.[]|{"Image": .Image, "Names": .Names[0]}]'
...<snip>...

[shadowman@aap-c1 ~]$
```

실행 중인 컨테이너 리스트 JSON 출력
```json
[
  {
    "Image": "registry.redhat.io/rhel8/redis-6:latest",
    "Names": "redis-unix"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/receptor-rhel8:latest",
    "Names": "receptor"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest",
    "Names": "automation-controller-rsyslog"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest",
    "Names": "automation-controller-task"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest",
    "Names": "automation-controller-web"
  }
]
```

#### 2.1.3 설치된 AAP

```bash
ls -ldh ~/aap
tree -F -L 2 ~/aap
```

실행 결과
```
[shadowman@aap-c1 ~]$ ls -ldh ~/aap
drwxrwx---. 7 shadowman shadowman 82 Oct 18 13:33 /home/shadowman/aap

[shadowman@aap-c1 ~]$ tree -F -L 2 ~/aap
/home/shadowman/aap
├── containers/
│   ├── podman*
│   ├── storage/
│   └── storage.conf
├── controller/
│   ├── data/
│   ├── etc/
│   ├── nginx/
│   ├── rsyslog/
│   └── supervisor/
├── receptor/
│   └── etc/
├── redis/
│   └── redis-unix.conf
└── tls/
    ├── ca.cert
    ├── ca.key
    └── extracted/

13 directories, 5 files

[shadowman@aap-c1 ~]$
```

#### 2.1.4 AAP 서비스

```bash
systemctl list-units --type=service --state=running --user
```

실행 결과
```
[shadowman@aap-c1 ~]$ systemctl list-units --type=service --state=running --user
  UNIT                                  LOAD   ACTIVE SUB     DESCRIPTION
  automation-controller-rsyslog.service loaded active running Podman automation-controller-rsyslog.service
  automation-controller-task.service    loaded active running Podman automation-controller-task.service
  automation-controller-web.service     loaded active running Podman automation-controller-web.service
  dbus-broker.service                   loaded active running D-Bus User Message Bus
  receptor.service                      loaded active running Podman receptor.service
  redis-unix.service                    loaded active running Podman redis-unix.service

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.
6 loaded units listed.

[shadowman@aap-c1 ~]$
```

#### 2.1.5 호스트 상의 컨테이너 사용 리소스

```bash
podman container stats -a
```

실행 결과
```
[shadowman@aap-c1 ~]$ podman container stats -a
ID            NAME                           CPU %       MEM USAGE / LIMIT  MEM %       NET IO      BLOCK IO    PIDS        CPU TIME    AVG CPU %
53886cc1808d  redis-unix                     0.28%       9.339MB / 8.058GB  0.12%       0B / 0B     0B / 0B     5           9.458591s   0.28%
829f518ea5ea  receptor                       0.01%       10.55MB / 8.058GB  0.13%       0B / 0B     0B / 0B     8           270.761ms   0.01%
52108e57d17a  automation-controller-rsyslog  0.40%       186.3MB / 8.058GB  2.31%       0B / 0B     0B / 0B     7           6.539283s   0.40%
09999e865e35  automation-controller-task     2.84%       662.4MB / 8.058GB  8.22%       0B / 0B     0B / 0B     26          46.476578s  2.84%
e2459e3d4a01  automation-controller-web      2.57%       1.638GB / 8.058GB  20.33%      0B / 0B     0B / 0B     21          41.793051s  2.57%

Ctrl+C

[shadowman@aap-c1 ~]$
```
<br>

### 2.2 aap-c2.thinkmore.net 노드 확인

#### 2.2.1 이미지 리스트

```bash
podman image ls
podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
```

실행 결과
```
[shadowman@aap-c2 ~]$ podman image ls
REPOSITORY                                                          TAG         IMAGE ID      CREATED      SIZE
registry.redhat.io/rhel8/redis-6                                    latest      c99cedb5e4c1  13 days ago  329 MB
registry.redhat.io/ansible-automation-platform-25/receptor-rhel8    latest      ae9c0820d61d  2 weeks ago  598 MB
registry.redhat.io/ansible-automation-platform-25/controller-rhel8  latest      3d0cb3204030  2 weeks ago  1.74 GB

[shadowman@aap-c2 ~]$ podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
Names: registry.redhat.io/rhel8/redis-6:latest
Names: registry.redhat.io/ansible-automation-platform-25/receptor-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest

[shadowman@aap-c2 ~]$
```

#### 2.2.2 실행 중인 컨테이너 리스트

```bash
podman ps
podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
podman ps --format json | jq '[.[]|{"Image": .Image, "Names": .Names[0]}]'
```

실행 결과
```
[shadowman@aap-c2 ~]$ podman ps
CONTAINER ID  IMAGE                                                                      COMMAND               CREATED            STATUS            PORTS       NAMES
54bc84af2795  registry.redhat.io/rhel8/redis-6:latest                                    run-redis             About an hour ago  Up About an hour              redis-unix
f3817dc2ecbd  registry.redhat.io/ansible-automation-platform-25/receptor-rhel8:latest    /usr/bin/receptor...  38 minutes ago     Up 38 minutes                 receptor
949ce5111aa1  registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest  /usr/bin/launch_a...  37 minutes ago     Up 34 minutes                 automation-controller-rsyslog
4050604be901  registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest  /usr/bin/launch_a...  37 minutes ago     Up 34 minutes                 automation-controller-task
13a05bbd181a  registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest  /usr/bin/launch_a...  37 minutes ago     Up 33 minutes                 automation-controller-web

[shadowman@aap-c2 ~]$ podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
54bc84af2795    run-redis       redis-unix
f3817dc2ecbd    /usr/bin/receptor...    receptor
949ce5111aa1    /usr/bin/launch_a...    automation-controller-rsyslog
4050604be901    /usr/bin/launch_a...    automation-controller-task
13a05bbd181a    /usr/bin/launch_a...    automation-controller-web

[shadowman@aap-c2 ~]$ podman ps --format json | jq '[.[]|{"Image": .Image, "Names": .Names[0]}]'
...<snip>...

[shadowman@aap-c2 ~]$
```

실행 중인 컨테이너 리스트 JSON 출력
```json
[
  {
    "Image": "registry.redhat.io/rhel8/redis-6:latest",
    "Names": "redis-unix"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/receptor-rhel8:latest",
    "Names": "receptor"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest",
    "Names": "automation-controller-rsyslog"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest",
    "Names": "automation-controller-task"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest",
    "Names": "automation-controller-web"
  }
]
```

#### 2.2.3 설치된 AAP

```bash
ls -ldh ~/aap
tree -F -L 2 ~/aap
```

실행 결과
```
[shadowman@aap-c2 ~]$ ls -ldh ~/aap
drwxrwx---. 7 shadowman shadowman 82 Oct 18 13:33 /home/shadowman/aap

[shadowman@aap-c2 ~]$ tree -F -L 2 ~/aap
/home/shadowman/aap
├── containers/
│   ├── podman*
│   ├── storage/
│   └── storage.conf
├── controller/
│   ├── data/
│   ├── etc/
│   ├── nginx/
│   ├── rsyslog/
│   └── supervisor/
├── receptor/
│   └── etc/
├── redis/
│   └── redis-unix.conf
└── tls/
    ├── ca.cert
    ├── ca.key
    └── extracted/

13 directories, 5 files

[shadowman@aap-c2 ~]$
```

#### 2.2.4 AAP 서비스

```bash
systemctl list-units --type=service --state=running --user
```

실행 결과
```
[shadowman@aap-c2 ~]$ systemctl list-units --type=service --state=running --user
  UNIT                                  LOAD   ACTIVE SUB     DESCRIPTION
  automation-controller-rsyslog.service loaded active running Podman automation-controller-rsyslog.service
  automation-controller-task.service    loaded active running Podman automation-controller-task.service
  automation-controller-web.service     loaded active running Podman automation-controller-web.service
  dbus-broker.service                   loaded active running D-Bus User Message Bus
  receptor.service                      loaded active running Podman receptor.service
  redis-unix.service                    loaded active running Podman redis-unix.service

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.
6 loaded units listed.

[shadowman@aap-c2 ~]$
```

#### 2.2.5 호스트 상의 컨테이너 사용 리소스

```bash
podman container stats -a
```

실행 결과
```
[shadowman@aap-c2 ~]$ podman container stats -a
ID            NAME                           CPU %       MEM USAGE / LIMIT  MEM %       NET IO      BLOCK IO    PIDS        CPU TIME    AVG CPU %
54bc84af2795  redis-unix                     0.29%       9.658MB / 8.058GB  0.12%       0B / 0B     0B / 0B     5           11.472286s  0.29%
f3817dc2ecbd  receptor                       0.01%       10.89MB / 8.058GB  0.14%       0B / 0B     0B / 0B     8           364.665ms   0.01%
949ce5111aa1  automation-controller-rsyslog  0.32%       186.7MB / 8.058GB  2.32%       0B / 0B     0B / 0B     7           7.103114s   0.32%
4050604be901  automation-controller-task     2.69%       780.2MB / 8.058GB  9.68%       0B / 0B     0B / 0B     26          59.396059s  2.69%
13a05bbd181a  automation-controller-web      2.08%       1.647GB / 8.058GB  20.43%      0B / 0B     0B / 0B     21          45.736476s  2.08%

Ctrl+C

[shadowman@aap-c2 ~]$
```
<br>
<br>

## 3. Event-Driven Ansible 확인

### 3.1 aap-eda1.thinkmore.net 노드 확인

#### 3.1.1 이미지 리스트

```bash
podman image ls
podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
```

실행 결과
```
[shadowman@aap-eda1 ~]$ podman image ls
REPOSITORY                                                                 TAG         IMAGE ID      CREATED      SIZE
registry.redhat.io/rhel8/redis-6                                           latest      c99cedb5e4c1  13 days ago  329 MB
registry.redhat.io/ansible-automation-platform-25/eda-controller-ui-rhel8  latest      04ae1753b4d7  2 weeks ago  512 MB
registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8     latest      1f5b20f9596d  2 weeks ago  1.01 GB

[shadowman@aap-eda1 ~]$ podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
Names: registry.redhat.io/rhel8/redis-6:latest
Names: registry.redhat.io/ansible-automation-platform-25/eda-controller-ui-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest

[shadowman@aap-eda1 ~]$
```

#### 3.1.2 실행 중인 컨테이너 리스트

```bash
podman ps
podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
podman ps --format json | jq '[.[]|{"Image": .Image, "Names": .Names[0]}]'
```

실행 결과
```
[shadowman@aap-eda1 ~]$ podman ps
CONTAINER ID  IMAGE                                                                             COMMAND               CREATED            STATUS         PORTS       NAMES
2e64e4748733  registry.redhat.io/rhel8/redis-6:latest                                           run-redis             About an hour ago  Up 45 minutes              redis-tcp
f2151ee12847  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     gunicorn --bind 1...  37 minutes ago     Up 36 minutes              automation-eda-api
f299465e8c6d  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     daphne --bind 127...  36 minutes ago     Up 35 minutes              automation-eda-daphne
0c21d117548d  registry.redhat.io/ansible-automation-platform-25/eda-controller-ui-rhel8:latest  /bin/sh -c nginx ...  36 minutes ago     Up 35 minutes              automation-eda-web
e1de2bee29dd  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     aap-eda-manage rq...  36 minutes ago     Up 35 minutes              automation-eda-worker-1
a7ea81ff64e5  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     aap-eda-manage rq...  36 minutes ago     Up 35 minutes              automation-eda-worker-2
654d4051fa7e  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     aap-eda-manage rq...  36 minutes ago     Up 35 minutes              automation-eda-activation-worker-1
76e250ade176  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     aap-eda-manage rq...  36 minutes ago     Up 35 minutes              automation-eda-activation-worker-2
7bd05c3582bd  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     aap-eda-manage sc...  36 minutes ago     Up 35 minutes              automation-eda-scheduler

[shadowman@aap-eda1 ~]$ podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
2e64e4748733    run-redis       redis-tcp
f2151ee12847    gunicorn --bind 1...    automation-eda-api
f299465e8c6d    daphne --bind 127...    automation-eda-daphne
0c21d117548d    /bin/sh -c nginx ...    automation-eda-web
e1de2bee29dd    aap-eda-manage rq...    automation-eda-worker-1
a7ea81ff64e5    aap-eda-manage rq...    automation-eda-worker-2
654d4051fa7e    aap-eda-manage rq...    automation-eda-activation-worker-1
76e250ade176    aap-eda-manage rq...    automation-eda-activation-worker-2
7bd05c3582bd    aap-eda-manage sc...    automation-eda-scheduler

[shadowman@aap-eda1 ~]$ podman ps --format json | jq '[.[]|{"Image": .Image, "Names": .Names[0]}]'
...<snip>...

[shadowman@aap-eda1 ~]$
```

실행 중인 컨테이너 리스트 JSON 출력
```json
[
  {
    "Image": "registry.redhat.io/rhel8/redis-6:latest",
    "Names": "redis-tcp"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest",
    "Names": "automation-eda-api"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest",
    "Names": "automation-eda-daphne"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/eda-controller-ui-rhel8:latest",
    "Names": "automation-eda-web"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest",
    "Names": "automation-eda-worker-1"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest",
    "Names": "automation-eda-worker-2"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest",
    "Names": "automation-eda-activation-worker-1"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest",
    "Names": "automation-eda-activation-worker-2"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest",
    "Names": "automation-eda-scheduler"
  }
]
```

#### 3.1.3 설치된 AAP

```bash
ls -ldh ~/aap
tree -F -L 2 ~/aap
```

실행 결과
```
[shadowman@aap-eda1 ~]$ ls -ldh ~/aap
drwxrwx---. 6 shadowman shadowman 59 Oct 18 13:38 /home/shadowman/aap

[shadowman@aap-eda1 ~]$ tree -F -L 2 ~/aap
/home/shadowman/aap
├── containers/
│   ├── podman*
│   ├── storage/
│   └── storage.conf
├── eda/
│   ├── etc/
│   └── nginx/
├── redis/
│   ├── cluster.init
│   ├── redis-tcp.conf
│   ├── redis-users.acl
│   ├── redis_nodes.conf
│   ├── server.crt
│   └── server.key
└── tls/
    ├── ca.cert
    ├── ca.key
    └── extracted/

8 directories, 10 files

[shadowman@aap-eda1 ~]$
```

#### 3.1.4 AAP 서비스

```bash
systemctl list-units --type=service --state=running --user
```

실행 결과
```

```

#### 3.1.5 호스트 상의 컨테이너 사용 리소스

```bash
podman container stats -a
```

실행 결과
```
[shadowman@aap-eda1 ~]$ systemctl list-units --type=service --state=running --user
  UNIT                                       LOAD   ACTIVE SUB     DESCRIPTION
  automation-eda-activation-worker-1.service loaded active running Podman automation-eda-activation-worker-1.service
  automation-eda-activation-worker-2.service loaded active running Podman automation-eda-activation-worker-2.service
  automation-eda-api.service                 loaded active running Podman automation-eda-api.service
  automation-eda-daphne.service              loaded active running Podman automation-eda-daphne.service
  automation-eda-scheduler.service           loaded active running Podman automation-eda-scheduler.service
  automation-eda-web.service                 loaded active running Podman automation-eda-web.service
  automation-eda-worker-1.service            loaded active running Podman automation-eda-worker-1.service
  automation-eda-worker-2.service            loaded active running Podman automation-eda-worker-2.service
  dbus-broker.service                        loaded active running D-Bus User Message Bus
  redis-tcp.service                          loaded active running Podman redis-tcp.service

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.
10 loaded units listed.

[shadowman@aap-eda1 ~]$
```
<br>

### 3.2 aap-eda2.thinkmore.net 노드 확인

#### 3.2.1 이미지 리스트

```bash
podman image ls
podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
```

실행 결과
```
[shadowman@aap-eda2 ~]$ podman image ls
REPOSITORY                                                                 TAG         IMAGE ID      CREATED      SIZE
registry.redhat.io/rhel8/redis-6                                           latest      c99cedb5e4c1  13 days ago  329 MB
registry.redhat.io/ansible-automation-platform-25/eda-controller-ui-rhel8  latest      04ae1753b4d7  2 weeks ago  512 MB
registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8     latest      1f5b20f9596d  2 weeks ago  1.01 GB

[shadowman@aap-eda2 ~]$ podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
Names: registry.redhat.io/rhel8/redis-6:latest
Names: registry.redhat.io/ansible-automation-platform-25/eda-controller-ui-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest

[shadowman@aap-eda2 ~]$
```

#### 3.2.2 실행 중인 컨테이너 리스트

```bash
podman ps
podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
podman ps --format json | jq '[.[]|{"Image": .Image, "Names": .Names[0]}]'
```

실행 결과
```
[shadowman@aap-eda2 ~]$ podman ps
CONTAINER ID  IMAGE                                                                             COMMAND               CREATED            STATUS         PORTS       NAMES
4eb6b77d20ee  registry.redhat.io/rhel8/redis-6:latest                                           run-redis             About an hour ago  Up 49 minutes              redis-tcp
f94fa6418e40  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     gunicorn --bind 1...  41 minutes ago     Up 40 minutes              automation-eda-api
62a0c70d119f  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     daphne --bind 127...  40 minutes ago     Up 39 minutes              automation-eda-daphne
eeb2c4213258  registry.redhat.io/ansible-automation-platform-25/eda-controller-ui-rhel8:latest  /bin/sh -c nginx ...  40 minutes ago     Up 39 minutes              automation-eda-web
2d58dc32134c  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     aap-eda-manage rq...  40 minutes ago     Up 39 minutes              automation-eda-worker-1
1079cdf227ec  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     aap-eda-manage rq...  40 minutes ago     Up 39 minutes              automation-eda-worker-2
72321c8a4f9e  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     aap-eda-manage rq...  40 minutes ago     Up 39 minutes              automation-eda-activation-worker-1
d13080dcab28  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     aap-eda-manage rq...  40 minutes ago     Up 39 minutes              automation-eda-activation-worker-2
4988060dda9b  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     aap-eda-manage sc...  40 minutes ago     Up 39 minutes              automation-eda-scheduler

[shadowman@aap-eda2 ~]$ podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
4eb6b77d20ee    run-redis       redis-tcp
f94fa6418e40    gunicorn --bind 1...    automation-eda-api
62a0c70d119f    daphne --bind 127...    automation-eda-daphne
eeb2c4213258    /bin/sh -c nginx ...    automation-eda-web
2d58dc32134c    aap-eda-manage rq...    automation-eda-worker-1
1079cdf227ec    aap-eda-manage rq...    automation-eda-worker-2
72321c8a4f9e    aap-eda-manage rq...    automation-eda-activation-worker-1
d13080dcab28    aap-eda-manage rq...    automation-eda-activation-worker-2
4988060dda9b    aap-eda-manage sc...    automation-eda-scheduler

[shadowman@aap-eda2 ~]$ podman ps --format json | jq '[.[]|{"Image": .Image, "Names": .Names[0]}]'
...<snip>...

[shadowman@aap-eda2 ~]$
```

실행 중인 컨테이너 리스트 JSON 출력
```json
[
  {
    "Image": "registry.redhat.io/rhel8/redis-6:latest",
    "Names": "redis-tcp"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest",
    "Names": "automation-eda-api"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest",
    "Names": "automation-eda-daphne"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/eda-controller-ui-rhel8:latest",
    "Names": "automation-eda-web"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest",
    "Names": "automation-eda-worker-1"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest",
    "Names": "automation-eda-worker-2"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest",
    "Names": "automation-eda-activation-worker-1"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest",
    "Names": "automation-eda-activation-worker-2"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest",
    "Names": "automation-eda-scheduler"
  }
]
```

#### 3.2.3 설치된 AAP

```bash
ls -ldh ~/aap
tree -F -L 2 ~/aap
```

실행 결과
```
[shadowman@aap-eda2 ~]$ ls -ldh ~/aap
drwxrwx---. 6 shadowman shadowman 59 Oct 18 13:38 /home/shadowman/aap

[shadowman@aap-eda2 ~]$ tree -F -L 2 ~/aap
/home/shadowman/aap
├── containers/
│   ├── podman*
│   ├── storage/
│   └── storage.conf
├── eda/
│   ├── etc/
│   └── nginx/
├── redis/
│   ├── cluster.init
│   ├── redis-tcp.conf
│   ├── redis-users.acl
│   ├── redis_nodes.conf
│   ├── server.crt
│   └── server.key
└── tls/
    ├── ca.cert
    ├── ca.key
    └── extracted/

8 directories, 10 files

[shadowman@aap-eda2 ~]$
```

#### 3.2.4 AAP 서비스

```bash
systemctl list-units --type=service --state=running --user
```

실행 결과
```
[shadowman@aap-eda2 ~]$ systemctl list-units --type=service --state=running --user
  UNIT                                       LOAD   ACTIVE SUB     DESCRIPTION
  automation-eda-activation-worker-1.service loaded active running Podman automation-eda-activation-worker-1.service
  automation-eda-activation-worker-2.service loaded active running Podman automation-eda-activation-worker-2.service
  automation-eda-api.service                 loaded active running Podman automation-eda-api.service
  automation-eda-daphne.service              loaded active running Podman automation-eda-daphne.service
  automation-eda-scheduler.service           loaded active running Podman automation-eda-scheduler.service
  automation-eda-web.service                 loaded active running Podman automation-eda-web.service
  automation-eda-worker-1.service            loaded active running Podman automation-eda-worker-1.service
  automation-eda-worker-2.service            loaded active running Podman automation-eda-worker-2.service
  dbus-broker.service                        loaded active running D-Bus User Message Bus
  redis-tcp.service                          loaded active running Podman redis-tcp.service

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.
10 loaded units listed.

[shadowman@aap-eda2 ~]$
```

#### 3.2.5 호스트 상의 컨테이너 사용 리소스

```bash
podman container stats -a
```

실행 결과
```
[shadowman@aap-eda2 ~]$ podman container stats -a
ID            NAME                                CPU %       MEM USAGE / LIMIT  MEM %       NET IO      BLOCK IO    PIDS        CPU TIME      AVG CPU %
4eb6b77d20ee  redis-tcp                           0.42%       18.08MB / 3.837GB  0.47%       0B / 0B     0B / 0B     6           13.09897s     0.42%
f94fa6418e40  automation-eda-api                  0.52%       610.1MB / 3.837GB  15.90%      0B / 0B     0B / 0B     7           13.366466s    0.52%
62a0c70d119f  automation-eda-daphne               0.10%       119.5MB / 3.837GB  3.11%       0B / 0B     0B / 0B     2           2.470344s     0.10%
eeb2c4213258  automation-eda-web                  0.01%       3.932MB / 3.837GB  0.10%       0B / 0B     0B / 0B     3           237.496ms     0.01%
2d58dc32134c  automation-eda-worker-1             2.50%       128.7MB / 3.837GB  3.35%       0B / 0B     0B / 0B     3           1m4.410534s   2.50%
1079cdf227ec  automation-eda-worker-2             5.10%       127.1MB / 3.837GB  3.31%       0B / 0B     0B / 0B     3           2m11.159253s  5.10%
72321c8a4f9e  automation-eda-activation-worker-1  0.34%       128.3MB / 3.837GB  3.34%       0B / 0B     0B / 0B     3           8.853947s     0.34%
d13080dcab28  automation-eda-activation-worker-2  0.35%       127.8MB / 3.837GB  3.33%       0B / 0B     0B / 0B     3           8.926015s     0.35%
4988060dda9b  automation-eda-scheduler            0.20%       139.5MB / 3.837GB  3.64%       0B / 0B     0B / 0B     2           5.151457s     0.20%

Ctrl+C

[shadowman@aap-eda2 ~]$
```
<br>
<br>

## 4. Automation Hub 확인

### 4.1 aap-hub1.thinkmore.net 노드 확인

#### 4.1.1 이미지 리스트

```bash
podman image ls
podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
```

실행 결과
```
[shadowman@aap-hub1 ~]$ podman image ls
REPOSITORY                                                       TAG         IMAGE ID      CREATED      SIZE
registry.redhat.io/rhel8/redis-6                                 latest      c99cedb5e4c1  13 days ago  329 MB
registry.redhat.io/ansible-automation-platform-25/hub-web-rhel8  latest      741a253ea19e  2 weeks ago  508 MB
registry.redhat.io/ansible-automation-platform-25/hub-rhel8      latest      b9a82993ab1c  2 weeks ago  1.3 GB

[shadowman@aap-hub1 ~]$ podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
Names: registry.redhat.io/rhel8/redis-6:latest
Names: registry.redhat.io/ansible-automation-platform-25/hub-web-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest

[shadowman@aap-hub1 ~]$
```

#### 4.1.2 실행 중인 컨테이너 리스트

```bash
podman ps
podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
podman ps --format json | jq '[.[]|{"Image": .Image, "Names": .Names[0]}]'
```

실행 결과
```
[shadowman@aap-hub1 ~]$ podman ps
CONTAINER ID  IMAGE                                                                   COMMAND               CREATED            STATUS            PORTS       NAMES
341ec7e6e99b  registry.redhat.io/rhel8/redis-6:latest                                 run-redis             About an hour ago  Up 53 minutes                 redis-tcp
048e68a91a19  registry.redhat.io/rhel8/redis-6:latest                                 run-redis             About an hour ago  Up About an hour              redis-unix
728f22fa442a  registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest      pulpcore-api --na...  43 minutes ago     Up 41 minutes                 automation-hub-api
2fe52375dc6f  registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest      pulpcore-content ...  43 minutes ago     Up 41 minutes                 automation-hub-content
f1f2325011d5  registry.redhat.io/ansible-automation-platform-25/hub-web-rhel8:latest  /bin/sh -c nginx ...  43 minutes ago     Up 41 minutes                 automation-hub-web
cf3bd8c38261  registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest      pulpcore-worker       43 minutes ago     Up 41 minutes                 automation-hub-worker-1
d9d457f2001e  registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest      pulpcore-worker       43 minutes ago     Up 41 minutes                 automation-hub-worker-2

[shadowman@aap-hub1 ~]$ podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
341ec7e6e99b    run-redis       redis-tcp
048e68a91a19    run-redis       redis-unix
728f22fa442a    pulpcore-api --na...    automation-hub-api
2fe52375dc6f    pulpcore-content ...    automation-hub-content
f1f2325011d5    /bin/sh -c nginx ...    automation-hub-web
cf3bd8c38261    pulpcore-worker automation-hub-worker-1
d9d457f2001e    pulpcore-worker automation-hub-worker-2

[shadowman@aap-hub1 ~]$ podman ps --format json | jq '[.[]|{"Image": .Image, "Names": .Names[0]}]'
...<snip>...

[shadowman@aap-hub1 ~]$
```

실행 중인 컨테이너 리스트 JSON 출력
```json
[
  {
    "Image": "registry.redhat.io/rhel8/redis-6:latest",
    "Names": "redis-tcp"
  },
  {
    "Image": "registry.redhat.io/rhel8/redis-6:latest",
    "Names": "redis-unix"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest",
    "Names": "automation-hub-api"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest",
    "Names": "automation-hub-content"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/hub-web-rhel8:latest",
    "Names": "automation-hub-web"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest",
    "Names": "automation-hub-worker-1"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest",
    "Names": "automation-hub-worker-2"
  }
]
```

#### 4.1.3 설치된 AAP

```bash
ls -ldh ~/aap
tree -F -L 2 ~/aap
```

실행 결과
```
[shadowman@aap-hub1 ~]$ ls -ldh ~/aap
drwxr-x---. 5 shadowman shadowman 41 Oct 18 13:40 /home/shadowman/aap

[shadowman@aap-hub1 ~]$ tree -F -L 2 ~/aap
/home/shadowman/aap
├── hub/
│   ├── data/
│   ├── etc/
│   └── nginx/
├── redis/
│   ├── cluster.init
│   ├── redis-tcp.conf
│   ├── redis-unix.conf
│   ├── redis-users.acl
│   ├── redis_nodes.conf
│   ├── server.crt
│   └── server.key
└── tls/
    ├── ca.cert
    ├── ca.key
    └── extracted/

7 directories, 9 files

[shadowman@aap-hub1 ~]$
```

#### 4.1.4 AAP 서비스

```bash
systemctl list-units --type=service --state=running --user
```

실행 결과
```
[shadowman@aap-hub1 ~]$ systemctl list-units --type=service --state=running --user
  UNIT                            LOAD   ACTIVE SUB     DESCRIPTION
  automation-hub-api.service      loaded active running Podman automation-hub-api.service
  automation-hub-content.service  loaded active running Podman automation-hub-content.service
  automation-hub-web.service      loaded active running Podman automation-hub-web.service
  automation-hub-worker-1.service loaded active running Podman automation-hub-worker-1.service
  automation-hub-worker-2.service loaded active running Podman automation-hub-worker-2.service
  dbus-broker.service             loaded active running D-Bus User Message Bus
  redis-tcp.service               loaded active running Podman redis-tcp.service
  redis-unix.service              loaded active running Podman redis-unix.service

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.
8 loaded units listed.

[shadowman@aap-hub1 ~]$
```

#### 4.1.5 호스트 상의 컨테이너 사용 리소스

```bash
podman container stats -a
```

실행 결과
```
[shadowman@aap-hub1 ~]$ podman container stats -a
ID            NAME                     CPU %       MEM USAGE / LIMIT  MEM %       NET IO      BLOCK IO    PIDS        CPU TIME    AVG CPU %
341ec7e6e99b  redis-tcp                0.62%       17.63MB / 3.837GB  0.46%       0B / 0B     0B / 0B     6           20.756194s  0.62%
048e68a91a19  redis-unix               0.22%       9.417MB / 3.837GB  0.25%       0B / 0B     0B / 0B     5           10.017442s  0.22%
728f22fa442a  automation-hub-api       1.41%       647.6MB / 3.837GB  16.87%      0B / 0B     0B / 0B     6           37.28404s   1.41%
2fe52375dc6f  automation-hub-content   0.45%       163.9MB / 3.837GB  4.27%       0B / 0B     0B / 0B     5           11.992046s  0.45%
f1f2325011d5  automation-hub-web       0.01%       3.985MB / 3.837GB  0.10%       0B / 0B     0B / 0B     3           193.767ms   0.01%
cf3bd8c38261  automation-hub-worker-1  0.23%       120.8MB / 3.837GB  3.15%       0B / 0B     0B / 0B     1           6.106798s   0.23%
d9d457f2001e  automation-hub-worker-2  0.21%       121.1MB / 3.837GB  3.15%       0B / 0B     0B / 0B     1           5.486218s   0.21%

Ctrl+C

[shadowman@aap-hub1 ~]$
```
<br>

### 4.2 aap-hub2.thinkmore.net 노드 확인

#### 4.2.1 이미지 리스트

```bash
podman image ls
podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
```

실행 결과
```
[shadowman@aap-hub2 ~]$ podman image ls
REPOSITORY                                                       TAG         IMAGE ID      CREATED      SIZE
registry.redhat.io/rhel8/redis-6                                 latest      c99cedb5e4c1  13 days ago  329 MB
registry.redhat.io/ansible-automation-platform-25/hub-web-rhel8  latest      741a253ea19e  2 weeks ago  508 MB
registry.redhat.io/ansible-automation-platform-25/hub-rhel8      latest      b9a82993ab1c  2 weeks ago  1.3 GB

[shadowman@aap-hub2 ~]$ podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
Names: registry.redhat.io/rhel8/redis-6:latest
Names: registry.redhat.io/ansible-automation-platform-25/hub-web-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest

[shadowman@aap-hub2 ~]$
```

#### 4.2.2 실행 중인 컨테이너 리스트

```bash
podman ps
podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
podman ps --format json | jq '[.[]|{"Image": .Image, "Names": .Names[0]}]'
```

실행 결과
```
[shadowman@aap-hub2 ~]$ podman ps
CONTAINER ID  IMAGE                                                                   COMMAND               CREATED            STATUS            PORTS       NAMES
7bf29d3e9fb5  registry.redhat.io/rhel8/redis-6:latest                                 run-redis             About an hour ago  Up 57 minutes                 redis-tcp
3b80ac7d25c1  registry.redhat.io/rhel8/redis-6:latest                                 run-redis             About an hour ago  Up About an hour              redis-unix
7b27bc12aef1  registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest      pulpcore-api --na...  47 minutes ago     Up 45 minutes                 automation-hub-api
e5da3a71a02d  registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest      pulpcore-content ...  46 minutes ago     Up 45 minutes                 automation-hub-content
ddf6c538b648  registry.redhat.io/ansible-automation-platform-25/hub-web-rhel8:latest  /bin/sh -c nginx ...  46 minutes ago     Up 45 minutes                 automation-hub-web
3f3d36a6fa41  registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest      pulpcore-worker       46 minutes ago     Up 45 minutes                 automation-hub-worker-1
9425d2befb47  registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest      pulpcore-worker       46 minutes ago     Up 45 minutes                 automation-hub-worker-2

[shadowman@aap-hub2 ~]$ podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
7bf29d3e9fb5    run-redis       redis-tcp
3b80ac7d25c1    run-redis       redis-unix
7b27bc12aef1    pulpcore-api --na...    automation-hub-api
e5da3a71a02d    pulpcore-content ...    automation-hub-content
ddf6c538b648    /bin/sh -c nginx ...    automation-hub-web
3f3d36a6fa41    pulpcore-worker automation-hub-worker-1
9425d2befb47    pulpcore-worker automation-hub-worker-2

[shadowman@aap-hub2 ~]$ podman ps --format json | jq '[.[]|{"Image": .Image, "Names": .Names[0]}]'
...<snip>...

[shadowman@aap-hub2 ~]$
```

실행 중인 컨테이너 리스트 JSON 출력
```json
[
  {
    "Image": "registry.redhat.io/rhel8/redis-6:latest",
    "Names": "redis-tcp"
  },
  {
    "Image": "registry.redhat.io/rhel8/redis-6:latest",
    "Names": "redis-unix"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest",
    "Names": "automation-hub-api"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest",
    "Names": "automation-hub-content"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/hub-web-rhel8:latest",
    "Names": "automation-hub-web"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest",
    "Names": "automation-hub-worker-1"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest",
    "Names": "automation-hub-worker-2"
  }
]
```

#### 4.2.3 설치된 AAP

```bash
ls -ldh ~/aap
tree -F -L 2 ~/aap
```

실행 결과
```
[shadowman@aap-hub2 ~]$ ls -ldh ~/aap
drwxr-x---. 5 shadowman shadowman 41 Oct 18 13:40 /home/shadowman/aap

[shadowman@aap-hub2 ~]$ tree -F -L 2 ~/aap
/home/shadowman/aap
├── hub/
│   ├── data/
│   ├── etc/
│   └── nginx/
├── redis/
│   ├── cluster.init
│   ├── redis-tcp.conf
│   ├── redis-unix.conf
│   ├── redis-users.acl
│   ├── redis_nodes.conf
│   ├── server.crt
│   └── server.key
└── tls/
    ├── ca.cert
    ├── ca.key
    └── extracted/

7 directories, 9 files

[shadowman@aap-hub2 ~]$
```

#### 2.2.4 AAP 서비스

```bash
systemctl list-units --type=service --state=running --user
```

실행 결과
```
[shadowman@aap-hub2 ~]$ systemctl list-units --type=service --state=running --user
  UNIT                            LOAD   ACTIVE SUB     DESCRIPTION
  automation-hub-api.service      loaded active running Podman automation-hub-api.service
  automation-hub-content.service  loaded active running Podman automation-hub-content.service
  automation-hub-web.service      loaded active running Podman automation-hub-web.service
  automation-hub-worker-1.service loaded active running Podman automation-hub-worker-1.service
  automation-hub-worker-2.service loaded active running Podman automation-hub-worker-2.service
  dbus-broker.service             loaded active running D-Bus User Message Bus
  redis-tcp.service               loaded active running Podman redis-tcp.service
  redis-unix.service              loaded active running Podman redis-unix.service

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.
8 loaded units listed.

[shadowman@aap-hub2 ~]$
```

#### 4.2.5 호스트 상의 컨테이너 사용 리소스

```bash
podman container stats -a
```

실행 결과
```
[shadowman@aap-hub2 ~]$ podman container stats -a
ID            NAME                     CPU %       MEM USAGE / LIMIT  MEM %       NET IO      BLOCK IO    PIDS        CPU TIME    AVG CPU %
7bf29d3e9fb5  redis-tcp                0.43%       18.25MB / 3.837GB  0.48%       0B / 0B     0B / 0B     6           15.22589s   0.43%
3b80ac7d25c1  redis-unix               0.21%       9.417MB / 3.837GB  0.25%       0B / 0B     0B / 0B     5           10.222176s  0.21%
7b27bc12aef1  automation-hub-api       1.40%       653.2MB / 3.837GB  17.02%      0B / 0B     0B / 0B     6           39.724449s  1.40%
e5da3a71a02d  automation-hub-content   0.45%       166.2MB / 3.837GB  4.33%       0B / 0B     0B / 0B     5           12.744281s  0.45%
ddf6c538b648  automation-hub-web       0.01%       3.977MB / 3.837GB  0.10%       0B / 0B     0B / 0B     3           222.454ms   0.01%
3f3d36a6fa41  automation-hub-worker-1  0.22%       120.8MB / 3.837GB  3.15%       0B / 0B     0B / 0B     1           6.143296s   0.22%
9425d2befb47  automation-hub-worker-2  0.21%       120.7MB / 3.837GB  3.14%       0B / 0B     0B / 0B     1           5.832503s   0.21%

Ctrl+C

[shadowman@aap-hub2 ~]$
```
<br>
<br>

## 5. Automation Gateway 확인

### 5.1 aap-gw1.thinkmore.net 노드 확인

#### 5.1.1 이미지 리스트

```bash
podman image ls
podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
```

실행 결과
```
[shadowman@aap-gw1 ~]$ podman image ls
REPOSITORY                                                             TAG         IMAGE ID      CREATED      SIZE
registry.redhat.io/rhel8/redis-6                                       latest      c99cedb5e4c1  13 days ago  329 MB
registry.redhat.io/ansible-automation-platform-25/gateway-rhel8        latest      f768a98a64be  2 weeks ago  940 MB
registry.redhat.io/ansible-automation-platform-25/gateway-proxy-rhel8  latest      6e53887286a7  2 weeks ago  497 MB

[shadowman@aap-gw1 ~]$ podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
Names: registry.redhat.io/rhel8/redis-6:latest
Names: registry.redhat.io/ansible-automation-platform-25/gateway-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/gateway-proxy-rhel8:latest

[shadowman@aap-gw1 ~]$
```

#### 5.1.2 실행 중인 컨테이너 리스트

```bash
podman ps
podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
podman ps --format json | jq '[.[]|{"Image": .Image, "Names": .Names[0]}]'
```

실행 결과
```
[shadowman@aap-gw1 ~]$ podman ps
CONTAINER ID  IMAGE                                                                         COMMAND               CREATED            STATUS            PORTS       NAMES
02bd4eddba17  registry.redhat.io/rhel8/redis-6:latest                                       run-redis             About an hour ago  Up About an hour              redis-tcp
e06b9df22aea  registry.redhat.io/ansible-automation-platform-25/gateway-proxy-rhel8:latest  /usr/bin/envoy --...  59 minutes ago     Up 58 minutes                 automation-gateway-proxy
01cb536a7125  registry.redhat.io/ansible-automation-platform-25/gateway-rhel8:latest        /usr/bin/supervis...  59 minutes ago     Up 58 minutes                 automation-gateway

[shadowman@aap-gw1 ~]$ podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
02bd4eddba17    run-redis       redis-tcp
e06b9df22aea    /usr/bin/envoy --...    automation-gateway-proxy
01cb536a7125    /usr/bin/supervis...    automation-gateway

[shadowman@aap-gw1 ~]$ podman ps --format json | jq '[.[]|{"Image": .Image, "Names": .Names[0]}]'
...<snip>...

[shadowman@aap-gw1 ~]$
```

실행 중인 컨테이너 리스트 JSON 출력
```json
[
  {
    "Image": "registry.redhat.io/rhel8/redis-6:latest",
    "Names": "redis-tcp"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/gateway-proxy-rhel8:latest",
    "Names": "automation-gateway-proxy"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/gateway-rhel8:latest",
    "Names": "automation-gateway"
  }
]
```

#### 5.1.3 설치된 AAP

```bash
ls -ldh ~/aap
tree -F -L 2 ~/aap
```

실행 결과
```
[shadowman@aap-gw1 ~]$ ls -ldh ~/aap
drwxr-x---. 6 shadowman shadowman 65 Oct 18 13:10 /home/shadowman/aap

[shadowman@aap-gw1 ~]$ tree -F -L 2 ~/aap
/home/shadowman/aap
├── gateway/
│   ├── etc/
│   ├── nginx/
│   └── supervisor/
├── gatewayproxy/
│   └── etc/
├── redis/
│   ├── cluster.init
│   ├── redis-tcp.conf
│   ├── redis-users.acl
│   ├── redis_nodes.conf
│   ├── server.crt
│   └── server.key
└── tls/
    ├── ca.cert
    ├── ca.key
    └── extracted/

9 directories, 8 files

[shadowman@aap-gw1 ~]$
```

#### 5.1.4 AAP 서비스

```bash
systemctl list-units --type=service --state=running --user
```

실행 결과
```
[shadowman@aap-gw1 ~]$ systemctl list-units --type=service --state=running --user
  UNIT                             LOAD   ACTIVE SUB     DESCRIPTION
  automation-gateway-proxy.service loaded active running Podman automation-gateway-proxy.service
  automation-gateway.service       loaded active running Podman automation-gateway.service
  dbus-broker.service              loaded active running D-Bus User Message Bus
  redis-tcp.service                loaded active running Podman redis-tcp.service

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.
4 loaded units listed.

[shadowman@aap-gw1 ~]$
```

#### 5.1.5 호스트 상의 컨테이너 사용 리소스

```bash
podman container stats -a
```

실행 결과
```
[shadowman@aap-gw1 ~]$ podman container stats -a
ID            NAME                      CPU %       MEM USAGE / LIMIT  MEM %       NET IO      BLOCK IO    PIDS        CPU TIME      AVG CPU %
02bd4eddba17  redis-tcp                 1.55%       21.95MB / 3.837GB  0.57%       0B / 0B     0B / 0B     6           59.273735s    1.55%
e06b9df22aea  automation-gateway-proxy  0.65%       24.07MB / 3.837GB  0.63%       0B / 0B     0B / 0B     13          24.01636s     0.65%
01cb536a7125  automation-gateway        2.62%       836.4MB / 3.837GB  21.80%      0B / 0B     0B / 0B     56          1m36.570669s  2.62%

Ctrl+C

[shadowman@aap-gw1 ~]$
```
<br>

### 5.2 aap-gw2.thinkmore.net 노드 확인

#### 5.2.1 이미지 리스트

```bash
podman image ls
podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
```

실행 결과
```
[shadowman@aap-gw2 ~]$ podman image ls
REPOSITORY                                                             TAG         IMAGE ID      CREATED      SIZE
registry.redhat.io/rhel8/redis-6                                       latest      c99cedb5e4c1  13 days ago  329 MB
registry.redhat.io/ansible-automation-platform-25/gateway-rhel8        latest      f768a98a64be  2 weeks ago  940 MB
registry.redhat.io/ansible-automation-platform-25/gateway-proxy-rhel8  latest      6e53887286a7  2 weeks ago  497 MB

[shadowman@aap-gw2 ~]$ podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
Names: registry.redhat.io/rhel8/redis-6:latest
Names: registry.redhat.io/ansible-automation-platform-25/gateway-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/gateway-proxy-rhel8:latest

[shadowman@aap-gw2 ~]$
```

#### 5.2.2 실행 중인 컨테이너 리스트

```bash
podman ps
podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
podman ps --format json | jq '[.[]|{"Image": .Image, "Names": .Names[0]}]'
```

실행 결과
```
[shadowman@aap-gw2 ~]$ podman ps
CONTAINER ID  IMAGE                                                                         COMMAND               CREATED            STATUS            PORTS       NAMES
2b839799d129  registry.redhat.io/rhel8/redis-6:latest                                       run-redis             About an hour ago  Up About an hour              redis-tcp
b197126933f8  registry.redhat.io/ansible-automation-platform-25/gateway-proxy-rhel8:latest  /usr/bin/envoy --...  About an hour ago  Up About an hour              automation-gateway-proxy
38f5cfc8caba  registry.redhat.io/ansible-automation-platform-25/gateway-rhel8:latest        /usr/bin/supervis...  About an hour ago  Up About an hour              automation-gateway

[shadowman@aap-gw2 ~]$ podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
2b839799d129    run-redis       redis-tcp
b197126933f8    /usr/bin/envoy --...    automation-gateway-proxy
38f5cfc8caba    /usr/bin/supervis...    automation-gateway

[shadowman@aap-gw2 ~]$ podman ps --format json | jq '[.[]|{"Image": .Image, "Names": .Names[0]}]'
...<snip>...

[shadowman@aap-gw2 ~]$
```

실행 중인 컨테이너 리스트 JSON 출력
```json
[
  {
    "Image": "registry.redhat.io/rhel8/redis-6:latest",
    "Names": "redis-tcp"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/gateway-proxy-rhel8:latest",
    "Names": "automation-gateway-proxy"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/gateway-rhel8:latest",
    "Names": "automation-gateway"
  }
]
```

#### 5.2.3 설치된 AAP

```bash
ls -ldh ~/aap
tree -F -L 2 ~/aap
```

실행 결과
```

```

#### 5.2.4 AAP 서비스

```bash
systemctl list-units --type=service --state=running --user
```

실행 결과
```
[shadowman@aap-gw2 ~]$ ls -ldh ~/aap
drwxr-x---. 6 shadowman shadowman 65 Oct 18 13:10 /home/shadowman/aap

[shadowman@aap-gw2 ~]$ tree -F -L 2 ~/aap
/home/shadowman/aap
├── gateway/
│   ├── etc/
│   ├── nginx/
│   └── supervisor/
├── gatewayproxy/
│   └── etc/
├── redis/
│   ├── cluster.init
│   ├── redis-tcp.conf
│   ├── redis-users.acl
│   ├── redis_nodes.conf
│   ├── server.crt
│   └── server.key
└── tls/
    ├── ca.cert
    ├── ca.key
    └── extracted/

9 directories, 8 files

[shadowman@aap-gw2 ~]$
```

#### 5.2.5 호스트 상의 컨테이너 사용 리소스

```bash
podman container stats -a
```

실행 결과
```
[shadowman@aap-gw2 ~]$ podman container stats -a
ID            NAME                      CPU %       MEM USAGE / LIMIT  MEM %       NET IO      BLOCK IO    PIDS        CPU TIME      AVG CPU %
2b839799d129  redis-tcp                 1.00%       17.87MB / 3.837GB  0.47%       0B / 0B     0B / 0B     6           39.879113s    1.00%
b197126933f8  automation-gateway-proxy  0.43%       21.66MB / 3.837GB  0.56%       0B / 0B     0B / 0B     12          16.758654s    0.43%
38f5cfc8caba  automation-gateway        2.30%       677.5MB / 3.837GB  17.65%      0B / 0B     0B / 0B     52          1m28.861357s  2.30%

Ctrl+C

[shadowman@aap-gw2 ~]$
```
<br>
<br>

## 6. PostgreSQL 데이터베이스 확인

### 6.1 생성된 AAP DB 확인

```bash
psql --username=postgres --host=aap-db.thinkmore.net
\l
\q
```

실행 결과
```
[shadowman@aap-c1 ~]$ psql --username=postgres --host=aap-db.thinkmore.net
Password for user postgres:
psql (13.16, server 15.8)
WARNING: psql major version 13, server major version 15.
         Some psql features might not work.
Type "help" for help.

postgres=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-----------+----------+----------+------------+------------+-----------------------
 awx       | awx      | UTF8     | en_US.utf8 | en_US.utf8 |
 eda       | eda      | UTF8     | en_US.utf8 | en_US.utf8 |
 gateway   | gateway  | UTF8     | en_US.utf8 | en_US.utf8 |
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 pulp      | pulp     | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(7 rows)

postgres=# \q

[shadowman@aap-c1 ~]$
```
<br>
<br>

------
[차례](../README.md)