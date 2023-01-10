# Introdução

   * O servidor de gateway pode através do NAT (Network Address Translation) fazer o roteamento todos as maquinas de uma rede interna (Ex.: Rede de MV no Virtualbox) para uma rede externa (Ex. Internet) utilizando somente um endereço de rede de saída. Para isso o NAT reescreve os enderecos da fonte (source address) de todos os pacotes de saída para o endereço da interface externa do gateway.

   * As definições de rede da rede externa ao gateway server estão na Tabela 1.

<p><center> Tabela 1: Definições da rede externa</center></p>

| DESCRIÇÃO   | IP            |
|:------------|:------------- |
| rede        | 10.0.2.0      |
| máscara     | 255.255.255.0 |
| VirtualBox (gateway)     | 10.0.2.2      |
| Broadcast   | 10.0.2.255    |

   * As definições de rede da rede interna ao gateway server estão na Tabela 2.

<p><center> Tabela 2: Definições da rede interna</center></p>

| DESCRIÇÃO   |  IP           |
|:------------|:------------- |
| rede        | 10.9.24.0     |
| máscara     | 255.255.255.0 |
| Broadcast   | 10.9.24.255   |
| Gateway     | 10.9.24.108   |
| samba       | 10.9.24.117   |
| NameServer1 | 10.9.24.121   |
| NameServer2 | 10.9.24.231   |





## Configuração do servidor Gateway como NAT:

### Configuração do firewall/NAT

   * Para configurar um servidor como gateway de rede é necessário configurar o firewall do linux (iptables). 
   * as regras do iptables podem ser digitadas no terminal ou podem ser executadas em um script.
   * Com um script, pode-se inicializar as regras do firewall todas as vezes que a máquina for reinicializada.

#### habilitar o firewall 
   * Vamos serguir os passos descritos em [1]:
   
   1. Habilite o firewall com o seguintes comandos, para permitir o acesso ssh:
```bash

 $ sudo ufw allow ssh
 $ sudo ufw enable
 $ sudo ufw status
 
```
   2. Habilite o encaminhamento de pacotes das interfaces WAN para LAN, descomentando a seguinte linha _# net/ipv4/ip_forwarding=1_

```bash
$ Comando: 
$ sudo nano /etc/ufw/sysctl.conf

...
net/ipv4/ip_forwarding=1
...
```


   3. Configurar as interfaces de rede (netplan) 

```bash
$ sudo nano /etc/netplan/50-cloud-init.yaml 
```

```
network:
    ethernets:
        enp0s3:
            dhcp4: true
        enp0s8:
            addresses: [10.9.24.1/24]
            dhcp4: false              
    version: 2
```

```bash
$ sudo netplan apply
$ ifconfig -a
```

   4. Crie o arquivo /etc/rc.local.
```bash
$ sudo nano /etc/rc.local
```

   5. Adicione o script abaixo no arquivo [/etc/rc.local](rc.local)

---
```bash
#!/bin/bash

# /etc/rc.local

# Default policy to drop all incoming packets.
# Politica padrão para bloquear (drop) todos os pacotes de entrada
iptables -P INPUT DROP
iptables -P FORWARD DROP

# Accept incoming packets from localhost and the LAN interface.
# Aceita pacotes de entrada a partir das interfaces localhost e the LAN.
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -i enp0s8 -j ACCEPT

# Accept incoming packets from the WAN if the router initiated the connection.
# Aceita pacotes de entrada a partir da WAN se o roteador iniciou a conexao
iptables -A INPUT -i enp0s3 -m conntrack \
--ctstate ESTABLISHED,RELATED -j ACCEPT

# Forward LAN packets to the WAN.
# Encaminha os pacotes da LAN para a WAN
iptables -A FORWARD -i enp0s8 -o enp0s3 -j ACCEPT

# Forward WAN packets to the LAN if the LAN initiated the connection.
# Encaminha os pacotes WAN para a LAN se a LAN inicar a conexao.
iptables -A FORWARD -i enp0s3 -o enp0s8 -m conntrack \
--ctstate ESTABLISHED,RELATED -j ACCEPT

# NAT traffic going out the WAN interface.
# Trafego NAT sai pela interface WAN
iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE

# rc.local needs to exit with 0
# rc.local precisa sair com 0
exit 0
```
---
   6. converte o arquivo em executável e o torna inicializável no boot
```bash
$ sudo chmod 755 /etc/rc.local
```
   7. verificar se o firewall está funcionando
```bash
$ sudo ufw status
```

   8.  reiniciar a máquina
```bash
$ sudo reboot
```
   9. Nas máquinas SAMBA, NS1 e NS2 ativar o gateway (gateway4: 10.9.24.1) na interface de rede:
```bash
$ sudo nano /etc/netplan/50-cloud-init.yaml
```
```
network:
    ethernets:
        enp0s3:
            addresses: [10.9.24.108/24]
            gateway4: 10.9.24.1
            dhcp4: false
            nameservers:
                addresses:
                - 8.8.8.8
                - 8.8.4.4
                search: []
    version: 2
```


```bash
$ sudo netplan apply
$ ifconfig -a
```

  10. Encaminhamento de portas para acesso externo à serviços da rede interna.
  
  * Para permitir que o serviço de compartilhamento de arquivos esteja disponível externamente, adicione as informações do IPTABLES sobre portas, IP e Interface no arquivo 
  * /etc/rc.local
  *  conforme o exemplo abaixo, depois reinicie a máquina:
  
   a. SAMBA: Para permitir que o serviço de compartilhamento de arquivos esteja disponível externamente:
        * Portas: 445 e 139
        
```bash
#Recebe pacotes na porta 445 da interface externa do gw e encaminha para o servidor interno na porta 445
iptables -A PREROUTING -t nat -i enp0s3 -p tcp --dport 445 -j DNAT --to 192.168.24.90:445
iptables -A FORWARD -p tcp -d 192.168.24.90 --dport 445 -j ACCEPT

#Recebe pacotes na porta 139 da interface externa do gw e encaminha para o servidor interno na porta 139
iptables -A PREROUTING -t nat -i enp0s3 -p tcp --dport 139 -j DNAT --to 192.168.24.90:139
iptables -A FORWARD -p tcp -d 192.168.24.90 --dport 139 -j ACCEPT
```
   b. DNS: Para permitir que o serviço de resolução de nomes (DNS) esteja disponível externamente:
        * Porta: 53
        
        
```bash
#Recebe pacotes na porta 53 da interface externa do gw e encaminha para o servidor DNS Master interno na porta 53
iptables -A PREROUTING -t nat -i enp0s3 -p udp --dport 53 -j DNAT --to 192.168.24.90:53
iptables -A FORWARD -p udp -d 192.168.24.90 --dport 53 -j ACCEPT
```


