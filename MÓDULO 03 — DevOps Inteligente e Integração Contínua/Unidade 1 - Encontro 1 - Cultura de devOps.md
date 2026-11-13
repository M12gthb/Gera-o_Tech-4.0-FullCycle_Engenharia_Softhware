# 1. DevOps começa com um problema de incentivos CULTURA E ENGENHARIA — FULL CYCLE

Quase toda equipe que pede "um pouco de DevOps" tem um problema de incentivo. O time de desenvolvimento é medido por entregar funcionalidades rápido. O time de operações é medido por estabilidade. Cada mudança é uma ameaça para um dos lados, e a reação natural é empurrar o risco para o outro. Desenvolvedores jogam código por cima do muro, operações segura o deploy, e as duas partes passam a trabalhar em ciclos separados.

DevOps é a tentativa de alinhar esses incentivos. Não é um cargo, uma ferramenta ou uma equipe nova chamada "DevOps". Quando uma empresa cria o cargo "engenheiro DevOps" e mantém o muro intacto, ela criou mais uma parte do muro.

O romance The Phoenix Project, de Gene Kim, Kevin Behr e George Spafford (2013), descreve esse problema com uma história de falha de TI numa fábrica fictícia. Vale a leitura pela clareza com que mostra gargalos, filas de trabalho e o custo de mudanças que ninguém consegue acompanhar. Os autores depois aprofundaram os princípios no DevOps Handbook (2016).

## 2. Os Três Caminhos

Gene Kim organizou os princípios do DevOps em três caminhos, descritos no DevOps Handbook.

- O **primeiro caminho** é o fluxo. O trabalho deve ir da ideia até o cliente o mais rápido possível, sem perder qualidade. A chave é reduzir o tamanho dos lotes e os tempos de espera. Um lote de mudança de três meses é difícil de testar, difícil de implantar e difícil de reverter. Um lote de uma tarefa de um dia é o oposto.

- O **segundo caminho** é o feedback. Problemas precisam ser detectados cedo e comunicados para quem pode corrigi-los. Testes automatizados, monitoramento e revisões de código são formas de feedback. Um defeito que chega ao cliente é um feedback caro e tardio.

- O **terceiro caminho** é o aprendizado contínuo. A equipe precisa de tempo e de um ambiente seguro para experimentar, errar e melhorar. Uma cultura que pune erros recebe relatórios de erro maquiados. Uma cultura que investiga erros sem culpar pessoas recebe informação verdadeira, e ela é o insumo de melhorias reais.

## 3. CALMS: cinco pilares culturais

Damon Edwards e John Willis propuseram, em 2010, o modelo CALMS para descrever a cultura DevOps: Cultura, Automação, Lean, Medição e Compartilhamento. Não é uma metodologia com passos. É um lembrete de que as práticas técnicas sem cultura colaborativa não produzem o resultado esperado.

**Cultura** significa responsabilidade compartilhada. Quem escreve o código também acompanha o que acontece com ele em produção.

**Automação** reduz o trabalho repetitivo e o risco de erro humano. Deploy manual feito por uma pessoa específica é um ponto único de falha.

**Lean** é a busca por eliminar desperdício: filas de aprovação, retrabalho, tarefas que ninguém usa. Limitar o trabalho em andamento é uma das práticas mais efetivas.

**Medição** dá base para decisões. Sem dados, a discussão vira opinião de quem fala mais alto.

**Compartilhamento** é a transferência de conhecimento entre times. Documentação, revisões cruzadas e rotação de plantão reduzem a dependência de uma única pessoa.

## 4. As métricas DORA

O programa de pesquisa DevOps Research and Assessment (DORA), liderado por Nicole Forsgren, Jez Humble e Gene Kim, identificou em centenas de organizações quatro métricas que se correlacionam com desempenho de entrega. O livro *Accelerate* (2018) apresenta a metodologia e os resultados.

As quatro métricas são:

| Métrica | Pergunta que responde | Como medir na prática |
| --- | --- | --- |
| Frequência de deploys | Com que frequência entregamos valor? | Número de deploys em produção por dia ou semana |
| Lead time para mudanças | Quanto tempo leva de um commit até produção? | Mediana entre o primeiro commit da mudança e o deploy |
| Taxa de falhas em mudanças | Quantas mudanças causam incidente ou rollback? | Percentual de deploys que exigem correção imediata |
| Tempo para restaurar o serviço | Quanto tempo leva para voltar após uma falha? | Mediana entre o início do incidente e a restauração |

O relatório DORA mais recente também considera a confiabilidade como quinta dimensão. A ideia central, no entanto, continua sendo a mesma: velocidade e estabilidade não são opostos. Equipes de elite entregam com frequência e falham pouco.

Um cuidado, porém: essas métricas medem o sistema, não as pessoas. Se você usar frequência de deploy para avaliar desenvolvedores individuais, o time vai dividir mudanças artificialmente para inflar o número, e a métrica deixa de significar algo. A regra de Goodhart resume o problema: quando uma medida vira alvo, ela deixa de ser uma boa medida.

## 5. Medir sem punir

Uma métrica de DevOps deve responder a uma pergunta que a equipe quer fazer. Se o objetivo é reduzir o tempo de restauração, mede-se o tempo de restauração e discute-se o que atrasa a resposta a incidentes. Se o objetivo é entender por que o lead time cresceu, separa-se o tempo de revisão, o tempo de espera do pipeline e o tempo de aprovação.

### Três práticas para uma medição saudável

- Publique as métricas do time, e não de indivíduos.
- Revise tendências em retrospectivas, sem transformar o número em meta de bônus.
- Sempre que um número piorar, pergunte o que mudou no sistema antes de perguntar quem errou.

Uma retrospectiva com foco em causas costuma se organizar em torno de perguntas simples: o que aconteceu, o que nos atrasou, o que podemos mudar na próxima semana. A resposta quase nunca é "fulano".
