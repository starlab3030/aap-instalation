# 운영 환경에서 Full-HA를 고려한 고성

## 1.

```ini
# This section is for your AAP Gateway host(s)
# -----------------------------------------------------
[automationgateway]
gateway1.thinkmore.net
gateway2.thinkmore.net

# This section is for your AAP Controller host(s)
# -----------------------------------------------------
[automationcontroller]
controller1.thinkmore.net
controller2.thinkmore.net

# This section is for your AAP Automation Hub host(s)
# -----------------------------------------------------
[automationhub]
hub1.thinkmore.net
hub2.thinkmore.net

# This section is for your AAP EDA Controller host(s)
# -----------------------------------------------------
[automationeda]
eda1.thinkmore.net
eda2.thinkmore.net

[redis]
gateway1.thinkmore.net
gateway2.thinkmore.net
hub1.thinkmore.net
hub2.thinkmore.net
eda1.thinkmore.net
eda2.thinkmore.net

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
<br>

------
[차례](../README.md)