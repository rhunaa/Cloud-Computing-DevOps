Global Solution — Cloud Computing & DevOps

### Trilha A — Deploy com Docker (Multi-Containers)

## Integrantes

  Bruna Pereira Cordeiro — RM: 563153
  Lucas Simões Silva — RM: 561436 
  Milena Beck Speria — RM: 565077
  Isabella Siqueira — RM: 563973

## Aplicação SPA — Carreiras do Futuro com IA

Aplicação estática servida via Nginx, utilizando MySQL 8 como banco de dados, ambos executando em containers Docker e orquestrados via Docker Compose.

## Descrição do Ambiente

Este projeto realiza o deploy de uma aplicação SPA (Single Page Application) construída na disciplina de RWD e alinhada ao tema: Trabalho com Inteligência Artificial: carreiras, competências e tendências.

O ambiente utiliza:

    Container 1 – Frontend (Nginx + SPA):
    Servido via Nginx
    Arquivos copiados para /usr/share/nginx/html

    Container 2 – Banco de Dados MySQL 8
    Base gs_ai criada automaticamente
    Tabela carreiras definida via schema.sql
    Dados carregados via seed.sql
    Permite consultas de validação
    Orquestração via Docker Compose

## Estrutura do Repositório

/
|-- app/       # Arquivos da SPA (HTML, CSS, JS)
|-- ops/       # Dockerfile e docker-compose.yml
|-- db/        # schema.sql e seed.sql
|-- README.md  # Documentação principal


## Descrição da Solução

O deploy consiste em:

Construir uma imagem Nginx personalizada contendo os arquivos da SPA

Subir um container MySQL com schema + seed

Iniciar todo o ambiente com docker compose

Acessar a SPA via http://localhost:8080

A aplicação exibe conteúdos sobre carreiras em Inteligência Artificial, reforçando a temática da Global Solution.

## Pré-requisitos

Antes de executar o ambiente, instale:

✔ Docker
✔ Docker Compose

## Arquivos Importantes

  --- Dockerfile – (ops/Dockerfile)

FROM nginx:stable
COPY ./app /usr/share/nginx/html

  --- docker-compose.yml – (ops/docker-compose.yml)

version: "3.9"

services:
  web:
    build:
      context: ..             
      dockerfile: ops/Dockerfile
    container_name: spa_frontend
    ports:
      - "8080:80"
    depends_on:
      - db

  db:
    image: mysql:8
    container_name: spa_database
    environment:
      MYSQL_ROOT_PASSWORD: senha
      MYSQL_DATABASE: gs_ai
    volumes:
      - dbdata:/var/lib/mysql
      - ../db:/docker-entrypoint-initdb.d
    ports:
      - "3306:3306"

volumes:
  dbdata:

   --- schema.sql – (db/schema.sql)
CREATE TABLE carreiras (
    id INT AUTO_INCREMENT PRIMARY KEY,
    titulo VARCHAR(255) NOT NULL,
    area VARCHAR(255),
    demanda VARCHAR(100)
);

   --- seed.sql – (db/seed.sql)
INSERT INTO carreiras (titulo, area, demanda) VALUES
('Engenheiro de Ética em IA', 'Tecnologia e Inovação', 'Alta'),
('Especialista em Prompt Engineering', 'IA Generativa', 'Alta'),
('Analista de Automação Inteligente', 'Processos e Robótica', 'Média');


## Passo a Passo do Deploy (Comandos Obrigatórios)

1- Preparar o ambiente

Certifique-se de ter instalado no seu computador:

Docker
Docker Compose

 2- Navegar até a pasta do projeto

No terminal, entre na pasta onde está o docker-compose.yml:

cd ~/SeuLocal/SuaPasta/ops

cd ~/Desktop/Cp-Cloud/ops

Obs.: A pasta ops deve conter o docker-compose.yml e o Dockerfile.

 3- Construir a imagem do frontend

docker compose build
Este comando cria a imagem personalizada do Nginx com a SPA dentro.

 4 - Subir os containers

docker compose up -d

O parâmetro -d faz com que os containers rodem em modo background.

 5 -  Verificar se os containers estão ativos

docker compose ps

Você deve ver algo assim (os nomes podem variar):
Name	Command	State	Ports
spa_frontend	nginx -g ...	Up	0.0.0.0:8080->80/tcp
spa_database	docker-entrypoint	Up	0.0.0.0:3306->3306/tcp

✅ Se aparecer "Up" em ambos, os containers estão rodando corretamente.

 6 - Validar a SPA no navegador

Abra o navegador e acesse:
http://localhost:8080

 7 - Conectar ao banco MySQL

 Usando o MySQL CLI direto no terminal:

docker exec -it spa_database mysql -uroot -psenha

 8 - Validar as tabelas e dados
No cliente SQL ou MySQL CLI, execute os comandos:

USE gs_ai;
SHOW TABLES;
SELECT * FROM carreiras;

Saída esperada do SELECT * FROM carreiras:
id	titulo	area	demanda
1	Engenheiro de Ética em IA	Tecnologia e Inovação	Alta
2	Especialista em Prompt Engineering	IA Generativa	Alta
3	Analista de Automação Inteligente	Processos e Robótica	Média

✅ Se os dados aparecerem, o seed foi aplicado corretamente.

 9 - Parar os containers (quando terminar)
docker compose down

Para remover volumes também (caso queira reiniciar do zero):
docker compose down -v

## Como Validar a Aplicação

✔ 1. Acessar a SPA

    Abra no navegador: http://localhost:8080

✔ 2. Validar o Banco de Dados
    
    Comandos SQL de validação:
    USE gs_ai;
    SHOW TABLES;
    SELECT * FROM carreiras;

## Solução de Problemas Comuns

O MySQL não inicia
✔ Verifique se a porta 3306 já está em uso
✔ Apague o volume e tente novamente:
docker volume rm ops_dbdata

A SPA não aparece
✔ Verifique se o container spa_frontend está ativo
✔ Cheque se os arquivos estão realmente em /usr/share/nginx/html
✔ Rodar:
docker logs spa_frontend

O banco não recebe o seed
✔ Garanta que o volume não estava criado antes
✔ Remova os containers e volumes:
docker compose down -v
docker compose up -d


## Diagrama de Arquitetura

Usuário → Navegador → Container Nginx (SPA) → Docker Network → Container MySQL

Descrição dos Componentes:

Usuário: acessa a aplicação via navegador.
Navegador: envia requisições HTTP para o Nginx.
Nginx (Container Web): hospeda a SPA.
Docker Network: permite comunicação entre containers.
MySQL (Container DB): armazena dados das carreiras de IA e executa consultas.

                        Usuário
                           │
                           ▼
                      Navegador
                           │ 8080
                           ▼
          ┌────────────────────────────────────────┐
          │        Docker Compose  (Bridge)        │
          └───────────┬────────────────────────────┘
                      │
        ┌─────────────┴──────────────┐
        │                            │
        ▼                            ▼
┌────────────────────┐     ┌──────────────────────┐
│  Container Nginx   │     │   Container MySQL    │
│   spa_frontend     │     │    spa_database      │
│  Servindo a SPA    │     │ Banco gs_ai          │
└────────────────────┘     └──────────────────────┘



## Vídeo no YouTube

https://youtu.be/hZTnwQynAOc?si=nygGpEQLCq77mZxK

## Github

https://github.com/rhunaa/Cloud-Computing-DevOps

## Conclusão

Este projeto demonstra o domínio de conceitos essenciais de Cloud Computing & DevOps, incluindo:

Conteinerização com Docker
Orquestração multi-containers
Servidor web com Nginx
Banco de dados MySQL com schema + seed automatizados
Organização e documentação profissional
Validação do ambiente completo
A solução reflete o tema Trabalho com Inteligência Artificial, mostrando como aplicações modernas podem ser preparadas, empacotadas e publicadas em ambientes cloud-ready.


