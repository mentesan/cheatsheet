# Dicas e truques

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
