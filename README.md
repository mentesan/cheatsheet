# Dicas e truques

## Search Wazuh/OSSEC json logs
```
cat ossec-alerts-25.json | jq -r -c 'select(.agent.ip=="16.50.20.14")' > filtered.json
```

## Deleting Wazuh indexes

https://www.ibm.com/docs/en/cloud-private/3.1.2?topic=logging-manually-removing-log-indices
Removendo através da API

Execute os seguintes passos, logado no Kibana.

* Listar os índices.
Logar no console do Kibana e ir em Dev Tools.

No painel da esquerda, apague o conteúdo e digite o seguinte para listar os índices:
```
GET /_cat/indices?v
```
Clique no triângulo verde para executar a chamada de API. Será apresentada a lista de índices do lado direito, assim como os dados de alocação, ex:

health status index                             uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   logstash-2019.02.05               nbkLRGXqQ6enWMbLeYIO1w   5   1     932127            0    571.8mb        571.8mb

* Delete os índices.

Note: Jamais remova os índices searchguard e .kibana pois são essenciais para o funcionamento.

Identifique os índices que deseja deletar, com base na lista apresentada.
Digite o seguinte para deletar os índices desejados, pode ser usado o caracter "*" para deletar todos os índices correspondentes ao mês em questão.
```
DELETE /{your index name}
DELETE /security-auditlog-2022.05*
DELETE /wazuh-alerts-4.x-2022.06*
```

## Testar redis com curl
(printf "AUTH <password>\r\nPING\r\nQUIT\r\n";) | nc localhost 6379
  
## Windows 11
### Alterar variável PATH no PowerShell permanentemente
```
[Environment]::SetEnvironmentVariable("PATH", $Env:PATH + ";C:\Users\user\Scripts", [EnvironmentVariableTarget]::Machine)
```

## Suricata monitoring
```
tail -f /var/log/suricata/eve.json | jq -r -c 'select(.event_type=="alert")'
```
## LDAP
ldapsearch  -H ldap://172.7.6.5 -x -W -D "user@domain.local" -b "dc=domain,dc=local" "(sAMAccountName=user)"
LDAPTLS_REQCERT=never ldapsearch -Z -H ldap://172.5.6.7 -x -W -D "user@domain.local" -b "dc=domain,dc=local" "(sAMAccountName=user)"

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
