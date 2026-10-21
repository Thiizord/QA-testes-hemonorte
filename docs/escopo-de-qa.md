# Escopo de QA no Hemonorte e filtro de confidencialidade

## O que QA faz no projeto

As atividades efetivamente exercidas até aqui:

1. **Análise de requisitos e rastreabilidade** — leitura das issues de feature antes de testar, extração dos critérios de aceite como roteiro de teste e apontamento de lacunas e ambiguidades (perguntas de negócio registradas como decisões pendentes).
2. **Testes automatizados** — suíte unitária em três frentes: regras de domínio, Bean Validation de DTOs e contrato HTTP standalone; regressão garantida pela suíte existente a cada onda de código novo.
3. **Execução e evidência** — rodada completa via Maven a cada missão, com relatório de execução por classe e cobertura JaCoCo medida de verdade.
4. **Gestão de defeitos** — relatório vivo no repositório do sistema com identificação estável (QA-XXX), severidade, evidência (execução ou leitura de código), correção sugerida e verificação de correção em rodada posterior.
5. **Entrega colaborativa** — branch de QA → pull request → CI → aprovação de code owner → merge; resumo da rodada no corpo do PR para a revisão dos desenvolvedores.
6. **Métricas e leitura** — cobertura por pacote com interpretação honesta (o número só significa algo com o contexto do que entrou de novo).

## Filtro de confidencialidade — aplicado a todo artefato antes de entrar

Checklist obrigatório antes de qualquer inclusão:

1. O conteúdo está relacionado ao trabalho de QA?
2. Sua inclusão é útil e justificável?
3. Contém código-fonte do sistema, dados pessoais, credenciais ou outros dados sensíveis?
4. Revela detalhes internos que não estão autorizados para divulgação?
5. Existe restrição de propriedade intelectual, contrato ou política interna?
6. É possível substituir o que é restrito por exemplo genérico, descrição abstrata ou resultado agregado?
7. A divulgação está autorizada?

**Se houver dúvida, o original não entra.** Prepara-se versão sanitizada ou solicita-se autorização.

Aplicação prática deste repositório:

- Testes que importam classes do sistema **não são "artefatos independentes"** — ficam de fora; a técnica vem como exemplo genérico.
- O relatório de defeitos do sistema **não é copiado** — a rodada é resumida aqui com severidades, contagens e link público para o pull request que contém o detalhe.
- Métricas entram **com o contexto de medição** (o quê, quando, com qual suíte).

## Ritual de atualização pós-atividade

Obrigatório ao final de **cada** atividade de QA no Hemonorte, envolva testes ou não:

1. Identificar artefatos novos ou alterados.
2. Avaliar quais podem entrar com segurança (checklist acima).
3. Atualizar arquivos existentes ou criar novos quando necessário.
4. Revisar consistência e confidencialidade.
5. Versionar conforme as permissões disponíveis.
6. **Reportar**: o que foi incluído, o que ficou pendente e o que não entrou — com o motivo.

Se nenhuma atualização for necessária, a conclusão é registrada no resumo da atividade. Nada é criado artificialmente só para gerar movimentação.
