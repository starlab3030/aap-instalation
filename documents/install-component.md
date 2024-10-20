# AAP 컴포넌트 선택 설치

## 1. Automation Controller 설치

### 1.1 설치 인벤토리 파일

```ini
# This section is for your AAP Gateway host(s)
# -----------------------------------------------------
[automationgateway]
aap-c.thinkmore.net ansible_connection=local

# This section is for your AAP Controller host(s)
# -----------------------------------------------------
[automationcontroller]
aap-c.thinkmore.net ansible_connection=local

# This section is for the AAP database
# -----------------------------------------------------
[database]
aap-c.thinkmore.net ansible_connection=local

[all:vars]

# Common variables
postgresql_admin_username=postgres
postgresql_admin_password=redhat

bundle_install=true
bundle_dir=/home/shadowman/temp/ansible-setup/bundle

redis_mode=standalone

# AAP Gateway
# 
gateway_admin_password=redhat
gateway_pg_host=aap-c.thinkmore.net
gateway_pg_password=redhat

# AAP Controller
# 
controller_admin_password=redhat
controller_pg_host=aap-c.thinkmore.net
controller_pg_password=redhat
```
<br>

### 1.2 설치 실행

```bash
cd ~/temp/ansible-setup/
ansible-playbook -i inventory ansible.containerized_installer.install
```

실행 결과
```
[shadowman@aap-c ~]$ cd ~/temp/ansible-setup/

[shadowman@aap-c ansible-setup]$ ansible-playbook -i inventory ansible.containerized_installer.install

...<snip>...

PLAY [Run the Automation Hub postinstall] ******************************************************************************************
skipping: no hosts matched

PLAY [Delete the ansible base shared secret] ***************************************************************************************

TASK [Delete the ansible base shared secret] ***************************************************************************************
ok: [aap-c.thinkmore.net]

PLAY RECAP *************************************************************************************************************************
aap-c.thinkmore.net        : ok=401  changed=143  unreachable=0    failed=0    skipped=107  rescued=0    ignored=0
localhost                  : ok=18   changed=0    unreachable=0    failed=0    skipped=17   rescued=0    ignored=0

[shadowman@aap-c ansible-setup]$
```
<br>
<br>

## 2. Event-Driven Ansible 설치

### 2.1 설치 인벤토리 파일

```ini
# This section is for your AAP Gateway host(s)
# -----------------------------------------------------
[automationgateway]
aap-c.thinkmore.net ansible_connection=local

# This section is for your AAP EDA Controller host(s)
# -----------------------------------------------------
[automationeda]
aap-c.thinkmore.net ansible_connection=local

# This section is for the AAP database
# -----------------------------------------------------
[database]
aap-c.thinkmore.net ansible_connection=local

[all:vars]

# Common variables
postgresql_admin_username=postgres
postgresql_admin_password=redhat

bundle_install=true
bundle_dir=/home/shadowman/temp/ansible-setup/bundle

redis_mode=standalone

# AAP Gateway
# 
gateway_admin_password=redhat
gateway_pg_host=aap-c.thinkmore.net
gateway_pg_password=redhat

# AAP EDA Controller
# -----------------------------------------------------
eda_admin_password=redhat
eda_pg_host=aap-c.thinkmore.net
eda_pg_password=redhat
```
<br>
<br>

## 3. Automation Hub 설치

### 3.1 설치 인벤토리 파일

```ini
# This section is for your AAP Gateway host(s)
# -----------------------------------------------------
[automationgateway]
aap-c.thinkmore.net ansible_connection=local

# This section is for your AAP Automation Hub host(s)
# -----------------------------------------------------
[automationhub]
aap-c.thinkmore.net ansible_connection=local

# This section is for the AAP database
# -----------------------------------------------------
[database]
aap-c.thinkmore.net ansible_connection=local

[all:vars]

# Common variables
postgresql_admin_username=postgres
postgresql_admin_password=redhat

bundle_install=true
bundle_dir=/home/shadowman/temp/ansible-setup/bundle

redis_mode=standalone

# AAP Gateway
# 
gateway_admin_password=redhat
gateway_pg_host=aap-c.thinkmore.net
gateway_pg_password=redhat

# AAP Automation Hub
# -----------------------------------------------------
hub_admin_password=redhat
hub_pg_host=aap-c.thinkmore.net
hub_pg_password=redhat
```
<br>
<br>

------
[차례](../README.md)