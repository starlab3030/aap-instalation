# 2대의 노드에 AAP 구성

## 1. 설치 준비

### 1.1 설치 파일 확인

설치를 위해 해당 위치로 이동
```bash
cd ~/temp/ansible-setup/
```

실행 결과
```
[shadowman@aap-c1 ~]$ cd ~/temp/ansible-setup/

[shadowman@aap-c1 ansible-setup]$
```
<br>

### 1.2 인벤토리 파일 확인

Two-Node 설치를 위한 인벤토리 파일
```bash
cat inventory
```

실행 결과
```
[shadowman@aap-c1 ansible-setup]$ cat inventory
...<snip>...

[shadowman@aap-c1 ansible-setup]$ 
```

inventory 파일
```ini
[automationgateway]
aap-c1.thinkmore.net
aap-c2.thinkmore.net

[automationcontroller]
aap-c1.thinkmore.net
aap-c2.thinkmore.net

[automationhub]
aap-c1.thinkmore.net
aap-c2.thinkmore.net

[automationeda]
aap-c1.thinkmore.net
aap-c2.thinkmore.net

[redis]
aap-c1.thinkmore.net
aap-c2.thinkmore.net

[all:vars]
# For PostgreSQL
#
postgresql_admin_username=postgres
postgresql_admin_password=redhat

# For Redis
#
redis_mode=standalone

# To install AAP w/ bundles
#
bundle_install=true
bundle_dir=/home/shadowman/temp/ansible-setup/bundle

# For Gateway
#
gateway_admin_password=redhat
gateway_pg_host=aap-db.thinkmore.net
gateway_pg_password=redhat

# For Automation Controller
#
controller_admin_password=redhat
controller_pg_host=aap-db.thinkmore.net
controller_pg_password=redhat

# For Automation Hub
#
hub_admin_password=redhat
hub_pg_host=aap-db.thinkmore.net
hub_pg_password=redhat
hub_shared_data_path="192.168.0.3:/ext/nfs/aap/hub"

# For Event-Drive Ansible
#
eda_admin_password=redhat
eda_pg_host=aap-db.thinkmore.net
eda_pg_password=redhat
```
<br>
<br>

## 2. Two-Node AAP 설치

```bash
ansible-playbook -i inventory ansible.containerized_installer.install
```

실행 결과
```
[shadowman@aap-c1 ansible-setup]$ ansible-playbook -i inventory ansible.containerized_installer.install

...<snip>...

PLAY [Delete the ansible base shared secret] ***************************************************************************************

TASK [Delete the ansible base shared secret] ***************************************************************************************
ok: [aap-c1.thinkmore.net]
ok: [aap-c2.thinkmore.net]

PLAY RECAP *************************************************************************************************************************
aap-c1.thinkmore.net       : ok=582  changed=205  unreachable=0    failed=0    skipped=144  rescued=0    ignored=0
aap-c2.thinkmore.net       : ok=449  changed=146  unreachable=0    failed=0    skipped=109  rescued=0    ignored=0
localhost                  : ok=25   changed=0    unreachable=0    failed=0    skipped=38   rescued=0    ignored=0

[shadowman@aap-c1 ansible-setup]$
```
<br>
<br>

## 3. 설치된 AAP 확인

### 3.1 aap-c1 노드 확인

#### 3.1.1 AAP 이미지 리스트

```bash
podman image ls
podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
```

실행 환경
```
[shadowman@aap-c1 ~]$ podman image ls
REPOSITORY                                                                 TAG         IMAGE ID      CREATED      SIZE
registry.redhat.io/rhel8/redis-6                                           latest      c99cedb5e4c1  13 days ago  329 MB
registry.redhat.io/ansible-automation-platform-25/hub-web-rhel8            latest      741a253ea19e  2 weeks ago  508 MB
registry.redhat.io/ansible-automation-platform-25/eda-controller-ui-rhel8  latest      04ae1753b4d7  2 weeks ago  512 MB
registry.redhat.io/ansible-automation-platform-25/receptor-rhel8           latest      ae9c0820d61d  2 weeks ago  598 MB
registry.redhat.io/ansible-automation-platform-25/hub-rhel8                latest      b9a82993ab1c  2 weeks ago  1.3 GB
registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8     latest      1f5b20f9596d  2 weeks ago  1.01 GB
registry.redhat.io/ansible-automation-platform-25/controller-rhel8         latest      3d0cb3204030  2 weeks ago  1.74 GB
registry.redhat.io/ansible-automation-platform-25/gateway-rhel8            latest      f768a98a64be  2 weeks ago  940 MB
registry.redhat.io/ansible-automation-platform-25/gateway-proxy-rhel8      latest      6e53887286a7  2 weeks ago  497 MB

[shadowman@aap-c1 ~]$ podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
Names: registry.redhat.io/rhel8/redis-6:latest
Names: registry.redhat.io/ansible-automation-platform-25/hub-web-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/eda-controller-ui-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/receptor-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/gateway-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/gateway-proxy-rhel8:latest

[shadowman@aap-c1 ~]$
```

#### 3.1.2 실행 중인 컨테이너 리스트

```bash
podman ps
podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
podman ps --format json | jq '[.[]|{"Image": .Image, "Names": .Names[0]}]'
```

실행 환경
```
[shadowman@aap-c1 ~]$ podman ps
CONTAINER ID  IMAGE                                                                             COMMAND               CREATED         STATUS         PORTS       NAMES
e35ea8ece2ec  registry.redhat.io/rhel8/redis-6:latest                                           run-redis             58 minutes ago  Up 58 minutes              redis-unix
6d99b32c8961  registry.redhat.io/rhel8/redis-6:latest                                           run-redis             58 minutes ago  Up 58 minutes              redis-tcp
59d1eeb759d5  registry.redhat.io/ansible-automation-platform-25/gateway-proxy-rhel8:latest      /usr/bin/envoy --...  57 minutes ago  Up 55 minutes              automation-gateway-proxy
1b224be037cd  registry.redhat.io/ansible-automation-platform-25/gateway-rhel8:latest            /usr/bin/supervis...  57 minutes ago  Up 55 minutes              automation-gateway
d8b82ffc7fda  registry.redhat.io/ansible-automation-platform-25/receptor-rhel8:latest           /usr/bin/receptor...  55 minutes ago  Up 55 minutes              receptor
f7168510c252  registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest         /usr/bin/launch_a...  54 minutes ago  Up 50 minutes              automation-controller-rsyslog
368896a7dde9  registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest         /usr/bin/launch_a...  54 minutes ago  Up 50 minutes              automation-controller-task
039407642bbd  registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest         /usr/bin/launch_a...  54 minutes ago  Up 50 minutes              automation-controller-web
74a165cafd48  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     gunicorn --bind 1...  49 minutes ago  Up 48 minutes              automation-eda-api
e81321e9f1d6  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     daphne --bind 127...  49 minutes ago  Up 48 minutes              automation-eda-daphne
3f9e62eb7d2a  registry.redhat.io/ansible-automation-platform-25/eda-controller-ui-rhel8:latest  /bin/sh -c nginx ...  49 minutes ago  Up 48 minutes              automation-eda-web
a1a483c0e795  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     aap-eda-manage rq...  49 minutes ago  Up 48 minutes              automation-eda-worker-1
ffec382e42b9  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     aap-eda-manage rq...  49 minutes ago  Up 48 minutes              automation-eda-worker-2
5bd90aa51ee0  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     aap-eda-manage rq...  49 minutes ago  Up 48 minutes              automation-eda-activation-worker-1
52ab64f19123  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     aap-eda-manage rq...  49 minutes ago  Up 48 minutes              automation-eda-activation-worker-2
d7dd9643f0d4  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     aap-eda-manage sc...  49 minutes ago  Up 48 minutes              automation-eda-scheduler
bb284cd995d0  registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest                pulpcore-api --na...  47 minutes ago  Up 45 minutes              automation-hub-api
6f83a94234d8  registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest                pulpcore-content ...  47 minutes ago  Up 45 minutes              automation-hub-content
0e0291e20bab  registry.redhat.io/ansible-automation-platform-25/hub-web-rhel8:latest            /bin/sh -c nginx ...  47 minutes ago  Up 45 minutes              automation-hub-web
1497aeb18a13  registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest                pulpcore-worker       47 minutes ago  Up 45 minutes              automation-hub-worker-1
ffc238ee41e2  registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest                pulpcore-worker       47 minutes ago  Up 45 minutes              automation-hub-worker-2

[shadowman@aap-c1 ~]$ podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
e35ea8ece2ec    run-redis       redis-unix
6d99b32c8961    run-redis       redis-tcp
59d1eeb759d5    /usr/bin/envoy --...    automation-gateway-proxy
1b224be037cd    /usr/bin/supervis...    automation-gateway
d8b82ffc7fda    /usr/bin/receptor...    receptor
f7168510c252    /usr/bin/launch_a...    automation-controller-rsyslog
368896a7dde9    /usr/bin/launch_a...    automation-controller-task
039407642bbd    /usr/bin/launch_a...    automation-controller-web
74a165cafd48    gunicorn --bind 1...    automation-eda-api
e81321e9f1d6    daphne --bind 127...    automation-eda-daphne
3f9e62eb7d2a    /bin/sh -c nginx ...    automation-eda-web
a1a483c0e795    aap-eda-manage rq...    automation-eda-worker-1
ffec382e42b9    aap-eda-manage rq...    automation-eda-worker-2
5bd90aa51ee0    aap-eda-manage rq...    automation-eda-activation-worker-1
52ab64f19123    aap-eda-manage rq...    automation-eda-activation-worker-2
d7dd9643f0d4    aap-eda-manage sc...    automation-eda-scheduler
bb284cd995d0    pulpcore-api --na...    automation-hub-api
6f83a94234d8    pulpcore-content ...    automation-hub-content
0e0291e20bab    /bin/sh -c nginx ...    automation-hub-web
1497aeb18a13    pulpcore-worker automation-hub-worker-1
ffc238ee41e2    pulpcore-worker automation-hub-worker-2

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

#### 3.1.3 설치된 AAP

```bash
ls -ldh ~/aap
tree -F -L 1 ~/aap
```

실행 환경
```
[shadowman@aap-c1 ~]$ ls -ldh ~/aap
drwxrwx---. 11 shadowman shadowman 139 Oct 18 08:11 /home/shadowman/aap

[shadowman@aap-c1 ~]$ tree -F -L 1 ~/aap
/home/shadowman/aap
├── containers/
├── controller/
├── eda/
├── gateway/
├── gatewayproxy/
├── hub/
├── receptor/
├── redis/
└── tls/

9 directories, 0 files

[shadowman@aap-c1 ~]$
```

#### 3.1.4 AAP 서비스

```bash
systemctl list-units --type=service --state=running --user
```

실행 결과
```
[shadowman@aap-c1 ~]$ systemctl list-units --type=service --state=running --user
  UNIT                                                LOAD   ACTIVE SUB     DESCRIPTION                                            >
  at-spi-dbus-bus.service                             loaded active running Accessibility services bus
  automation-controller-rsyslog.service               loaded active running Podman automation-controller-rsyslog.service
  automation-controller-task.service                  loaded active running Podman automation-controller-task.service
  automation-controller-web.service                   loaded active running Podman automation-controller-web.service
  automation-eda-activation-worker-1.service          loaded active running Podman automation-eda-activation-worker-1.service
  automation-eda-activation-worker-2.service          loaded active running Podman automation-eda-activation-worker-2.service
  automation-eda-api.service                          loaded active running Podman automation-eda-api.service
  automation-eda-daphne.service                       loaded active running Podman automation-eda-daphne.service
  automation-eda-scheduler.service                    loaded active running Podman automation-eda-scheduler.service
  automation-eda-web.service                          loaded active running Podman automation-eda-web.service
  automation-eda-worker-1.service                     loaded active running Podman automation-eda-worker-1.service
  automation-eda-worker-2.service                     loaded active running Podman automation-eda-worker-2.service
  automation-gateway-proxy.service                    loaded active running Podman automation-gateway-proxy.service
  automation-gateway.service                          loaded active running Podman automation-gateway.service
  automation-hub-api.service                          loaded active running Podman automation-hub-api.service
  automation-hub-content.service                      loaded active running Podman automation-hub-content.service
  automation-hub-web.service                          loaded active running Podman automation-hub-web.service
  automation-hub-worker-1.service                     loaded active running Podman automation-hub-worker-1.service
  automation-hub-worker-2.service                     loaded active running Podman automation-hub-worker-2.service
  dbus-:1.1097-org.a11y.atspi.Registry@0.service      loaded active running dbus-:1.1097-org.a11y.atspi.Registry@0.service
  dbus-:1.2-org.freedesktop.portal.IBus@1.service     loaded active running dbus-:1.2-org.freedesktop.portal.IBus@1.service
  dbus-:1.2-org.gnome.Identity@0.service              loaded active running dbus-:1.2-org.gnome.Identity@0.service
  dbus-:1.2-org.gnome.OnlineAccounts@0.service        loaded active running dbus-:1.2-org.gnome.OnlineAccounts@0.service
  dbus-:1.2-org.gnome.ScreenSaver@0.service           loaded active running dbus-:1.2-org.gnome.ScreenSaver@0.service
  dbus-:1.2-org.gnome.Shell.CalendarServer@0.service  loaded active running dbus-:1.2-org.gnome.Shell.CalendarServer@0.service
  dbus-:1.2-org.gnome.Shell.Notifications@0.service   loaded active running dbus-:1.2-org.gnome.Shell.Notifications@0.service
  dbus-broker.service                                 loaded active running D-Bus User Message Bus
  dconf.service                                       loaded active running User preferences database
  evolution-addressbook-factory.service               loaded active running Evolution address book service
  evolution-calendar-factory.service                  loaded active running Evolution calendar service
  evolution-source-registry.service                   loaded active running Evolution source registry
  gnome-session-manager@gnome.service                 loaded active running GNOME Session Manager (session: gnome)
  gnome-session-monitor.service                       loaded active running Monitor Session leader for GNOME Session
  gvfs-daemon.service                                 loaded active running Virtual filesystem service
  gvfs-goa-volume-monitor.service                     loaded active running Virtual filesystem service - GNOME Online Accounts moni>
  gvfs-gphoto2-volume-monitor.service                 loaded active running Virtual filesystem service - digital camera monitor
  gvfs-metadata.service                               loaded active running Virtual filesystem metadata service
  gvfs-mtp-volume-monitor.service                     loaded active running Virtual filesystem service - Media Transfer Protocol mo>
  gvfs-udisks2-volume-monitor.service                 loaded active running Virtual filesystem service - disk device monitor
  org.gnome.SettingsDaemon.A11ySettings.service       loaded active running GNOME accessibility service
  org.gnome.SettingsDaemon.Color.service              loaded active running GNOME color management service
  org.gnome.SettingsDaemon.Datetime.service           loaded active running GNOME date & time service
  org.gnome.SettingsDaemon.Housekeeping.service       loaded active running GNOME maintenance of expirable data service
  org.gnome.SettingsDaemon.Keyboard.service           loaded active running GNOME keyboard configuration service
  org.gnome.SettingsDaemon.MediaKeys.service          loaded active running GNOME keyboard shortcuts service
  org.gnome.SettingsDaemon.Power.service              loaded active running GNOME power management service
  org.gnome.SettingsDaemon.PrintNotifications.service loaded active running GNOME printer notifications service
  org.gnome.SettingsDaemon.Rfkill.service             loaded active running GNOME RFKill support service
  org.gnome.SettingsDaemon.ScreensaverProxy.service   loaded active running GNOME FreeDesktop screensaver service
  org.gnome.SettingsDaemon.Sharing.service            loaded active running GNOME file sharing service
  org.gnome.SettingsDaemon.Smartcard.service          loaded active running GNOME smartcard service
  org.gnome.SettingsDaemon.Sound.service              loaded active running GNOME sound sample caching service
  org.gnome.SettingsDaemon.Subscription.service       loaded active running GNOME subscription management service
  org.gnome.SettingsDaemon.UsbProtection.service      loaded active running GNOME USB protection service
  org.gnome.SettingsDaemon.Wacom.service              loaded active running GNOME Wacom tablet support service
  org.gnome.SettingsDaemon.XSettings.service          loaded active running GNOME XSettings service
  org.gnome.Shell@wayland.service                     loaded active running GNOME Shell on Wayland
  pipewire-pulse.service                              loaded active running PipeWire PulseAudio
  pipewire.service                                    loaded active running PipeWire Multimedia Service
  receptor.service                                    loaded active running Podman receptor.service
  redis-tcp.service                                   loaded active running Podman redis-tcp.service
  redis-unix.service                                  loaded active running Podman redis-unix.service
  wireplumber.service                                 loaded active running Multimedia Service Session Manager
  xdg-desktop-portal-gnome.service                    loaded active running Portal service (GNOME implementation)
  xdg-desktop-portal-gtk.service                      loaded active running Portal service (GTK/GNOME implementation)
  xdg-desktop-portal.service                          loaded active running Portal service
  xdg-document-portal.service                         loaded active running flatpak document portal service
  xdg-permission-store.service                        loaded active running sandboxed app permission store

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.
68 loaded units listed.

[shadowman@aap-c1 ~]$
```

#### 3.1.5 호스트 상의 컨테이너 사용 리소스

```bash
podman container stats -a
```

실행 환경
```
[shadowman@aap-c1 ~]$ podman container stats -a
ID            NAME                                CPU %       MEM USAGE / LIMIT  MEM %       NET IO      BLOCK IO    PIDS        CPU TIME      AVG CPU %
e35ea8ece2ec  redis-unix                          0.36%       3.129MB / 8.058GB  0.04%       0B / 0B     0B / 0B     5           12.540024s    0.34%
6d99b32c8961  redis-tcp                           1.07%       11.32MB / 8.058GB  0.14%       0B / 0B     0B / 0B     5           36.736627s    1.00%
59d1eeb759d5  automation-gateway-proxy            0.78%       22.09MB / 8.058GB  0.27%       0B / 0B     0B / 0B     17          43.620466s    1.24%
1b224be037cd  automation-gateway                  3.09%       886MB / 8.058GB    10.99%      0B / 0B     0B / 0B     78          2m26.751633s  4.18%
d8b82ffc7fda  receptor                            0.01%       8.143MB / 8.058GB  0.10%       0B / 0B     0B / 0B     8           465.768ms     0.01%
f7168510c252  automation-controller-rsyslog       0.04%       32.96MB / 8.058GB  0.41%       0B / 0B     0B / 0B     7           8.063093s     0.25%
368896a7dde9  automation-controller-task          3.12%       271.4MB / 8.058GB  3.37%       0B / 0B     0B / 0B     26          1m25.10056s   2.64%
039407642bbd  automation-controller-web           0.16%       942.1MB / 8.058GB  11.69%      0B / 0B     0B / 0B     22          1m15.937825s  2.37%
74a165cafd48  automation-eda-api                  0.05%       302.5MB / 8.058GB  3.75%       0B / 0B     0B / 0B     11          29.555232s    0.96%
e81321e9f1d6  automation-eda-daphne               0.04%       5.566MB / 8.058GB  0.07%       0B / 0B     0B / 0B     2           2.820133s     0.09%
3f9e62eb7d2a  automation-eda-web                  0.00%       4.342MB / 8.058GB  0.05%       0B / 0B     0B / 0B     5           527.839ms     0.02%
a1a483c0e795  automation-eda-worker-1             0.12%       16.95MB / 8.058GB  0.21%       0B / 0B     0B / 0B     3           32.379118s    1.05%
ffec382e42b9  automation-eda-worker-2             0.12%       32.1MB / 8.058GB   0.40%       0B / 0B     0B / 0B     3           31.511078s    1.02%
5bd90aa51ee0  automation-eda-activation-worker-1  0.29%       6.418MB / 8.058GB  0.08%       0B / 0B     0B / 0B     3           9.233474s     0.30%
52ab64f19123  automation-eda-activation-worker-2  0.27%       2.99MB / 8.058GB   0.04%       0B / 0B     0B / 0B     3           9.135548s     0.30%
d7dd9643f0d4  automation-eda-scheduler            0.13%       9.273MB / 8.058GB  0.12%       0B / 0B     0B / 0B     2           4.930636s     0.16%
bb284cd995d0  automation-hub-api                  3.35%       536.6MB / 8.058GB  6.66%       0B / 0B     0B / 0B     10          1m46.112836s  3.67%
6f83a94234d8  automation-hub-content              0.15%       162.6MB / 8.058GB  2.02%       0B / 0B     0B / 0B     9           23.180666s    0.80%
0e0291e20bab  automation-hub-web                  0.01%       6.267MB / 8.058GB  0.08%       0B / 0B     0B / 0B     5           430.811ms     0.01%
1497aeb18a13  automation-hub-worker-1             0.34%       109.7MB / 8.058GB  1.36%       0B / 0B     0B / 0B     1           6.756801s     0.23%
ffc238ee41e2  automation-hub-worker-2             0.00%       120MB / 8.058GB    1.49%       0B / 0B     0B / 0B     1           6.31343s      0.22%

Ctrl+C

[shadowman@aap-c1 ~]$
```
<br>

### 3.2 aap-c2 노드 확인

#### 3.2.1 AAP 이미지 리스트

```bash
podman image ls
podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
```

실행 환경
```
[shadowman@aap-c2 ~]$ podman image ls
REPOSITORY                                                                 TAG         IMAGE ID      CREATED      SIZE
registry.redhat.io/rhel8/redis-6                                           latest      c99cedb5e4c1  13 days ago  329 MB
registry.redhat.io/ansible-automation-platform-25/hub-web-rhel8            latest      741a253ea19e  2 weeks ago  508 MB
registry.redhat.io/ansible-automation-platform-25/eda-controller-ui-rhel8  latest      04ae1753b4d7  2 weeks ago  512 MB
registry.redhat.io/ansible-automation-platform-25/receptor-rhel8           latest      ae9c0820d61d  2 weeks ago  598 MB
registry.redhat.io/ansible-automation-platform-25/hub-rhel8                latest      b9a82993ab1c  2 weeks ago  1.3 GB
registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8     latest      1f5b20f9596d  2 weeks ago  1.01 GB
registry.redhat.io/ansible-automation-platform-25/controller-rhel8         latest      3d0cb3204030  2 weeks ago  1.74 GB
registry.redhat.io/ansible-automation-platform-25/gateway-rhel8            latest      f768a98a64be  2 weeks ago  940 MB
registry.redhat.io/ansible-automation-platform-25/gateway-proxy-rhel8      latest      6e53887286a7  2 weeks ago  497 MB

[shadowman@aap-c2 ~]$ podman image ls --format json | jq -r '.[]|"Names: \(.Names[])"'
Names: registry.redhat.io/rhel8/redis-6:latest
Names: registry.redhat.io/ansible-automation-platform-25/hub-web-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/eda-controller-ui-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/receptor-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/gateway-rhel8:latest
Names: registry.redhat.io/ansible-automation-platform-25/gateway-proxy-rhel8:latest

[shadowman@aap-c2 ~]$
```

#### 3.2.2 실행 중인 컨테이너 리스트

```bash
podman ps
podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
podman ps --format json | jq '[.[]|{"Image": .Image, "Names": .Names[0]}]'
```

실행 환경
```
[shadowman@aap-c2 ~]$ podman ps
CONTAINER ID  IMAGE                                                                             COMMAND               CREATED            STATUS            PORTS       NAMES
4518fd9f56ec  registry.redhat.io/rhel8/redis-6:latest                                           run-redis             About an hour ago  Up About an hour              redis-unix
ef3c7f085d05  registry.redhat.io/ansible-automation-platform-25/gateway-proxy-rhel8:latest      /usr/bin/envoy --...  About an hour ago  Up About an hour              automation-gateway-proxy
2cce90413ece  registry.redhat.io/ansible-automation-platform-25/gateway-rhel8:latest            /usr/bin/supervis...  About an hour ago  Up About an hour              automation-gateway
09e16f796bce  registry.redhat.io/ansible-automation-platform-25/receptor-rhel8:latest           /usr/bin/receptor...  About an hour ago  Up About an hour              receptor
61a09ceeaf1f  registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest         /usr/bin/launch_a...  About an hour ago  Up 57 minutes                 automation-controller-rsyslog
ca86d2a20aba  registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest         /usr/bin/launch_a...  About an hour ago  Up 57 minutes                 automation-controller-task
93b24c0b956d  registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest         /usr/bin/launch_a...  About an hour ago  Up 56 minutes                 automation-controller-web
543c31c577c2  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     gunicorn --bind 1...  55 minutes ago     Up 54 minutes                 automation-eda-api
71422bd1ed4b  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     daphne --bind 127...  55 minutes ago     Up 54 minutes                 automation-eda-daphne
3eb67e470a68  registry.redhat.io/ansible-automation-platform-25/eda-controller-ui-rhel8:latest  /bin/sh -c nginx ...  55 minutes ago     Up 54 minutes                 automation-eda-web
1a64ebc178a5  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     aap-eda-manage rq...  55 minutes ago     Up 54 minutes                 automation-eda-worker-1
74f19c671d44  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     aap-eda-manage rq...  55 minutes ago     Up 54 minutes                 automation-eda-worker-2
9f26cc389422  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     aap-eda-manage rq...  55 minutes ago     Up 54 minutes                 automation-eda-activation-worker-1
521413bf5aa3  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     aap-eda-manage rq...  55 minutes ago     Up 54 minutes                 automation-eda-activation-worker-2
ba29dde31d5b  registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest     aap-eda-manage sc...  55 minutes ago     Up 54 minutes                 automation-eda-scheduler
11f6109e29be  registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest                pulpcore-api --na...  53 minutes ago     Up 51 minutes                 automation-hub-api
a7214b0f79df  registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest                pulpcore-content ...  53 minutes ago     Up 51 minutes                 automation-hub-content
3778506c4863  registry.redhat.io/ansible-automation-platform-25/hub-web-rhel8:latest            /bin/sh -c nginx ...  53 minutes ago     Up 51 minutes                 automation-hub-web
87e23fa5fcab  registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest                pulpcore-worker       53 minutes ago     Up 51 minutes                 automation-hub-worker-1
059b2db7a424  registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest                pulpcore-worker       53 minutes ago     Up 51 minutes                 automation-hub-worker-2

[shadowman@aap-c2 ~]$ podman ps --format "{{.ID}}\t{{.Command}}\t{{.Names}}"
4518fd9f56ec    run-redis       redis-unix
ef3c7f085d05    /usr/bin/envoy --...    automation-gateway-proxy
2cce90413ece    /usr/bin/supervis...    automation-gateway
09e16f796bce    /usr/bin/receptor...    receptor
61a09ceeaf1f    /usr/bin/launch_a...    automation-controller-rsyslog
ca86d2a20aba    /usr/bin/launch_a...    automation-controller-task
93b24c0b956d    /usr/bin/launch_a...    automation-controller-web
543c31c577c2    gunicorn --bind 1...    automation-eda-api
71422bd1ed4b    daphne --bind 127...    automation-eda-daphne
3eb67e470a68    /bin/sh -c nginx ...    automation-eda-web
1a64ebc178a5    aap-eda-manage rq...    automation-eda-worker-1
74f19c671d44    aap-eda-manage rq...    automation-eda-worker-2
9f26cc389422    aap-eda-manage rq...    automation-eda-activation-worker-1
521413bf5aa3    aap-eda-manage rq...    automation-eda-activation-worker-2
ba29dde31d5b    aap-eda-manage sc...    automation-eda-scheduler
11f6109e29be    pulpcore-api --na...    automation-hub-api
a7214b0f79df    pulpcore-content ...    automation-hub-content
3778506c4863    /bin/sh -c nginx ...    automation-hub-web
87e23fa5fcab    pulpcore-worker automation-hub-worker-1
059b2db7a424    pulpcore-worker automation-hub-worker-2

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
    "Image": "registry.redhat.io/ansible-automation-platform-25/gateway-proxy-rhel8:latest",
    "Names": "automation-gateway-proxy"
  },
  {
    "Image": "registry.redhat.io/ansible-automation-platform-25/gateway-rhel8:latest",
    "Names": "automation-gateway"
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

#### 3.2.3 설치된 AAP

```bash
ls -ldh ~/aap
tree -F -L 1 ~/aap
```

실행 환경
```
[shadowman@aap-c2 ~]$ ls -ldh ~/aap
drwxrwx---. 11 shadowman shadowman 139 Oct 18 08:11 /home/shadowman/aap

[shadowman@aap-c2 ~]$ tree -F -L 1 ~/aap
/home/shadowman/aap
├── containers/
├── controller/
├── eda/
├── gateway/
├── gatewayproxy/
├── hub/
├── receptor/
├── redis/
└── tls/

9 directories, 0 files

[shadowman@aap-c2 ~]$
```

#### 3.2.4 AAP 서비스

```bash
systemctl list-units --type=service --state=running --user
```

실행 결과
```
[shadowman@aap-c2 ~]$ systemctl list-units --type=service --state=running --user
  UNIT                                       LOAD   ACTIVE SUB     DESCRIPTION
  automation-controller-rsyslog.service      loaded active running Podman automation-controller-rsyslog.service
  automation-controller-task.service         loaded active running Podman automation-controller-task.service
  automation-controller-web.service          loaded active running Podman automation-controller-web.service
  automation-eda-activation-worker-1.service loaded active running Podman automation-eda-activation-worker-1.service
  automation-eda-activation-worker-2.service loaded active running Podman automation-eda-activation-worker-2.service
  automation-eda-api.service                 loaded active running Podman automation-eda-api.service
  automation-eda-daphne.service              loaded active running Podman automation-eda-daphne.service
  automation-eda-scheduler.service           loaded active running Podman automation-eda-scheduler.service
  automation-eda-web.service                 loaded active running Podman automation-eda-web.service
  automation-eda-worker-1.service            loaded active running Podman automation-eda-worker-1.service
  automation-eda-worker-2.service            loaded active running Podman automation-eda-worker-2.service
  automation-gateway-proxy.service           loaded active running Podman automation-gateway-proxy.service
  automation-gateway.service                 loaded active running Podman automation-gateway.service
  automation-hub-api.service                 loaded active running Podman automation-hub-api.service
  automation-hub-content.service             loaded active running Podman automation-hub-content.service
  automation-hub-web.service                 loaded active running Podman automation-hub-web.service
  automation-hub-worker-1.service            loaded active running Podman automation-hub-worker-1.service
  automation-hub-worker-2.service            loaded active running Podman automation-hub-worker-2.service
  dbus-broker.service                        loaded active running D-Bus User Message Bus
  receptor.service                           loaded active running Podman receptor.service
  redis-unix.service                         loaded active running Podman redis-unix.service

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.
21 loaded units listed.

[shadowman@aap-c2 ~]$
```

#### 3.2.5 호스트 상의 컨테이너 사용 리소스

```bash
podman container stats -a
```

실행 환경
```
[shadowman@aap-c2 ~]$ podman container stats -a
ID            NAME                                CPU %       MEM USAGE / LIMIT  MEM %       NET IO      BLOCK IO    PIDS        CPU TIME      AVG CPU %
4518fd9f56ec  redis-unix                          0.34%       9.478MB / 8.058GB  0.12%       0B / 0B     0B / 0B     5           13.688111s    0.34%
ef3c7f085d05  automation-gateway-proxy            0.51%       12.51MB / 8.058GB  0.16%       0B / 0B     0B / 0B     16          19.71255s     0.51%
2cce90413ece  automation-gateway                  2.71%       970MB / 8.058GB    12.04%      0B / 0B     0B / 0B     68          1m44.377156s  2.71%
09e16f796bce  receptor                            0.01%       12.52MB / 8.058GB  0.16%       0B / 0B     0B / 0B     8           500.272ms     0.01%
61a09ceeaf1f  automation-controller-rsyslog       0.21%       112.6MB / 8.058GB  1.40%       0B / 0B     0B / 0B     7           7.460135s     0.21%
ca86d2a20aba  automation-controller-task          2.62%       377.4MB / 8.058GB  4.68%       0B / 0B     0B / 0B     26          1m33.064494s  2.62%
93b24c0b956d  automation-controller-web           2.27%       1.251GB / 8.058GB  15.52%      0B / 0B     0B / 0B     22          1m20.358746s  2.27%
543c31c577c2  automation-eda-api                  0.87%       518.7MB / 8.058GB  6.44%       0B / 0B     0B / 0B     11          29.623463s    0.87%
71422bd1ed4b  automation-eda-daphne               0.09%       111.3MB / 8.058GB  1.38%       0B / 0B     0B / 0B     2           2.941098s     0.09%
3eb67e470a68  automation-eda-web                  0.02%       6.095MB / 8.058GB  0.08%       0B / 0B     0B / 0B     5           615.315ms     0.02%
1a64ebc178a5  automation-eda-worker-1             1.25%       113.6MB / 8.058GB  1.41%       0B / 0B     0B / 0B     3           42.753474s    1.25%
74f19c671d44  automation-eda-worker-2             1.27%       113.5MB / 8.058GB  1.41%       0B / 0B     0B / 0B     3           43.296454s    1.27%
9f26cc389422  automation-eda-activation-worker-1  0.30%       113.4MB / 8.058GB  1.41%       0B / 0B     0B / 0B     3           10.196491s    0.30%
521413bf5aa3  automation-eda-activation-worker-2  0.30%       113.4MB / 8.058GB  1.41%       0B / 0B     0B / 0B     3           10.121023s    0.30%
ba29dde31d5b  automation-eda-scheduler            0.07%       113.6MB / 8.058GB  1.41%       0B / 0B     0B / 0B     2           2.383628s     0.07%
11f6109e29be  automation-hub-api                  3.36%       1.138GB / 8.058GB  14.13%      0B / 0B     0B / 0B     10          1m48.650202s  3.36%
a7214b0f79df  automation-hub-content              0.75%       206.9MB / 8.058GB  2.57%       0B / 0B     0B / 0B     9           24.150196s    0.75%
3778506c4863  automation-hub-web                  0.01%       6.357MB / 8.058GB  0.08%       0B / 0B     0B / 0B     5           481.544ms     0.01%
87e23fa5fcab  automation-hub-worker-1             0.22%       120.3MB / 8.058GB  1.49%       0B / 0B     0B / 0B     1           7.167187s     0.22%
059b2db7a424  automation-hub-worker-2             0.21%       120.3MB / 8.058GB  1.49%       0B / 0B     0B / 0B     1           6.705983s     0.21%

Ctrl+C

[shadowman@aap-c2 ~]$
```
<br>

### 3.3 AAP 데이터베이스 

#### 3.3.1 데이터베이스 연결 확인

DB 연결 확인
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
    Name     |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-------------+----------+----------+------------+------------+-----------------------
 awx         | awx      | UTF8     | en_US.utf8 | en_US.utf8 |
 eda         | eda      | UTF8     | en_US.utf8 | en_US.utf8 |
 gateway     | gateway  | UTF8     | en_US.utf8 | en_US.utf8 |
 my_database | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 postgres    | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 pulp        | pulp     | UTF8     | en_US.utf8 | en_US.utf8 |
 template0   | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
             |          |          |            |            | postgres=CTc/postgres
 template1   | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
             |          |          |            |            | postgres=CTc/postgres
(8 rows)

postgres=# \q

[shadowman@aap-c1 ~]$
```
<br>
<br>

## 4. AAP 연결 테스트

### 4.1 앤서블 게이트웨이 연결 

|$\color{lime}{\texttt{항목}}$|$\color{lime}{\texttt{값}}$|
|:---|:---|
|`URL`|aap-c1.thinkmore.net<br>aap-c2.thinkmore.net|
|`사용자`|admin|
|`암호`|redhat|

* 각각의 앤서블 게이트웨이에 접속하여 테스트
* 설정 값이 공유되는 것을 확인

> [!NOTE]
> 앤서블 게이트웨이가 active-active로 구성되어 있으며, 앞 단에 로드밸런싱 장비 혹은 HAProxy로 고가용성을 구성할 수 있습니다.
<br>
<br>

------
[차례](../README.md)