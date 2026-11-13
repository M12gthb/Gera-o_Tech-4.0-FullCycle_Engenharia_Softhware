# 1. Diagnóstico começa com um sintoma bem descrito

Quando um incidente é deflagrado, a pior abordagem possível é recorrer a um assistente de IA com uma frase vaga como "o sistema está lento". Sem saber o que foi medido, o horário exato, a rota afetada ou o contexto operacional, o modelo emitirá apenas hipóteses genéricas. A qualidade do diagnóstico técnico está diretamente atrelada à precisão da pergunta, e a pergunta depende de fatos baseados em dados concretos.

## As Quatro Dimensões de um Sintoma Útil

- O que está errado: A métrica ou comportamento anômalo observado.
- Desde quando: O registro temporal exato do início da falha.
- Onde: A delimitação precisa de serviço, rota ou região afetada.
- O que mudou: Eventos correlatos recentes (como o deploy de uma nova versão).

Uma declaração rica em contexto — como "o p95 da rota de checkout subiu de 300 ms para 2,4 s a partir das 14h10, apenas na região Sul, logo após o deploy da versão 2.7.1" — fornece o insumo ideal para investigações assertivas. A inteligência artificial entrega seu máximo desempenho quando recebe fatos estruturados, e não quando é forçada a adivinhá-los.

## 2. Preparar o contexto sem vazar dados

Os dados provenientes de incidentes operacionais são inerentemente sensíveis. Os logs costumam concentrar identificadores, endereços de IP e eventuais fragmentos de payloads. Antes de submeter qualquer trecho a um assistente de inteligência artificial, aplique o mesmo rigor de sanitização exigido pelas políticas de segurança da corporação: remova tokens, e-mails, CPFs e cabeçalhos de autorização, substituindo-os por placeholders padronizados.

**bash**
grep "ERROR" pedidos.log \
  | sed -E 's/[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+//g' \
  | sed -E 's/Bearer [A-Za-z0-9._-]+/Bearer /g' \
  | head -200 > trecho_sanitizado.log

**Filtros Automatizados vs. Revisão Humana**
Expressões regulares (Regex) capturam padrões previsíveis de dados, mas falham em identificar exceções contextuais, como o nome de um cliente inserido acidentalmente em um campo livre. Realize sempre uma inspeção manual do trecho antes do envio.

Além da privacidade, é preciso gerenciar o volume de dados. Um arquivo com um milhão de linhas ultrapassa a janela de contexto e contém majoritariamente ruído operacional. Filtre o escopo por janela temporal, nível de severidade e identificador de requisição, enviando apenas amostras representativas delimitadas pelo início e fim do incidente.

## 3. Análise de tendências e anomalias

Os assistentes baseados em inteligência artificial são ferramentas poderosas para interpretar séries temporais que já foram extraídas e consolidadas. O formato ideal de comando instrui o modelo a comparar janelas distintas, isolar o ponto exato de inflexão da curva e listar hipóteses classificadas de acordo com as evidências. Um prompt altamente eficaz divide-se em três frentes: o fato observado, a hipótese levantada e o teste prático capaz de confirmá-la ou refutá-la.

**Exemplo de Prompt Estruturado**
Dados: latência p95 por minuto de 13h às 15h, rota /checkout, região sul.
Versões: 2.6.9 até 14h10, 2.7.1 depois.
Pedido: 1 descreva a mudança observada, sem interpretar; 2 liste três
hipóteses com a evidência a favor e contra; 3 para cada uma, diga qual
consulta ou medição confirmaria ou descartaria a hipótese.

**Evitando a Armadilha da Plausibilidade**
A IA tende a gerar narrativas altamente convincentes que muitas vezes aceitamos apenas por soarem coerentes. Ao exigir a separação rígida entre fato, hipótese e teste, a interação com o modelo deixa de ser uma mera consulta e transforma-se em um processo rigoroso de investigação técnica.

## 4. Correlação não é causa

Um modelo analítico ou ferramenta de monitoramento pode apontar que a latência de uma aplicação disparou exatamente no mesmo minuto em que um job de backup foi iniciado. Trata-se de uma correlação temporal. No entanto, isso pode representar a causa raiz, uma mera coincidência ou o reflexo de um terceiro fator oculto, como o pico simultâneo de tráfego geral naquele horário.

A única maneira de separar esses cenários é através de isolamento ou reprodução controlada: pause o backup em um ambiente de homologação e meça o comportamento, ou desvie uma fração do tráfego para observação. Um diagnóstico só deve ser validado como causa após passar por um experimento rigoroso que poderia tê-lo refutado.

Essa disciplina metodológica aplica-se a qualquer instrumento de análise, seja humano ou automatizado. Um painel exibindo duas linhas convergindo não comprova uma relação de causalidade por si só.

**Atenção: Cuidado com Ações Reativas em Produção**
Jamais aplique mudanças diretas em produção unicamente porque um assistente sugeriu uma causa plausível. Executar um reinício de serviço às cegas ou um rollback precipitadamente pode mascarar o problema original e desencadear um novo incidente ainda mais grave.

## 5. Performance guiada por medição

Tentar otimizar um sistema sem antes mensurar o seu comportamento real é a maneira mais eficiente de desperdiçar tempo de engenharia. Antes de aplicar qualquer modificação no código, meça exatamente onde os recursos e o tempo estão sendo consumidos. Enquanto um profiler aponta quais funções consomem mais CPU, um trace distribuído revela qual serviço ou chamada de rede domina a latência da transação.

Uma vez que o gargalo é isolado, a correção costuma ser cirúrgica e direta: a criação de um índice ausente no banco de dados, a remoção de consultas executadas dentro de laços de repetição, a conversão de chamadas síncronas em fluxos assíncronos ou a implementação de uma estratégia adequada de cache. O ganho obtido deve ser medido novamente, utilizando o mesmo método anterior, para confirmar empiricamente a eficácia da alteração.

Um exemplo clássico e recorrente de gargalo é o problema do N+1, que ocorre quando a aplicação recupera uma lista principal e, em seguida, dispara uma consulta isolada para cada item iterado:

```python
# N+1: uma consulta para os pedidos e uma consulta adicional por pedido
pedidos = repo.listar_pedidos()
for p in pedidos:
    p.cliente = repo.buscar_cliente(p.cliente_id)
```

**Abordagem otimizada:** Uma única consulta com junção executada no banco
``python
pedidos = repo.listar_pedidos_com_cliente()
``

## 6. Pós-incidente assistido

Assim que o incidente é contornado e estabilizado, a sessão de revisão pós-incidente (post-mortem) representa o momento ideal para converter a experiência em aprendizado e melhoria estrutural. A inteligência artificial pode atuar como um facilitador valioso, auxiliando na organização cronológica da linha do tempo com base no histórico de mensagens do canal de crise, na elaboração do rascunho de resumo e na formulação de perguntas críticas que acabaram não sendo levantadas durante o calor do momento.

**Validação Humana Rigorosa**
O rascunho gerado por IA exige checagem minuciosa frente aos fatos reais, e as causas precisam ser validadas por quem conduziu a investigação técnica. Linhas do tempo automáticas podem inverter eventos cronologicamente, e suposições de causa raiz geradas pela ferramenta podem não encontrar respaldo em nenhum log do sistema.

Uma excelente prática metodológica consiste em solicitar ao assistente que relacione todos os pontos que permaneceram sem resposta clara ao longo da investigação. Esse exercício costuma expor lacunas profundas de instrumentação e monitoramento que merecem se transformar em ações concretas de melhoria no backlog.
