# 레거시 시스템 연결

Ansible Automation Platform(이하 AAP)은 관리 대상 노드에 대한 연결성 및 모듈을 통한 관리가 가능합니다.
AAP는 이를 위해 SSH 연결과 파이썬 기반 모듈을 사용하는 데, 레거시 시스템의 경우에 보안 및 버전 호환성에 따라 기본 제공 기능으로 지원이 되지 않을 수 있습니다.

이 경우에 여러가지 방안으로 레거시 시스템을 관리할 수 있습니다.
<br>
<br>

## 1. SSH 연결

### 1.1 레거시 시스템으로 SSH 연결 시 에러

#### 1.1.1 SSH 연결 테스트

RHEL92에서 RHEL55로 SSH 연결
```bash
ssh rhel55.thinkmore.net
```

실행 결과
```
[root@rhel92 ~]# ssh rhel55.thinkmore.net
Unable to negotiate with 192.168.0.20 port 22: no matching key exchange method found. Their offer: diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1

[root@rhel92 ~]#
```
* 레거시 시스템에서 사용하는 SSH 키 교환 알고리즘과 SSH 클라이언트 시스템에서 사용하는 알고리즘이 달라 생기는 이슈
* SSH 클라이언트에서 보안상의 이유로 레거시 알고리즘이 지원되지 않을 수 있음

RHEL92에서 RHEL64로 SSH 연결
```bash
ssh rhel64.thinkmore.net
```

실행 결과
```
[root@rhel92 ~]# ssh rhel64.thinkmore.net
Unable to negotiate with 192.168.0.21 port 22: no matching host key type found. Their offer: ssh-rsa,ssh-dss

[root@rhel92 ~]#
```
* 레거시 시스템에서 제공하는 호스트의 기본 SSH 키가 레거시 키만 제공하는 경우에 생기는 이슈

#### 1.1.2 SSH 연결에서 CLI 옵션 사용

SSH 키 교환 알고리즘 지정
```bash
ssh -o KexAlgorithms=diffie-hellman-group-exchange-sha1 rhel55.thinkmore.net
```

실행 환경
```
[root@rhel92 ~]# ssh -o KexAlgorithms=diffie-hellman-group-exchange-sha1 rhel55
Unable to negotiate with 192.168.0.20 port 22: no matching host key type found. Their offer: ssh-rsa,ssh-dss

[root@rhel92 ~]#
```
* 레거시 시스템의 SSH 키 타입이 매치되지 않음

SSH 키 타입까지 지정
```bash
ssh -o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 -o PubkeyAcceptedKeyTypes=ssh-rsa rhel55.thinkmore.net
ssh -o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 -o PubkeyAcceptedKeyTypes=ssh-rsa rhel64.thinkmore.net
```

실행 환경
```
[root@rhel92 ~]# ssh -o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 -o PubkeyAcceptedKeyTypes=ssh-rsa rhel55.thinkmore.net
ssh_dispatch_run_fatal: Connection to 192.168.0.20 port 22: error in libcrypto

[root@rhel92 ~]# ssh -o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 -o PubkeyAcceptedKeyTypes=ssh-rsa rhel64.thinkmore.net
ssh_dispatch_run_fatal: Connection to 192.168.0.21 port 22: error in libcrypto

[root@rhel92 ~]#
```
* SSH 클라이언트의 알고리즘(예:SHA-256)이 RHEL55/RHEL64에서 지원되지 않는 경우, 타겟 노드의 ssh-rsa(RSA/SHA-1)로 시도
* RHEL9에서는 SHA-1가 ***DEFAULT*** 보안 정책에서 허용되지 않아 이슈 발생

#### 1.1.3 보안 정책 변경 후 SSH 연결 시도

```bash
update-crypto-policies --show
update-crypto-policies --set DEFAULT:SHA1
update-crypto-policies --show
update-crypto-policies --is-applied
ssh -o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 -o PubkeyAcceptedKeyTypes=ssh-rsa rhel64.thinkmore.net
ssh -o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 -o PubkeyAcceptedKeyTypes=ssh-rsa rhel55.thinkmore.net
```

실행 환경
```
[root@rhel92 ~]# update-crypto-policies --show
DEFAULT

[root@rhel92 ~]# update-crypto-policies --set DEFAULT:SHA1
Setting system policy to DEFAULT:SHA1
Note: System-wide crypto policies are applied on application start-up.
It is recommended to restart the system for the change of policies
to fully take place.

[root@rhel92 ~]# update-crypto-policies --show
DEFAULT:SHA1

[root@rhel92 ~]# update-crypto-policies --is-applied
The configured policy is applied

[root@rhel92 ~]# ssh -o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 -o PubkeyAcceptedKeyTypes=ssh-rsa rhel64.thinkmore.net
...<snip>...

[root@rhel64 ~]# exit
logout
Connection to rhel64.thinkmore.net closed.

[root@rhel92 ~]# ssh -o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 -o PubkeyAcceptedKeyTypes=ssh-rsa rhel55.thinkmore.net
...<snip>...

[root@rhel55 ~]# exit
logout
Connection to rhel55.thinkmore.net closed.

[root@rhel92 ~]#
```
* 보안 정책으로 ***LEGACY***를 사용할 수 있음
* 보안 정책 변경 후 전체 적용을 위해서는 시스템 재부팅이 필요
<br>

> [!NOTE]
> SSH 서버가 ECDSA 호스트 키를 지원하는 경우, 해당 키를 생성하여 설정 후 연결
<br>

### 1.2 SSH_CONFIG 설정으로 연결

#### 1.2.1 SSH 클라이언트 시스템 레벨 설정

```bash
update-crypto-policies --show
cat /etc/ssh/ssh_config.d/45-legacy.conf
ssh rhel64
exit
ssh rhel55
exit
```

실행 결과
```
[root@rhel92 ~]# update-crypto-policies --show
DEFAULT:SHA1

[root@rhel92 ~]# cat /etc/ssh/ssh_config.d/45-legacy.conf
Host rhel64.thinkmore.net,rhel55.thinkmore.net
        KexAlgorithms +diffie-hellman-group1-sha1
        HostKeyAlgorithms +ssh-rsa
        PubkeyAcceptedKeyTypes +ssh-rsa

[root@rhel92 ~]# ssh rhel64
...<snip>...

[root@rhel64 ~]# exit
logout
Connection to rhel64 closed.

[root@rhel92 ~]# ssh rhel55
...<snip>...

[root@rhel55 ~]# exit
logout
Connection to rhel55 closed.

[root@rhel92 ~]#
```

#### 1.2.2 SSH 클라이언트 사용자 레벨 설정

```bash
update-crypto-policies --show
cat ~/.ssh/config
ssh rhel64
exit
ssh rhel55
exit
```

실행 결과
```
[root@rhel92 ~]# update-crypto-policies --show
DEFAULT:SHA1

[root@rhel92 ~]# cat ~/.ssh/config
Host rhel64.thinkmore.net,rhel55.thinkmore.net
        KexAlgorithms +diffie-hellman-group1-sha1
        HostKeyAlgorithms +ssh-rsa
        PubkeyAcceptedKeyTypes +ssh-rsa

[root@rhel92 ~]# ssh rhel64
...<snip>...

[root@rhel64 ~]# exit
logout
Connection to rhel64 closed.

[root@rhel92 ~]# ssh rhel55
...<snip>...

[root@rhel55 ~]# exit
logout
Connection to rhel55 closed.

[root@rhel92 ~]#
```
<br>
<br>

## 2. 파이썬 버전

### 2.1 앤서블 설정

#### 2.1.1 앤서블 설치

```bash
dnf install -y ansible
```

실행 결과
```
[root@rhel92 ~]# dnf install -y ansible
...<snip>...

[root@rhel92 ~]#
```

#### 2.1.2 인벤토리 구성

```bash
cat /etc/ansible/hosts
```

실행 결과
```
[root@rhel92 ~]# cat /etc/ansible/hosts
[legacy]
rhel64
rhel55

[legacy:vars]
ansible_ssh_common_args = "-o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 -o PubkeyAcceptedKeyTypes=ssh-rsa"
ansible_ssh_pass = 'jdngajdw73'

[root@rhel92 ~]#
```
<br>

### 2.2 파이썬 버전 체크

#### 2.2.1 앤서블 호스트 버전 체크

```bash
ansible --version
```

실행 결과
```
[root@rhel92 ~]# ansible --version
ansible [core 2.14.14]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.9/site-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.9.16 (main, Dec  8 2022, 00:00:00) [GCC 11.3.1 20221121 (Red Hat 11.3.1-4)] (/usr/bin/python3)
  jinja version = 3.1.2
  libyaml = True

[root@rhel92 ~]#
```

#### 2.2.2 타겟 노드 파이썬 버전 체크

```bash
ssh rhel55 "python -V"
ssh rhel64 "python -V"
```

실행 결과
```
[root@rhel92 ~]# ssh rhel55 "python -V"
Python 2.4.3

[root@rhel92 ~]# ssh rhel64 "python -V"
Python 2.6.6

[root@rhel92 ~]#
```

#### 2.2.3 앤서블 관리대상 노드 버전 체크

```bash
ansible legacy -m ping
```

실행 결과
```
[root@rhel92 ~]# ansible legacy -m ping
[WARNING]: Unhandled error in Python interpreter discovery for host rhel55: Expecting value: line 1 column 1 (char 0)

rhel64 | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "msg": "ansible-core requires a minimum of Python2 version 2.7 or Python3 version 3.5. Current version: 2.6.6 (r266:84292, Oct 12 2012, 14:23:48) [GCC 4.4.6 20120305 (Red Hat 4.4.6-4)]"
}

[WARNING]: Platform linux on host rhel55 is using the discovered Python interpreter at /usr/bin/python, but future installation of another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-core/2.14/reference_appendices/interpreter_discovery.html for more information.

rhel55 | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "module_stderr": "Shared connection to rhel55 closed.\r\n",
    "module_stdout": "  File \"/root/.ansible/tmp/ansible-tmp-1729348277.9066813-34026-208152617816384/AnsiballZ_ping.py\", line 77\r\n    with open(args_path, 'rb') as f:\r\n            ^\r\nSyntaxError: invalid syntax\r\n",
    "msg": "MODULE FAILURE\nSee stdout/stderr for the exact error",
    "rc": 1
}

[root@rhel92 ~]#
```
<br>
<br>

## 3. 실행 환경 (Execution Environment) 준비

### 3.1 ***ansible-navigator*** 설정

#### 3.1.1 채널 활성화 및 설치

```bash
dnf install --enablerepo=ansible-automation-platform-2.5-for-rhel-9-x86_64-rpms ansible-navigator
ansible-navigator --version
```

실행 결과
```
[root@rhel92 ~]# dnf install --enablerepo=ansible-automation-platform-2.5-for-rhel-9-x86_64-rpms ansible-navigator
...<snip>...

[root@rhel92 ~]# ansible-navigator --version
ansible-navigator 24.9.0

[root@rhel92 ~]#
```

#### 3.1.2 레지스트리 로그인

```bash
podman login registry.redhat.io -u='USER_ID' -p='USER_PASSWD'
```

실행 결과
```
[root@rhel92 ~]# podman login registry.redhat.io -u='USER_ID' -p='USER_PASSWD'
Login Succeeded!

[root@rhel92 ~]#
```

#### 3.1.3 EE 이미지 확인

```bash
podman search registry.redhat.io/ansible-automation-platform-25/ee-supported
```

실행 결과
```
[root@rhel92 ~]# podman search registry.redhat.io/ansible-automation-platform-25/ee-supported
NAME                                                                  DESCRIPTION
registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel9  rhcc_registry.access.redhat.com_ansible-auto...
registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel8  rhcc_registry.access.redhat.com_ansible-auto...

[root@rhel92 ~]# 
```

#### 3.1.3 EE 이미지 다운로드

```bash
podman pull registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel8:latest
podman pull registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel9:latest
podman images
```

실행 결과
```
[root@rhel92 ~]# podman pull registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel8:latest
Trying to pull registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel8:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob 133388727492 done
Copying blob 23742cb03d23 done
Copying blob 6033a3819524 done
Copying blob ef94ff6b6d8b done
Copying config a0524338a6 done
Writing manifest to image destination
Storing signatures
a0524338a6857e513e23a56159dd240eb1c63b955046b285907989854e8aa7a0

[root@rhel92 ~]# podman pull registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel9:latest
Trying to pull registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel9:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob 4d5d1cbd7ece done
Copying blob 615a7d40bab5 done
Copying blob 8e9af33c4e3b done
Copying blob efb4fb097d72 done
Copying config f8c64f51b9 done
Writing manifest to image destination
Storing signatures
f8c64f51b975df43f66d9ed35c62f683a80136983cd419385c4c7f90c006844a

[root@rhel92 ~]# podman images
REPOSITORY                                                            TAG         IMAGE ID      CREATED      SIZE
registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel8  latest      a0524338a685  10 days ago  2.25 GB
registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel9  latest      f8c64f51b975  10 days ago  2.26 GB

[root@rhel92 ~]#
```
<br>

### 3.2 인벤토리 확인

#### 3.2.1 인벤토리 리스트 확인

인벤토리 파일
```bash
ansible-navigator inventory -m stdout -i /etc/ansible/hosts --list --pp=missing | jq -c . | jq '.'
```

실행 결과 JSON 형식
```json
{
  "_meta": {
    "hostvars": {
      "rhel55": {
        "ansible_ssh_common_args": "-o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 -o PubkeyAcceptedKeyTypes=ssh-rsa",
      },
      "rhel64": {
        "ansible_ssh_common_args": "-o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 -o PubkeyAcceptedKeyTypes=ssh-rsa",
      }
    }
  },
  "all": {
    "children": [
      "ungrouped",
      "common",
      "legacy"
    ]
  },
  "common": {
    "hosts": [
      "rhel92"
    ]
  },
  "legacy": {
    "hosts": [
      "rhel64",
      "rhel55"
    ]
  }
}
```
* ***--pp***: --pull-policy이며, EE 이미지에 대한 정책 설정 (always | missing | never | tag)
* ***-m***: --mode이며, 사용자 인터페이스 모드 지정 (stdout | interactive)

#### 3.2.2 RHEL92 호스트 대상 ping 테스트

ping-host.yaml 파일
```yaml
---
- name: Validate inventory hosts
  hosts: rhel92
  gather_facts: no

  tasks:

    - name: Ping any host
      ansible.builtin.ping:
...
```

플레이북 실행
```bash
ansible-navigator run -m stdout ping-host.yaml -i /etc/ansible/hosts --pp missing
```

실행 결과
```
[root@rhel92 ~]# ansible-navigator run -m stdout ping-host.yaml -i /etc/ansible/hosts --pp missing

PLAY [Validate inventory hosts] ************************************************

TASK [Ping any host] ***********************************************************
ok: [rhel92]

PLAY RECAP *********************************************************************
rhel92                     : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

[root@rhel92 ~]#
```

#### 3.2.3 레거시 호스트 대상 ping 테스트

ping-host.yaml 파일
```yaml
---
- name: Validate inventory hosts
  hosts: legacy
  gather_facts: no

  tasks:

    - name: Ping any host
      ansible.builtin.ping:
...
```

플레이북 실행
```bash
ansible-navigator run -m stdout ping-host.yaml -i /etc/ansible/hosts --pp missing
```

실행 결과
```
[root@rhel92 ~]# ansible-navigator run -m stdout ping-host.yaml -i /etc/ansible/hosts --pp missing

PLAY [Validate inventory hosts] ************************************************

TASK [Ping any host] ***********************************************************
[WARNING]: Unhandled error in Python interpreter discovery for host rhel55:
Expecting value: line 1 column 1 (char 0)
fatal: [rhel64]: FAILED! => {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "changed": false, "msg": "ansible-core requires a minimum of Python2 version 2.7 or Python3 version 3.6. Current version: 2.6.6 (r266:84292, Oct 12 2012, 14:23:48) [GCC 4.4.6 20120305 (Red Hat 4.4.6-4)]"}
[WARNING]: Platform linux on host rhel55 is using the discovered Python
interpreter at /usr/bin/python, but future installation of another Python
interpreter could change the meaning of that path. See
https://docs.ansible.com/ansible-
core/2.16/reference_appendices/interpreter_discovery.html for more information.
fatal: [rhel55]: FAILED! => {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "changed": false, "module_stderr": "Shared connection to rhel55 closed.\r\n", "module_stdout": "  File \"/root/.ansible/tmp/ansible-tmp-1729414235.1023953-25-33211598709319/AnsiballZ_ping.py\", line 77\r\n    with open(args_path, 'rb') as f:\r\n            ^\r\nSyntaxError: invalid syntax\r\n", "msg": "MODULE FAILURE\nSee stdout/stderr for the exact error", "rc": 1}

PLAY RECAP *********************************************************************
rhel55                     : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
rhel64                     : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
Please review the log for errors.

[root@rhel92 ~]#
```
* EE 이미지 내의 파이썬 버전과 타겟 노드의 파이썬 버전이 맞지 않아 이슈 발생
<br>

### 3.3 ansible-navigator를 위한 구성 파일

#### 3.3.1 구성 파일 설정

**ansible-navigator의 설정 관리**
* ansible-navigator에 대한 구성 파일(또는 설정 파일)을 생성하여 해당 구성 설정의 기본값을 재정의
* 설정 파일은 JSON(.json) 또는 YAML(.yml 또는 .yaml) 형식

**설정 파일 우선 순위**
* ANSIBLE_NAVIGATOR_CONFIG 환경 변수가 설정된 경우 환경 변수가 지정하는 위치에서 구성 파일을 사용
* 현재 Ansible 프로젝트 디렉터리의 ansible-navigator.yml 파일
* ~/.ansible-navigator.yml 파일(홈 디렉터리에 있음). 파일 이름이 '.'으로 시작

#### 3.3.2 구성 파일 예

필요 패키지 설치
```bash
dnf install python3-pip
pip3 install yq
```

실행 결과
```
[root@rhel92 ~]# dnf install python3-pip
...<snip>...

[root@rhel92 ~]# pip3 install yq
...<snip>...

[root@rhel92 ~]# yq '.' ~/.ansible-navigator.yml
...<snip>...

[root@rhel92 ~]#
```

~/.ansible-navigator.yml 파일
```yaml
---
ansible-navigator:
  execution-environment:
    image: registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel9:latest
    pull:
      policy: missing
```

#### 3.3.3 ping 테스트

~/ansible.cfg 파일
```ini
[defaults]
inventory = ./inventory
```

~/inventory 파일
```ini
[common]
rhel92

[common:vars]
ansible_ssh_pass = 'SSH_USER_PW'

[legacy]
rhel64
rhel55

[legacy:vars]
ansible_ssh_common_args = "-o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 -o PubkeyAcceptedKeyTypes=ssh-rsa"
ansible_ssh_pass = 'SSH_USER_PW'
```

~/ping-host.yaml
```yaml
---
- name: Validate inventory hosts
  hosts: "{{ HOST_NAME }}"
  gather_facts: no

  tasks:

    - name: Ping any host
      ansible.builtin.ping:
```

ping 테스트
```bash
ansible-navigator run -m stdout ping-host.yaml -e HOST_NAME=rhel92
ansible-navigator run -m stdout ping-host.yaml -e HOST_NAME=legacy
```

실행 결과
```
[root@rhel92 ~]# ansible-navigator run -m stdout ping-host.yaml -e HOST_NAME=rhel92

PLAY [Validate inventory hosts] ************************************************

TASK [Ping any host] ***********************************************************
ok: [rhel92]

PLAY RECAP *********************************************************************
rhel92                     : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

[root@rhel92 ~]# ansible-navigator run -m stdout ping-host.yaml -e HOST_NAME=legacy

PLAY [Validate inventory hosts] ************************************************

TASK [Ping any host] ***********************************************************
fatal: [rhel64]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: ssh_dispatch_run_fatal: Connection to 192.168.0.21 port 22: error in libcrypto", "unreachable": true}
fatal: [rhel55]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: ssh_dispatch_run_fatal: Connection to 192.168.0.20 port 22: error in libcrypto", "unreachable": true}

PLAY RECAP *********************************************************************
rhel55                     : ok=0    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0
rhel64                     : ok=0    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0
Please review the log for errors.

[root@rhel92 ~]#
```
* *ansible-navigator*의 구성 파일에 설정된 EE가 RHEL9 기반이며, SHA1을 지원하지 않아 생긴 이슈

#### 3.3.4 EE 이미지를 지정하여 ping 테스트

```bash
ansible-navigator run -m stdout ping-host.yaml -e HOST_NAME=legacy --eei registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel8:latest
ansible-navigator run -m stdout ping-host.yaml -e HOST_NAME=legacy --eei registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel9:latest
```

실행 결과
```
[root@rhel92 ~]# ansible-navigator run -m stdout ping-host.yaml -e HOST_NAME=legacy --eei registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel8:latest

PLAY [Validate inventory hosts] ************************************************

TASK [Ping any host] ***********************************************************
[WARNING]: Unhandled error in Python interpreter discovery for host rhel55:
Expecting value: line 1 column 1 (char 0)
fatal: [rhel64]: FAILED! => {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "changed": false, "msg": "ansible-core requires a minimum of Python2 version 2.7 or Python3 version 3.6. Current version: 2.6.6 (r266:84292, Oct 12 2012, 14:23:48) [GCC 4.4.6 20120305 (Red Hat 4.4.6-4)]"}
[WARNING]: Platform linux on host rhel55 is using the discovered Python
interpreter at /usr/bin/python, but future installation of another Python
interpreter could change the meaning of that path. See
https://docs.ansible.com/ansible-
core/2.16/reference_appendices/interpreter_discovery.html for more information.
fatal: [rhel55]: FAILED! => {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "changed": false, "module_stderr": "Shared connection to rhel55 closed.\r\n", "module_stdout": "  File \"/root/.ansible/tmp/ansible-tmp-1729418712.9864047-25-252305002253057/AnsiballZ_ping.py\", line 77\r\n    with open(args_path, 'rb') as f:\r\n            ^\r\nSyntaxError: invalid syntax\r\n", "msg": "MODULE FAILURE\nSee stdout/stderr for the exact error", "rc": 1}

PLAY RECAP *********************************************************************
rhel55                     : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
rhel64                     : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
Please review the log for errors.

[root@rhel92 ~]# ansible-navigator run -m stdout ping-host.yaml -e HOST_NAME=legacy --eei registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel9:latest

PLAY [Validate inventory hosts] ************************************************

TASK [Ping any host] ***********************************************************
fatal: [rhel64]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: ssh_dispatch_run_fatal: Connection to 192.168.0.21 port 22: error in libcrypto", "unreachable": true}
fatal: [rhel55]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: ssh_dispatch_run_fatal: Connection to 192.168.0.20 port 22: error in libcrypto", "unreachable": true}

PLAY RECAP *********************************************************************
rhel55                     : ok=0    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0
rhel64                     : ok=0    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0
Please review the log for errors.

[root@rhel92 ~]#
```
* EE 이미지가 RHEL8일 때는 SHA1은 지원하나 파이썬 버전 차이 지원 이슈
* EE 이미지를 CLI에서 RHEL9로 지정하면, SHA1 지원 이슈 발생은 같음
<br>

### 3.4 레드햇 지원 EE로 테스트

#### 3.4.1 레드햇 지원 EE 이미지 리스트

```bash
podman search --format "{{.Name}}" registry.redhat.io/*/ee-supported | sort -u
```

실행 결과
```
[root@rhel92 ~]# podman search --format "{{.Name}}" registry.redhat.io/*/ee-supported | sort -u
registry.redhat.io/ansible-automation-platform-20-early-access/ee-supported-rhel8
registry.redhat.io/ansible-automation-platform-21/ee-supported-rhel8
registry.redhat.io/ansible-automation-platform-22/ee-supported-rhel8
registry.redhat.io/ansible-automation-platform-23/ee-supported-rhel8
registry.redhat.io/ansible-automation-platform-24/ee-supported-rhel8
registry.redhat.io/ansible-automation-platform-24/ee-supported-rhel9
registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel8
registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel9

[root@rhel92 ~]#
```

#### 3.4.2 이미지 태그 확인

```bash
podman search --format json --list-tags registry.redhat.io/ansible-automation-platform-21/ee-supported-rhel8 | jq -r '.[].Tags[]'
```

실행 결과
```
[root@rhel92 ~]# podman search --format json --list-tags registry.redhat.io/ansible-automation-platform-21/ee-supported-rhel8 | jq -r '.[].Tags[]'
1.0
1.0.0
1.0.0-17
1.0.0-21
1.0.0-30
1.0.0-37
1.0.0-37-source
1.0.0-6
1.0.1
1.0.1-102
1.0.1-102-source
1.0.1-105
1.0.1-105-source
1.0.1-109
1.0.1-109-source
1.0.1-11
1.0.1-114
1.0.1-114-source
1.0.1-119
1.0.1-119-source
1.0.1-11-source
1.0.1-124
1.0.1-124-source
1.0.1-15
1.0.1-15.1645819575

[root@rhel92 ~]#
```

#### 3.4.3 초기 EE 이미지 다운로드

```bash
podman pull registry.redhat.io/ansible-automation-platform-21/ee-supported-rhel8:1.0
podman images
```

실행 결과
```
[root@rhel92 ~]# podman pull registry.redhat.io/ansible-automation-platform-21/ee-supported-rhel8:1.0
Trying to pull registry.redhat.io/ansible-automation-platform-21/ee-supported-rhel8:1.0...
Getting image source signatures
Checking if image destination supports signatures
Copying blob 2c217bd1bbcd done
Copying blob 28ff5ee6facb done
Copying blob 2ad3b9180fba done
Copying blob 6e88b19c938a done
Copying config f8c31ee72b done
Writing manifest to image destination
Storing signatures
f8c31ee72b394b80438f8949e63de590af7f7c192982bf8287dd5a4d6459b17b

[root@rhel92 ~]# podman images
REPOSITORY                                                            TAG         IMAGE ID      CREATED        SIZE
registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel8  latest      a0524338a685  10 days ago    2.25 GB
registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel9  latest      f8c64f51b975  10 days ago    2.26 GB
registry.redhat.io/ansible-automation-platform-21/ee-supported-rhel8  1.0         f8c31ee72b39  18 months ago  1.44 GB

[root@rhel92 ~]#
```

#### 3.4.4 다운로드한 이미지로 legacy 시스템 ping 테스트

```bash
ansible-navigator run -m stdout ping-host.yaml -e HOST_NAME=legacy --eei registry.redhat.io/ansible-automation-platform-21/ee-supported-rhel8:1.0
```

실행 결과
```
[root@rhel92 ~]# ansible-navigator run -m stdout ping-host.yaml -e HOST_NAME=legacy --eei registry.redhat.io/ansible-automation-platform-21/ee-supported-rhel8:1.0

PLAY [Validate inventory hosts] ************************************************

TASK [Ping any host] ***********************************************************
[WARNING]: Unhandled error in Python interpreter discovery for host rhel55:
Expecting value: line 1 column 1 (char 0)
[DEPRECATION WARNING]: ansible-core 2.13 will require Python 2.7 or newer on
the target. Current version: 2.6.6 (r266:84292, Oct 12 2012, 14:23:48) [GCC
4.4.6 20120305 (Red Hat 4.4.6-4)]. This feature will be removed in version
2.13. Deprecation warnings can be disabled by setting
deprecation_warnings=False in ansible.cfg.
ok: [rhel64]
[WARNING]: Platform linux on host rhel55 is using the discovered Python
interpreter at /usr/bin/python, but future installation of another Python
interpreter could change the meaning of that path. See
https://docs.ansible.com/ansible-
core/2.12/reference_appendices/interpreter_discovery.html for more information.
fatal: [rhel55]: FAILED! => {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "changed": false, "module_stderr": "Shared connection to rhel55 closed.\r\n", "module_stdout": "  File \"/root/.ansible/tmp/ansible-tmp-1729420559.6597173-29-242278753588330/AnsiballZ_ping.py\", line 77\r\n    with open(args_path, 'rb') as f:\r\n            ^\r\nSyntaxError: invalid syntax\r\n", "msg": "MODULE FAILURE\nSee stdout/stderr for the exact error", "rc": 1}

PLAY RECAP *********************************************************************
rhel55                     : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
rhel64                     : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
Please review the log for errors.

[root@rhel92 ~]#
```
* 레거시 시스템인 RHEL55의 경우에는 파이썬 버전이 2.6.6으로 ansible-core 2.13에서는 지원이 안됨
<br>

### 3.5 파이썬 설치 없이 실행

#### 3.5.1 앤서블 CLI로 ansible.builtin.raw 모듈 테스트

```bash
ansible all -m ansible.builtin.raw -a 'hostname'
```

실행 결과
```
[root@rhel92 ~]# ansible all -m ansible.builtin.raw -a 'hostname'
rhel55 | CHANGED | rc=0 >>
rhel55.thinkmore.net
Shared connection to rhel55 closed.

rhel64 | CHANGED | rc=0 >>
rhel64.thinkmore.net
Shared connection to rhel64 closed.

rhel92 | CHANGED | rc=0 >>
rhel92.thinkmore.net
Shared connection to rhel92 closed.

[root@rhel92 ~]#
```

#### 3.5.2 앤서블 플레이북에서 raw 모듈 테스트

~/run-raw-on-host.yaml
```yaml
---
- name: Validate inventory hosts
  hosts: "{{ HOST_NAME }}"
  gather_facts: no

  tasks:

    - name: Run raw module on any host
      ansible.builtin.raw: "{{ RUN_CMD }}"
...
```

플레이북을 통해 명령어 실행
```bash
ansible-playbook run-raw-on-host.yaml -e HOST_NAME=all -e RUN_CMD=hostname
```

실행 결과
```
[root@rhel92 ~]# ansible-playbook run-raw-on-host.yaml -e HOST_NAME=all -e RUN_CMD=hostname

PLAY [Validate inventory hosts] *************************************************************************************************************************************************************

TASK [Run raw module on any host] ***********************************************************************************************************************************************************
changed: [rhel55]
changed: [rhel64]
changed: [rhel92]

PLAY RECAP **********************************************************************************************************************************************************************************
rhel55                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rhel64                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rhel92                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[root@rhel92 ~]#
```
<br>

### 3.6 ansible.builtin.raw 모듈 사용

앤서블 raw 모듈에 대한 정보는 다음에서 확인할 수 있습니다.
* [ansible.builtin.raw](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/raw_module.html) 모듈
<br>

> [!IMPORTANT]
> ansible.builtin.raw 모듈은 타겟 시스템에 파이썬이 없어, 파이썬을 설치하거나 혹은 파이썬을 사용하지 못하는 경우에만 사용하는 것이 좋습니다. 앤서블 *raw* 모듈은 단순 형태로 관리 대상 노드에서 주어진 인수 기반의 명령어를 실행할 뿐으로, 실행 결과 확인 및 적합성 등등은 제공되지 않습니다.
<br>

#### 3.6.1 CLI ad-hoc 사용 시

```bash
ansible all -m raw -a 'ping -c 1 192.168.0.1'
```

실행 결과
```
[root@rhel92 ~]# ansible all -m raw -a 'ping -c 1 192.168.0.1'
rhel55 | CHANGED | rc=0 >>
PING 192.168.0.1 (192.168.0.1) 56(84) bytes of data.
64 bytes from 192.168.0.1: icmp_seq=1 ttl=64 time=2.45 ms

--- 192.168.0.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 2.456/2.456/2.456/0.000 ms
Shared connection to rhel55 closed.

rhel64 | CHANGED | rc=0 >>
PING 192.168.0.1 (192.168.0.1) 56(84) bytes of data.
64 bytes from 192.168.0.1: icmp_seq=1 ttl=64 time=0.627 ms

--- 192.168.0.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.627/0.627/0.627/0.000 ms
Shared connection to rhel64 closed.

rhel92 | CHANGED | rc=0 >>
PING 192.168.0.1 (192.168.0.1) 56(84) bytes of data.
64 bytes from 192.168.0.1: icmp_seq=1 ttl=64 time=0.533 ms

--- 192.168.0.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.533/0.533/0.533/0.000 ms
Shared connection to rhel92 closed.

[root@rhel92 ~]#
```
* 여러 인수를 전달해도 문제 없음

#### 3.6.2 플레이북 상에 raw 모듈 인수로 1개 전달

~/run-command-via-raw.yaml 파일
```yaml
---
- name: Validate inventory hosts
  hosts: "{{ HOST_NAME }}"
  gather_facts: no

  tasks:

    - name: Run raw module on any host
      ansible.builtin.raw: ping -c 1 {{ TGT_HOST }}
...
```

```bash
ansible-playbook run-command-via-raw.yaml -e HOST_NAME=all -e TGT_HOST=192.168.0.1
ansible-playbook run-command-via-raw.yaml -e HOST_NAME=all -e TGT_HOST=192.168.0.1 -vvv
```

실행 결과
```
[root@rhel92 ~]# ansible-playbook run-command-via-raw.yaml -e HOST_NAME=all -e TGT_HOST=192.168.0.1

PLAY [Validate inventory hosts] *************************************************************************************************************************************************************

TASK [Run raw module on any host] ***********************************************************************************************************************************************************
changed: [rhel55]
changed: [rhel64]
changed: [rhel92]

PLAY RECAP **********************************************************************************************************************************************************************************
rhel55                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rhel64                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rhel92                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[root@rhel92 ~]# ansible-playbook run-command-via-raw.yaml -e HOST_NAME=all -e TGT_HOST=192.168.0.1 -vvv

...<snip>...

TASK [Run raw module on any host] ***********************************************************************************************************************************************************
task path: /root/run-command-via-raw.yaml:8
<rhel92> ESTABLISH SSH CONNECTION FOR USER: None
<rhel92> SSH: EXEC sshpass -d12 ssh -C -o ControlMaster=auto -o ControlPersist=60s -o ConnectTimeout=10 -o 'ControlPath="/root/.ansible/cp/566b8abf85"' -tt rhel92 'ping -c 1 192.168.0.1'
<rhel64> ESTABLISH SSH CONNECTION FOR USER: None
<rhel64> SSH: EXEC sshpass -d16 ssh -C -o ControlMaster=auto -o ControlPersist=60s -o ConnectTimeout=10 -o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 -o PubkeyAcceptedKeyTypes=ssh-rsa -o 'ControlPath="/root/.ansible/cp/5b9323ce8f"' -tt rhel64 'ping -c 1 192.168.0.1'
<rhel55> ESTABLISH SSH CONNECTION FOR USER: None
<rhel55> SSH: EXEC sshpass -d20 ssh -C -o ControlMaster=auto -o ControlPersist=60s -o ConnectTimeout=10 -o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 -o PubkeyAcceptedKeyTypes=ssh-rsa -o 'ControlPath="/root/.ansible/cp/1c353a7831"' -tt rhel55 'ping -c 1 192.168.0.1'

...<snip>...

PLAY RECAP **********************************************************************************************************************************************************************************
rhel55                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rhel64                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rhel92                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[root@rhel92 ~]#
```

#### 3.6.3 플레이북 상에 raw 모듈로 여러 인수 전달 세 에러

```bash
ansible-playbook run-raw-on-host.yaml -e HOST_NAME=all -e RUN_CMD='ping -c1 192.168.0.1'
ansible-playbook run-raw-on-host.yaml -e HOST_NAME=all -e RUN_CMD='ping -c1 192.168.0.1' -vvv
```

실행 결과
```
[root@rhel92 ~]# ansible-playbook run-raw-on-host.yaml -e HOST_NAME=all -e RUN_CMD='ping -c1 192.168.0.1'

PLAY [Validate inventory hosts] *************************************************************************************************************************************************************

TASK [Run raw module on any host] ***********************************************************************************************************************************************************
fatal: [rhel64]: FAILED! => {"changed": true, "msg": "non-zero return code", "rc": 2, "stderr": "Shared connection to rhel64 closed.\r\n", "stderr_lines": ["Shared connection to rhel64 closed."], "stdout": "Usage: ping [-LRUbdfnqrvVaA] [-c count] [-i interval] [-w deadline]\r\n            [-p pattern] [-s packetsize] [-t ttl] [-I interface or address]\r\n            [-M mtu discovery hint] [-S sndbuf]\r\n            [ -T timestamp option ] [ -Q tos ] [hop1 ...] destination\r\n", "stdout_lines": ["Usage: ping [-LRUbdfnqrvVaA] [-c count] [-i interval] [-w deadline]", "            [-p pattern] [-s packetsize] [-t ttl] [-I interface or address]", "            [-M mtu discovery hint] [-S sndbuf]", "            [ -T timestamp option ] [ -Q tos ] [hop1 ...] destination"]}
fatal: [rhel55]: FAILED! => {"changed": true, "msg": "non-zero return code", "rc": 2, "stderr": "Shared connection to rhel55 closed.\r\n", "stderr_lines": ["Shared connection to rhel55 closed."], "stdout": "Usage: ping [-LRUbdfnqrvVaA] [-c count] [-i interval] [-w deadline]\r\n            [-p pattern] [-s packetsize] [-t ttl] [-I interface or address]\r\n            [-M mtu discovery hint] [-S sndbuf]\r\n            [ -T timestamp option ] [ -Q tos ] [hop1 ...] destination\r\n", "stdout_lines": ["Usage: ping [-LRUbdfnqrvVaA] [-c count] [-i interval] [-w deadline]", "            [-p pattern] [-s packetsize] [-t ttl] [-I interface or address]", "            [-M mtu discovery hint] [-S sndbuf]", "            [ -T timestamp option ] [ -Q tos ] [hop1 ...] destination"]}
fatal: [rhel92]: FAILED! => {"changed": true, "msg": "non-zero return code", "rc": 1, "stderr": "Shared connection to rhel92 closed.\r\n", "stderr_lines": ["Shared connection to rhel92 closed."], "stdout": "ping: usage error: Destination address required\r\n", "stdout_lines": ["ping: usage error: Destination address required"]}

PLAY RECAP **********************************************************************************************************************************************************************************
rhel55                     : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
rhel64                     : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
rhel92                     : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   

[root@rhel92 ~]# ansible-playbook run-raw-on-host.yaml -e HOST_NAME=all -e RUN_CMD='ping -c1 192.168.0.1' -vvv

...<snip>...

TASK [Run raw module on any host] ***********************************************************************************************************************************************************
task path: /root/run-raw-on-host.yaml:8
<rhel92> ESTABLISH SSH CONNECTION FOR USER: None
<rhel92> SSH: EXEC sshpass -d12 ssh -C -o ControlMaster=auto -o ControlPersist=60s -o ConnectTimeout=10 -o 'ControlPath="/root/.ansible/cp/566b8abf85"' -tt rhel92 ping
<rhel64> ESTABLISH SSH CONNECTION FOR USER: None
<rhel64> SSH: EXEC sshpass -d16 ssh -C -o ControlMaster=auto -o ControlPersist=60s -o ConnectTimeout=10 -o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 -o PubkeyAcceptedKeyTypes=ssh-rsa -o 'ControlPath="/root/.ansible/cp/5b9323ce8f"' -tt rhel64 ping
<rhel55> ESTABLISH SSH CONNECTION FOR USER: None
<rhel55> SSH: EXEC sshpass -d20 ssh -C -o ControlMaster=auto -o ControlPersist=60s -o ConnectTimeout=10 -o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 -o PubkeyAcceptedKeyTypes=ssh-rsa -o 'ControlPath="/root/.ansible/cp/1c353a7831"' -tt rhel55 ping

...<snip>...

PLAY RECAP **********************************************************************************************************************************************************************************
rhel55                     : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
rhel64                     : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
rhel92                     : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   

[root@rhel92 ~]#
```
* 실제 전달은 "ping -c1 192.168.0.1"이나 raw 모듈에 전달된 인수는 "ping"이어서 이슈 발생

#### 3.6.3 플레이북 상에 raw 모듈로 여러 인수 전달 방법

```bash
ansible-playbook run-raw-on-host.yaml -e HOST_NAME=all -e RUN_CMD='"ping -c1 192.168.0.1"'
ansible-playbook run-raw-on-host.yaml -e HOST_NAME=all -e RUN_CMD='"ping -c1 192.168.0.1"' -vvv
```
* 전달 인수를 1개로 인식시키지 위해 한 번 더 묶음

실행 결과
```
[root@rhel92 ~]# ansible-playbook run-raw-on-host.yaml -e HOST_NAME=all -e RUN_CMD='"ping -c1 192.168.0.1"'

PLAY [Validate inventory hosts] *************************************************************************************************************************************************************

TASK [Run raw module on any host] ***********************************************************************************************************************************************************
changed: [rhel55]
changed: [rhel64]
changed: [rhel92]

PLAY RECAP **********************************************************************************************************************************************************************************
rhel55                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rhel64                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rhel92                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[root@rhel92 ~]# ansible-playbook run-raw-on-host.yaml -e HOST_NAME=all -e RUN_CMD='"ping -c1 192.168.0.1"' -vvv

...<snip>...

TASK [Run raw module on any host] ***********************************************************************************************************************************************************
task path: /root/run-raw-on-host.yaml:8
<rhel92> ESTABLISH SSH CONNECTION FOR USER: None
<rhel92> SSH: EXEC sshpass -d12 ssh -C -o ControlMaster=auto -o ControlPersist=60s -o ConnectTimeout=10 -o 'ControlPath="/root/.ansible/cp/566b8abf85"' -tt rhel92 'ping -c1 192.168.0.1'
<rhel64> ESTABLISH SSH CONNECTION FOR USER: None
<rhel64> SSH: EXEC sshpass -d16 ssh -C -o ControlMaster=auto -o ControlPersist=60s -o ConnectTimeout=10 -o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 -o PubkeyAcceptedKeyTypes=ssh-rsa -o 'ControlPath="/root/.ansible/cp/5b9323ce8f"' -tt rhel64 'ping -c1 192.168.0.1'
<rhel55> ESTABLISH SSH CONNECTION FOR USER: None
<rhel55> SSH: EXEC sshpass -d20 ssh -C -o ControlMaster=auto -o ControlPersist=60s -o ConnectTimeout=10 -o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 -o PubkeyAcceptedKeyTypes=ssh-rsa -o 'ControlPath="/root/.ansible/cp/1c353a7831"' -tt rhel55 'ping -c1 192.168.0.1'

...<snip>...

PLAY RECAP **********************************************************************************************************************************************************************************
rhel55                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rhel64                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rhel92                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[root@rhel92 ~]#
```

#### 3.6.4 ansible-navigator로 raw 모듈로 테스트

```bash
ansible-navigator run -m stdout run-raw-on-host.yaml -e HOST_NAME=all -e RUN_CMD='"ping -c1 192.168.0.1"' --eei registry.redhat.io/ansible-automation-platform-21/ee-supported-rhel8:1.0
ansible-navigator run -m stdout run-raw-on-host.yaml -e HOST_NAME=all -e RUN_CMD='"ping -c1 192.168.0.1"' --eei registry.redhat.io/ansible-automation-platform-21/ee-supported-rhel8:1.0 -vvv
```

실행 결과
```
[root@rhel92 ~]# ansible-navigator run -m stdout run-raw-on-host.yaml -e HOST_NAME=all -e RUN_CMD='"ping -c1 192.168.0.1"' --eei registry.redhat.io/ansible-automation-platform-21/ee-supported-rhel8:1.0

PLAY [Validate inventory hosts] ************************************************

TASK [Run raw module on any host] **********************************************
changed: [rhel55]
changed: [rhel64]
changed: [rhel92]

PLAY RECAP *********************************************************************
rhel55                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rhel64                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rhel92                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[root@rhel92 ~]# ansible-navigator run -m stdout run-raw-on-host.yaml -e HOST_NAME=all -e RUN_CMD='"ping -c1 192.168.0.1"' --eei registry.redhat.io/ansible-automation-platform-21/ee-supported-rhel8:1.0 -vvv

...<snip>...

TASK [Run raw module on any host] **********************************************
task path: /root/run-raw-on-host.yaml:8
<rhel92> ESTABLISH SSH CONNECTION FOR USER: None
<rhel92> SSH: EXEC sshpass -d10 ssh -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o ConnectTimeout=10 -o 'ControlPath="/home/runner/.ansible/cp/566b8abf85"' -tt rhel92 'ping -c1 192.168.0.1'
<rhel64> ESTABLISH SSH CONNECTION FOR USER: None
<rhel64> SSH: EXEC sshpass -d12 ssh -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o ConnectTimeout=10 -o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 -o PubkeyAcceptedKeyTypes=ssh-rsa -o 'ControlPath="/home/runner/.ansible/cp/5b9323ce8f"' -tt rhel64 'ping -c1 192.168.0.1'
<rhel55> ESTABLISH SSH CONNECTION FOR USER: None
<rhel55> SSH: EXEC sshpass -d14 ssh -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o ConnectTimeout=10 -o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 -o PubkeyAcceptedKeyTypes=ssh-rsa -o 'ControlPath="/home/runner/.ansible/cp/1c353a7831"' -tt rhel55 'ping -c1 192.168.0.1'

...<snip>...

PLAY RECAP *********************************************************************
rhel55                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rhel64                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rhel92                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[root@rhel92 ~]#
```
<br>
<br>

#############################################
\# 이하는 작업 중
#############################################

### 3.X (Under-Construction)

#### 3.X.1 RHEL6 이미지 확인

```bash
podman search --format "{{.Name}}" registry.redhat.io/rhel6 | sort -u
```

실행 결과
```
[root@rhel92 ~]# podman search --format "{{.Name}}" registry.redhat.io/rhel6 | sort -u
registry.redhat.io/rhel6
registry.redhat.io/rhel6.10
registry.redhat.io/rhel6.5
registry.redhat.io/rhel6.6
registry.redhat.io/rhel6.7
registry.redhat.io/rhel6.8
registry.redhat.io/rhel6.9
registry.redhat.io/rhel6-init
registry.redhat.io/rhel6/rhel

[root@rhel92 ~]#
```

#### 3.X.2 이미지 태그 확인

```bash
podman search --format json --list-tags registry.redhat.io/rhel6 | jq -r '.[].Tags[]'
```

실행 환경
```
[root@rhel92 ~]# podman search --format json --list-tags registry.redhat.io/rhel6 | jq -r '.[].Tags[]'
6.10
6.10-500
6.10-503
6.10-505
6.10-507
6.10-508
6.10-511
6.10-512
6.10-513
6.10-514
6.10-521
6.10-522
6.10-530
6.10-532
6.10-533
6.10-534
6.10-541
6.10-543
6.10-545
6.10-548
6.10-548-source
6.5-11
6.5-12
6.5-15
6.6-6

[root@rhel92 ~]#
```

#### 3.X.3 RHEL6 이미지 다운로드

```bash
podman pull registry.redhat.io/rhel6/rhel:6.10-548
```

실행 결과
```
[root@rhel92 ~]# podman pull registry.redhat.io/rhel6/rhel:6.10-548
Trying to pull registry.redhat.io/rhel6/rhel:6.10-548...
Getting image source signatures
Checking if image destination supports signatures
Copying blob 9c4cf6c2176d done
Copying blob 4ca77ff8a54a done
Copying config 0d05d5d1f2 done
Writing manifest to image destination
Storing signatures
0d05d5d1f28d374f012a30e55931aaf878ae461da9925cb5d71cd82ae09f707f

[root@rhel92 ~]#
```
<br>
<br>


```bash

```

실행 결과
```

```
<br>
<br>

------
[차례](../README.md)