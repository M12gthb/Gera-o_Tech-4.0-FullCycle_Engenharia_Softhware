# 1. Onde a IA ajuda e onde ela inventa

Os assistentes de inteligência artificial são extremamente úteis na redação de documentações a partir de bases concretas já existentes — como bases de código, diffs, suítes de testes ou registros de reuniões de design. Contudo, tornam-se altamente perigosos quando alimentados com poucos dados e obrigados a preencher lacunas por conta própria. Nessas situações, a IA tende a descrever um sistema puramente plausível, e não o sistema real em produção: não é incomum ver um endpoint fictício que jamais existiu aparecer no contrato acompanhado de parâmetros estruturados de forma impecável.

**A Regra Prática: A IA Redige, o Código Prova**
Toda e qualquer afirmação presente na documentação deve ser estritamente verificável contra o código fonte, os testes automatizados ou o comportamento real em execução. Informações que não possam ser verificadas dessa forma precisam ser explicitamente sinalizadas como decisões de design, acompanhadas de seu respectivo autor e data.

# Encontro 2: Gerando e mantendo documentação com IA

Para instruir uma IA a gerar a documentação precisa de um endpoint, o ideal é fornecer um conjunto restrito e relevante de insumos: o arquivo do roteador, os schemas utilizados e os testes de integração correspondentes àquela rota. Os testes desempenham um papel fundamental por apresentarem exemplos reais e executáveis de payloads de entrada e saída.

**Exemplo de Prompt Contextualizado**
Gere a seção OpenAPI para o endpoint POST /pedidos a partir dos arquivos abaixo. Use apenas o que está nos arquivos. Se um código de resposta não aparecer no código ou nos testes, não o inclua e liste a dúvida ao final. Formato: YAML OpenAPI 3.1, sem comentários adicionais. [conteúdo de app/routes/pedidos.py] [conteúdo de app/schemas/pedido.py] [conteúdo de tests/test_pedidos_integracao.py]

A diretriz restritiva "use apenas o que está nos arquivos" combinada com a exigência de listar dúvidas pendentes é o que converte uma resposta genérica de IA em um rascunho técnico realmente confiável. Na ausência dessas travas, o modelo tende a inventar informações para preencher lacunas silenciosamente.

Sempre cruze o resultado gerado com os testes automatizados. Se a suíte de testes espera o código 422 para um campo ausente, mas o rascunho documenta o código 400, há uma divergência clara entre a documentação e a implementação, cabendo à verificação técnica determinar qual lado está correto.

# 3. Documentação como parte da mudança

A documentação que realmente sobrevive ao longo do tempo é aquela que evolui lado a lado com o código-fonte. Uma estratégia altamente eficaz para assegurar essa sincronia consiste em estabelecer uma regra no pipeline de Integração Contínua (CI) que interrompa o fluxo sempre que um arquivo de rota for alterado sem que o contrato correspondente sofra modificações concomitantes. Essa checagem de diff pode ser implementada de forma simples com um script direto:

**bash**
git diff --name-only origin/main...HEAD | grep -q "app/routes/" \ && ! git diff --name-only origin/main...HEAD | grep -q "openapi.yaml" \ && echo "Rota alterada sem atualizar openapi.yaml" && exit 1 exit 0

Evidentemente, abordagens automatizadas desse tipo podem gerar falsos positivos — como em cenários onde uma refatoração interna é realizada sem causar impacto algum no contrato da API. Por esse motivo, é fundamental prever exceções mediante uma justificativa clara redigida na descrição do Pull Request. O propósito central da automação é tornar a decisão explícita e consciente, e nunca engessar ou impedir o trabalho da equipe.

Aqui, o auxílio de um assistente inteligente mostra-se valioso: diante do diff de uma rota, a IA pode analisar e sugerir se o contrato precisa ser modificado e quais seções devem ser atualizadas. Trata-se, contudo, de um lembrete útil para orientar o autor da mudança, e nunca de um veredito absoluto.

# 4. Verificação com exemplos executáveis

A documentação verdadeiramente confiável é aquela que pode ser executada de ponta a ponta. Exemplos de requisições e respostas redigidos em um README ou em um contrato OpenAPI podem ser convertidos diretamente em suítes de teste. Se o exemplo de uso deixa de funcionar na prática, o teste falha, impedindo que a documentação permaneça desatualizada por mais tempo do que o ciclo de um pipeline.

A ferramenta Schemathesis — citada na unidade anterior — assume o papel de gerar automaticamente cenários de teste rigorosos a partir da especificação OpenAPI, disparando-os contra a API em execução. Esse processo descobre divergências críticas, como um campo documentado como obrigatório que o servidor aceita silenciosamente sem ele, ou uma resposta que retorna um tipo de dado diferente daquele declarado no contrato.

**bash**
schemathesis run openapi.yaml --base-url http://localhost:8000

Trate cada falha apontada pela ferramenta como uma de duas constatações possíveis: ou trata-se de um bug no servidor que precisa ser corrigido, ou de um contrato incorreto que diverge da realidade. Identifique qual das duas premissas está incorreta, faça a correção devida e documente o aprendizado.

# 5. Documentação para onboarding e operação

Além do contrato técnico de APIs, duas frentes documentais possuem valor inestimável para a engenharia: o guia de onboarding, responsável por capacitar um novo colaborador a compreender o sistema em até uma semana, e a documentação operacional, consubstanciada no runbook de plantão. Ambas ganham muita agilidade com o apoio de IA na redação, desde que a base de conteúdo reflita os fatos e a vivência real da equipe.

Para estruturar o onboarding, propor perguntas-chave respondidas pela própria equipe no repositório funciona muito melhor do que textos genéricos: onde residem as principais responsabilidades, como uma alteração típica percorre a arquitetura e qual é o ciclo de vida completo de uma transação. O assistente inteligente atua organizando esse material em um guia coerente, mas a substância deve vir dos engenheiros.

**A Estrutura Essencial de um Runbook**
No caso dos guias de operação (runbooks), a regra de ouro do DevOps permanece inalterada: identifique o sintoma, apresente a forma de confirmação, liste os passos de mitigação e estabeleça o critério de escalonamento. Um runbook gerado sem o respaldo de quem já enfrentou e resolveu o problema em ambiente produtivo tende a descrever caminhos genéricos que falham no momento crítico.

# 6. Manutenção: a documentação envelhece

A documentação técnica não deve ser tratada como um projeto pontual com data de término, mas sim como um ativo vivo. É fundamental atribuir um responsável claro pela curadoria de cada documento, estipular prazos periódicos de revisão e definir gatilhos automáticos de atualização — como a alteração de um contrato de API ou uma falha em incidente real onde o runbook se mostrou insuficiente.

**Ciclo de Feedback na Retrospectiva**
Uma rotina simples e altamente eficaz consiste em auditar a documentação durante as reuniões de retrospectiva. Cada post-mortem de incidente e cada processo de onboarding produzem naturalmente uma lista de lacunas informacionais. Sempre que uma deficiência documental se repete, ela deve ser tratada como um item legítimo de backlog — com responsável atribuído e prazo de entrega, exatamente como qualquer outro bug de software.

