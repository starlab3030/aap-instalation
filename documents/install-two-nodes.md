# 2대의 노드에 AAP 구성


```yaml

```


```json

```


```bash
egrep -v "^$|^#" inventory
```

실행 결과
```
[shadowman@aap-c1 ansible-setup]$ egrep -v "^$|^#" inventory
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
postgresql_admin_username=postgres
postgresql_admin_password=redhat
redis_mode=standalone
bundle_install=true
bundle_dir=/home/shadowman/temp/ansible-setup/bundle
gateway_admin_password=redhat
gateway_pg_host=aap-db.thinkmore.net
gateway_pg_password=<set your own>
controller_admin_password=redhat
controller_pg_host=aap-db.thinkmore.net
controller_pg_password=redhat
hub_admin_password=redhat
hub_pg_host=aap-db.thinkmore.net
hub_pg_password=redhat
hub_shared_data_path="192.168.0.3:/ext/nfs/aap/hub"
eda_admin_password=redhat
eda_pg_host=aap-db.thinkmore.net
eda_pg_password=redhat

[shadowman@aap-c1 ansible-setup]$ 
```