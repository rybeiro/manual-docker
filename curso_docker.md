# Curso Docker

Docker é uma estrutura virtualizada de sistemas em contêiner. O contâneir pode ser considerado um micro serviço. O contâiner encapsula todo o ambiente necessário para executar o sistema, incluindo o código, as bibliotecas e as dependências.

Comandos úteis do docker
- docker ps ou docker container ls ( listas os serviços - containers - em execução )
- docker ps -a ou docker container ls -a ( listas os serviços - containers -  em parados )
- docker image ls ou docker images ( lista todas as imagens )

Como executar imagem do docker
```shell
docker run --rm php:apache
```

- run ( cmd para executar )
- --rm ( instrução que remove o container após parar o serviço )

```shell
docker run --rm -d php:apache
```

- -d ( daemon, que instrui o docker a executar em segundo plano ).


```shell
docker run -name teste-dev -d php:apache
```
- -name ( parâmetro para dar um nome o container )


```shell
docker run -name teste-dev -p 8000:80 php:apache
```

- -p ( Mapeamento e exportar a porta - consiste em abri portas do container )


```shell
docker run -d --name meu-projeto -p 8005:80 -v $(pwd):/var/www/html php:apache
```
- -v ( espelha o diretório local para o container ) Toda alteração efetuada no diretório local refletira no container.


```shell
docker run --name meu-db -e MYSQL_ROOT_PASSWORD=mypass -p 3310:3306 -d mysql
```

dificuldade em comunicar entre os containers

#### Informações detalhadas do container
```shell
docker inspect meu-projeto
docker inspect meu-db
```


#### Configurando e definido redes
```shell
docker network create --driver bridge minha_rede
```

```shell
docker run -d --name meu-projeto -p 8005:80 -v $(pwd):/var/www/html --net minha_rede php:apache
docker run --name meu-db -e MYSQL_ROOT_PASSWORD=mypass -p 3310:3306 -d --net minha_rede mysql
```

#### Entrar no container
```shell
docker exec -it meu-projeto bash
```
- -i (interativo)
- -t (tty ou terminal=true)
- bash (terminal do linux)

#### Remover o container pelo nome ou id 
```shell
docker container rm meu-projeto 
# ou
docker container rm <ContainerId>
```

#### Remover todos os container parados
```shell
docker container prune
```

## Criar minha imagem após fazer alterações no container
```shell
# criar um container com base na imagem do php:apache
docker run -d --name meu-projeto -p 8005:80 -v $(pwd):/var/www/html php:apache

# acessar o container para instalar pacotes necessários ou criar arquivos
# instalar o iputils-ping, vim e wget
docker exec -it meu-projeto bash

# criar uma imagem a partir desse container
docker commit meu-projeto minha-imagem-docker:latest

```

### Dockerfile
Contém uma lista de instruções para criar uma imagem Docker e a partir da imagem criar o container. O uso do Dockerfile é para os casos em que o seu projeto necessita de alguma pacote para funcionar.

docker run -d --name meu-projeto -p 8005:80 -v $(pwd):/var/www/html --net minha_rede php:apache

```shell
# imagem base
FROM php:apache

# executar comandos dentro do container
RUN apt update
RUN apt upgrade -y
RUN apt install wget -y

# Informações do autor
LABEL maintainer 'Fábio Ribeiro <Fabio Ribeiro at rybeiro@gmail.com>'

# Passando argumentos para o container
ARG S3_BUCKET=FABIO_RIBEIRO

# Passando variáveis de ambiente para o container
ENV S3_BUCKET=${S3_BUCKET}

# copia conteúdo para o container
COPY ./ /var/www/html/

# cria um usuario no container
RUN useradd -m -s /bin/bash fabio

# usuario ativo e logado no sistema
USER fabio

# cria o volume que poderá ser acessado por outro container
VOLUME /meus-dados

# diretorio padrao de trabalho
WORKDIR /var/www/html

# expoe a porta
EXPOSE 80 443

# Ponto de entrada quando o container for iniciado
# ENTRYPOINT ["/usr/local/bin/python"]

# comando que será executado logo após
# CMD ["run.py"]

```
# =============================================================================

#### Como criar a imagem com base no Dockerfile

- O primeiro passo é criar a imagem

```shell
docker build -t minha-imagem-dockerfile .
```

- O segundo passo nós já aprendemos, vamos relembrar
```shell
docker run -d --name meu-projeto-com-dockerfile -p 8000:80 -v $(pwd):/var/www/html minha-imagem-dockerfile
```


#### Criando a imagem personalizada
```shell
docker image tag minha-imagem-dockerfile fabiorybeiro/curso-docker:1.0
```

#### Efetuando o login

```shell
docker login --username=fabiorybeiro
```

Entre com a senha: ***
*output:* Login Succeeded

#### Fazendo o push
```shell
docker image push fabiorybeiro/curso-docker:1.0
```

#### Confirmar o upload da imagem
Acesse o site do docker: https://hub.docker.com/u/fabiorybeiro


### docker-compose

É um recurso que facilita a criação e o gerenciamento de sistemas compostos por um ou vários contêineres, permitindo iniciar, parar e escalar todo o aplicativo com um único comando.

Passos para criar um Serviço a partir do docker-compose:

```shell
services:
  webserver:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: webserver
    ports:
      - "8080:80"
    volumes:
      - ./:/var/www/html
    environment:
      - DB_HOST=mysql
      - DB_PORT=3306
      - DB_DATABASE=applications
      - DB_USERNAME=fabio
      - DB_PASSWORD=mypass
    networks:
  		- rede_compartilhada

  mysql:
    image: mysql
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: Cmypass
      MYSQL_DATABASE: applications
      MYSQL_USER: fabio
      MYSQL_PASSWORD: mypass
    networks:
  		- rede_compartilhada
      

networks:
  rede_compartilhada:
    name: rede_compartilhada
``` 

#### Comandos 
O comando _docker-compose build_ é usado para criar ou reconstruir imagens de contêiner Docker com base nos serviços definidos em um arquivo.
```shell
docker-compose build
```

O comando _docker-compose up_ é usado para iniciar os contêineres definidos em um arquivo.
```shell
docker-compose up
```

Para iniciar apenas um contêiner definido no docker-compose.

```shell
docker-compose up webserver
```

Para parar todos os contêineres
```shell
docker-compose stop
```

Para parar um contêiner
```shell
docker-compose stop webserver
```

Para iniciar todos os contêineres
```shell
docker-compose start
```

Para start um contêiner
```shell
docker-compose start webserver
```

Para remover os contêineres
```shell
docker-compose down
```

### Conectar um contêiner externo na mesma rede

```shell
services:
  redis:
    image: redis
      

networks:
  rede_compartilhada
    external: true
    
``` 
