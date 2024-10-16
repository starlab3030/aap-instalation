# AAP 설치 제거

## 1. 컨테이너화 된 AAP 제거

### 1.1 uninstall.yml 플레이북 확인

컨테이너화 된 AAP 설치/구성/제거 관련 파일 확인
```bash
ls -lh temp/ansible-setup/collections/ansible_collections/ansible/containerized_installer/playbooks/
cat ~/temp/ansible-setup/collections/ansible_collections/ansible/containerized_installer/playbooks/uninstall.yml
```

실행 결과
```
[shadowman@aap-c ~]$ ls -lh temp/ansible-setup/collections/ansible_collections/ansible/containerized_installer/playbooks/
total 32K
-rw-r--r--. 1 shadowman shadowman 4.0K Oct  8 03:56 backup.yml
-rw-r--r--. 1 shadowman shadowman  234 Oct  8 03:56 bundle.yml
-rw-r--r--. 1 shadowman shadowman 6.8K Oct  8 03:56 install.yml
-rw-r--r--. 1 shadowman shadowman 5.0K Oct  8 03:56 restore.yml
-rw-r--r--. 1 shadowman shadowman 4.2K Oct  8 03:56 uninstall.yml

[shadowman@aap-c ~]$ cat ~/temp/ansible-setup/collections/ansible_collections/ansible/containerized_installer/playbooks/uninstall.yml
...<snip>...

[shadowman@aap-c ~]$ 
```

uninstall.yml 파일
```yaml
---
- name: Collect services facts
  hosts: automationcontroller:automationeda:automationgateway:automationhub:database:execution_nodes:redis
  gather_facts: false
  become: false
  tasks:
    - name: Gather regular user uid and home dir
      ansible.builtin.setup:
        filter:
          - 'ansible_user_uid'
          - 'ansible_user_dir'

    - name: Get service facts
      ansible.builtin.service_facts:
      become: true

- name: Uninstall the Automation Controller
  hosts: automationcontroller
  gather_facts: false
  become: false
  tasks:
    - name: Uninstall the automation controller
      ansible.builtin.import_role:
        name: automationcontroller
        tasks_from: uninstall.yml

- name: Uninstall the Automation Hub
  hosts: automationhub
  gather_facts: false
  become: false
  tasks:
    - name: Uninstall the automation hub
      ansible.builtin.import_role:
        name: automationhub
        tasks_from: uninstall.yml

- name: Uninstall the Automation EDA
  hosts: automationeda
  gather_facts: false
  tasks:
    - name: Uninstall the automation eda
      ansible.builtin.import_role:
        name: automationeda
        tasks_from: uninstall.yml

- name: Uninstall receptor
  hosts: automationcontroller:execution_nodes
  gather_facts: false
  become: false
  tasks:
    - name: Uninstall receptor
      ansible.builtin.import_role:
        name: receptor
        tasks_from: uninstall.yml

- name: Uninstall the Automation Gateway
  hosts: automationgateway
  any_errors_fatal: true
  gather_facts: false
  become: false
  tasks:
    - name: Uninstall Automation Gateway
      ansible.builtin.import_role:
        name: automationgateway
        tasks_from: uninstall.yml

- name: Uninstall redis cluster
  hosts: redis
  gather_facts: false
  become: false
  tasks:
    - name: Uninstall redis cluster tcp socket
      ansible.builtin.include_role:
        name: redis
        tasks_from: uninstall.yml
      vars:
        redis_unix_socket: false
        redis_cluster: true
      when: redis_mode | default('cluster') == 'cluster'

- name: Uninstall redis
  hosts: automationcontroller:automationeda:automationgateway:automationhub
  gather_facts: false
  become: false
  tasks:
    - name: Uninstall redis unix socket
      ansible.builtin.include_role:
        name: redis
        tasks_from: uninstall.yml
      when: >
        inventory_hostname in groups.get('automationcontroller', []) or
        inventory_hostname in groups.get('automationhub', []) or
        (inventory_hostname in groups.get('automationeda', []) and groups.get('automationeda', []) | length == 1)

    - name: Uninstall redis tcp socket
      ansible.builtin.include_role:
        name: redis
        tasks_from: uninstall.yml
      vars:
        redis_unix_socket: false
      when:
        - redis_mode | default('cluster') == 'standalone'
        - inventory_hostname == groups['automationgateway'] | first

- name: Uninstall pcp
  hosts: automationcontroller:automationhub:automationeda:automationgateway:database:redis
  gather_facts: false
  become: false
  tasks:
    - name: Uninstall pcp
      ansible.builtin.include_role:
        name: pcp
        tasks_from: uninstall.yml

- name: Uninstall the database
  hosts: database
  gather_facts: false
  become: false
  tasks:
    - name: Uninstall postgresql
      ansible.builtin.import_role:
        name: postgresql
        tasks_from: uninstall.yml

- name: Remove service leftovers
  hosts: automationcontroller:automationeda:automationgateway:automationhub
  gather_facts: false
  become: false
  tasks:
    - name: Run ostree tasks
      ansible.builtin.import_role:
        name: common
        tasks_from: ostree.yml

    - name: Remove python packages used by collections
      ansible.builtin.package:
        name:
          - python3-psycopg2
        state: absent
      become: true
      when: not ostree | bool

- name: Uninstall common container components
  hosts: automationcontroller:automationeda:automationgateway:automationhub:database:execution_nodes:redis
  gather_facts: false
  become: false
  tasks:
    - name: Uninstall podman
      ansible.builtin.import_role:
        name: common
        tasks_from: uninstall.yml
...
```
* 앤서블 플레이북에서 사용하기 위해서는 FQDN을 사용
* 설치 제거를 위한 FQDN은 *ansible.containerized_installer.uninstall*
<br>

### 1.2 기본 제거 명령

```bash
ansible-playbook -i inventory ansible.containerized_installer.uninstall
```
* 모든 systemd 장치 및 컨테이너가 중지
* 컨테이너화된 설치 프로그램에서 사용하는 모든 리소스가 삭제
  - 구성 및 데이터 디렉터리/파일
  - systemd 장치 파일
  - Podman 컨테이너 및 이미지
  - RPM 패키지
<br>

### 1.3 컨테이너 이미지 유지

```bash
ansible-playbook -i inventory ansible.containerized_installer.uninstall -e container_keep_images=true
```
* container_keep_images 변수를 true로 설정
<br>

### 1.4 postgresql 데이터베이스 유지

```bash
ansible-playbook -i </path/to/inventory> ansible.containerized_installer.uninstall -e postgresql_keep_databases=true
```
* postgresql_keep_databases 변수를 true로 설정
<br>
<br>

## 2. 설치 제거 실행

### 2.1 uninstall.yml 플레이북 실행

컨테이너화 된 AAP의 설치를 제거합니다.
```bash
cd temp/ansible-setup/
ansible-playbook -i inventory ansible.containerized_installer.uninstall
```
* 명령어는 앤서블 설치 파일 위치 (inventory 파일이 있는)에서 실행

실행 결과
```
[shadowman@aap-c ~]$ cd temp/ansible-setup/
[shadowman@aap-c ansible-setup]$ ansible-playbook -i inventory ansible.containerized_installer.uninstall
PLAY [Collect services facts] ***********************************************************************************************************

TASK [Gather regular user uid and home dir] *********************************************************************************************
ok: [aap-c.thinkmore.net]

TASK [Get service facts] ****************************************************************************************************************
ok: [aap-c.thinkmore.net]

PLAY [Uninstall the Automation Controller] **********************************************************************************************

TASK [ansible.containerized_installer.automationcontroller : Set facts for containers] **************************************************
ok: [aap-c.thinkmore.net]

TASK [ansible.containerized_installer.automationcontroller : Set facts for systemd services and containers] *****************************
ok: [aap-c.thinkmore.net]

TASK [ansible.containerized_installer.automationcontroller : Set controller port list] **************************************************
ok: [aap-c.thinkmore.net]

TASK [ansible.containerized_installer.automationcontroller : Add https port to controller port list] ************************************
ok: [aap-c.thinkmore.net]

TASK [ansible.containerized_installer.automationcontroller : Ensure systemd units are disabled and stopped] *****************************
changed: [aap-c.thinkmore.net] => (item=automation-controller-rsyslog.service)
changed: [aap-c.thinkmore.net] => (item=automation-controller-task.service)
changed: [aap-c.thinkmore.net] => (item=automation-controller-web.service)

TASK [ansible.containerized_installer.automationcontroller : Delete the containers] *****************************************************
changed: [aap-c.thinkmore.net] => (item=automation-controller-rsyslog)
changed: [aap-c.thinkmore.net] => (item=automation-controller-task)
changed: [aap-c.thinkmore.net] => (item=automation-controller-web)

TASK [ansible.containerized_installer.automationcontroller : Delete the systemd unit files] *********************************************
changed: [aap-c.thinkmore.net] => (item=automation-controller-rsyslog.service)
changed: [aap-c.thinkmore.net] => (item=automation-controller-task.service)
changed: [aap-c.thinkmore.net] => (item=automation-controller-web.service)

TASK [ansible.containerized_installer.automationcontroller : Delete podman volumes] *****************************************************
included: /home/shadowman/temp/ansible-automation-platform-containerized-setup-bundle-2.5-2-x86_64/collections/ansible_collections/ansible/containerized_installer/roles/automationcontroller/tasks/volumes.yml for aap-c.thinkmore.net

...<snip>...

TASK [ansible.containerized_installer.common : Remove the executionplane container images] **********************************************
changed: [aap-c.thinkmore.net] => (item=registry.redhat.io/ansible-automation-platform-25/de-supported-rhel8:latest)
changed: [aap-c.thinkmore.net] => (item=registry.redhat.io/ansible-automation-platform-25/ee-minimal-rhel8:latest)
changed: [aap-c.thinkmore.net] => (item=registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel8:latest)

TASK [ansible.containerized_installer.common : Delete the execution plane storage] ******************************************************
changed: [aap-c.thinkmore.net]

TASK [ansible.containerized_installer.common : Disable podman socket] *******************************************************************
changed: [aap-c.thinkmore.net]

TASK [ansible.containerized_installer.common : Delete the podman files] *****************************************************************
ok: [aap-c.thinkmore.net] => (item=/home/shadowman/aap/containers/podman)
changed: [aap-c.thinkmore.net] => (item=/home/shadowman/.config/systemd/user/podman.service.d)
changed: [aap-c.thinkmore.net] => (item=/run/user/1001/podman/podman.sock)

TASK [ansible.containerized_installer.common : Reload systemd] **************************************************************************
ok: [aap-c.thinkmore.net]

TASK [ansible.containerized_installer.common : Log out of the registry] *****************************************************************
skipping: [aap-c.thinkmore.net]

TASK [ansible.containerized_installer.common : Remove the container images] *************************************************************
changed: [aap-c.thinkmore.net] => (item=registry.redhat.io/ansible-automation-platform-25/gateway-rhel8:latest)
changed: [aap-c.thinkmore.net] => (item=registry.redhat.io/ansible-automation-platform-25/gateway-proxy-rhel8:latest)
changed: [aap-c.thinkmore.net] => (item=registry.redhat.io/ansible-automation-platform-25/controller-rhel8:latest)
changed: [aap-c.thinkmore.net] => (item=registry.redhat.io/ansible-automation-platform-25/receptor-rhel8:latest)
changed: [aap-c.thinkmore.net] => (item=registry.redhat.io/ansible-automation-platform-25/eda-controller-rhel8:latest)
changed: [aap-c.thinkmore.net] => (item=registry.redhat.io/ansible-automation-platform-25/eda-controller-ui-rhel8:latest)
changed: [aap-c.thinkmore.net] => (item=registry.redhat.io/ansible-automation-platform-25/hub-rhel8:latest)
changed: [aap-c.thinkmore.net] => (item=registry.redhat.io/ansible-automation-platform-25/hub-web-rhel8:latest)
changed: [aap-c.thinkmore.net] => (item=registry.redhat.io/rhel8/redis-6:latest)
changed: [aap-c.thinkmore.net] => (item=registry.redhat.io/rhel8/postgresql-15:latest)

TASK [ansible.containerized_installer.common : Remove the containers config directory] **************************************************
changed: [aap-c.thinkmore.net]

TASK [ansible.containerized_installer.common : Remove the TLS CA and container certs directories] ***************************************
ok: [aap-c.thinkmore.net] => (item=/home/shadowman/.config/containers/certs.d)
changed: [aap-c.thinkmore.net] => (item=/home/shadowman/aap/tls)

TASK [ansible.containerized_installer.common : Delete the ansible base shared secret] ***************************************************
ok: [aap-c.thinkmore.net]

PLAY RECAP ******************************************************************************************************************************
aap-c.thinkmore.net        : ok=132  changed=60   unreachable=0    failed=0    skipped=12   rescued=0    ignored=0   

[shadowman@aap-c ansible-setup]$
```
<br>

### 2.2 설치 제거 확인

#### 2.2.1 컨테이너 확인

```bash
podman ps -a
podman images
```

실행 결과
```
[shadowman@aap-c ansible-setup]$ podman ps -a
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES

[shadowman@aap-c ansible-setup]$ podman images
REPOSITORY  TAG         IMAGE ID    CREATED     SIZE

[shadowman@aap-c ansible-setup]$ 
```

#### 2.2.2 서비스 확인

```bash
systemctl list-units --type=service --state=running --user
systemctl list-units --user
```

실행 결과
```
[shadowman@aap-c ansible-setup]$ systemctl list-units --type=service --state=running --user
Failed to connect to bus: No medium found

[shadowman@aap-c ansible-setup]$ systemctl list-units --user
Failed to connect to bus: No medium found

[shadowman@aap-c ansible-setup]$ 
```
<br>
<br>
