## 1. Fundamentos e Comparativo (O que é Docker e por que usar?)

**Definição:** É uma plataforma que permite empacotar sua aplicação e todas as suas dependências em um container — uma unidade padronizada que funciona da mesma forma em qualquer lugar.

**POR QUE DOCKER?**

- **Consistência:** "Funciona na minha máquina" se torna "funciona em qualquer lugar"

- **Isolamento:** Aplicações não interferem uma na outra

- **Portabilidade:** Mesmo container roda no seu PC, servidor Linux, ou nuvem

- **Eficiência:** Menos overhead que máquinas virtuais

- **Escalabilidade:** Fácil iniciar múltiplas cópias da aplicação


## 2. Conceitos e Arquitetura

**Container:** Um container é uma instância em execução de uma imagem. É como um processo isolado que contém seu código, runtime, bibliotecas e dependências.

**Imagem:** Uma imagem é um template de read-only que define como criar um container. É como um "snapshot" que inclui o sistema operacional base, dependências e código.

**Diferenças principais:**
- Imagem é estática, não muda. Container é uma instância em execução.
- Uma imagem pode criar muitos containers diferentes.
- Imagens são armazenadas em registries (Docker Hub). Containers rodam localmente ou em servidores.

**Dockerfile:** Um Dockerfile é um arquivo de texto que contém instruções para construir uma imagem. É como uma receita.

`
FROM python:3.9-slim WORKDIR /app COPY . . RUN pip install -r requirements.txt CMD ["python", "app.py"]
`

**REGISTRY / DOCKER HUB:** Um registry é um repositório centralizado onde imagens são armazenadas e compartilhadas. Docker Hub é o registry público mais popular.

`
Exemplo: A imagem nginx:latest está no Docker Hub. Qualquer um pode fazer docker run nginx e ela é baixada automaticamente.
`
**Fluxo Básico**

`
1. Você escreve um Dockerfile
2. docker build cria uma Imagem
3. docker run executa um Container a partir da imagem
`
## 3. ARQUITETURA DOCKER

Docker usa uma arquitetura cliente-servidor:

- **Docker Client:** Ferramenta CLI que você usa (docker run, docker build, etc)
- **Docker Daemon:** Serviço que roda em background e gerencia containers, imagens e recursos
- **Registry:**Repositório de imagens (Docker Hub, repositórios privados, etc)

`
**O fluxo é:** você digita um comando no Docker Client, que comunica com o Docker Daemon (o serviço), que por sua vez gerencia os containers em execução e interage com registries quando precisa baixar imagens.
`

## 4. Primeiros Passos Práticos

**Instalação**

docker --version docker run hello-world

**RODANDO SEU PRIMEIRO CONTAINER**

docker run nginx


Para acessar no navegador, use a flag -p para mapear portas:

docker run -p 8080:80 nginxS

`
**O que acontece:**
1. Docker procura a imagem nginx localmente
2. Se não encontrar, baixa do Docker Hub
3. Cria um container a partir da imagem
4. Executa o container
`

`
**COMANDOS ESSENCIAIS**

- **docker run** — Executar container
- **docker build** — Criar imagem a partir de um Dockerfile
- **docker ps** — Listar containers em execução
- **docker ps -a** — Listar todos os containers (incluindo parados)
- **docker images** — Listar imagens locais
- **docker logs CONTAINER_ID** — Ver logs de um container
- **docker exec -it CONTAINER_ID bash** — Entrar em um container em execução
- **docker stop CONTAINER_ID** — Parar um container
- **docker rm CONTAINER_ID** — Remover um container
- **docker rmi IMAGE_ID** — Remover uma imagem
`

## Bildando e rodando imagem

**Build a imagem:**
`
docker build -t meu-app:1.0 .
`
**Executar container**
`
docker run -p 5000:5000 meu-app:1.0
`

## VOLUME (PERSISTÊNCIA DE DADOS)

Containers são efêmeros — quando deletados, os dados dentro são perdidos. Para persistir dados ou compartilhar arquivos entre seu PC e o container, use volumes.

**VOLUME (PERSISTÊNCIA DE DADOS)**
`
docker run -v /caminho/host:/caminho/container nginx

**Caso de uso: Desenvolvimento. Você edita código no seu editor, e o container reflete as mudanças em tempo real. Assim você não precisa reconstruir a imagem toda vez.**
`

## Links Importântes


- [Guia Prático: Dockerizando Spring Boot + AWS](https://medium.com/@amandapazettiperes/guia-pr%C3%A1tico-dockerizando-uma-aplica%C3%A7%C3%A3o-spring-boot-e-banco-de-dados-e-implantando-na-aws-8dba24dcd313)
- [Let's Stop "It Works on My Machine"](https://medium.com/@senaunalmis/lets-stop-the-it-works-on-my-machine-drama-together-a-docker-guide-3642a32df746)
- [Docker: Get Started](https://www.docker.com/get-started/?at_exp=DO105.B-DO106.A)
- [Documentação Oficial do Docker](https://docs.docker.com/?at_exp=DO106.A&_gl=1*1v9ivvu*_gcl_au*MTg5MjYwOTczOS4xNzg3NTg0MDcw*_ga*MTkwMDA1ODY4MC4xNzg3NTg0MDcw*_ga_XJWPQMJYHQ*czE3ODgyNTkwNDkkbzUkZzEkdDE3ODgyNTkwNjYkajQzJGwwJGgw)
- [Docker Hub](https://hub.docker.com/)

