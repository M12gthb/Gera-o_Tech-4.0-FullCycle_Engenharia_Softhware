# 1. Tipos de teste de carga

O conceito de teste de carga não constitui uma abordagem única e monolítica. Cada modalidade responde a um questionamento distinto sobre o comportamento do sistema, de modo que confundir seus objetivos conduz inevitavelmente a diagnósticos equivocados.

- **Smoke Test:** Executado com volume reduzido de requisições, tem como finalidade confirmar se a aplicação responde e se o script de teste opera corretamente. Deve ser sempre o primeiro a ser rodado no pipeline.
- **Teste de Carga (Load):** Simula o volume estimado de pico operacional ao longo de um intervalo representativo. Responde se o sistema atinge as metas de desempenho sob condições normais de uso intenso.
- **Teste de Estresse (Stress):** Eleva a carga de forma gradual e contínua até o ponto de ruptura do sistema. Identifica o limite máximo de capacidade e o padrão de falha: se a aplicação degrada de forma graciosa ou colapsa abruptamente.
- **Teste de Resistência (Soak Test):** Submete o ambiente a uma carga moderada e contínua por várias horas ou dias. É o único capaz de revelar vazamentos de memória (memory leaks), esgotamento de conexões e saturação de disco que jamais aparecem em testes de curta duração.

**A Armadilha dos Testes Rápidos**
O erro mais frequente na engenharia é executar unicamente um teste de carga relâmpago de dois minutos e declarar publicamente que o sistema "suporta a carga". Essa prática não oferece absolutamente nenhuma garantia sobre a estabilidade e a integridade da aplicação ao longo do tempo de operação contínua.

# 2. Um teste de carga com k6

O k6 consolida-se como uma ferramenta moderna de código aberto na qual os cenários de carga são redigidos em JavaScript. Essa abordagem permite versionar os scripts de teste diretamente junto com o código-fonte da aplicação e executá-los de forma automatizada no pipeline.

**javascript**
import http from "k6/http"; import { check, sleep } from "k6"; export const options = { stages: [ { duration: "1m", target: 50 }, { duration: "5m", target: 50 }, { duration: "1m", target: 0 }, ], thresholds: { http_req_duration: ["p(95)<200"], http_req_failed: ["rate<0.01"], }, }; export default function () { const res = http.get("http://localhost:8000/pedidos?cliente_id=42"); check(res, { "status 200": (r) => r.status === 200 }); sleep(1); }

Os thresholds funcionam como critérios objetivos de aprovação. Caso o percentil p95 ultrapasse 200 ms ou a taxa de falhas supere 1%, o k6 encerra a execução com código de erro, reprovando automaticamente o pipeline de CI. Trata-se de um genuíno quality gate de desempenho, que deve ser disparado contra um ambiente estritamente representativo de produção, e jamais contra a máquina de desenvolvimento do autor.

Fique atento a um viés metodológico muito comum: rodar testes de carga em uma máquina local com o banco de dados hospedado na mesma máquina afere exclusivamente os limites do hardware local, e não o comportamento real do sistema distribuído. Documente e registre sempre as características do ambiente juntamente com os relatórios de resultados.

# 3. Profiling: onde o tempo é gasto dentro de um processo

Quando o gargalo de desempenho reside internamente na lógica da aplicação, a ferramenta ideal para rastreá-lo é o profiler, responsável por expor quais funções consomem mais ciclos de CPU ou alocações de memória. Diferentes ecossistemas dispõem de utilitários consagrados para essa finalidade, como o cProfile (Python), o profiler nativo do Node.js (--cpu-prof) e o pprof (Go), todos capazes de gerar relatórios detalhados que apontam o caminho quente (hot path) de execução.

**bash**
python -m cProfile -s cumtime app.py > perfil.txt

É fundamental compreender que o profiler responde ao questionamento "onde" o tempo é consumido, e nunca "por quê". Uma função que aparece no topo do relatório pode estar figurando ali devido a um laço de repetição desnecessário, a rotinas excessivas de serialização ou ao uso de uma biblioteca externa mal configurada. A interpretação correta exige a leitura atenta do código-fonte e a formulação de uma hipótese lógica, que deverá ser rigorosamente testada por meio de uma nova medição.

**A armadilha do tempo de espera por I/O**
Um detalhe que frequentemente induz a equipe ao erro: o tempo de espera por operações de I/O — como uma consulta demorada ao banco de dados ou uma requisição de rede — não figura como consumo de CPU. Um profiler de CPU exibirá uma aplicação como "ociosa" mesmo quando ela estiver lenta devido a bloqueios de espera. Para diagnosticar esse cenário com precisão, o rastreamento distribuído (distributed tracing), abordado na unidade de observabilidade, é a ferramenta correta.

# 4. Trade-offs de consistência: CAP e PACELC

O teorema CAP, proposto por Eric Brewer em 2000 e formalmente demonstrado por Gilbert e Lynch em 2002, estabelece que um sistema distribuído tolerante a partições de rede precisa escolher obrigatoriamente entre consistência e disponibilidade durante uma falha de partição. Embora o enunciado seja conciso, o que ele deixa de fora é crucial: as partições de rede são raras, porém inevitáveis a longo prazo, e o dilema de escolha restringe-se estritamente aos momentos de falha, e não a todo o tempo de operação.

Para suprir essa lacuna, o modelo PACELC (proposto por Daniel Abadi) expande o raciocínio: se houver partição (P), escolha entre disponibilidade (A) e consistência (C); caso contrário (E), mesmo operando sem falhas, escolha entre latência (L) e consistência (C). É justamente esse segundo aspecto que impacta diretamente a engenharia no dia a dia, visto que a adoção de replicação síncrona garante consistência absoluta ao custo de introduzir latência em cada operação de escrita.

**A Decisão de Negócio no Dia a Dia**
Na prática da engenharia, essas escolhas se traduzem em questionamentos fundamentais: o saldo exibido na conta do usuário pode estar desatualizado por um segundo? Um pedido de compra pode ser confirmado sem que o estoque tenha sido validado de forma síncrona? A resposta para essas indústrias depende exclusivamente das diretrizes do negócio e deve ser obrigatoriamente formalizada em um ADR (Registro de Decisão Arquitetural), nos moldes discutidos nas unidades anteriores.

# 5. Assistência de IA para hipóteses de gargalo

Os assistentes inteligentes são excelentes aliados na geração rápida de hipóteses a partir dos relatórios e métricas de um teste de carga: uma curva de latência que cresce proporcionalmente ao número de conexões ativas com o banco de dados sugere o esgotamento do pool de conexões; por outro lado, uma curva que escala em função do tamanho do payload sugere ineficiências de serialização, e assim por diante. No entanto, essas deduções geradas por IA devem ser tratadas estritamente como um ponto de partida investigativo, e jamais como um diagnóstico conclusivo.

Adote rigorosamente o mesmo método visto na unidade de observabilidade: separe com clareza o fato observado da hipótese levantada e defina, para cada suposição, qual métrica ou medição será responsável por confirmá-la ou refutá-la. Um assistente que sugere superficialmente "aumente o pool de conexões" sem antes averiguar o dimensionamento real atual está apenas prescrevendo uma solução para um problema que pode nem existir.

**Cuidado com o Efeito de Deslocamento**
Aumentar limites operacionais de forma arbitrária — como expandir o pool de conexões, elevar o número de threads ou injetar mais memória — geralmente apenas desloca o gargalo para outra camada do sistema, em vez de eliminá-lo. Meça e valide o impacto real de cada modificação de forma isolada.