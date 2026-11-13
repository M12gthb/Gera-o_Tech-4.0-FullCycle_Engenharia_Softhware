# 1. Monitoramento responde perguntas conhecidas

O monitoramento consiste no acompanhamento contínuo de sinais cujos limiares já são conhecidos de antemão: uso de CPU elevado, taxa de erro superior a 1%, ou uma fila de mensageria acumulando mais de mil mensagens. Para cada uma dessas condições previamente mapeadas, define-se um alerta específico.

Por outro lado, a observabilidade é a capacidade do sistema de responder a perguntas que você não conseguiu prever. Um exemplo clássico: por que clientes de uma região específica estão enfrentando timeouts apenas às quintas-feiras? Para solucionar cenários como este, a aplicação precisa emitir telemetria rica o suficiente que permita a investigação aprofundada sem a necessidade de injetar novo código no meio de um incidente ativo.

**A Distinção Fundamental**
Um painel de monitoramento totalmente verde não comprova que os usuários estão tendo uma boa experiência; ele prova apenas que as falhas previstas não aconteceram. A observabilidade é o pilar que capacita a equipe a investigar todo o restante. Como destacam Charity Majors, Liz Fong-Jones e George Miranda no livro Observability Engineering (2022), esse conceito é a base da engenharia de confiabilidade moderna.

Os tradicionais três pilares da telemetria — logs, métricas e traces — atuam de forma sinérgica e complementar, sendo cada um estruturado para responder a naturezas distintas de questionamentos operacionais.

## 2. Logs estruturados e correlação

Um log é, essencialmente, um registro de evento enriquecido com contexto. Enquanto o formato de texto livre (como Erro ao processar pedido) pode até ser útil para um humano inspecionar um arquivo estático, ele se torna quase inútil para consultas e agregações automatizadas. A adoção de logs estruturados em JSON transforma cada atributo em um campo pesquisável e indexável.

**python**
``import json
import logging
import sys
import time
import uuid

def registrar(evento: str, **campos):
    linha = {"ts": time.time(), "evento": evento,**campos}
    sys.stdout.write(json.dumps(linha, ensure_ascii=False) + "\n")

request_id = str(uuid.uuid4())
registrar("pedido_recebido", request_id=request_id, usuario_id=42, valor=189.90)
registrar("gateway_timeout", request_id=request_id, gateway="cartao", tentativa=2)``

O campo request_id atua como o elo fundamental que permite reconstruir a jornada completa de uma transação, mesmo quando ela transita por múltiplos microsserviços. O identificador deve ser gerado na borda (entrada) do sistema, propagado via cabeçalhos HTTP e injetado em todas as camadas de log. Sem ele, o registro de cada serviço permanece isolado em uma ilha de dados.

### Diretrizes de Conteúdo e Severidade

- Privacidade Absoluta: Nunca registre segredos, senhas, tokens de acesso ou dados pessoais sensíveis (PII) nos logs.
- Critério de Severidade: Utilize DEBUG para diagnóstico técnico detalhado, INFO para marcos importantes do fluxo, WARNING para desvios que o sistema conseguiu contornar, e ERROR estritamente para falhas que exigem intervenção humana. Se tudo vira ERROR, nenhum evento é tratado como prioridade.

Conforme preconiza o Fator XI do manifesto The Twelve-Factor App, os logs devem ser tratados como fluxos contínuos de eventos direcionados estritamente à saída padrão (stdout). A responsabilidade pela coleta, agregação e rotação desses arquivos cabe integralmente à plataforma de execução, eximindo a aplicação de gerenciar armazenamento local.

## 3. Métricas: números agregados

As métricas representam valores numéricos agregados ao longo do tempo. Por serem extremamente econômicas de armazenar e rápidas de consultar, tornam-se a escolha ideal para a construção de painéis operacionais e sistemas de alerta. Os três tipos fundamentais mais comuns são: contador (acumulador estritamente crescente, como o total de requisições atendidas), medidor (valor flutuante que sobe e desce, como conexões ativas), e histograma (que distribui amostras em faixas de valores, ideal para rastrear latências).

Duas metodologias consagradas auxiliam na escolha correta do que monitorar:

- **Método RED (Rate, Errors, Duration):** Proposto por Tom Wilkie, é voltado especificamente para serviços orientados a requisições (mede taxa de requisições por segundo, quantidade de erros e duração/latência).
- **Método USE (Utilization, Saturation, Errors):** Desenvolvido por Brendan Gregg, foca na análise de recursos de infraestrutura (como CPU, disco e rede).

A latência de serviços deve ser avaliada sempre por meio de percentis, nunca por médias aritméticas. Uma média geral de 200 ms pode mascarar perfeitamente o fato de que 5% dos usuários reais estão enfrentando tempos de espera superiores a 3 segundos. Monitore métricas como p50, p95 e p99 — sendo o p99 habitualmente o primeiro indicador a expor gargalos que impactam diretamente a experiência do cliente.

**Alerta de Arquitetura: O Custo da Alta Cardinalidade**
A cardinalidade elevada é o vilão oculto no consumo de recursos de métricas. Utilizar um rótulo dinâmico contendo o ID do usuário gera uma série temporal inédita para cada cliente novo, fazendo o armazenamento explodir. Utilize apenas rótulos com baixa variação (como rota HTTP, método e código de status). Para rastrear dados detalhados por usuário, recorra sempre aos logs estruturados ou aos traces.

## 4. Traces distribuídos

Um trace representa o caminho completo percorrido por uma requisição através de múltiplos microsserviços. Ele é composto por unidades menores chamadas spans, que encapsulam operações individuais contendo carimbo de início, duração e atributos customizados. Por exemplo, o span principal de um checkout pode conter spans filhos para as chamadas subsequentes ao microsserviço de estoque, ao gateway de pagamento e ao banco de dados. Quando uma transação apresenta lentidão, o trace aponta instantaneamente qual dos componentes concentrou o gargalo de tempo.

O OpenTelemetry consolida-se como o padrão aberto da indústria para a geração desses dados de telemetria, mantendo a independência em relação a fornecedores de ferramentas. Instrumentar a aplicação com OpenTelemetry garante que a eventual substituição do backend de observabilidade ocorra sem a necessidade de reescrever o código-fonte.

**python**
``from opentelemetry import trace

tracer = trace.get_tracer(**name**)

def finalizar_pedido(pedido_id: str) -> None:
    with tracer.start_as_current_span("finalizar_pedido") as span:
        span.set_attribute("pedido.id", pedido_id)
        verificar_estoque(pedido_id)``

**Instrumentação Automática vs. Manual**
Enquanto a instrumentação automática cobre bibliotecas e frameworks comuns (como clientes HTTP e drivers de banco de dados), a instrumentação manual é indispensável para capturar operações complexas de negócio, camada onde o valor analítico se torna mais evidente durante o diagnóstico de incidentes.

## 5. SLI, SLO e orçamento de erro

Existe um dilema clássico na engenharia: alertas que disparam sem parar acabam sendo ignorados pelas equipes, enquanto alertas que nunca disparam não oferecem proteção real. Para resolver esse equilíbrio, as diretrizes de SRE estabelecem o uso integrado de três conceitos fundamentais:

- **SLI (Indicador de Nível de Serviço):** Métrica quantificável da experiência real do usuário. Um exemplo clássico é a proporção de requisições de checkout que respondem com sucesso em menos de 800 ms.
- **SLO (Objetivo de Nível de Serviço):** A meta estipulada para o SLI dentro de uma janela temporal definida (por exemplo: garantir que 99,5% das requisições atendam ao critério anterior em um período móvel de 30 dias).
- **Orçamento de Erro (Error Budget):** A margem de falha tolerada sem que o SLO seja violado (complementar à meta — com um SLO de 99,5%, o orçamento é de 0,5%).

**Dinâmica de Gestão do Orçamento de Erro**
Quando o orçamento de erro passa a ser consumido de forma acelerada, a equipe reduz o ritmo de novas mudanças e redireciona o foco totalmente para a estabilidade. Por outro lado, quando sobra orçamento, há margem controlada para assumir mais riscos e acelerar entregas.

Essa abordagem transforma discussões subjetivas e emocionais — como a afirmação vaga de que "o sistema está instável" — em decisões matemáticas precisas e objetivas, pactuadas previamente entre as áreas de produto e engenharia antes mesmo da ocorrência de incidentes.
