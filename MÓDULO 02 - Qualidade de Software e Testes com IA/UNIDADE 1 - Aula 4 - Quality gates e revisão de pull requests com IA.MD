# 1. Quality gate: uma regra que bloqueia a entrega

Um quality gate é uma condição objetiva que a mudança precisa cumprir para avançar. Ele é o ponto em que o pipeline deixa de ser informativo e passa a ter poder de decisão. Um teste vermelho sem consequência é só um aviso. Um teste vermelho que impede o merge é um gate.

**OS GATES MAIS COMUNS**
Os gates mais comuns são estes: o código compila, o lint passa, a checagem de tipos passa, os testes passam, a cobertura não caiu, e a análise estática não encontrou vulnerabilidades de severidade alta. Cada um tem um custo e um valor diferente, e a escolha deve ser feita com a equipe, não copiada de um template.

**A ARMADILHA DA MÉTRICA DE COBERTURA**
Há um erro frequente nesse ponto. Times adotam uma meta de cobertura, como 80%, e a cumprem com testes sem asserções. O gate passa, a qualidade não melhora e a confiança na métrica diminui. A recomendação é exigir que a cobertura não diminua em relação à branch principal, em vez de exigir um número absoluto. Essa regra é mais difícil de enganar e impede regressões sem punir código legado antigo.

# 2. Proteção de branch e checks obrigatórios

Um gate só funciona se for impossível contorná-lo. No GitHub, isso é configurado em ``Settings > Branches``, com uma regra de proteção para a `main`. As configurações que importam são:

**REGRAS ESSENCIAIS DE PROTEÇÃO**
- Exigir Pull Request antes de fazer merge, com pelo menos uma aprovação.
- Exigir que os checks de CI passem antes do merge, marcando os jobs específicos como obrigatórios.
- Exigir que a branch esteja atualizada com a principal antes do merge.
Impedir que administradores contornem a regra, ou pelo menos registrar quando isso acontece.

**BOAS PRÁTICAS E CUIDADOS OPERACIONAIS**
A última configuração é a mais negligenciada. Se administradores podem fazer push direto, a proteção vale apenas até a primeira emergência, e as emergências se repetem.

Uma distinção útil: um check obrigatório precisa ter um nome estável. Se você renomeia o job no workflow, a regra de proteção continua apontando para o nome antigo e a PR fica bloqueada para sempre. Esse problema aparece com frequência em times que reorganizam pipelines sem revisar as regras.

# 3. Análise estática e segurança como gate

Análise estática examina o código sem executá-lo. Ela encontra padrões perigosos, como concatenação de SQL, uso de funções de execução de comandos com entrada do usuário e comparação insegura de senhas. Ferramentas como o Semgrep permitem escrever regras próprias e aplicar conjuntos prontos da comunidade.

**EXEMPLO DE JOB DE SEGURANÇA**
**yaml**
``seguranca: runs-on: ubuntu-latest permissions: contents: read steps: - uses: actions/checkout@v4 - uses: semgrep/semgrep-action@v1 with: config: p/default``

**VERIFICAÇÃO DE DEPENDÊNCIAS E DEPENDABOT**
A verificação de dependências é o outro lado. Uma biblioteca pode ter uma vulnerabilidade publicada depois que você a instalou. O GitHub oferece o Dependabot, que abre PRs de atualização e alertas automáticos. Trate esses alertas com a mesma seriedade de um teste vermelho, e não como ruído.

**PROGRESSIVIDADE DOS GATES**
Um cuidado: não configure o gate para bloquear qualquer achado de qualquer severidade no dia um. Isso faz o time desativar a ferramenta. Comece bloqueando apenas o que é alto e crítico, documente as exceções com justificativa e aumente o rigor aos poucos.

# 4. Falsos positivos e a política de exceções

Toda ferramenta de análise produz falsos positivos. Um achado que não é explorável no contexto real precisa de uma saída documentada. A saída errada é desabilitar a regra globalmente. A saída certa é uma exceção local, com comentário explicando o motivo, revisada em PR.

*python*
``nosec: a entrada é um inteiro validado na linha 12, e este trecho não é alcançado com outra entrada``
``resultado = subprocess.run(["ls", str(diretorio_validado)], check=True)``

Essa prática tem um efeito colateral positivo: a exceção vira documentação. Dois meses depois, quando alguém mudar a validação da linha 12, a revisão de código tende a notar que a justificativa deixou de valer.

Acompanhe a taxa de falsos positivos por ferramenta. Se um conjunto de regras gera mais de metade de achados descartados, ele está consumindo a atenção do time sem proteger nada. Revise a configuração trimestralmente.

# 5. Revisão de PRs com apoio de IA

Assistentes de IA podem atuar em PRs de três formas: como comentaristas que apontam problemas no diff, como geradores de descrição e resumo da mudança, e como verificadores de regras do projeto. Os três são úteis, e os três precisam de limites claros.

**ATUAÇÃO DOS ASSISTENTES EM PRS**
Como comentarista, o assistente é bom em encontrar problemas mecânicos: tratamento de erro ausente, consulta dentro de laço, variável não usada, nome que não segue o padrão. Ele é fraco em avaliar se a mudança resolve o problema certo, se a abordagem combina com a arquitetura e se o impacto em outros serviços foi considerado. Essas perguntas continuam com o revisor humano.

Como gerador de resumo, o assistente acelera a leitura. Peça a ele que liste o que mudou, o que não mudou e quais riscos a revisão deve olhar, e confira se o resumo corresponde ao diff. Um resumo que descreve algo que não está no diff é um sinal de alerta.

Como verificador de regras, o assistente pode ler as instruções do repositório e checar a PR contra elas. Isso só funciona se as regras estiverem escritas de forma verificável: "não use any" é verificável, "escreva código limpo" não é.

``Atenção Crítica``
``Comentários automáticos não aprovam PRs. Configure a ferramenta como comentarista e nunca como aprovadora obrigatória. Um bot que aprova sozinho remove a revisão humana do processo sem que ninguém perceba.``

**PROMPT NO TEMPLATE DE PR**
Uma PR pode incluir um prompt de revisão no próprio template. Assim, o autor já responde às perguntas antes d


# 6. Métricas do próprio pipeline

**Métricas do próprio pipeline**
Um pipeline também é um sistema que precisa de métricas. Acompanhe pelo menos quatro indicadores:

| Indicador | O que revela | Sinal de alerta |
|---|---|---|
| Tempo médio do pipeline | Custo de feedback para o desenvolvedor | Acima de 10 minutos para PRs pequenas |
| Taxa de falhas por causa do próprio pipeline | Instabilidade da infraestrutura ou de testes | Mais de 5% das execuções falham sem mudança de código |
| Tempo entre abertura e merge | Gargalo de revisão | Crescimento sustentado ao longo de semanas |
| Número de bypasses de proteção | Pressão para contornar o processo | Qualquer ocorrência sem registro |