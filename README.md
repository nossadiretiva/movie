# Micro Serviço de Filmes

O micro serviço de filmes é uma aplicação construída em ***NodeJS*** que faz a tratativa do cadastro das filmes da aplicação ***Rotten-Potatoes***.

## Estrutura do projeto

Esse projeto trabalha com uma base de dados *MongDB*
![Diagrama micro serviço de reviews](https://github.com/nossadiretiva/imagens/blob/master/diagrama_microservico_movie.png?raw=true)

## Configuração

É preciso configurar a *connection string* pra que o serviço acesse o banco de dados *MongoDB*. Isso pode ser realizado com a variável de ambiente ***MONGODB_URI***

Exemplo:

    MONGODB_URI: mongodb://mongouser:mongopwd@localhost:27017/admin

## Criando a imagem para executar o container

Foi criado dentro do projeto um arquivo  `Dockerfile`  contendo o passo-a-passo (receita de bolo) de criação da imagem  _Docker_  que posteriormente será utilizada para execução da aplicação em um container.

    FROM node:17.3.1
    WORKDIR /app
    COPY ./package*.json ./
    RUN npm install
    COPY . .
    EXPOSE 8181
    CMD [ "node", "./server.js" ]

## Ignorando arquivos desnecessários na criação da imagem

No diretório da aplicação podem existir arquivos indesejados e/ou desnecessários que seriam copiados com o comando  `COPY`  e que poderiam ser ignorados durante o processo de criação da imagem  _Docker_.

Para isso foi criado no mesmo diretório onde se encontram os arquivos do projeto o arquivo  `.dockerignore`  que possui uma lista (com o mesmo padrão do arquivo  `.gitignore`) com os diretórios/arquivos que serão ignorados no momento da execução da cópia dos arquivos para dentro da imagem.

## Executando ambiente da aplicação

Como o ambiente é formado por uma [aplicação web principal](https://github.com/nossadiretiva/rotten-potatoes-ms), um [micro serviço de *review*](https://github.com/nossadiretiva/review) e um [micro serviço de filme](https://github.com/nossadiretiva/movie), a configuração deste ambiente ficou a cargo do *Docker Compose*, que se encontra no diretório da aplicação principal.

Segue abaixo o trecho que configura o micro serviço de *filmes*:

      movie-service:
        build:
          context: ../movie/src
          dockerfile: Dockerfile
        container_name: movie-service
        image: nossadiretiva/movie-service:v1
        ports:
          - 8181:8181
        networks:
          - movie_network
        depends_on:
          - mongodb
        environment:
          MONGODB_URI: mongodb://mongouser:mongopwd@mongodb-movie:27017/admin
    
      mongodb:
        container_name: mongodb-movie
        image: mongo:5.0.5
        ports: 
          - 27017:27017
        networks:
          - movie_network
        volumes:
          - mongodb_movie_vol:/data/db
        environment:
          MONGO_INITDB_ROOT_USERNAME: mongouser
          MONGO_INITDB_ROOT_PASSWORD: mongopwd
