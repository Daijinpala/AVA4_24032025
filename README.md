# Projeto:
```
1. Instalação e configuração do DOCKER ou CONTAINERD no host EC2; SEGUIR DESENHO TOPOLOGIA DISPOSTA. `Ponto adicional para o trabalho utilizar a instalação via script deStart Instance (user_data.sh)`

2. Efetuar Deploy de uma aplicação Wordpress com: container de aplicação RDS database Mysql. 

3. Configuração da utilização do serviço EFS AWS para estáticos do container de aplicação Wordpress .

4. Configuração do serviço de Load Balancer AWS para a aplicação Wordpress.

```
<hr>

## Estágios do Projeto de forma resumida:

1. Rodar o wordpress local ✅
2. Criar a VPC, EC2 ✅
3. Criou o RDS ✅
4. Instalou o Docker na EC2 ✅
5. Rodou o Wordpress na EC2 ✅
6. Criou um script de inicialização no User Data e o testou ✅
7. Criou o auto-scaling group e balanceador de Carga ✅
8. Criou regras de scaling ✅
9. Monitoramento no Cloudwatch ✅

![1](png/print.png)

### Pontos de atenção:

- Não utilizar ip público para saída do serviços WP (Evitem publicar o serviço WP via IP Público) 
- Sugestão para o tráfego de internet sair pelo LB (Load Balancer Classic)
- Pastas públicas e estáticos do wordpress sugestão de utilizar o EFS (Elastic File Sistem)
- Fica a critério de cada integrante usar Dockerfile ou Dockercompose;
- Necessário demonstrar a aplicação wordpress funcionando (tela de login) 
- Aplicação Wordpress precisa estar rodando na porta 80 ou 8080;
- Utilizar repositório git para versionamento; 
- Criar documentação.

## Conclusão:

### Primeira etapa `local`: Criação do ambiente de testes.

<div>
<details align="left">
    <summary></summary>
1 - Baixe o wsl (Pela loja da Microsoft || Por linha de comando).

```
wsl --install
```


2 - Instale a versão mais atual do ubuntu (Pela loja da Microsoft || Por linha de comando).

```
wsl --install -d Ubuntu-24.04
```

3 - Instale o um editor de código de sua preferência (VScode).

```
https://code.visualstudio.com/download
```

4 - Faça um conexão ao wsl.

![2](png/base.png)

```
Assim que você entrar no vscode já com o wsl instalado ele vai te recomendar que baixe uma extensão chamada wsl, após a instalação você podera clicar no simbolo >< no canto inferior esquerdo e após isso é só apertar em "Connect to WSL" e estará dentro da maquina !! 
```

5 - Atualize a maquina.

```
sudo apt update && sudo apt upgrade -y
```

6 - Instale o docker && docker compose.

```
1- sudo apt install -y ca-certificates curl gnupg

2- sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

3- echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

4- sudo apt update

5- sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

6- sudo usermod -aG docker $USER

7- sudo systemctl enable docker
sudo systemctl start docker

----------------------------------------------------------------------------------------

1- Instale as dependências necessárias.
2- Adicione a chave GPG oficial do Docker.
3- Adicione o repositório do Docker ao APT.
4- Atualize novamente.
5- Instale o Docker && o docker compose.
6- Adicione seu usuário ao grupo `docker` (Opcional).
7- Habilite o Docker para iniciar com o sistema (Opcional).
```
</div>

### Segunda etapa `local`: Teste de implementação do *Wordpress*.

<div>
<details align="left">
    <summary></summary>
1-  Criar uma abstração dos volumes e redes (Opcional).

```
----------------------------------------------------------------------------------------

|- NETWORK
|-- NAME 'tunel'

----------------------------------------------------------------------------------------

|- VOLUMES
|-- NAME wordp
|-- NAME dbm

----------------------------------------------------------------------------------------

Projeto: foi definido que o nome da rede será 'tunel' e que será criado dois volumes, um para armazenar os arquivos do site(wordp) e o outro, arquivos referentes ao banco de dados(dbm).
```

2- Criar volumes para arquivos do site e do banco de dados.

```
1- docker volume create wordp

2- docker volume create dbm

3- docker volume ls

----------------------------------------------------------------------------------------

1- Cria um volume para o Wordpress.
2- Cria um volume para o Banco de dados mysql.
3- Listas os volumes.
```


3- Criar uma rede para permitir uma conexão entre o banco e o site.

```
1- docker network create tunel

2- docker network ls

----------------------------------------------------------------------------------------

1- Cria uma rede com o nome 'tunel'.
2- Lista as redes.
```

4- Criar pastas para armazenar arquivos referentes ao projeto.

```
1- mkdir projetinho
2- cd projetinho

----------------------------------------------------------------------------------------

1- Cria uma diretório com o nome 'projetinho'
2- Entra no diretório especificado (Por mais leigo que seja, fazer uma estrutura para um projeto é essencial).
```

5- Checar a documentação  do docker-hub

```
https://hub.docker.com/_/wordpress
```

6- Criar uma abstração do banco de dados (Opcional).

```
|- DB NAME 'projetinho'
|-- DB USER 'cariani'
|-- DB PASSWORD '0311'
```

7- Criar um compose seguindo a documentação acima.

nano `docker-compose.yml`: 
```
services:
  web:
    image: wordpress
    restart: always
    ports:
      - "80:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: cariani
      WORDPRESS_DB_PASSWORD: 0311
      WORDPRESS_DB_NAME: projetinho
    volumes:
      - wordp:/var/www/html
    networks:
      - tunel

  db:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_DATABASE: projetinho
      MYSQL_USER: cariani
      MYSQL_PASSWORD: 0311
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - dbm:/var/lib/mysql
    networks:
      - tunel

networks:
  tunel:
    driver: bridge

volumes:
  wordp:
  dbm:

```

8- Executar o compose, e após o teste apagar ele.

```
1- docker compose up -d

2- docker compose down

----------------------------------------------------------------------------------------
 
1- Executa o arquivo 'docker-compose.yml'
2- Exclui os containers gerados(Caso não tenha criado os volumes e a rede antes de executar, o mesmo irá criar as redes serão excluidas más os volumes permanecerão)
```

9- Resultado.

![3](png/Result.png)

<br>

![4](png/posinst.png)
</div>

### Primeira etapa `AWS`: Criação da topologia da rede e seus grupos de segurança com teste sem ALB && ASG.

<div>
<details align="left">
    <summary></summary>
1- Criar a VPC

![5](png/vcp.png)

2- Criar os grupos de segurança.

![6](png/gpsg.png)

3- Criar o banco de dados(RDS).

```
Especificações: 

- RDS com MySQL, sem Multi-AZ e instâncias db.t3.micro

----------------------------------------------------------------------------------------

|- NAME DB 'db-wordpress'
|-- NAME USER 'flavor'
|-- PASSWORD '998049352'
|--- DATABASE NAME 'db_projetinho'

```
![7](png/rds_mysql.png)

![8](png/rds_gratuito.png)

![9](png/rds_config.png)


```
Após a criação do banco de dados, RDS > escolha o seu banco > security > altere a regra de entrada pro grupo de segurança que vai estar sua ec2.
```

4- Criar uma EC2.

![10](png/ec2_image.png)

![11](png/ec2_network.png)

![12](png/ec2_userdata.png)


Documento utilizado no`userdata`:
```
#!/bin/bash

echo "Atualizando repositórios e instalando atualizações"
sudo yum update -y

sudo yum install docker -y

sudo yum install wget -y

sudo yum install curl -y

sudo yum install amazon-efs-utils -y

echo "Repositórios e atualizações instalados"
 
echo "Habilitando Docker como serviço e passando usuário pro grupo docker"
sudo service docker start
sudo systemctl enable docker.service
sudo usermod -aG docker ec2-user
echo "Docker habilitado e usuário passado para o grupo docker"

echo "Testando instalação Docker"
docker --version
echo "Docker testado"

echo "Instalando Docker compose" 
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
echo "Instalação feita"

echo "Permissionamento do docker-compose sendo executado"
sudo chmod +x /usr/local/bin/docker-compose
echo "Permisisonamento feito"

echo "Criação de pasta wordpress"
sudo mkdir wordpress
echo "Pasta criada COM SUCESSO"

echo "MONTAGEM SENDO FEITA NO EFS"
sudo mount -t efs -o tls fs-06887e858d43acc91:/ wordpress
echo "MONTAGEM FEITA COM SUCESSO"

echo "Requisitando arquivo docker-compose.yml"
wget https://raw.githubusercontent.com/Daijinpala/AVA4_24032025/refs/heads/main/POTATO%20SCRIPT/docker-compose.yml
echo "Arquivo docker-compose.yml requisitado"

echo "Executando docker-compose up"
cd /home/ec2-user/docker-compose.yml
sudo docker-compose up -d
echo "Docker compose executado corretamente!"
```

5- Entar na EC2 via ssh.

6- Baixar o *mysql-client* para testar a conectividade com o banco de dados

```
sudo apt install -y mysql-client

----------------------------------------------------------------------------------------

mysql -h [ENDEREÇO_DO_BANCO] -u [USUÁRIO] -p -e "SHOW DATABASES;"
```

nano `docker-compose.yml`: 
```
services:
  web:
    image: wordpress
    restart: always
    ports:
      - "80:80"
    environment:
      WORDPRESS_DB_HOST: db-wordpress.c98i000mqf2o.us-east-1.rds.amazonaws.com
      WORDPRESS_DB_USER: flavor
      WORDPRESS_DB_PASSWORD: 998049352
      WORDPRESS_DB_NAME: db_projetinho
    networks:
      - tunel

networks:
  tunel:
    driver: bridge
```

![13](png/rds_fim.png)

</div>

### Segunda etapa `AWS`: Aplicação do auto-scaling group com balanceador de carga, regras de scaling e CloudWatch.

<div>
<details align="left">
    <summary></summary>
1- Criar uma `VPC` (apagar todas as outras existentes para maior facilidade).

![14](png/vpc.png)

**IMPORTANTE**: Acessar a subnetes publicas e colocar para elas o ipv4 publico automaticamente.

2- Criar um grupo de segurança para os servidores web.

`sg_webservers:`
![15](png/sgweb-entrada.png)
![16](png/sgweb-saida.png)

3- Criar o grupo de segurança RDS.

`sg_mysql:`
![f1](png/rds-entrada.png)
![f2](png/rds-saida.png)


4- Criar o banco de dados(RDS).

```
Cria do mesmo jeito que o teste anterior.
```

5- Cria um target group para o `ALB`.

![17](png/tg-alb.png)

6- Criar um grupo de segurança para o `ALB`.

`sg_alb`:
![18](png/sgalb-entrada.png)
![19](png/sgalb-saida.png)

7- Criar um `Launch template`.

![20](png/lt_1.png)
![21](png/lt_2.png)

`Userdata` utilizado:
```
#!/bin/bash

echo "Atualizando repositórios e instalando atualizações"
sudo yum update -y

sudo yum install docker -y

sudo yum install wget -y

sudo yum install curl -y

sudo yum install amazon-efs-utils -y

echo "Repositórios e atualizações instalados"
 
echo "Habilitando Docker como serviço e passando usuário pro grupo docker"
sudo service docker start
sudo systemctl enable docker.service
sudo usermod -aG docker ec2-user
echo "Docker habilitado e usuário passado para o grupo docker"

echo "Testando instalação Docker"
docker --version
echo "Docker testado"

echo "Instalando Docker compose" 
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
echo "Instalação feita"

echo "Permissionamento do docker-compose sendo executado"
sudo chmod +x /usr/local/bin/docker-compose
echo "Permisisonamento feito"

echo "Criação de pasta wordpress"
sudo mkdir wordpress
echo "Pasta criada COM SUCESSO"

echo "MONTAGEM SENDO FEITA NO EFS"
sudo mount -t efs -o tls fs-06887e858d43acc91:/ wordpress
echo "MONTAGEM FEITA COM SUCESSO"

echo "Requisitando arquivo docker-compose.yml"
wget https://raw.githubusercontent.com/Daijinpala/AVA4_24032025/refs/heads/main/POTATO%20SCRIPT/docker-compose.yml
echo "Arquivo docker-compose.yml requisitado"

echo "Executando docker-compose up"
cd /home/ec2-user/docker-compose.yml
sudo docker-compose up -d
echo "Docker compose executado corretamente!"
```

![22](png/lt_3.png)

8- Faça um `ASG` com o `ALB`

![23](png/asg-1.png)

<hr>

![24](png/asg-2.png)

<hr>

![25](png/asg-3.png)
![26](png/asg-3.1.png)
![27](png/asg-3.2.png)

<hr>

![28](png/asg-4.png)
![29](png/asg-4.1.png)
![30](png/asg-4.2.png)
![31](png/asg-4.3.png)

<hr>

![31](png/asg-6.png)

<hr>

9- Verifique se foi criada as instancias.

![32](png/f1.png)

10- Tente acessar o Wordpress pelo IP publico das maquinas.

EC2 1 IP: 54.86.187.15

![33](png/f2.png)

EC2 2 IP: 3.89.221.227

![34](png/f3.png)

11- Tente entrar pelo DNS do `LB`.

![35](png/f4.png)

12- Analisar os dados no `CloudWatch`.

![35](png/cloudwatch.png)

</div>

### EFS on Linux

<div>
<details align="left">
    <summary></summary>

1- Criar o grupo de segurança para o EFS

![00](png/efs-entrada.png)
![01](png/efs-saida.png)

2- Criar um EFS.

![02](png/efs-cri-2.png)
![03](png/efs-cri-2.1.png)


<hr>

![04](png/efs-cri-3.png)
![05](png/efs-cri-fim.png)

<hr>
3- Montagem manualmente da pasta

![06](png/mount-1.png)
![07](png/mount-2.png)

4- Entrar na EC2 via ssh

5- Instale os pacotes necessários

Documentação Linux: https://docs.aws.amazon.com/pt_br/efs/latest/ug/using-amazon-efs-utils.html
Documentação oficial para outras distribuições: https://docs.aws.amazon.com/pt_br/efs/latest/ug/installing-amazon-efs-utils.html (não funciona, testei somente na distribuição do ubuntu)

no `Linux`:
```
sudo yum install amazon-efs-utils -y
```

![08](png/dw-efs.png)

no `Ubuntu`:
```
$ sudo apt-get update
$ sudo apt-get -y install git binutils rustc cargo pkg-config libssl-dev gettext
$ git clone https://github.com/aws/efs-utils
$ cd efs-utils
$ ./build-deb.sh
$ sudo apt-get -y install ./build/amazon-efs-utils*deb
```

6- Monte uma pasta

```
Crie uma pasta para fazer a montagem

mkdir wordpress

Cole o que copiamos do nosso efs para um montagem manual

Exemplo: sudo mount -t efs -o tls fs-06887e858d43acc91:/ wordpress
```

![09](png/efs-mount-ec2.png)

7- Execute o Wordpress e seja feliz

nano `docker-compose.yml`:
```
services:
  web:
    image: wordpress
    restart: always
    ports:
      - "80:80"
    environment:
      WORDPRESS_DB_HOST: 
      WORDPRESS_DB_USER: flavor
      WORDPRESS_DB_PASSWORD: 998049352
      WORDPRESS_DB_NAME: db_projetinho
    volumes:
      - /home/ec2-user/wordpress:/var/www/html
    networks:
      - tunel

networks:
  tunel:
    driver: bridge
```

comando pra executar o container 
```
Linux:
docker-compose up -d

---------------------------------------------------------
Ubuntu:
docker compose up -d
```
Antes de executar o wordpress:
![010](png/teste-efs-1.png)

<hr>

Executando a instalação do wordpress:
![011](png/PósInst.png)

No monitoramento:
![012](png/monitoraefs.png)

Conteúdo na pasta:
![013](png/conteudopasta.png)

- Testando em uma Ec2 em outra região

Adicione a região nas configurações de rede do EFS:
![014](png/testeec2azB.png)

Após isso é só entrar na outra ec2, criar uma pasta com o mesmo nome e por fim entrar nela e verificar se o conteúdo está lá:
![015](png/férias.png)

</div>
