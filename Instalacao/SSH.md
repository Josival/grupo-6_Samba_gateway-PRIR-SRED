# Solução de Possíveis Problemas com o SSH

 Se ao fazer um SSH para um ip e ele não fizer nenhuma conexão, e você não tinha feito nenhuma configuração/conexão do OpenVPN 3 antes, você deve ir para este [Repositório](https://github.com/alaelson/2022-924-notasdeaula/blob/main/Aula.924-2022.11.04.md). Mas se você já tinha feito anteriormente as configurações, e mesmo assim não está funcionando você deve realizar esses passos de verificação (Utilizaremos a instalações designadas para a turma 924):

### > Passo 1

> Ver se há alguma configuração do VPN da sua turma instalada/conectada. Você irá dar esta comando:

```bash
openvpn3 configs-list
```
> Se tiver uma configuração da sua turma, você irá para o [Passo 2](https://github.com/Josival/grupo-6_Samba_gateway-PRIR-SRED/edit/main/Instalacao/SSH.md#-passo-2). Se não houver nenhuma configuração da sua turma instalada, você irá fazer estes seguintes passos:

#### Importar o certificado para o OpenVPN 

* Baixe o arquivo com extensão ``.ovpn`` disponível no classroom
* Ou faça o download direto pelo terminal.

```bash
curl -fsSL https://www.dropbox.com/s/hb8ee3kiwkhutbl/vpn924.labredes.arapiraca.ifal.edu.br.ovpn?dl=0 > ~/vpn924.labredes.arapiraca.ifal.edu.br.ovpn
```


* ``CONFIG_FILE`` = arquivo de configuração .ovpn
* ``CONFIG_NAME``= nome da configuração

```bash
openvpn3 config-import --config CONFIG_FILE --name CONFIG_NAME --persistent
openvpn3 config-manage --show --config CONFIG_NAME --dco true
```
* exemplo
```bash
openvpn3 config-import --config vpn924.labredes.arapiraca.ifal.edu.br.ovpn --name vpn924.labredes --persistent
openvpn3 config-manage --show --config vpn924.labredes --dco true
```

### > Passo 2

> Após conferirmos que há uma conexão da turma instalada, iremos ver se há sessões abertas, com este comando:

    openvpn3 sessions-list
   
> Se houver 1 sessão aberta, então você irá para o [Passo 3](https://github.com/Josival/grupo-6_Samba_gateway-PRIR-SRED/edit/main/Instalacao/SSH.md#-passo-3). Se não houver nenhuma sessão aberta você irá iniciar a conexão (Irá pedir o username e a senha que o professor designou):

#### Iniciar a conexão
```bash
openvpn3 session-start --config CONFIG_NAME
```
* Exemplo
```bash
openvpn3 session-start --config vpn924.labredes
```

> Não pode haver mais de uma sessão aberta da mesma turma. Para remover uma das sessões, você irá dar este comando:

#### Finalizar a conexão
```bash
openvpn3 session-manage --path CONFIG_PATH --disconnect
```
* Exemplo
```bash
openvpn3 session-manage --path <Você irá colocar aqui o path da sessão que você deseja remover, que foi apresentado ao dar comando sessions-list (Sem esses "< >")> --disconnect
```

### > Passo 3

> Faça o teste agora com o SSH, para ver se irá funcionar. Se mesmo assim não funcionar, deve ser algum bug do servidor. Então, você irá dar o comando SSH com outro ip que funcione, ao entrar neste ip, ai sim você irá fazer o ssh com o ip que estava querendo entrar.
