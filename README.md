# Para configurar um cluster MySQL de alta disponibilidade em dois servidores usando Docker, siga os passos abaixo.

## Servidor Principal:

### Passo 1: Configuração no Servidor Principal

#### Acesse a pasta /home:
```
cd /home
```
#### Criar o diretorio do docker (zbx7-mysql-ha-01):
```
mkdir zbx7-mysql-ha-01
```
```
cd /home/zbx7-mysql-ha-01
```
#### Copiar o script de setup inicial da maquina para dentro do diretorio (setup.sh)
```
escrever gitclone
```
#### Copiar o docker compose para dentro do diretorio (docker-compose.yml)
```
escrever gitclone
```
#### Acessar o diretorio e dar permissão para o arquivo setup.sh com:
```
chmod +x setup.sh
```
#### Rodar o script para realizar a preparação da maquina para utilizar o docker:
```
./setup.sh
```
#### Aceite todas as solicitações com "Y" ou yes para instalar todas as dependencias.

#### Após a maquina reiniciar acesse novamente o diretorio criado no /home/zbx7-mysql-ha-01

#### Construa a maquina com o docker-compose com o comando:
```
docker-compose up -d
```
#### Conectar ao Docker:
```
docker exec -it <NOME-DO-CONTAINER> bash
```
#### Adicionar as configurações de report_host nos seus servidores usando o comando echo:
```
echo -e "[mysqld]\nreport_host=<IP-DO-SERVIDOR-PRINCIPAL>" | tee -a /etc/my.cnf
```
#### Reiniciar o serviço de dentro do container:
```
mysqlsh --sql -e "SHUTDOWN;"
```
#### Veja se voltou o serviço
```
docker-compose ps
```
#### Caso e somente se, o serviço nao fique up sozinho novamente execute:
```
docker-compose stop
docker-compose start
```
#### Caso esteja up ignore esses dois comandos

#### Conectar ao MySQL Shell (substitua ROOT_PASSWORD pela senha do root):
```
docker exec -it <NOME-DO-CONTAINER> mysqlsh root@localhost --password=<SENHA-ROOT>
```
#### Configure a instância:
```
dba.configureInstance('root@localhost:3306', {password: '<SENHA-ROOT>'});
```
#### Verifique a configuração da instância, Após a configuração, execute::
```
dba.checkInstanceConfiguration('root@localhost:3306', {password: '<SENHA-ROOT>'});
```
#### Criar o cluster no servidor principal:
#### Dentro do MySQL Shell:
```
var cluster = dba.createCluster('ZabbixCluster');
```
#### Verificar o group_replication e setar os IPs de replicação:
```
\sql
SHOW VARIABLES LIKE 'group_replication%';
SET GLOBAL group_replication_group_seeds = '<IP-DO-SERVIDOR-PRINCIPAL>:3306,<IP-DO-SERVIDOR-SECUNDARIO>:3306';
START GROUP_REPLICATION;
```
##############################################################################################

## Servidor Secundario:

### Passo 2: Configuração no Servidor Secundario

#### Libere o ip do servidor principal no firewall para permitir acesso na porta 3306:
```
sudo ufw allow from <IP-DO-SERVIDOR-PRINCIPAL> to any port 3306 proto tcp
```
#### Acesse a pasta /home:
```
cd /home
```
#### Criar o diretorio do docker (zbx7-mysql-ha-02):
```
mkdir zbx7-mysql-ha-02
```
```
cd /home/zbx7-mysql-ha-02
```
#### Copiar o script de setup inicial da maquina para dentro do diretorio (setup.sh)
#### Copiar o docker compose para dentro do diretorio (docker-compose.yml)

#### Acessar o diretorio e dar permissão para o arquivo setup.sh com:
```
chmod +x setup.sh
```
#### Rodar o script para realizar a preparação da maquina para utilizar o docker:
```
./setup.sh
```
#### Aceite todas as solicitações com "Y" ou yes para instalar todas as dependencias.

#### Após a maquina reiniciar acesse novamente o diretorio criado no /home/zbx7-mysql-ha-02
```
cd /home/zbx7-mysql-ha-02
```
#### Construa a maquina com o docker-compose com o comando:
```
docker-compose up -d
```
#### Conectar ao Docker:
```
docker exec -it <NOME-DO-CONTAINER> bash
```
#### Adicionar as configurações de report_host nos seus servidores usando o comando echo:
```
echo -e "[mysqld]\nreport_host=177.137.208.38" | tee -a /etc/my.cnf
```
#### Reiniciar o serviço de dentro do container:
```
mysqlsh --sql -e "SHUTDOWN;"
```
#### Veja se voltou o serviço
```
docker-compose ps
```
#### Caso e somente se, o serviço nao fique up sozinho novamente execute:
```
docker-compose stop
docker-compose start
```
#### Caso esteja up ignore esses dois comandos

#### Conectar ao MySQL Shell (substitua ROOT_PASSWORD pela senha do root):
```
docker exec -it <NOME-DO-CONTAINER> mysqlsh root@localhost --password=7xJQFBSceexH
```
#### Configure a instância:
```
dba.configureInstance('root@localhost:3306', {password: '7xJQFBSceexH'});
```
#### Verifique a configuração da instância, Após a configuração, execute::
```
dba.checkInstanceConfiguration('root@localhost:3306', {password: '7xJQFBSceexH'});
```
##############################################################################################

## Servidor Principal:

### Passo 3: Adicione a instância ao cluster com a opção de clone para corrigir os GTIDs:

#### No MySQL Shell do servidor principal:
```
cluster.addInstance('root@177.137.208.38:3306', {password: '7xJQFBSceexH', recoveryMethod: 'clone'});
```
### Passo 4: Para criar o banco de dados 'zabbix-ha' após configurar o cluster MySQL, siga os passos abaixo:

#### 1- Conecte-se ao MySQL no container principal:
```
docker exec -it mysql-zabbix-01 mysql -u root -p
```
#### 2- Desative o super-read-only:
```
SET GLOBAL super_read_only = OFF;
```
#### 3- Crie o banco de dados:
```
CREATE DATABASE `zabbix-ha`;
```
#### 4- Crie o usuário zabbix com as permissões necessárias:
```
CREATE USER 'zabbix'@'%' IDENTIFIED BY '7xJQFBSceexH';
```
```
GRANT ALL PRIVILEGES ON `zabbix-ha`.* TO 'zabbix'@'%';
```
```
FLUSH PRIVILEGES;
```
#### 5- Verifique a criação:
```
SHOW DATABASES;
```
#### 6- Verifique as permissões do usuário zabbix:
```
SHOW GRANTS FOR 'zabbix'@'%';
```
#### 7- Ative o super-read-only:
```
SET GLOBAL super_read_only = ON;
```
#### 8- Teste a conexão com o MySQL:
```
mysql -u <USUARIO> -p<SENHA-ROOT> -h <IP do servidor MySQL>
```
#### No DBeaver ou na string de conexão, adicione o parâmetro allowPublicKeyRetrieval=true.

#### Por exemplo:
```
jdbc:mysql://<IP do servidor MySQL>:3306/<NOME-DATABASE>?allowPublicKeyRetrieval=true&useSSL=false
```
#### Isso garantirá que o banco de dados e o usuário estejam configurados corretamente no cluster.

##############################################################################################

### Resolução de problemas ao configurar o cluster MySQL:

#### 1 - Limpe o esquema de metadados da instância:
```
dba.dropMetadataSchema();
```
#### 2- Reinicie o cluster caso já tenha um:
```
dba.rebootClusterFromCompleteOutage();
```
#### 3- Crie o cluster novamente:
```
var cluster = dba.createCluster('ZabbixCluster');
```
#### 4- Adicione a instância secundária:
```
cluster.addInstance('root@<IP-DO-HOST-SECUNDARIO>', {password: '<SENHA-ROOT>'});
```
#### 5- Verifique se o Group Replication está ativado:
```
START GROUP_REPLICATION;
```

cluster.status();
pct enter vmid


SHOW VARIABLES LIKE 'group_replication%';
SET GLOBAL group_replication_group_seeds = '45.224.129.55:3306,177.137.208.34:3306';
START GROUP_REPLICATION;



var cluster = dba.getCluster();
typeof cluster;
cluster.addInstance("root@177.137.208.34:3306");
