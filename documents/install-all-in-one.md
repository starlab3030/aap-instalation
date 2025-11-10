# 로컬에 All-In-One 설치

## 1. 인벤토리 파일 구성

### 1.1 all-in-one 설치를 위한 인벤토리 파일

all-in-one 설치를 위한 인벤토리 파일
```ini
[automationgateway]
aap-c.thinkmore.net


[automationcontroller]
aap-c.thinkmore.net


[automationhub]
aap-c.thinkmore.net


[automationeda]
aap-c.thinkmore.net


[database]
aap-c.thinkmore.net


[all:vars]
ansible_connection=local

postgresql_admin_username=postgres
postgresql_admin_password=redhat


# Registry
#
registry_username='$<USER_ID>'
registry_password='$<USER_PW>'


# REDIS
#
redis_mode=standalone


# Automation Gateway
#
gateway_admin_password=redhat
gateway_pg_host=aap-c.thinkmore.net
gateway_pg_password=redhat


# Automation Controller
#
controller_admin_password=redhat
controller_pg_host=aap-c.thinkmore.net
controller_pg_password=redhat
controller_percent_memory_capacity=0.5


# Automation Hub
#
hub_admin_password=redhat
hub_pg_host=aap-c.thinkmore.net
hub_pg_password=redhat
hub_seed_collections=false


# Event-Driven Ansible
#
eda_admin_password=redhat
eda_pg_host=aap-c.thinkmore.net
eda_pg_password=redhat
```
* *ansible_connection=`local`*
  + AAP를 호스팅하는 동일한 노드에서 실행되는 all-in-one 설치에 사용
* *bundle_dir*의 값은 절대 경로로 입력해야 함
* *redis_mode*는 ***standalone***으로 설정

인벤토리 파일 확인
```bash
egrep -v "^#|^$" inventory
```

실행 결과
```
[shadowman@aap-c ansible-setup]$ egrep -v "^#|^$" inventory
[automationgateway]
aap-c.thinkmore.net
[automationcontroller]
aap-c.thinkmore.net
[automationhub]
aap-c.thinkmore.net
[automationeda]
aap-c.thinkmore.net
[database]
aap-c.thinkmore.net
[all:vars]
ansible_connection=local
postgresql_admin_username=postgres
postgresql_admin_password=redhat
registry_username='$<USER_ID>'
registry_password='$<USER_PW>'
redis_mode=standalone
gateway_admin_password=redhat
gateway_pg_host=aap-c.thinkmore.net
gateway_pg_password=redhat
controller_admin_password=redhat
controller_pg_host=aap-c.thinkmore.net
controller_pg_password=redhat
controller_percent_memory_capacity=0.5
hub_admin_password=redhat
hub_pg_host=aap-c.thinkmore.net
hub_pg_password=redhat
hub_seed_collections=false
eda_admin_password=redhat
eda_pg_host=aap-c.thinkmore.net
eda_pg_password=redhat

[shadowman@aap-c ansible-setup]$
```
<br>

### 1.2 설치 파일 확인

#### 번들 설치

번들 설치 시, *bundle_dir* 변수에 입력 값을 기반으로 확인

~/collections/ansible_collections/ansible/containerized_installer/roles/preflight/tasks/main.yml
```yaml
#...<snip>...

- name: Include bundle checks
  when: bundle_install | default(false) | bool
  block:
    - name: Ensure bundle_dir is provided
      ansible.builtin.assert:
        that:
          - bundle_dir is defined
          - bundle_dir | length
        fail_msg: 'bundle_dir must be set when bundle_install=true'

    - name: Check the images directory
      ansible.builtin.stat:
        path: '{{ bundle_dir }}/images'
      register: _bundle_images

    - name: Ensure the images directory exists
      ansible.builtin.assert:
        that:
          - _bundle_images.stat.exists | bool
          - _bundle_images.stat.isdir | bool
        fail_msg: 'The bundle directory must contain an images directory'
...
```

#### redis 모드

설치 시, *redis_mode*에서 모드 설정

~/collections/ansible_collections/ansible/containerized_installer/roles/preflight/tasks/redis.yml
```yaml
---
- name: Ensure redis mode is a valid choice
  ansible.builtin.assert:
    that:
      - redis_mode in ['cluster', 'standalone']
    fail_msg: 'Invalid redis mode value.'
  when: redis_mode is defined

- name: Ensure redis cluster is correctly configured
  ansible.builtin.assert:
    that:
      - groups.get('redis', []) | length >= 6
    fail_msg: 'A [redis] group with at least 6 nodes if required for redis cluster mode'
  when: redis_mode | default('cluster') == 'cluster'
...
```

#### 메모리 크기 체크

설치 시, 해당 노드의 메모리 크기 체크

~/collections/ansible_collections/ansible/containerized_installer/roles/preflight/tasks/nodes.yml
```yaml
---
#...<snip>...

- name: Fail if this machine lacks sufficient RAM
  ansible.builtin.assert:
    that:
      - ansible_memtotal_mb >= 15000
    fail_msg: >
      This machine does not have sufficient RAM to run Ansible Automation Platform.
      Required RAM is 16GB but this machine only has {{ ansible_memtotal_mb }}MB.

#...<snip>...
```
* 메모리가 15,000 MiB 이상이어야 함
<br>
<br>

## 2. 앤서블 설치

### 2.1 환경 변수 설정

#### 2.1.1 기본 변수 파일

~/collections/ansible_collections/ansible/containerized_installer/roles/automationgateway/defaults/main.yml
```yaml
### envoy
envoy_conf_dir: '{{ aap_volumes_dir }}/gatewayproxy/etc'
envoy_disable_https: false
envoy_http_port: 80
envoy_https_port: 443

#...<snip>...

### automation gateway
gateway_container_requires: []
gateway_conf_dir: '{{ aap_volumes_dir }}/gateway/etc'
gateway_firewall_zone: public
gateway_admin_user: admin
gateway_admin_email: admin@example.com
#...<snip>...
```

#### 2.1.2 사용자 ID와 암호 설정

```bash
gateway_admin_username='admin'
gateway_admin_password='redhat'
```

#### 2.1.3 AAP 기본 포트 설정

```bash
envoy_http_port=80
envoy_https_port=443
```

#### 2.1.4 HTTPS 비활성화

```bash
envoy_disable_https: true
```
<br>

### 2.2 컨테이너 형 앤서블 설치

```bash
ansible-playbook -i inventory ansible.containerized_installer.install
```

실행 결과
```
[shadowman@aap-c ansible-setup]$ ansible-playbook -i inventory ansible.containerized_installer.install

...<snip>...

PLAY RECAP **********************************************************************************************************************************************************
aap-c.thinkmore.net        : ok=621  changed=227  unreachable=0    failed=0    skipped=292  rescued=0    ignored=0   
localhost                  : ok=30   changed=0    unreachable=0    failed=0    skipped=63   rescued=0    ignored=0   

[shadowman@aap-c ansible-setup]$ 
```
<br>
<br>

## 3. 설치된 AAP 확인

### 3.1 컨테이너 환경

#### 3.1.1 AAP 이미지 리스트

```bash
podman image ls
podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
```

실행 환경
```
[shadowman@aap-c ~]$ podman image ls
REPOSITORY                                                                 TAG         IMAGE ID      CREATED      SIZE
registry.redhat.io/rhel9/redis-6                                           latest      884c9eab94ec  4 days ago   337 MB
registry.redhat.io/ansible-automation-platform-26/eda-controller-ui-rhel9  latest      59601280aacd  6 days ago   606 MB
registry.redhat.io/ansible-automation-platform-26/gateway-rhel9            latest      f79d345c5aa7  6 days ago   981 MB
registry.redhat.io/ansible-automation-platform-26/hub-rhel9                latest      07352ccfe11f  6 days ago   1.19 GB
registry.redhat.io/ansible-automation-platform-26/eda-controller-rhel9     latest      b62edeea6b63  6 days ago   1.09 GB
registry.redhat.io/ansible-automation-platform-26/receptor-rhel9           latest      62c2b669ae6b  6 days ago   587 MB
registry.redhat.io/ansible-automation-platform-26/controller-rhel9         latest      70ee04219145  11 days ago  2 GB
registry.redhat.io/ansible-automation-platform-26/gateway-proxy-rhel9      latest      46a31eca8a1b  3 weeks ago  367 MB
registry.redhat.io/ansible-automation-platform-26/hub-web-rhel9            latest      133b616a78d0  3 weeks ago  602 MB
registry.redhat.io/rhel9/postgresql-15                                     latest      b861c547e3ba  3 weeks ago  531 MB

[shadowman@aap-c ~]$ podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
Names: registry.redhat.io/rhel9/redis-6:latest
Names: registry.redhat.io/ansible-automation-platform-26/eda-controller-ui-rhel9:latest
Names: registry.redhat.io/ansible-automation-platform-26/gateway-rhel9:latest
Names: registry.redhat.io/ansible-automation-platform-26/hub-rhel9:latest
Names: registry.redhat.io/ansible-automation-platform-26/eda-controller-rhel9:latest
Names: registry.redhat.io/ansible-automation-platform-26/receptor-rhel9:latest
Names: registry.redhat.io/ansible-automation-platform-26/controller-rhel9:latest
Names: registry.redhat.io/ansible-automation-platform-26/gateway-proxy-rhel9:latest
Names: registry.redhat.io/ansible-automation-platform-26/hub-web-rhel9:latest
Names: registry.redhat.io/rhel9/postgresql-15:latest

[shadowman@aap-c ~]$ 
```

#### 3.1.2 실행 중인 컨테이너 리스트

```bash
podman ps
podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
podman ps --format json | jq '[.[]|{"Image": .Image, "Names": .Names[0]}]'
```

실행 환경
```
[shadowman@aap-c ~]$ podman ps
...<snip>...

[shadowman@aap-c ~]$ podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
2ffe34effab3    run-postgresql  postgresql
bde916805460    run-redis       redis-unix
8291e2ed7278    run-redis       redis-tcp
50b3d2f70893    /usr/bin/envoy --...    automation-gateway-proxy
c101bca53df5    /usr/bin/supervis...    automation-gateway
221bb07d4639    /usr/bin/receptor...    receptor
90be25fac271    /usr/bin/launch_a...    automation-controller-rsyslog
e97b56dbcf1e    /usr/bin/launch_a...    automation-controller-task
29e1fef146c1    /usr/bin/launch_a...    automation-controller-web
359000befa81    gunicorn --bind 1...    automation-eda-api
1e19e7ac5cde    daphne --bind 127...    automation-eda-daphne
4c361bcae441    /bin/sh -c nginx ...    automation-eda-web
3795031f5f1f    aap-eda-manage rq...    automation-eda-worker-1
f36110a2a2ee    aap-eda-manage rq...    automation-eda-worker-2
0e92d6a52974    aap-eda-manage rq...    automation-eda-activation-worker-1
87738cb387ca    aap-eda-manage rq...    automation-eda-activation-worker-2
c63fcbb3a02e    aap-eda-manage sc...    automation-eda-scheduler
1d6765fccdfa    pulpcore-api --na...    automation-hub-api
0008efe74b09    pulpcore-content ...    automation-hub-content
ced1a391b0fd    /bin/sh -c nginx ...    automation-hub-web
36dce89bda02    pulpcore-worker automation-hub-worker-1
0a7a7501021f    pulpcore-worker automation-hub-worker-2

[shadowman@aap-c ~]$ podman ps --format json | jq '[.[]|{"Image": .Image, "Names": .Names[0]}]'
...<snip>...

[shadowman@aap-c ~]$ 
```

실행 중인 컨테이너 리스트 JSON 출력
```json
[
  {
    "Image": "registry.redhat.io/rhel9/postgresql-15:latest",
    "Names": "postgresql"
  },
  {
    "Image": "registry.redhat.io/rhel9/redis-6:latest",
    "Names": "redis-unix"
  },
  {
    "Image": "registry.redhat.io/rhel9/redis-6:latest",
    "Names": "redis-tcp"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-26/gateway-proxy-rhel9:latest",
    "Names": "automation-gateway-proxy"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-26/gateway-rhel9:latest",
    "Names": "automation-gateway"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-26/receptor-rhel9:latest",
    "Names": "receptor"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-26/controller-rhel9:latest",
    "Names": "automation-controller-rsyslog"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-26/controller-rhel9:latest",
    "Names": "automation-controller-task"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-26/controller-rhel9:latest",
    "Names": "automation-controller-web"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-26/eda-controller-rhel9:latest",
    "Names": "automation-eda-api"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-26/eda-controller-rhel9:latest",
    "Names": "automation-eda-daphne"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-26/eda-controller-ui-rhel9:latest",
    "Names": "automation-eda-web"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-26/eda-controller-rhel9:latest",
    "Names": "automation-eda-worker-1"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-26/eda-controller-rhel9:latest",
    "Names": "automation-eda-worker-2"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-26/eda-controller-rhel9:latest",
    "Names": "automation-eda-activation-worker-1"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-26/eda-controller-rhel9:latest",
    "Names": "automation-eda-activation-worker-2"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-26/eda-controller-rhel9:latest",
    "Names": "automation-eda-scheduler"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-26/hub-rhel9:latest",
    "Names": "automation-hub-api"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-26/hub-rhel9:latest",
    "Names": "automation-hub-content"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-26/hub-web-rhel9:latest",
    "Names": "automation-hub-web"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-26/hub-rhel9:latest",
    "Names": "automation-hub-worker-1"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-26/hub-rhel9:latest",
    "Names": "automation-hub-worker-2"
  }
]
```
<br>

### 3.2 설치된 AAP 접속

#### 3.2.1 AAP Proxy 접속

|$\color{lime}{\texttt{항목}}$|$\color{lime}{\texttt{설명}}$|
|:---|:---|
|`URL`|https://aap-c.thinkmore.net|
|`사용자`|admin|
|`암호`|사용자_암호|

AAP proxy의 대시보드
<img src="./images/aap_proxy.png" title="100px" alt="AAP 프록시 대시보드"></img>

* 왼편 메뉴에 AAP 인스턴스 리스트가 있음
  - Automation Execution (Automation Controller)
  - Automation Decisions (Event-Driven Ansible)
  - Automation Content (Automation Hub)
  - Ansible Lightspeed
* AAP 인스턴스 별 기능이 분산되어 있음
<br>

### 3.3 AAP 구조

#### 3.3.1 설치된 AAP

```bash
ls -ldh ~/aap
tree -F -L 1 ~/aap
```

실행 환경
```
[shadowman@aap-c ~]$ ls -ldh ~/aap
drwxrwx---. 12 shadowman shadowman 157 Oct 14 13:54 /home/shadowman/aap

[shadowman@aap-c ~]$ tree -F -L 1 ~/aap
/home/shadowman/aap
├── containers/
├── controller/
├── eda/
├── gateway/
├── gatewayproxy/
├── hub/
├── postgresql/
├── receptor/
├── redis/
└── tls/

10 directories, 0 files

[shadowman@aap-c ~]$ 
```

#### 3.3.2 컨테이너화 된 AAP를 위한 podman 구성 사항

```bash
tree -F -L 3 ~/aap/containers/
```

실행 환경
```
[shadowman@aap-c ~]$ tree -F -L 3 ~/aap/containers/
/home/shadowman/aap/containers/
├── podman*
├── storage/
│   ├── db.sql
│   ├── defaultNetworkBackend
│   ├── libpod/
│   ├── networks/
│   │   └── netavark.lock
│   ├── overlay/
│   │   ├── 526835743b80e63bc87c0f1800388f36568c6b158fff4efe6a6b75e664cf035d/
│   │   ├── 9d1b281cd61b82682769233ec86fae0affd0cbe1b816e84edcced3a0dbd2a3b1/
│   │   ├── df2641cdeeb38df7b9c5eb7601607ba82080bba82496bdb2e4a5de7813000ae3/
│   │   └── l/
│   ├── overlay-containers/
│   │   └── containers.lock
│   ├── overlay-images/
│   │   ├── 70b9df44d11fa1362bcef75ab02a1c349e9b683b95cd9128cd9000884d959386/
│   │   ├── 884f5223cbb47fdb820d54a073d6d69bdafdb811c44e18319f867d970d2b604e/
│   │   ├── a2386c4d84d9f8c71963e4d26eddb14417d3a416bb6116f6115adbfce2b221d1/
│   │   ├── images.json
│   │   └── images.lock
│   ├── overlay-layers/
│   │   ├── 526835743b80e63bc87c0f1800388f36568c6b158fff4efe6a6b75e664cf035d.tar-split.gz
│   │   ├── 9d1b281cd61b82682769233ec86fae0affd0cbe1b816e84edcced3a0dbd2a3b1.tar-split.gz
│   │   ├── df2641cdeeb38df7b9c5eb7601607ba82080bba82496bdb2e4a5de7813000ae3.tar-split.gz
│   │   ├── layers.json
│   │   └── layers.lock
│   ├── storage.lock
│   ├── userns.lock
│   └── volumes/
└── storage.conf

15 directories, 15 files

[shadowman@aap-c ~]$
```
* ~/aap/containers 디렉터리에는 실행-플레인에서 사용되고 설치도는 podman 특정 사항이 포함됨

#### 3.3.3 컨테이너화 된 Automation Controller

```bash
tree -F -L 3 ~/aap/controller/
```

실행 환경
```
[shadowman@aap-c ~]$ tree -F -L 3 ~/aap/controller/
/home/shadowman/aap/controller/
├── data/
│   ├── job_execution/
│   ├── job_status/
│   ├── logs/
│   ├── projects/
│   └── rsyslog/
│       └── rsyslog.conf
├── etc/
│   ├── conf.d/
│   │   ├── callback_receiver_workers.py
│   │   ├── cluster_host_id.py
│   │   ├── container_groups.py
│   │   ├── execution_environments.py
│   │   ├── insights.py
│   │   ├── redis.py
│   │   └── subscription_usage_model.py
│   ├── launch_awx_task.sh*
│   ├── settings.py
│   ├── tower.cert
│   ├── tower.key
│   └── uwsgi.ini
├── nginx/
│   └── etc/
│       ├── controller.conf
│       └── redirect-page.html
├── rsyslog/
│   └── run/
│       ├── rsyslog.pid
│       └── rsyslog.sock=
└── supervisor/
    └── run/
        ├── supervisor.rsyslog.pid
        ├── supervisor.rsyslog.sock=
        ├── supervisor.task.pid
        ├── supervisor.task.sock=
        ├── supervisor.web.pid
        └── supervisor.web.sock=

14 directories, 23 files

[shadowman@aap-c ~]$ 
```
* 디렉터리에는 설치된 구성 및 런타임 데이터 지점 중 일부가 있음

#### 3.3.4 컨테이너화 된 eda

```bash
tree -F -L 3 ~/aap/eda/
```

실행 환경
```
[shadowman@aap-c ~]$ tree -F ~/aap/eda/
/home/shadowman/aap/eda/
├── etc/
│   ├── eda.cert
│   ├── eda.key
│   └── settings.yaml
└── nginx/
    └── etc/
        ├── eda.conf
        └── redirect-page.html

3 directories, 5 files

[shadowman@aap-c ~]$
```

#### 3.3.5 컨테이너화 된 hub

```bash
tree -F -L 3 ~/aap/hub/
```

실행 환경
```
[shadowman@aap-c ~]$ tree -F -L 3 ~/aap/hub/
/home/shadowman/aap/hub/
├── etc/
│   ├── keys/
│   │   ├── container_auth_private_key.pem
│   │   └── container_auth_public_key.pem
│   ├── pulp.cert
│   └── pulp.key
└── nginx/
    └── etc/
        ├── hub.conf
        └── redirect-page.html

4 directories, 6 files

[shadowman@aap-c ~]$
```

#### 3.3.6 컨테이너화 된 proxy

```bash
tree -F -L 3 ~/aap/gateway
tree -F -L 3 ~/aap/gatewayproxy/
```

실행 환경
```
[shadowman@aap-c ~]$ tree -F -L 3 ~/aap/gateway
/home/shadowman/aap/gateway
├── etc/
│   ├── gateway.cert
│   ├── gateway.key
│   ├── redis.cert
│   ├── redis.key
│   ├── settings.py
│   ├── supervisord.conf
│   └── uwsgi.ini
├── nginx/
│   └── etc/
│       └── nginx.conf
└── supervisor/
    └── run/
        ├── supervisor.sock=
        └── supervisord.pid

5 directories, 10 files

[shadowman@aap-c ~]$ tree -F -L 3 ~/aap/gatewayproxy/
/home/shadowman/aap/gatewayproxy/
└── etc/
    └── envoy.yaml

1 directory, 1 file

[shadowman@aap-c ~]$ 
```

#### 3.3.7 컨테이너화 된 receptor

```bash
tree -F -L 3 ~/aap/receptor/
```

실행 환경
```
[shadowman@aap-c ~]$ tree -F -L 3 ~/aap/receptor/
/home/shadowman/aap/receptor/
└── etc/
    ├── mesh-CA.crt
    ├── receptor.conf
    ├── receptor.crt
    ├── receptor.key
    ├── signing_private.pem
    └── signing_public.pem

1 directory, 6 files

[shadowman@aap-c ~]$ 
```
* 디렉터리에는 자동화 메시 구성 있음
<br>

### 3.4 AAP 서비스

```bash
tree -F .config/systemd/
```

실행 결과
```
[shadowman@aap-c ~]$ tree -F .config/systemd/
.config/systemd/
└── user/
    ├── automation-controller-rsyslog.service
    ├── automation-controller-task.service
    ├── automation-controller-web.service
    ├── automation-eda-activation-worker-1.service
    ├── automation-eda-activation-worker-2.service
    ├── automation-eda-api.service
    ├── automation-eda-daphne.service
    ├── automation-eda-scheduler.service
    ├── automation-eda-web.service
    ├── automation-eda-worker-1.service
    ├── automation-eda-worker-2.service
    ├── automation-gateway-proxy.service
    ├── automation-gateway.service
    ├── automation-hub-api.service
    ├── automation-hub-content.service
    ├── automation-hub-web.service
    ├── automation-hub-worker-1.service
    ├── automation-hub-worker-2.service
    ├── default.target.wants/
    │   ├── automation-controller-rsyslog.service -> /home/aap/.config/systemd/user/automation-controller-rsyslog.service
    │   ├── automation-controller-task.service -> /home/aap/.config/systemd/user/automation-controller-task.service
    │   ├── automation-controller-web.service -> /home/aap/.config/systemd/user/automation-controller-web.service
    │   ├── automation-eda-activation-worker-1.service -> /home/aap/.config/systemd/user/automation-eda-activation-worker-1.service
    │   ├── automation-eda-activation-worker-2.service -> /home/aap/.config/systemd/user/automation-eda-activation-worker-2.service
    │   ├── automation-eda-api.service -> /home/aap/.config/systemd/user/automation-eda-api.service
    │   ├── automation-eda-daphne.service -> /home/aap/.config/systemd/user/automation-eda-daphne.service
    │   ├── automation-eda-scheduler.service -> /home/aap/.config/systemd/user/automation-eda-scheduler.service
    │   ├── automation-eda-web.service -> /home/aap/.config/systemd/user/automation-eda-web.service
    │   ├── automation-eda-worker-1.service -> /home/aap/.config/systemd/user/automation-eda-worker-1.service
    │   ├── automation-eda-worker-2.service -> /home/aap/.config/systemd/user/automation-eda-worker-2.service
    │   ├── automation-gateway-proxy.service -> /home/aap/.config/systemd/user/automation-gateway-proxy.service
    │   ├── automation-gateway.service -> /home/aap/.config/systemd/user/automation-gateway.service
    │   ├── automation-hub-api.service -> /home/aap/.config/systemd/user/automation-hub-api.service
    │   ├── automation-hub-content.service -> /home/aap/.config/systemd/user/automation-hub-content.service
    │   ├── automation-hub-web.service -> /home/aap/.config/systemd/user/automation-hub-web.service
    │   ├── automation-hub-worker-1.service -> /home/aap/.config/systemd/user/automation-hub-worker-1.service
    │   ├── automation-hub-worker-2.service -> /home/aap/.config/systemd/user/automation-hub-worker-2.service
    │   ├── postgresql.service -> /home/aap/.config/systemd/user/postgresql.service
    │   ├── receptor.service -> /home/aap/.config/systemd/user/receptor.service
    │   ├── redis-tcp.service -> /home/aap/.config/systemd/user/redis-tcp.service
    │   └── redis-unix.service -> /home/aap/.config/systemd/user/redis-unix.service
    ├── podman.service.d/
    │   └── override.conf
    ├── postgresql.service
    ├── receptor.service
    ├── redis-tcp.service
    ├── redis-unix.service
    └── sockets.target.wants/
        └── podman.socket -> /usr/lib/systemd/user/podman.socket

4 directories, 46 files

[shadowman@aap-c ~]$ 
```
<br>

### 3.5 호스트 상의 컨테이너 사용 리소스

```bash
podman container stats -a
```

실행 환경
```
[shadowman@aap-c ~]$ podman container stats -a
ID            NAME                                CPU %       MEM USAGE / LIMIT  MEM %       NET IO      BLOCK IO    PIDS        CPU TIME      AVG CPU %
542122c594fd  postgresql                          2.88%       36.86MB / 3.837GB  0.96%       0B / 0B     0B / 0B     32          2m17.676589s  2.47%
aefd54102495  redis-unix                          0.45%       5.132MB / 3.837GB  0.13%       0B / 0B     0B / 0B     5           21.653127s    0.39%
f3175301d6bd  redis-tcp                           0.26%       5.452MB / 3.837GB  0.14%       0B / 0B     0B / 0B     5           12.586096s    0.23%
9b55ff708786  automation-gateway-proxy            0.50%       13.56MB / 3.837GB  0.35%       0B / 0B     0B / 0B     13          37.839952s    0.69%
452f26885288  automation-gateway                  2.53%       486.4MB / 3.837GB  12.68%      0B / 0B     0B / 0B     59          2m26.386377s  2.69%

...<snip>...

Ctrl+C

[shadowman@aap-c ~]$
```
<br>
<br>

## 4. AAP 데이터베이스: PostgreSQL

### 4.1 컨테이너 확인

```bash
podman ps -a --format "{{.ID}}\t{{.Status}}\t{{.Command}}\t{{.Names}}" | grep -i postgresql
```

실행 결과
```
[shadowman@aap-c ~]$ podman ps -a --format "{{.ID}}\t{{.Status}}\t{{.Command}}\t{{.Names}}" | grep -i postgre
ffe34effab3    Up 5 hours      run-postgresql  postgresql

[shadowman@aap-c ~]$ 
```
<br>

### 4.2 데이터베이스 컨테이너가 사용하는 포트 확인

```bash
podman container inspect postgresql | jq '.[].Config|{"Config.Labels.usage": .Labels.usage, "Config.CreateCommand": .CreateCommand}'
```

실행 결과
```
[shadowman@aap-c ~]$ podman container inspect postgresql | jq '.[].Config|{"Config.Labels.usage": .Labels.usage, "Config.CreateCommand": .CreateCommand}'
...<snip>...

[shadowman@aap-c ~]$ 
```

실행 결과 JSON 출력
```json
{
  "Config.Labels.usage": "podman run -d --name postgresql_database -e POSTGRESQL_USER=user -e POSTGRESQL_PASSWORD=pass -e POSTGRESQL_DATABASE=db -p 5432:5432 rhel9/postgresql-15",
  "Config.CreateCommand": [
    "podman",
    "container",
    "create",
    "--name",
    "postgresql",
    "--log-driver",
    "journald",
    "--network",
    "host",
    "--secret",
    "postgresql_admin_password,type=env,target=POSTGRESQL_ADMIN_PASSWORD",
    "--volume",
    "/home/aap/aap/tls/extracted:/etc/pki/ca-trust/extracted:z",
    "--volume",
    "/home/aap/aap/postgresql/postgresql.conf:/usr/share/container-scripts/postgresql/openshift-custom-postgresql.conf.template:ro,z",
    "--volume",
    "postgresql:/var/lib/pgsql/data:Z",
    "--volume",
    "/home/aap/aap/postgresql/server.crt:/var/lib/pgsql/server.crt:ro,z",
    "--volume",
    "/home/aap/aap/postgresql/server.key:/var/lib/pgsql/server.key:ro,z",
    "--env",
    "PGPORT=5432",
    "--env",
    "POSTGRESQL_EFFECTIVE_CACHE_SIZE=3146MB",
    "--env",
    "POSTGRESQL_MAX_CONNECTIONS=1024",
    "--env",
    "POSTGRESQL_SHARED_BUFFERS=1573MB",
    "--env",
    "POSTGRESQL_LOG_DESTINATION=/dev/stderr",
    "--uidmap",
    "26:0:1",
    "--uidmap",
    "0:1:26",
    "--uidmap",
    "27:27:65510",
    "--gidmap",
    "26:0:1",
    "--gidmap",
    "0:1:26",
    "--gidmap",
    "27:27:65510",
    "--label",
    "io.containers.autoupdate=registry",
    "--label",
    "io.containers.autoupdate.authfile=/home/aap/.config/containers/auth.json",
    "registry.redhat.io/rhel9/postgresql-15:latest"
  ]
}
```
* 데이터베이스 포트는 5432를 그대로 매핑
<br>

### 4.3 데이터베이스 접속 및 리스트 확인

```bash
podman exec -it postgresql /bin/bash
psql
\l
```

실행 결과
```
[shadowman@aap-c ~]$ podman exec -it postgresql /bin/bash

bash-4.4$ psql
psql (15.14)
Type "help" for help.

postgres=# \l
                                                List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    | ICU Locale | Locale Provider |   Access privileges
   
-----------+----------+----------+------------+------------+------------+-----------------+--------------------
---
 awx       | awx      | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | 
 eda       | eda      | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | 
 gateway   | gateway  | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | 
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | 
 pulp      | pulp     | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/postgres        
  +
           |          |          |            |            |            |                 | postgres=CTc/postgr
es
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/postgres        
  +
           |          |          |            |            |            |                 | postgres=CTc/postgr
es
(7 rows)

postgres=# 
```
* 데이터베이스 버전이 15.14
* DB 리스트에는 awx, eda, gateway, pulp 등이 있음

> [!IMPORTANT]
> All-In-One 설치의 경우에 PostgreSQL 데이터베이스 컨테이너에는 AAP의 모든 인스턴스의 DB가 같이 설치되어 있습니다. 이런 경우에는 각 인스턴스의 사용률 증가에 따라 DB 크기 조정 및 성능 튜닝이 필요할 수 있습니다.
<br>

### 4.4 awx 데이터베이스 확인

```bash
\c awx
\dt
```

실행 결과
```
postgres=# \c awx
You are now connected to database "awx" as user "postgres".

awx=# \dt
                                       List of relations
 Schema |                           Name                            |       Type        | Owner 
--------+-----------------------------------------------------------+-------------------+-------
 public | _unpartitioned_main_adhoccommandevent                     | table             | awx
 public | _unpartitioned_main_inventoryupdateevent                  | table             | awx
 public | _unpartitioned_main_jobevent                              | table             | awx
 public | _unpartitioned_main_projectupdateevent                    | table             | awx
 public | _unpartitioned_main_systemjobevent                        | table             | awx

 ...<snip>...

 public | oauth2_provider_grant                                     | table             | awx
 public | oauth2_provider_idtoken                                   | table             | awx
 public | oauth2_provider_refreshtoken                              | table             | awx
 public | social_auth_association                                   | table             | awx
 public | social_auth_code                                          | table             | awx
 public | social_auth_nonce                                         | table             | awx
 public | social_auth_partial                                       | table             | awx
 public | social_auth_usersocialauth                                | table             | awx
 public | sso_userenterpriseauth                                    | table             | awx
(175 rows)

awx=#         
```
<br>

### 4.5 eda 데이터베이스 확인

```bash
\c eda
\dt
```

실행 결과
```
awx=# \c eda
You are now connected to database "eda" as user "postgres".

eda=# \dt
                       List of relations
 Schema |                 Name                  | Type  | Owner 
--------+---------------------------------------+-------+-------
 public | auth_group                            | table | eda
 public | auth_group_permissions                | table | eda
 public | auth_permission                       | table | eda

...<snip>...

 public | django_content_type                   | table | eda
 public | django_migrations                     | table | eda
 public | django_session                        | table | eda
 public | flags_flagstate                       | table | eda
(50 rows)

eda=#     
```
<br>

### 4.6 hub 데이터베이스 확인

```bash
\c pulp
\dt
```

실행 결과
```
eda=# \c pulp
You are now connected to database "pulp" as user "postgres".

pulp=# \dt
                              List of relations
 Schema |                        Name                         | Type  | Owner 
--------+-----------------------------------------------------+-------+-------
 public | ansible_ansiblecollectiondeprecated                 | table | pulp
 public | ansible_ansibledistribution                         | table | pulp
 public | ansible_ansiblenamespace                            | table | pulp
 public | ansible_ansiblenamespacemetadata                    | table | pulp
 public | ansible_ansiblerepository                           | table | pulp
 public | ansible_collection                                  | table | pulp
 public | ansible_collectiondownloadcount                     | table | pulp
 public | ansible_collectionimport                            | table | pulp
 public | ansible_collectionremote                            | table | pulp
 public | ansible_collectionversion                           | table | pulp
 public | ansible_collectionversion_tags                      | table | pulp
 public | ansible_collectionversionmark                       | table | pulp
 public | ansible_collectionversionsignature                  | table | pulp
 public | ansible_crossrepositorycollectionversionindex       | table | pulp
 public | ansible_downloadlog                                 | table | pulp
 public | ansible_gitremote                                   | table | pulp
 public | ansible_role                                        | table | pulp
 public | ansible_roleremote                                  | table | pulp
 public | ansible_tag                                         | table | pulp

 ...<snip>...

 public | galaxy_aiindexdenylist                              | table | pulp
 public | galaxy_collectionimport                             | table | pulp
 public | galaxy_containerdistroreadme                        | table | pulp
 public | galaxy_containerregistryremote                      | table | pulp
 public | galaxy_containerregistryrepos                       | table | pulp
 public | galaxy_legacynamespace                              | table | pulp
 public | galaxy_legacynamespace_owners                       | table | pulp
 public | galaxy_legacyrole                                   | table | pulp
 public | galaxy_legacyrole_tags                              | table | pulp
 public | galaxy_legacyroledownloadcount                      | table | pulp
 public | galaxy_legacyroleimport                             | table | pulp
 public | galaxy_legacyrolesearchvector                       | table | pulp
 public | galaxy_legacyroletag                                | table | pulp
 public | galaxy_namespace                                    | table | pulp
 public | galaxy_namespacelink                                | table | pulp
 public | galaxy_organization                                 | table | pulp
 public | galaxy_setting                                      | table | pulp
 public | galaxy_synclist                                     | table | pulp
 public | galaxy_synclist_collections                         | table | pulp
 public | galaxy_synclist_namespaces                          | table | pulp
 public | galaxy_team                                         | table | pulp
 public | galaxy_user                                         | table | pulp
 public | galaxy_user_groups                                  | table | pulp
 public | galaxy_user_user_permissions                        | table | pulp
 public | social_auth_association                             | table | pulp
 public | social_auth_code                                    | table | pulp
 public | social_auth_nonce                                   | table | pulp
 public | social_auth_partial                                 | table | pulp
 public | social_auth_usersocialauth                          | table | pulp
(154 rows)

pulp=#   
```
<br>

### 4.7 proxy 데이터베이스 확인

```bash
\c gateway
\dt
```

실행 결과
```
pulp=# \c gateway
You are now connected to database "gateway" as user "postgres".

gateway=# \dt
                          List of relations
 Schema |                   Name                    | Type  |  Owner  
--------+-------------------------------------------+-------+---------
 public | aap_gateway_api_additionalroute           | table | gateway
 public | aap_gateway_api_httpport                  | table | gateway
 public | aap_gateway_api_migratedusermetadata      | table | gateway
 public | aap_gateway_api_migrateservicedatahasran  | table | gateway
 public | aap_gateway_api_organization              | table | gateway
 public | aap_gateway_api_preference                | table | gateway
 public | aap_gateway_api_route                     | table | gateway
 public | aap_gateway_api_serviceapiroute           | table | gateway
 public | aap_gateway_api_servicecluster            | table | gateway
 public | aap_gateway_api_servicekey                | table | gateway
 public | aap_gateway_api_servicenode               | table | gateway
 public | aap_gateway_api_servicetype               | table | gateway
 public | aap_gateway_api_team                      | table | gateway
 public | aap_gateway_api_team_parents              | table | gateway
 public | aap_gateway_api_uipluginroute             | table | gateway
 public | aap_gateway_api_user                      | table | gateway
 public | aap_gateway_api_user_groups               | table | gateway
 public | aap_gateway_api_user_user_permissions     | table | gateway

 ...<snip>...

 public | social_auth_association                   | table | gateway
 public | social_auth_code                          | table | gateway
 public | social_auth_nonce                         | table | gateway
 public | social_auth_partial                       | table | gateway
 public | social_auth_usersocialauth                | table | gateway
(54 rows)

gateway=#    
```
<br>
<br>

## 5. Automation Controller

### 5.1 automation-controller 컨테이너 리스트 확인

```bash
podman ps -a --format "{{.ID}}\t{{.Status}}\t{{.Command}}\t{{.Names}}" | grep -i "automation-controller"
```

실행 결과
```
[shadowman@aap-c ~]$ podman ps -a --format "{{.ID}}\t{{.Status}}\t{{.Command}}\t{{.Names}}" | grep -i "automation-controller"
90be25fac271    Up 5 hours      /usr/bin/launch_a...    automation-controller-rsyslog
e97b56dbcf1e    Up 5 hours      /usr/bin/launch_a...    automation-controller-task
29e1fef146c1    Up 5 hours      /usr/bin/launch_a...    automation-controller-web

[shadowman@aap-c ~]$ 
```
<br>

### 5.2 automation-controller-web 컨테이너

#### 5.2.1 컨테이너 구성 확인

```bash
podman container inspect automation-controller-web | jq '.[].Config|{"Config.Labels.usage": .Labels.usage, "Config.CreateCommand": .CreateCommand}'
```

실행 결과
```
[shadowman@aap-c ~]$ podman container inspect automation-controller-web | jq '.[].Config|{"Config.Labels.usage": .Labels.usage, "Config.CreateCommand": .CreateCommand}'

...<snip>...

[shadowman@aap-c ~]$ 
```

실행 결과 JSON 출력
```json
{
  "Config.Labels.usage": null,
  "Config.CreateCommand": [
    "podman",
    "container",
    "create",
    "--name",
    "automation-controller-web",
    "--log-driver",
    "journald",
    "--user",
    "1001",
    "--userns",
    "keep-id",
    "--network",
    "host",
    "--mount",
    "type=tmpfs,destination=/run/nginx,U=true",
    "--mount",
    "type=tmpfs,destination=/run/uwsgi,U=true",
    "--secret",
    "controller_secret_key,target=/etc/tower/SECRET_KEY,mode=0400,uid=1001",
    "--secret",
    "controller_channels,target=/etc/tower/conf.d/channels.py,mode=0400,uid=1001",
    "--secret",
    "controller_postgres,target=/etc/tower/conf.d/postgres.py,mode=0400,uid=1001",
    "--secret",
    "controller_resource_server,type=env,target=AWX_RESOURCE_SERVER__SECRET_KEY",
    "--volume",
    "/home/aap/aap/tls/extracted:/etc/pki/ca-trust/extracted:z",
    "--volume",
    "/home/aap/aap/controller/etc/settings.py:/etc/tower/settings.py:ro,z",
    "--volume",
    "/home/aap/aap/controller/etc/conf.d/callback_receiver_workers.py:/etc/tower/conf.d/callback_receiver_workers.py:ro,z",
    "--volume",
    "/home/aap/aap/controller/etc/conf.d/cluster_host_id.py:/etc/tower/conf.d/cluster_host_id.py:ro,z",
    "--volume",
    "/home/aap/aap/controller/etc/conf.d/container_groups.py:/etc/tower/conf.d/container_groups.py:ro,z",
    "--volume",
    "/home/aap/aap/controller/etc/conf.d/execution_environments.py:/etc/tower/conf.d/execution_environments.py:ro,z",
    "--volume",
    "/home/aap/aap/controller/etc/conf.d/insights.py:/etc/tower/conf.d/insights.py:ro,z",
    "--volume",
    "/home/aap/aap/controller/etc/conf.d/redis.py:/etc/tower/conf.d/redis.py:ro,z",
    "--volume",
    "/home/aap/aap/controller/etc/conf.d/subscription_usage_model.py:/etc/tower/conf.d/subscription_usage_model.py:ro,z",
    "--volume",
    "/home/aap/aap/controller/data/job_execution:/home/aap/aap/controller/data/job_execution:z",
    "--volume",
    "/home/aap/aap/controller/data/job_status:/home/aap/aap/controller/data/job_status:z",
    "--volume",
    "/home/aap/aap/controller/data/logs:/var/log/tower:z",
    "--volume",
    "/home/aap/aap/controller/data/projects:/home/aap/aap/controller/data/projects:z",
    "--volume",
    "/home/aap/aap/controller/data/rsyslog:/var/lib/awx/rsyslog:z",
    "--volume",
    "/home/aap/aap/controller/etc/launch_awx_task.sh:/usr/bin/launch_awx_task.sh:ro,z",
    "--volume",
    "/home/aap/aap/receptor/etc/receptor.conf:/etc/receptor/receptor.conf:ro,z",
    "--volume",
    "receptor_run:/run/receptor:U",
    "--volume",
    "redis_run:/run/redis:U",
    "--volume",
    "/home/aap/aap/controller/rsyslog/run:/run/awx-rsyslog:z",
    "--volume",
    "/home/aap/aap/controller/supervisor/run:/run/supervisor:z",
    "--volume",
    "/home/aap/aap/controller/etc/uwsgi.ini:/etc/tower/uwsgi.ini:ro,z",
    "--volume",
    "/home/aap/aap/controller/nginx/etc/controller.conf:/etc/nginx/nginx.conf:ro,z",
    "--volume",
    "/home/aap/aap/controller/nginx/etc/redirect-page.html:/var/lib/awx/venv/awx/lib/python3.11/site-packages/awx/ui/build/awx/index_awx.html:ro,z",
    "--volume",
    "controller_nginx:/var/lib/nginx:U",
    "--volume",
    "/home/aap/aap/controller/etc/tower.cert:/etc/tower/tower.cert:ro,z",
    "--volume",
    "/home/aap/aap/controller/etc/tower.key:/etc/tower/tower.key:ro,z",
    "--stop-timeout",
    "30",
    "--env",
    "SUPERVISOR_CONFIG_PATH=/etc/supervisord_web.conf",
    "--label",
    "io.containers.autoupdate=registry",
    "--label",
    "io.containers.autoupdate.authfile=/home/aap/.config/containers/auth.json",
    "registry.redhat.io/ansible-automation-platform-26/controller-rhel9:latest",
    "/usr/bin/launch_awx_web.sh"
  ]
}
```

#### 5.2.2 웹 구성 확인

```bash
podman exec -it automation-controller-web /bin/bash
cat /etc/nginx/nginx.conf
exit
```

실행 결과
```
[shadowman@aap-c ~]$ podman exec -it automation-controller-web /bin/bash

bash-4.4$ cat /etc/nginx/nginx.conf

......

http {
    
    ......

    server {
        listen 8443 default_server ssl http2;
        ......
    }

    server {
        listen 8080 default_server;
        listen [::]:8080 default_server;
        server_name _;
        return 301 https://$host:8443$request_uri;
    }
}

bash-4.4$ exit
exit

[shadowman@aap-c ~]$ 
```
* http는 8080 포트, https는 8443 포트 사용
<br>
<br>

## 6. Automation Hub

### 6.1 automation-hub 컨테이너 리스트

```bash
podman ps -a --format "{{.ID}}\t{{.Status}}\t{{.Command}}\t{{.Names}}" | grep -i "automation-hub"
```

실행 결과
```
[shadowman@aap-c ~]$ podman ps -a --format "{{.ID}}\t{{.Status}}\t{{.Command}}\t{{.Names}}" | grep -i "automation-hub"
1d6765fccdfa    Up 5 hours      pulpcore-api --na...    automation-hub-api
0008efe74b09    Up 5 hours      pulpcore-content ...    automation-hub-content
ced1a391b0fd    Up 5 hours      /bin/sh -c nginx ...    automation-hub-web
36dce89bda02    Up 5 hours      pulpcore-worker automation-hub-worker-1
0a7a7501021f    Up 5 hours      pulpcore-worker automation-hub-worker-2

[shadowman@aap-c ~]$ 
```
<br>

### 6.2 automation-hub-web 컨테이너

#### 6.2.1 컨테이너 구성 확인

```bash
podman container inspect automation-hub-web | jq '.[].Config|{"Config.Labels.usage": .Labels.usage, "Config.CreateCommand": .CreateCommand}'
```

실행 결과
```
[shadowman@aap-c ~]$ podman container inspect automation-hub-web | jq '.[].Config|{"Config.Labels.usage": .Labels.usage, "Config.CreateCommand": .CreateCommand}'
{
  "Config.Labels.usage": "s2i build <SOURCE-REPOSITORY> ubi9/nginx-124:latest <APP-NAME>",
  "Config.CreateCommand": [
    "podman",
    "container",
    "create",
    "--name",
    "automation-hub-web",
    "--log-driver",
    "journald",
    "--user",
    "1001",
    "--userns",
    "keep-id",
    "--network",
    "host",
    "--mount",
    "type=tmpfs,destination=/run/nginx,U=true",
    "--volume",
    "/home/aap/aap/hub/nginx/etc/hub.conf:/etc/nginx/nginx.conf:ro,z",
    "--volume",
    "hub_nginx:/var/lib/nginx:U",
    "--volume",
    "/home/aap/aap/hub/etc/pulp.cert:/etc/pulp/pulp.cert:ro,z",
    "--volume",
    "/home/aap/aap/hub/etc/pulp.key:/etc/pulp/pulp.key:ro,z",
    "--label",
    "io.containers.autoupdate=registry",
    "--label",
    "io.containers.autoupdate.authfile=/home/aap/.config/containers/auth.json",
    "registry.redhat.io/ansible-automation-platform-26/hub-web-rhel9:latest"
  ]
}

[shadowman@aap-c ~]$ 
```

#### 6.2.2 웹 구성 확인

```bash
podman exec -it automation-hub-web /bin/bash
cat /etc/nginx/nginx.conf
exit
```

실행 결과
```
[shadowman@aap-c ~]$ podman exec -it automation-hub-web /bin/bash

bash-4.4$ cat /etc/nginx/nginx.conf

......

http {
    
    ......

    server {
        listen 8444 default_server ssl http2;
        ......
    }

    server {
        listen 8081 default_server;
        listen [::]:8081 default_server;
        server_name _;
        return 301 https://$host:8444$request_uri;
    }
}

bash-4.4$ exit
exit

[shadowman@aap-c ~]$
```
* http는 8081 포트, https는 8444 포트 사용
<br>
<br>

## 7. Event-Driven Ansible

### 7.1 EDA 컨테이너 리스트 확인

```bash
podman ps -a --format "{{.ID}}\t{{.Status}}\t{{.Command}}\t{{.Names}}" | grep -i "automation-eda"
```

실행 결과
```
[shadowman@aap-c ~]$ podman ps -a --format "{{.ID}}\t{{.Status}}\t{{.Command}}\t{{.Names}}" | grep -i "automation-eda"
359000befa81    Up 5 hours      gunicorn --bind 1...    automation-eda-api
1e19e7ac5cde    Up 5 hours      daphne --bind 127...    automation-eda-daphne
4c361bcae441    Up 5 hours      /bin/sh -c nginx ...    automation-eda-web
3795031f5f1f    Up 5 hours      aap-eda-manage rq...    automation-eda-worker-1
f36110a2a2ee    Up 5 hours      aap-eda-manage rq...    automation-eda-worker-2
0e92d6a52974    Up 5 hours      aap-eda-manage rq...    automation-eda-activation-worker-1
87738cb387ca    Up 5 hours      aap-eda-manage rq...    automation-eda-activation-worker-2
c63fcbb3a02e    Up 5 hours      aap-eda-manage sc...    automation-eda-scheduler

[shadowman@aap-c ~]$ 
```
<br>

### 7.2 automation-eda-web 컨테이너

#### 7.2.1 컨테이너 구성 확인

```bash
podman container inspect automation-eda-web | jq '.[].Config|{"Config.Labels.usage": .Labels.usage, "Config.CreateCommand": .CreateCommand}'
```

실행 결과
```
[shadowman@aap-c ~]$ podman container inspect automation-eda-web | jq '.[].Config|{"Config.Labels.usage": .Labels.usage, "Config.CreateCommand": .CreateCommand}'
{
  "Config.Labels.usage": "s2i build <SOURCE-REPOSITORY> ubi9/nginx-124:latest <APP-NAME>",
  "Config.CreateCommand": [
    "podman",
    "container",
    "create",
    "--name",
    "automation-eda-web",
    "--log-driver",
    "journald",
    "--user",
    "1001",
    "--userns",
    "keep-id",
    "--network",
    "host",
    "--mount",
    "type=tmpfs,destination=/run/nginx,U=true",
    "--volume",
    "/home/aap/aap/eda/nginx/etc/eda.conf:/etc/nginx/nginx.conf:ro,z",
    "--volume",
    "/home/aap/aap/eda/nginx/etc/redirect-page.html:/var/lib/ansible-automation-platform/eda/index.html:ro,z",
    "--volume",
    "eda_nginx:/var/lib/nginx:U",
    "--volume",
    "/home/aap/aap/eda/etc/eda.cert:/etc/eda/eda.cert:ro,z",
    "--volume",
    "/home/aap/aap/eda/etc/eda.key:/etc/eda/eda.key:ro,z",
    "--label",
    "io.containers.autoupdate=registry",
    "--label",
    "io.containers.autoupdate.authfile=/home/aap/.config/containers/auth.json",
    "registry.redhat.io/ansible-automation-platform-26/eda-controller-ui-rhel9:latest"
  ]
}

[shadowman@aap-c ~]$ 
```

#### 7.2.2 웹 구성 확인

```bash
podman exec -it automation-eda-web /bin/bash
cat /etc/nginx/nginx.conf
exit
```

실행 결과
```
[shadowman@aap-c ~]$ podman exec -it automation-eda-web /bin/bash

bash-4.4$ cat /etc/nginx/nginx.conf

......

http {
    
    ......

    server {
        listen 8445 default_server ssl http2;
        ......
    }

    server {
        listen 8082 default_server;
        listen [::]:8082 default_server;
        server_name _;
        return 301 https://$host:8445$request_uri;
    }
}

bash-4.4$ exit
exit

[shadowman@aap-c ~]$ 
```
<br>
<br>

------
[차례](../README.md)