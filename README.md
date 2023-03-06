
# Atividade - AWS - Docker.
- Erik Alexandre Bezerra
- Alex Lopes
- Antonio Bezerra

# Requisitos : 
- instalação e configuração do DOCKER ou 
CONTAINERD no host EC2;
- Ponto adicional para o trabalho utilizar a 
instalação via script de Start Instance 
(user_data.sh)
- Efetuar Deploy de uma aplicação Wordpress 
com: 
  - container de aplicação
  - container database Mysql
- configuração da utilização do serviço EFS 
AWS para estáticos do container de aplicação 
Wordpress
- configuração do serviço de Load Balancer 
AWS para a aplicação Wordpress

# sumário
1. [Preparação do ambiente](#preparação-do-ambiente)
- [Sub-redes](#sub-redes)
- [Gateways da internet](#gateways-da-internet)
- [Gateways NAT](#gateways-nat)
- [Configuração da Tabelas de rotas](#configuração-da-tabelas-de-rotas)
- [Criar par de chaves](#criar-par-de-chaves)
- [Criação das instâncias](#criação-das-instâncias)
- [Criação do Load balance](#criação-do-load-balance)
- [Configuração dos grupos de segurança](#configuração-dos-grupos-de-segurança)
2. [Criando a aplicação Wordpress]()
- [Instalação e configuração do docker](#instalação-e-configuração-do-docker)
- [Instalação do Docker Compose](#instalação-do-docker-compose)
- [Montagem do EFS](#montagem-do-efs)
- [Criação de docker-compose para subir nossa aplicação](#criação-de-docker-compose-para-subir-nossa-aplicação)
- [Subindo os containers](#subindo-os-containers)
- [inicializando aplicação de forma automática](#inicializando-aplicação-de-forma-automatica)

# Preparação do ambiente 

## Sub-redes

1. Acesse o console VPC pelo link https://us-east-1.console.aws.amazon.com/vpc/home
2. Va na seção sub-redes
3. Clique em criar sub-rede
4. Preencha os campos da seguinte forma:

|||
|---|---|
|Nome | aws-controltower-Private |
|Zona de disponibilidade|us-east-1a|
|Bloco CIDR|172.31.64.0/20|

5. Clique em adicionar nova sub-rede e preencha da seguite forma:

|||
|---|---|
|Nome | aws-controltower-Publica |
|Zona de disponibilidade|us-east-1a|
|Bloco CIDR|172.31.0.0/20|

# **[*Gateways da internet*](https://docs.aws.amazon.com/pt_br/vpc/latest/userguide/VPC_Internet_Gateway.html)**
Um gateway da Internet é um roteador virtual que conecta uma VPC à Internet.

Para realizar a criação de seu Gateway da internet siga os seguintes passos:
1. Acesse o console do serviço de VPC por meio do link https://us-east-1.console.aws.amazon.com/vpc/;
2. Clique em `Gateways da internet`;
3. Clique em `criar Gateways da internet`;
4. De o nome `GatewayErik`;
5. Clique em `criar Gateway de internet`;

## Gateways NAT

1. Clique em `Gateways NAT`;
2. Clique em `Criar gateway NAT`;
3. De o nome `NATGateway`;
4. Selecione sua sub-rede pública;
5. Em tipo de conectividade selecione `Público`;
6. Clique em `alocar IP elástico`;
7. Por fim clique em `criar gateway NAT`.


# Configuração da **[*Tabelas de rotas*](https://docs.aws.amazon.com/pt_br/vpc/latest/userguide/VPC_Route_Tables.html)**
Precisaremos editar a tabela de rotas para cada sub-rede que criamos.

### Sub-rede pública
1. Clique em `tabelas de rotas`;
2. Selecione primeiramente a tabela de rotas de sua sub-rede pública;
3. Clique em `ações` e selecione editar rotas;
4. Na página de edição clique em `adicionar rota`;
5. Preencha os campos da seguinte forma:

| Destino | Alvo  |
| ---     | ---   |
|0.0.0.0/0|GatewayErik|

6. Clique em `salvar alterações`

### Sub-rede privada

1. Clique em `tabelas de rotas`;
2. Selecione primeiramente a tabela de rotas de sua sub-rede pública;
3. Clique em `ações` e selecione editar rotas;
4. Na página de edição clique em `adicionar rota`;
5. Preencha os campos da seguinte forma:

| Destino | Alvo  |
| ---     | ---   |
|0.0.0.0/0|NATGateway|

6. Clique em `salvar alterações`

# Criar **[*par de chaves*](https://docs.aws.amazon.com/pt_br/AWSEC2/latest/UserGuide/ec2-key-pairs.html)**

Para criar o par de chaves siga os seguintes passos:

1. Acesse o serviço Ec2 pelo console da AWS por meio do link https://us-east-1.console.aws.amazon.com/ec2; 
2. No tópico rede e segurança clique em `pares de chaves`;
3. Clique em `criar par de chaves`;
4. Preencha os campos da seguinte forma:

Nome
```
ChaveErik
```
Tipo de par de chaves
```
RSA
```
Formato de arquivo de chave privada
```
.pem
```

# Criação das instâncias
Neste ponto iremos realizar a criação de duas instâncias uma em cada sub-rede que criamos.

### Instância da aplicação Wordpress
1. Acesse o console de serviços ec2;
2. Clique em `instâcias`;
3. Clique em `executar instâncias`;
4. Agora preencha os campos da seguinte forma:

TAGS
| Chave | Valor  |
| ---     | ---   |
|Name|Wordpress|
|Project|PB|
|CostCenter|PBCompass|

Imagens de aplicação e de sistema operacional

```
Amazon Linux 2 AMI
```

Tipo de instância

```
t3.small
```
Nome do par de chaves
```
ChaveErik
```

### Configurações de rede
 
1. Clique em editar;
2. Selecione sua sub-rede privada;
3. Mantenha desabilitado o IP público;
4. Em firewall selecione o grupo de segurança `Default`.



### Instância que fará o acesso a nossa aplicação


TAGS
| Chave | Valor  |
| ---     | ---   |
|Name|Bastion|
|Project|PB|
|CostCenter|PBCompass|

Imagens de aplicação e de sistema operacional

```
Amazon Linux 2 AMI
```

Tipo de instância

```
t3.small
```
Nome do par de chaves
```
ChaveErik
```

### Configurações de rede
 
1. Clique em editar;
2. Selecione sua sub-rede pública;
3. Habilite o IP público;
4. Em firewall crie um novo grupo de segurança com o nome Bastion.



# Criação do Load balance

1. Na seção Balanceamento de carga clique em `grupos de destino`;
2. Clique em `Criar grupo de destino`;
3. No campo nome preencha com `Target`;
4. Mantenha a configuração padrão;
5. Clique em `próximo`;
6. Em `Registrar destinos` selecione sua instâcia Wordpress;
7. Clique em `incluir como pendente abaixo`;
8. Clique em `criar grupo de destino`.

Retornando para seção Balanceamento de carga

1. Clique em `Load balancers`;
2. Clique em criar `Application Load Balancer`;
3. De o nome `LoadBalancer`;
4. Em `Esquema` deixe marcado como Voltado para a Internet;
5. Em `Tipo de endereço IP` selecione IPv4;
6. Selecione sua VPC;
7. Em `Mapeamentos` marque as zonas de suas sub-redes;
8. Crie um novo grupo de segurança com o nome `SGwordpress`;
9. Selecione o grupo de destino criado anteriormente;
10. Por fim clique em `criar load balancer`.

# Configuração dos grupos de segurança
Após realizados todos os passos, temos criado no nosso ambiente 3 grupos de segurança. Agora iremos realizar a configuração das portas para que tudo funcione corretamente.

1. vá para seção `Rede e segurança`;
2. Selecione `Security groups`;
3. Selecione cada um dos grupos e clique em editar regras de entrada;
4. Deixa as regras de entrada de cada grupo da seguinte forma:

- Grupo de segurança do Bastion

|Porta|Protocolo|Origem|
|---|---|---|
|22222|TCP|"Meu-Ip"|

- Grupo de segurança do balanceador de carga

|Porta|Protocolo|Origem|
|---|---|---|
|80|TCP|0.0.0.0/0|

- Grupo de segurança da aplicação

|Porta|Protocolo|Origem|
|---|---|---|
|22|TCP|Grupo de segurança do Bastion Host|
|2049|TCP|172.31.0.0/16|
|2049|TCP|172.31.0.0/16|
|80|TCP|Grupo de segurança do balanceador de carga

Após realizar estás configurações teremos o ambiente pronto para que possamos construir nossa aplicação.

## Acessando nossa instância Wordpress
Agora iremos acessar a nossa instância Wordpress utilizando o bastion como ponte.

Vamos utilizar o ssh-agent para conseguirmos acessar a instância privada sem necessitar copiar a chave de acesso para dentro do bastion. Então, na sua máquina local execute o seguinte:
```
ssh-agent # Executando agente SSH
ssh-add "NomeDaChave.pem" # Adicionando chave ao agente
```
Com isso podemos acessar a chave de acesso dentro do Bastion. Vamos acessar o bastion utilizando o seguinte comando:

```
ssh -A -i "NomeDaChave.pem" ec2-user@ip-bastion -p 22222 # Acessando instância e encaminhando chave para o bastion 
```
Dado que estamos acessando o bastion, podemos acessa a instância da aplicação. Então, como já foi copiado a chave pelo agente ssh, podemos acessar a instância da aplicação pelo seguinte:

```
ssh ec2-user@ip-privado-wp # Acessando instância da aplicação utilizando IP privado
```
## Instalação e configuração do docker
Para realizar a instalação do docker execute os seguintes comandos:
```
#atualizar os pacotes para a última versão
sudo yum update -y
#instalar o docker
sudo yum install docker
#iniciar o serviço do docker
sudo systemctl start docker
#habilitar o serviço do docker para iniciar automaticamente
sudo systemctl enable docker
#adicionar o usuário ec2-user ao grupo docker
sudo usermod -a -G docker ec2-user

```
## Instalação do Docker Compose
Para realizar a instalação do docker-compose execute os seguintes comandos:

```
# baixar o docker-compose para a pasta /usr/local/bin
sudo curl -L "https://github.com/docker/compose/releases/download/v2.15.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# dar permissão de execução ao binário do docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## Montagem do EFS

```
# criar o diretório para o EFS
mkdir -p /mnt/nfs
# adicionar o EFS no fstab
echo "IP_OU_DNS_DO_NFS:/ /mnt/nfs nfs defaults 0 0" >> /etc/fstab
# montar o EFS
mount -a

```

## Criação de docker-compose para subir nossa aplicação
Dentro de sua instância crie um arquivo docker-compose.yml e cole o seguinte trecho:

```
version: '3.4'
services:
  db:
    image: mysql:5.7.22 #imagem do banco de dados MySQL
    command: mysqld --default_authentication_plugin=mysql_native_password
    restart: always #ativando a reinicialização automatica em caso de algum erro
    environment: #variáveis de ambiente do MySQL
      TZ: America/Sao_Paulo
      MYSQL_ROOT_PASSWORD: docker
      MYSQL_USER: docker
      MYSQL_PASSWORD: docker
      MYSQL_DATABASE: wordpress
    ports:
      - "3306:3306"
    networks:
      - wordpress-network  #rede em que o serviço db será executado
    volumes:
      - /mnt/nfs/mysql_data:/var/lib/mysql #permitindo que os dados do MySQL sejam persistentes
  wordpress:
    image: wordpress:latest #imagem do Wordpress
    restart: always 
    environment: #variáveis de ambiente do Wordpress
      TZ: America/Sao_Paulo
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: docker
    ports:
      - 80:80
    volumes:
      - /mnt/nfs/wordpress:/var/www/html #permitindo que os dados do Wordpress sejam persistentes
    depends_on: 
      - db  
    networks:
      - wordpress-network
networks:
  wordpress-network:
    driver: bridge

```

## Subindo os containers
Execulte o seguinte comando:
```
docker-compose -f docker-compose.yml up -d
```
# inicializando aplicação de forma automática
Para que não seja necessário ficar repetindo sempre os passos para a instalação, realize os seguintes passos:


1. Quando estiver realizando a criação da instância;
2. Role a página até dados avançados;
3. Em dados do usuário copie e cole os seguintes comandos:

Para a instância Wordpress
```
#!/bin/bash

# Instalação e configuração do Docker
yum update -y
yum install docker -y
systemctl start docker
systemctl enable docker
usermod -a -G docker ec2-user

# Instalação do docker-compose
curl -L "https://github.com/docker/compose/releases/download/v2.15.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

# Montagem do efs
mkdir -p /mnt/nfs
echo "172.31.11.203:/ /mnt/nfs nfs defaults 0 0" >> /etc/fstab
mount -a

# Executando o docker-compose do repositorio
yum install git -y
git clone https://github.com/Alexandreerik/Compass.uol-PB-Atv3.git /home/ec2-user/atividade_aws_docker
docker-compose -f /home/ec2-user/Compass.uol-PB-Atv3/docker-compose.yml up -d
```

Para a instância Bastion

```
#!/bin/bash

yum update -y

# Configuração da porta SSH
echo "Port 22222" >> /etc/ssh/sshd_config
systemctl restart sshd.service

```
Seguindo esses passos, ao acessar o endereço fornecido pelo Load balance na rede, a aplicação Wordpress já estará funcionando.

