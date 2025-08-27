# My personal cheat sheet
- [Remove PEM password from certificate](#remove-pem-password-from-certificate)
- [Change any Wazuh password](#change-any-wazuh-password)
- [Get VM info from Azure hosts](#get-vm-info-from-azure-hosts)
- [Search Wazuh json logs](#search-wazuh-json-logs)
- [Create exception rules on NAXSI WAF](#create-exception-rules-on-naxsi-waf)
- [Deleting Elasticsearch indexes](#deleting-elasticsearch-indexes)
- [Test redis with curl](#test-redis-with-curl)
- [Windows 11](#windows-11)
    * [Change PATH on PowerShell permanently](#change-path-on-powershell-permanently)
- [Suricata monitoring](#suricata-monitoring)
- [LDAP](#ldap)
- [OpenBSD Firewall na Azure](#openbsd-firewall-na-azure)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

## Extract private key from PFX certificate
```
openssl pkcs12 -in cert.pfx -nocerts -out private.key
```

## Remove PEM password from certificate
```
openssl rsa -in futurestudio_with_pass.key -out futurestudio.key
```

## Change any Wazuh password
1. Access your master node and navigate to the python3 console:
```
root@wazuh-master:/# /var/ossec/framework/python/bin/python3
```
2. Once in the python3 console, import the update_user framework function and use it with the user_id and a new password. In this case, the user_id is 1 for the “wazuh” user.
```
>>> from wazuh.security import update_user >>> update_user(user_id="1", password="NewPassword1!").render()
```
3. If the process was successful, you will receive the following output:
{'data': {'affected_items': [{'id': 1, 'username': 'wazuh', 'allow_run_as': True, 'roles': [1]}], 'total_affected_items': 1, 'total_failed_items': 0, 'failed_items': []}, 'message': 'User was

Original post: https://groups.google.com/g/wazuh/c/zxhdkmSkclE


## Get VM info from Azure hosts
Linux (you can use "jq" to filter json, but maybe not every machine has it installed...
```
curl -s -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance?api-version=2021-02-01" | sed -n -e 's/^.*vmSize\"\:\"//p' | sed 's/".*//'
```
Windows
```
((Invoke-WebRequest -Headers @{ 'Metadata' = 'true'} -URI http://169.254.169.254/metadata/instance?api-version=2021-02-01).Content | ConvertFromJson).compute.vmSize
```
Zabbix system.run item
```
system.run[powershell.exe -NoProfile -ExecutionPolicy Bypass "((Invoke-WebRequest -UseBasicParsing -Headers @{ 'Metadata' = 'true'} -URI http://169.254.169.254/metadata/instance?api-version=2021-02-01).Content | ConvertFrom-Json).compute.vmSize",wait]
```


## Search Wazuh json logs
```
cat ossec-alerts-25.json | jq -r -c 'select(.agent.ip=="16.50.20.14")' > filtered.json
```
## Create exception rules on NAXSI WAF

* Reset log file contents
```
:> /var/log/nginx/site.com-error.log
```
* Monitor the log file and test the application
```
tail -f /var/log/nginx/site.com-error.log
 ```
* Execute nx_util.py to generate exception rules
```
nx_util.py -o -p 1  -l /var/log/nginx/site.com-error.log
or
nx_util.py -o -p 1  -l /var/log/nginx/site.com-error.log >> /etc/nginx/naxsi_rules/site.rules
```
* Include rules in the site rules file referenced in nginx_site.conf and adjust accordingly
```
vim /etc/nginx/conf.d/site.com.conf
vim /etc/nginx/naxsi_rules/site.rules
```
## Deleting Elasticsearch indexes

https://www.ibm.com/docs/en/cloud-private/3.1.2?topic=logging-manually-removing-log-indices

Execute the steps logged into Kibana.
* List indexes
Kibana-->Dev Tools.
On the left panel, add a line to list indexes
```
GET /_cat/indices?v
```

Click on the green play button to execute the API call. You'll be presented a index list on the right side, there will be allocation status as well.
```
health status index                    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   logstash-2019.02.05      nbkLRGXqQ6enWMbLeYIO1w   5   1     932127            0    571.8mb        571.8mb
```
* Delete indexes

Note: Never remove these indexes:
* searchguard
* .kibana
They are essential to the system.

Identify the indexes you want to delete.

You can use "*" to delete a range, let's say all June indexes.

```
DELETE /{your index name}
DELETE /security-auditlog-2022.05*
DELETE /wazuh-alerts-4.x-2022.06*
```

## Test redis with curl
```
(printf "AUTH <password>\r\nPING\r\nQUIT\r\n";) | nc localhost 6379
```

## Windows 11
### Change PATH on PowerShell permanently

```
[Environment]::SetEnvironmentVariable("PATH", $Env:PATH + ";C:\Users\user\Scripts", [EnvironmentVariableTarget]::Machine)
```

## Suricata monitoring

```
tail -f /var/log/suricata/eve.json | jq -r -c 'select(.event_type=="alert")'
```

## LDAP
```
ldapsearch  -H ldap://172.7.6.5 -x -W -D "user@domain.local" -b "dc=domain,dc=local" "(sAMAccountName=user)"
LDAPTLS_REQCERT=never ldapsearch -Z -H ldap://172.5.6.7 -x -W -D "user@domain.local" -b "dc=domain,dc=local" "(sAMAccountName=user)"
```

## OpenBSD Firewall na Azure
- Criar uma VM com OpenBSD
- Criar uma VNet gigantesca, ex: 172.30.0.0/16
- Criar uma subnet para o OpenBSD, nosso Virtual Appliance, ex: 172.30.0.0/29 *alocar ip estático para o OpenBSD, vamos chama-la de "vnet_default" nesse doc.
- Vincular um ip externo ao sistema, de preferência estático
- Criar subnets para teste ou produção de acordo, ex: 172.30.1/24, 172.30.2/24 vinculando o ip (estático) 172.30.1.254 e 172.30.2.254 a VNics do OpenBSD (uma Vnic para cada subnet)
- Criar subnets de acordo com a necessidade, atribuindo interfaces ao OpenBSD com ip interno estático também
- Criar um Security Group para cada subnet, vincular de acordo
- Criar regra prioritária nas subnets bloqueando o tráfego com origem nas outras subnets e destino à "Virtual Network" inclusive na subnet do OpenBSD
- Criar regra específica aceitando SSH com destino ao ip interno do OpenBSD na "vnet_default", para poder conectar externamente e outras portas que precisem de redirecionamento ex: 443, 3306, 1433, etc.
- Criar "Route tables" para cada subnet, menos a "vnet_default", apontando a rota default "0.0.0.0/0" para o IP interno do OpenBSD na respectiva SUBNET, ex: 172.30.1.254 e 172.30.2.254
- Adicionar rotas também para as demais subnets, obrigando o tráfego a passar pelo Firewall (apontando pro OpenBSD também...).
- Vincular Route tables nas respectivas subnets
-OBS: Não fuciona somente forçando o roteamento pelo OpenBSD, o NAT geral é obritgatório independente da origem de ip público ou privada, por isso a criação das regras de bloqueio+rotas específicas é redundante, mas certifica que o tráfego seguirá o fluxo desejado sem possibilidade de "bypass" no OpenBSD.
