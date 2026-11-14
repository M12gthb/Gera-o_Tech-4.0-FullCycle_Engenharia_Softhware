# 1. Medir antes de otimizar

Desempenho é uma propriedade que se mede com rigor, e não algo que se avalia por mera percepção subjetiva. Antes de iniciar qualquer esforço de refatoração ou melhoria, é fundamental estabelecer com clareza o que significa "rápido" para o ecossistema. Um padrão comum de mercado estipula, por exemplo, que a latência da rota de checkout no p95 permaneça abaixo de 800 ms mesmo sob horários de pico. Na ausência de uma meta numérica clara, qualquer intervenção parece útil, e nenhuma otimização pode ser considerada concluída com segurança.

**A Realidade da Cauda (Percentis p95 e p99)**
A latência deve ser avaliada sempre por meio de percentis. Enquanto a mediana (p50) revela a experiência comum da maioria dos usuários, o p95 e o p99 expõem a cauda da distribuição — exatamente onde os clientes enfrentam lentidão e onde sistemas com alta densidade de chamadas internas sofrem. Considere uma requisição que dependa de cinco serviços sequenciais, cada qual com p99 de 200 ms: a probabilidade de que pelo menos uma etapa estoure esse limite é consideravelmente alta. Em arquiteturas distribuídas, a cauda sempre se acumula.

# 2. Cache: velocidade com custo de consistência

O cache tem como função armazenar o resultado de uma operação computacionalmente custosa para reutilizá-lo em chamadas futuras. A estratégia padrão mais utilizada é o modelo cache-aside: a aplicação consulta o cache primeiro; em caso de ausência (cache miss), busca o registro diretamente no banco de dados e grava o resultado obtido para as próximas execuções.

**python**
def obter_produto(produto_id: str) -> dict: chave = f"produto:{produto_id}" cacheado = cache.get(chave) if cacheado is not None: return cacheado produto = repo.buscar_produto(produto_id) cache.set(chave, produto, ttl=300) # 5 minutos return produto

Três fatores fundamentais exigem análise antes de implementar qualquer camada de cache:

1. Tolerância à defasagem: O preço de exibição de um produto pode tolerar uma lacuna de cinco minutos, mas o saldo de estoque crítico no momento exato da compra jamais pode ser desatualizado.
2. Estratégia de invalidação: Quando o dado subjacente muda, o cache precisa ser atualizado ou removido — sendo a via mais simples permitir que o TTL expire naturalmente.
3. Efeito de estouro (Cache Stampede): Quando uma chave altamente concorrida expira, milhares de requisições identificam a falta ao mesmo tempo e sobrecarregam o banco. As mitigações incluem variação aleatória de TTL, bloqueio de concorrência (para que apenas uma thread recupere o valor) e atualização antecipada antes da expiração.

**Atenção Crítica**
O cache jamais corrige uma consulta mal otimizada ou lenta. Ele apenas mascara o problema enquanto a chave permanece aquecida, fazendo com que a lentidão reapareça de forma severa na primeira ocorrência de cache miss. Otimize primeiramente a consulta estrutural e utilize o cache apenas para o que continuar custoso após esse ajuste.

# 3. Filas e processamento assíncrono

O uso de filas promove o desacoplamento essencial entre a origem de uma requisição e a sua execução efetiva. Essa arquitetura atua diretamente na absorção de picos de carga: se um milhar de pedidos ingressa no sistema em questão de minutos e o serviço de envio de e-mails processa apenas uma centena no mesmo intervalo, a fila retém a diferença de volume. Como resultado, as mensagens são despachadas com pequeno atraso temporal, mas sem qualquer perda de dados.

Para projetar sistemas assíncronos sólidos, três conceitos fundamentais devem ser dominados:

- **Backpressure:** Mecanismo que permite ao consumidor sinalizar saturação de capacidade, impedindo que a fila cresça de maneira infinita. Lembre-se: uma fila sem limites operacionais configurados é apenas uma falha sistêmica adiada.
- **Idempotência:** Propriedade que garante que uma operação possa ser executada múltiplas vezes sem produzir efeitos colaterais duplicados — um requisito vital, visto que grande parte das filas opera sob a diretriz de entrega "pelo menos uma vez" (at-least-once).
- **Dead Letter Queue (DLQ):** Destino reservado para mensagens que falharam repetidamente no processamento, permitindo isolamento para auditoria em vez de travar o fluxo principal da fila.

**Alerta de Produção: O Risco da Cobrança Dupla**
Qualquer consumidor responsável por transações financeiras — como a cobrança de cartões de crédito — obriga-se a implementar idempotência fundamentada em chaves exclusivas por operação. Na ausência dessa salvaguarda, a repetição de uma mensagem após um timeout de rede pode resultar em cobranças duplicadas ao cliente, constituindo um dos defeitos de produção mais caros e críticos em arquiteturas assíncronas.

# 4. Banco de dados: a origem da maioria dos gargalos

Na grande maioria das arquiteturas de software, o banco de dados constitui o principal gargalo de performance. Três problemas de projeto e consulta costumam se manifestar com alta frequência:

O primeiro deles é a consulta realizada sem o auxílio de índices adequados. Uma instrução SELECT que varre milhões de linhas em disco apenas para retornar uma dezena de registros pode levar segundos preciosos. A ferramenta EXPLAIN ANALYZE expõe o plano de execução do banco, revelando claramente a ocorrência de varreduras completas (sequential scans).

**sql**
EXPLAIN ANALYZE SELECT id, status, total FROM pedidos WHERE cliente_id = 42 ORDER BY criado_em DESC LIMIT 10; -- Com índice composto, a consulta lê apenas as linhas necessárias CREATE INDEX idx_pedidos_cliente_data ON pedidos (cliente_id, criado_em DESC);

O segundo gargalo recorrente é o clássico problema N+1 (já explorado no Módulo 3), cenário em que a aplicação dispara uma consulta isolada para cada item integrante de uma lista. O terceiro ponto crítico reside na paginação baseada em deslocamento (OFFSET), cuja lentidão degrada exponencialmente à medida que o usuário avança para páginas mais distantes, pois o banco é forçado a percorrer e descartar todas as linhas anteriores. Para grandes volumes de dados, a adoção de paginação por cursor — utilizando o último registro visualizado como ponto de partida — garante um custo computacional constante.

Vale ressaltar que índices não são gratuitos: embora acelerem operações de leitura, eles encarecem a escrita e consomem espaço de armazenamento. Um índice que não é acionado por nenhuma consulta representa apenas custo adicional para a infraestrutura, sendo o EXPLAIN o mecanismo ideal para identificar quais estruturas são efetivamente utilizadas.

# 5. Escalar: vertical, horizontal e onde o estado mora

A escala vertical consiste basicamente em alocar uma máquina mais potente (mais CPU, memória e I/O). Trata-se de uma abordagem operacionalmente simples, mas que esbarra em limites físicos e financeiros intransponíveis. Já a escala horizontal envolve adicionar novas instâncias de servidores operando em paralelo atrás de um balanceador de carga. Embora seja o modelo ideal para o crescimento sustentado, ela só produz o efeito desejado se a aplicação for concebida sem dependência de estado local (stateless). Dados de sessão, arquivos temporários de upload e caches mantidos unicamente em memória precisam ser transferidos para uma camada de armazenamento compartilhada; caso contrário, o balanceador poderá direcionar uma segunda requisição do mesmo usuário para uma instância que desconhece totalmente o seu contexto.

**A Armadilha de Escalar sem Diagnóstico**
Antes de iniciar qualquer expansão de infraestrutura, é imprescindível identificar com precisão onde reside o verdadeiro gargalo. Multiplicar instâncias de uma aplicação cujo travamento real é causado por esperas no banco de dados apenas intensifica a pressão sobre o banco, tornando o sistema globalmente mais lento em vez de acelerá-lo. Esse efeito colateral é amplamente conhecido e costuma surpreender equipes que recorrem à escala horizontal por puro reflexo.

