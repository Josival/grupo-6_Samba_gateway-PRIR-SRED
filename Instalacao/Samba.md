# Compartilhamento de arquivos com Samba

## Objetivo:

   * Configurar um servidor compartilhamento de arquivos usando o serviço Samba no linux
   * Acessar o **Gateway Server** via Putty no Windows e depois acessar os servidores **samba**.

https://ubuntu.com/tutorials/install-and-configure-samba#3-setting-up-samba

https://www.youtube.com/watch?v=uryVhhgTn9g

## Nome da máquina

```
Tabela 1: Exemplo de nomes dos servidores
---------------------------------------------------------
|    Nome da VM     |                NOME               |
---------------------------------------------------------
| Gateway (gw)      | gw.grupo6.turma924.ifalara.local	|
| Samba (smb)       | smb.grupo6.turma924.ifalara.local	|
| NameServer1 (ns1) | ns1.grupo6.turma924.ifalara.local	|
| NameServer2 (ns2) | ns2.grupo6.turma924.ifalara.local	|
---------------------------------------------------------
```

## Passo-a-passo:

```
Tabela 2: Definições da rede interna da turma 924
-----------------------------------------------
|  DESCRICAO  |  IP / ens160  |  IP / ens192  |
-----------------------------------------------
| rede        | 10.9.24.0     | 192.168.24.80 |
| máscara     | 255.255.255.0 | 255.255.255.0 |
| Gateway     | 10.9.24.108   | 192.168.24.89 |
| Samba       | 10.9.24.117   | 192.168.24.90 |
| NameServer1 | 10.9.24.121   | 192.168.24.91 |
| NameServer2 | 10.9.24.231   | 192.168.24.92 |
-----------------------------------------------
```
### 1. Na máquina Host faça login via ssh (Use Putty no Windows ou o Terminal no Linux)

Exemplo: $ ssh usuário@ipremoto

> Iremos dar o SSH para o ip reservado para o Samba
```bash
$ ssh administrador@10.9.24.117
```
<img src="/Figuras/Samba/1.4.png" title="ssh" width="550" /> 

> Se o SSH não funcionar, pode ser que esteja faltando alguma conexão ser feita, acesse o [Repositório](https://github.com/Josival/grupo-6_Samba_gateway-PRIR-SRED/blob/main/Instalacao/SSH.md) que poderá ajudar a resolver os possiveis problemas

### 2. Definir o IP da rede interna para o Samba-SRV

```bash
$ sudo nano /etc/netplan/00-installer-config.yaml
```
> Iremos configurar este arquivo segundo as configurações reservadas para o nosso grupo:
```
network:
  renderer: networkd
  ethernets:
    ens160:
      dhcp4: false
      addresses: [10.9.24.117/24]
      gateway4: 10.9.24.108
      nameservers:
         addresses:
           - 10.9.24.1
```
> O arquivo ficará dessa forma (Na parte de baixo do arquivo terá o ip para a interface de rede ens192, que foi designado previamente para o nosso grupo) e servirá como rede interna):
<img src="/Figuras/Samba/1.1.png" title="arquivo no sudo nano" width="550" /> 

> Para salvar/aplicar as alterações feitas no arquivo, pelo sudo nano, precisamos dar este comando:
```bash
$ sudo netplan apply
```
> Para vermos se as alterações foram realmente feitas, damos este comando:
```bash
$ ifconfig -a
```
<img src="/Figuras/Samba/1.2.png" title="ifconfig -a" width="550" /> 

```bash
$ ping 10.9.24.1
```
<img src="/Figuras/Samba/1.3.png" title="ping" width="550" /> 

### 3. instalar o servidor samba na MV samba-srv

```bash
$ sudo apt update
```
<img src="/Figuras/Samba/1.5.1..png" title="sudo apt" width="1000" />

```bash
$ sudo apt install samba
```
<img src="/Figuras/Samba/1.5.2..png" title="sudo apt" width="600" />
  
### 4. Verfificar se o samba está rodando

```bash
$ whereis samba
samba: /usr/sbin/samba /usr/lib/x86_64-linux-gnu/samba /etc/samba /usr/share/samba /usr/share/man/man8/samba.8.gz /usr/share/man/man7/samba.7.gz
```
<img src="/Figuras/Samba/1.6.png" title="whereis" width="550" />

```bash
$ sudo systemctl status smbd
● smbd.service - Samba SMB Daemon
     Loaded: loaded (/lib/systemd/system/smbd.service; enabled; vendor preset: >
     Active: active (running) since Wed 2022-12-07 17:03:10 UTC; 2 weeks 6 days>
       Docs: man:smbd(8)
             man:samba(7)
             man:smb.conf(5)
   Main PID: 45360 (smbd)
     Status: "smbd: ready to serve connections..."
      Tasks: 4 (limit: 462)
     Memory: 11.4M
     CGroup: /system.slice/smbd.service
             ├─45360 /usr/sbin/smbd --foreground --no-process-group
             ├─45362 /usr/sbin/smbd --foreground --no-process-group
             ├─45363 /usr/sbin/smbd --foreground --no-process-group
             └─45364 /usr/sbin/smbd --foreground --no-process-group

Dec 07 17:03:08 smb.grupo6.turma924.ifalara.local systemd[1]: smbd.service: Suc>
Dec 07 17:03:08 smb.grupo6.turma924.ifalara.local systemd[1]: Stopped Samba SMB>
Dec 07 17:03:08 smb.grupo6.turma924.ifalara.local systemd[1]: Starting Samba SM>
Dec 07 17:03:10 smb.grupo6.turma924.ifalara.local systemd[1]: Started Samba SMB
```
<img src="/Figuras/Samba/1.7.png" title="systemctl" width="1000" />

```bash
$ netstat -an | grep LISTEN
tcp        0      0 0.0.0.0:445             0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:139             0.0.0.0:*               LISTEN   
```
<img src="/Figuras/Samba/1.8.png" title="netstat" width="550" />

### 5. Faça o backup do arquivo de configuração do samba e cria um arquivo novo somente com os comandos necessários.
    
```bash
$ sudo cp /etc/samba/smb.conf{,.backup}
```

> Acesse o diretório "/etc/samba/" com o comando:

	cd ./etc/samba/

```
$ ls -la
-rw-r--r--  1 root root 8942 Mar 22 20:55 smb.conf
-rw-r--r--  1 root root 8942 Mar 23 01:42 smb.conf.backup
```
<img src="/Figuras/Samba/1.9..png" title="ls -la" width="550" />

> Comando para remover comentários desnecessários do arquivo:

```
$ sudo bash -c 'grep -v -E "^#|^;" /etc/samba/smb.conf.backup | grep . > /etc/samba/smb.conf'
```

```bash
$ sudo nano /etc/samba/smb.conf
```
> Iremos configurar este arquivo com estas configurações:
```
[global]
   workgroup = WORKGROUP
   server string = %h server (Samba, Ubuntu)
   log file = /var/log/samba/log.%m
   max log size = 1000
   logging = file
   panic action = /usr/share/samba/panic-action %d
   server role = standalone server
   obey pam restrictions = yes
   unix password sync = yes
   passwd program = /usr/bin/passwd %u
   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .
   pam password change = yes
   map to guest = bad user
   usershare allow guests = yes
[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700
[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
```
> O arquivo ficará dessa forma:
<img src="/Figuras/Samba/1.10.png" title="arquivo no sudo nano" width="800" /> 

  
 ### 6. Edite o arquivo de configuração /etc/samba/smb.conf

	* adicione as interfaces da sua máquina na linha "interfaces = 127.0.0.1/8 enp0s3", separando os nomes das interfaces por espaços.
  
  
```bash
$ sudo nano /etc/samba/smb.conf
```
> Iremos configurar este arquivo com estas configurações:
```
[global]
   workgroup = WORKGROUP
   netbios name = samba-srv
   security = user
   server string = %h server (Samba, Ubuntu)
   interfaces = 127.0.0.1/8 ens160 ens192
   bind interfaces only = yes
   log file = /var/log/samba/log.%m
   max log size = 1000
   logging = file
   panic action = /usr/share/samba/panic-action %d
   server role = standalone server
   obey pam restrictions = yes
   unix password sync = yes
   passwd program = /usr/bin/passwd %u
   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .
   pam password change = yes
   map to guest = bad user
   usershare allow guests = yes
[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700
[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
[homes]
   comment = Home Directories
   browseable = yes
   read only = no
   create mask = 0700
   directory mask = 0700
   valid users = %S
[public]
   comment = public anonymous access
   path = /samba/public
   browsable =yes
   create mask = 0660
   directory mask = 0771
   writable = yes
   guest ok = yes
   guest only = yes
   force user = nobody
   force create mode = 0777
   force directory mode = 0777
```
> O arquivo ficará dessa forma:
<img src="/Figuras/Samba/1.11.png" title="arquivo no sudo nano" width="1000" /> 

    * Renicie o serviço smbd
    
```bash
$ sudo systemctl restart smbd
```

   * modifica a pasta /samba/public para acesso a somente usuários do grupo sambashare

    sudo nano /etc/samba/smb.conf

```
[public]
   comment = public anonymous access
   path = /samba/public
   browsable =yes
   create mask = 0660
   directory mask = 0771
   writable = yes
   guest ok = no
   valid users = @sambashare
   #guest only = yes
   #force user = nobody
   #force create mode = 0777
   #force directory mode = 0777
```
<img src="/Figuras/Samba/1.12.png" title="config public" width="1000" /> 


    * Crie um usuário do S.O para que possa utilizar o compartilhamento samba:
    * usuário: aluno
    * senha: alunoifal
    
```bash
$ sudo adduser aluno
Adding user `aluno' ...
Adding new group `aluno' (1001) ...
Adding new user `aluno' (1001) with group `aluno' ...
Creating home directory `/home/aluno' ...
Copying files from `/etc/skel' ...
New password: 
Retype new password: 
passwd: password updated successfully
Changing the user information for aluno
Enter the new value, or press ENTER for the default
	Full Name []: Aluno de SRED no IFAL Arapiraca
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 
Is the information correct? [Y/n] y
```
<img src="/Figuras/Samba/1.13.png" title="adduser" width="550" /> 

    * É necessário vincular o usuário do S.O. ao Serviço Samba. Repita a senha de aluno ou crie uma senha nova somente para acessar o compartilhamento de arquivo. Neste caso repetiremos a senha do usuário aluno
    
```bash
$ sudo smbpasswd -a aluno
New SMB password:
Retype new SMB password:
Added user aluno.
```

<img src="/Figuras/Samba/1.14.png" title="sudo user mod" width="550" /> 

```bash
$ sudo usermod -aG sambashare aluno
```
    * O Samba já está instalado, agora precisamos criar um diretório para compartilhá-lo em rede.
   
```bash
$ mkdir sambashare
$ sudo mkdir -p /samba/public
```

    * Configure as permissões para que qualquer um possa acessar o compartilhamento público.

```bash
sudo chown -R nobody:nogroup /samba/public
sudo chmod -R 0775 /samba/public
sudo chgrp sambashare /samba/public
```

### 7. Cliente do compartilhamento:
   
    * Em um máquina com Windows (também pode usar linux os MacOS) digite no Winndows Explorer o endereço IP do servidor samba da seguinte forma:
    **\\ip_do_maquina**. Exemplo: smb://10.9.24.117/
    
   <p><center> Figura 1: Tela do Arquivos do Ubuntu para conectar com o servidor.</center></p>   
   <img src="/Figuras/Samba/1.15.png" alt="acesso pelo cliente samba"
	title="Figura 1: acesso pelo cliente samba" width="800" height="540" /> 


## Referências

    1. https://ubuntu.com/tutorials/install-and-configure-samba#1-overview
    2. https://websiteforstudents.com/install-samba-on-ubuntu-20-04-18-04/
    3. https://linuxconfig.org/how-to-configure-samba-server-share-on-ubuntu-20-04-focal-fossa-linux
    4. https://www.techrepublic.com/article/how-to-create-a-samba-share-on-ubuntu-server-20-04/
    5. https://help.ubuntu.com/community/Samba/SambaServerGuide?_ga=2.233364766.1651094097.1616438807-722741666.1614875855
