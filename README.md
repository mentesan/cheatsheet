# Dicas e truques

## Windows 11
### Alterar variável PATH no PowerShell permanentemente
```
[Environment]::SetEnvironmentVariable("PATH", $Env:PATH + ";C:\Users\user\Scripts", [EnvironmentVariableTarget]::Machine)
```

## Suricata
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

## Information gathering
### DNS
- Mapeamento completo
  - Subdomain takeover
  - DNSRecon
```
            dnsrecon -d pixeon.com -k -b -z -y --iw
                -b Bing search
                -k crt.sh enum
                -z DNSSEC zone walk with standard enumeration
                -y Yandex search
                --iw Continue brute forcing even with wildcard records
                -x <file.xml> Save output in xml format
            dnsenum --noreverse --nocolor -w -p 5 pixeon.com > dnsenum_pixeon.com
                -p 5 Number of google pages to process
                -w Perform whois queries
            dnsmap pixeon.com -r dnsmap_pixeon.com -w /usr/share/dnsrecon/top1mil.txt
            
```
- Whois
  - Verificar se o ip pertence a sistemas de segurança ex: Cloudflare
  - Whois buscando por inetnum
```
            whois 204.225.42.33 | fgrep inetnum
```
- BGP
  - bgp.he.net
- Ping Sweep
```
        - masscan -p1-65535 --banners --http-user-agent "Mozilla/5.0 Firefox/42.0" --open-only 192.168.0.1
        - masscan -pU:5060-5061 --ping --banners --open-only
            - Levantar portas interessantes
            - Banner grabbing
```
- Traceroute
  - A todos os destinos levantados
  - Informativo, porém detectar anomalias
- MTR (My TraceRoute)

## Portscan
- Host discovery (ping scanning)
```
                -sL list targets to confirm ip addresses
                -PN skip ping
                -PE ICMP echo, timestamp and netmask request
                -PP Timestamp
                -PM Address mask
                -PA Ack ping
                -sP -n just host discovery
                -sP -PE icmp ping only
                -sP -PS22,80,443 -R TCP SYN ping
                -sP -PA80,21,110 TCP ACK ping
                -sP -PU53,38,466 UDP ping (to closed ports, default 31,338)
                -sP -PP ICMP timestamp
                -sP -PM ICMP address mask
                -sP -PO IP protocol ping
                -sP -PR ARP scan
                
                --source-port 53 useful for UDP/TCP pings
                --data-lenght 32 to evade IDS with rules for zero lenght pings
                    32 Windows echo request
                    56 Linux default ping
                    
                --max-parallelism defaults to the number of targets
                --randomize-hosts to further evade sensors
                --reason expands on why the hosts is up
                -D <decoy1,decoy2,...>

                Ex:
                nmap -sP -oX np_ips.xml 204.225.42.0/23
                nmap -sP -PE -PP -PS21,22,25,80,113,8080 -PA80113,443,10050 -T4 --source-port 53
```
- Target specification:
```
                scanme.nmap.org/16 192.168.1.2/24 10.20.1-16.1-254
                192.168.10.8 10.0.1,3,5,8.1-255
                -iL <file> (space, tabs or newlines) or "-iL -" to get from STDIN
                --exclude comma separeted list
                --excludefile <file>
```
- Reverse DNS resolution
```
                -n no resolution
                -R resolve eve down hosts
```
- Port scanning
```
                -A remote OS detection
                -p0- scan all ports
                -sS TCP Syn stealth
                -sT TCP Connect
                -sU UDP
                -sF TCP FIN
                -sX TCP XMASS
                -sN TCP NULL
                -sA TCP Ack - map firewall rulesets
                -sW TCP Window - like ACK, but detects openports agains some machines
                -sM Maimon - firewall-evading similar to FIN, works with a few systems
                -sI TCP Idle
                -sO -p <protocol> - Protocol scan
                By default scans the most popular 1.000 ports of the specified protocol.
                
                -F (fast) - scans only 100 most popular ports.
                --top-ports <num> - specify number
                -p <list of ports> (pg. 84)
                -T0 Very slow | -T5 extremely aggressive

                ex: nmap -T4 -PN -p80 --max-rtt-timeout 200 --initial-rtt-timeout 150 --min-hostgroup 512 -oG losg/openport80-%D.gmap 200.1.10.0/24
                %D - Numeric date
```
- Version detection
```
                -sV
                -A - Advanced and aggressive features
                nmap -sV --scan-delay 1 -sT -p<portas>
```
- OS detection
```
                -O
                -v - verbose
```
- Traceroute
```
                --traceroute
```                
- Script scanning
```
                -sC
                --script - to specify a custom set os scripts
                --script-args - arguments to scripts
                --script-trace - debug script

                ex: nmap -sC --script-args user=foo,pass=sapeca,whois={whodb=nofollow+ripe}
                    nmap --script=./showSSHVersion --script-trace example.com
                    nmap --script=mycuston,safe,discovery example.com
```
  - Bypass Snort
    - Connect Scan: 
```
                -sT
                    Apenas uma porta: -p80 ou -p443
```
  - Bypass Suricata
    - Alterar user-agent
    - Alterar strings 'sqlspider' no script
```
                nmap --scan-delay 1 -sT -p80 --script=http-sql-injection 192.168.0.1 --script-args http.useragent="Mozilla/5.0"
```
  - Categories:
```
                    auth
                    default
                    discovery
                    external
                    intrusive
                    malware
                    safe
                    version
                    vuln
```
 - Subverting Firewalls and IDSs
   - NMAP Network Scanning - pg 260

 - OS detection
   - Nmap
 - Service detection
   - Nmap
   - Scripts NSE somente em portas interessantes
 - WPScan
```
            wpscan --url https://200.183.138.72 --rua --force --disable-tls-checks
```
## HTTP/HTTPS
- SSL
  - Onde houver portas interessantes 443 8443 4443 etc
  - Verificar se tem ssl
  - Executar ferramentas de SSL scan
    - sslscan
    - sslyze


