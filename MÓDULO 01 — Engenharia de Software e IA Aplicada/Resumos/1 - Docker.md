# Docker

## 1. Fundamentos e Comparativo (O que é Docker e por que usar?)

### O problema que o Docker resolve

Quase todo bug de "funciona na minha máquina" tem a mesma origem: o ambiente de execução é diferente do ambiente de desenvolvimento. A versão do Python muda, uma biblioteca de sistema operacional está ausente, uma variável de ambiente foi configurada à mão meses atrás e ninguém lembra. Você não percebe essas diferenças enquanto o código roda no seu computador. Elas aparecem no CI, no servidor de homologação ou na máquina do colega novo, quando já custam horas.

Docker resolve isso empacotando a aplicação junto com tudo o que ela precisa para rodar (interpretador, bibliotecas, arquivos de configuração) em uma unidade chamada **imagem**. Quem executa a imagem recebe o mesmo ambiente, independentemente da máquina.

É importante entender o que Docker **não** faz. Um container compartilha o kernel do sistema operacional hospedeiro. Num Linux, isso significa que um container é um processo isolado (com namespaces e cgroups), não uma máquina virtual completa. Por isso ele inicia em segundos e consome pouca memória. Em macOS e Windows, o Docker Desktop roda uma VM Linux por baixo, e os containers rodam dentro dela. O isolamento também não é uma fronteira de segurança suficiente sozinho: uma imagem vulnerável continua vulnerável dentro de um container.

> **Nota**
>
> A frase "funciona em qualquer lugar" é verdadeira para a mesma arquitetura de processador. Uma imagem construída em um Mac com chip ARM não roda igual num servidor x86 sem uma flag de plataforma (`--platform linux/amd64`). Ao montar pipelines, isso aparece.

## 2. Conceitos e Arquitetura

### Imagens, containers e camadas

São três conceitos que aparecem o tempo todo e são facilmente confundidos.

Uma **imagem** é um pacote imutável, formado por camadas empilhadas. Cada instrução do Dockerfile que altera o sistema de arquivos gera uma camada. Um **container** é uma instância em execução de uma imagem: o Docker adiciona uma camada gravável por cima das camadas da imagem, e é ali que ficam os arquivos criados durante a execução. Quando o container é removido, essa camada gravável some. Um **registry** é o lugar onde as imagens são armazenadas e baixadas, como o Docker Hub.

Essa separação tem uma consequência prática: uma mesma imagem pode gerar vários containers independentes, e apagar um container não apaga a imagem. Também explica por que dados importantes não devem ficar na camada gravável, assunto que volta na seção de volumes.

Experimente os comandos abaixo na ordem. Eles mostram o ciclo completo sem precisar de nenhum código seu:

```bash
docker pull python:3.12-slim
docker images
docker run --rm python:3.12-slim python -c "import sys; print(sys.version)"
docker ps -a
```

O `--rm` remove o container quando ele termina. Sem essa flag, o container parado continua existindo e aparece no `docker ps -a`. Esse detalhe confunde quem começa, porque o disco vai enchendo com containers mortos.

### Escrevendo o Dockerfile de uma API

Vamos containerizar uma API mínima em FastAPI. Crie uma pasta com três arquivos.

#### `main.py`

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/health")
def health() -> dict[str, str]:
    return {"status": "ok"}
```

#### `requirements.txt`, com versões fixadas

```text
fastapi==0.115.0
uvicorn==0.30.6
```

#### `Dockerfile`

```dockerfile
FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN useradd --create-home appuser
USER appuser

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

A ordem das instruções importa. O Docker reaproveita camadas já construídas quando nada mudou. Por isso copiamos primeiro o `requirements.txt` e instalamos as dependências, e só depois copiamos o código. Quando você altera o `main.py`, somente as camadas a partir do `COPY . .` são refeitas. A instalação de dependências, que costuma ser a parte mais lenta, continua em cache. Se o `COPY . .` viesse antes, qualquer edição de uma linha reinstalaria tudo.

O `--host 0.0.0.0` é obrigatório. Dentro do container, `127.0.0.1` significa o próprio container, e a porta publicada pelo Docker não alcançaria a aplicação. Esse é um dos erros mais comuns de quem começa: o container sobe, o log diz que o servidor iniciou, e o navegador não consegue conectar.

#### Construindo e executando

Agora construa e execute:

```bash
docker build -t minha-api:1.0.0 .
docker run --rm -p 8000:8000 minha-api:1.0.0
```

Em outro terminal, `curl http://localhost:8000/health` deve responder `{"status":"ok"}`. A flag `-p 8000:8000` mapeia a porta do seu computador (esquerda) para a porta do container (direita).

> **Dica**
>
> O ponto final em `docker build -t minha-api:1.0.0 .` é o contexto de build, a pasta enviada ao daemon. Tudo o que estiver nessa pasta pode entrar na imagem, e isso inclui arquivos sensíveis. Veja `.dockerignore` mais adiante.

## 3. Primeiros Passos Práticos

### Volumes, redes e Docker Compose

A aplicação quase nunca vive sozinha. Ela precisa de um banco de dados, e subir cada serviço com comandos longos fica inviável. O Docker Compose descreve o conjunto de serviços em um arquivo YAML.

```yaml
services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql://app:senha-local@db:5432/app
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: senha-local
      POSTGRES_DB: app
    volumes:
      - dbdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app -d app"]
      interval: 5s
      retries: 5

volumes:
  dbdata:
```

Três pontos merecem atenção:

1. O nome `db` funciona como hostname dentro da rede criada pelo Compose. A aplicação não precisa saber o IP do banco.
2. `depends_on` sem condição só garante a ordem de inicialização, não que o banco já aceita conexões. Por isso usamos o `healthcheck` com `condition: service_healthy`.
3. O volume nomeado `dbdata` sobrevive a `docker compose down`. Já `docker compose down -v` apaga o volume junto, e os dados se vão. Use o `-v` com a consciência de que ele é destrutivo.

A senha em texto puro aparece aqui apenas porque é um ambiente local descartável. Não repita esse padrão em nenhum ambiente compartilhado. Esse assunto é tratado no Módulo 3.

Para desenvolvimento, é comum montar o código do host dentro do container com um bind mount (`./:/app`). Assim, cada alteração no editor aparece no container na hora. Isso cria um problema clássico: se o seu diretório tem uma pasta de dependências do host (como `.venv` ou `node_modules` de outro sistema operacional), ela substitui a que foi instalada na imagem. A solução usual é montar um volume anônimo sobre essa pasta.

## 4. Criando sua própria imagem

### Boas práticas de imagem

Uma imagem é um artefato que vai para produção. Os cuidados abaixo evitam problemas que só aparecem depois de meses.

#### Use tags fixas

`FROM python:latest` significa que a imagem base muda sem aviso, e o build de hoje pode não ser igual ao de amanhã. Prefira `python:3.12-slim`, e para produção considere fixar até o digest (`@sha256:...`).

#### Rode como usuário não-root

O `USER appuser` do exemplo limita o estrago caso alguém consiga executar código dentro do container. Imagens que rodam como root são comuns e são um risco evitável.

#### Use `.dockerignore`

Sem ele, arquivos como `.git`, `.env`, logs e pastas de dependências locais entram na imagem. Um `.env` com credenciais dentro de uma imagem publicada é um vazamento.

```text
.git
.env
__pycache__/
*.pyc
.venv/
tests/
```

#### Use multi-stage build

Use quando precisar de ferramentas para compilar que não devem ir para produção. O primeiro estágio compila, e o segundo copia só o resultado:

```dockerfile
FROM python:3.12 AS build
WORKDIR /src
COPY requirements.txt .
RUN pip wheel --no-cache-dir --wheel-dir /wheels -r requirements.txt

FROM python:3.12-slim
WORKDIR /app
COPY --from=build /wheels /wheels
RUN pip install --no-cache-dir /wheels/*
COPY . .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

#### Escreva logs na saída padrão

O container não deve gravar logs em arquivos internos. O Docker captura o stdout e o stderr, e é daí que as ferramentas de observabilidade coletam os dados. O mesmo vale para o estado: se a aplicação guarda uploads dentro do container, eles desaparecem no próximo deploy.

### Comandos de diagnóstico

Um comando de diagnóstico que você vai usar todos os dias é `docker logs -f <container>`. Para entrar num container em execução, use `docker exec -it <container> sh`. Quando o container não sobe, `docker logs` e `docker inspect` costumam dar a resposta em menos de um minuto.
