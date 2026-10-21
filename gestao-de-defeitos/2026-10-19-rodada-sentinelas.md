# Rodada 2026-10-19 — provas de contrato e análise da onda de features

> Resumo sanitizado da rodada. O detalhe técnico completo dos achados vive no relatório de QA do repositório do sistema, entregue via pull request público — linkado na seção de rastreabilidade.

## Contexto

Onpa de entregas dos desenvolvedores: as features das issues #6 e #8–#14 (agendamento com capacidade, triagem clínica, coleta, exames sorológicos, estoque e expiração diária), um CI novo via GitHub Actions e regras de repositório passando a exigir pull request + aprovação de code owner + check de build para a branch de desenvolvimento. O QA sincronizou, rodou a suíte contra o código novo, atualizou o relatório de defeitos e entregou a própria rodada pelo novo fluxo.

## Atividades realizadas

1. **Sincronização e leitura do diff** — 19 commits desde a última rodada de QA, com autoria e propósito mapeados antes de qualquer teste.
2. **Execução da suíte completa** — 90 testes: 89 verdes, 1 não executável na sessão (integração que exige Docker).
3. **Conversão de sentinelas em provas de contrato** — dois testes que registravam um defeito de rota (405) viraram as provas da correção (PUT por id → 200 com corpo atualizado; DELETE por id → 204), atendendo a critério de aceite literal da issue de testes #15.
4. **Análise dos services novos** — leitura de concorrência, transacionalidade, fluxo de estado e consistência de contrato.
5. **Medição de cobertura** — JaCoCo executado na rodada, lido pacote a pacote e comparado com a medição anterior.
6. **Entrega pelo fluxo novo** — branch de QA → pull request → CI verde → revisão de code owner solicitada.

## Achados da rodada (sanitizados)

Contagem real registrada no relatório do sistema: **5 achados fechados por correção verificada em execução** (incluindo o crítico de cadastro bloqueado por anotação de validação incompatível e a rota de atualização/exclusão inacessível) e **9 achados novos**, sendo:

| Severidade | Achados | Destaques |
|---|---|---|
| Crítico | 2 | Boot da aplicação impossível por query malformada na camada de persistência; API de dados clínicos sem autenticação (recorrente) |
| Alto | 6 | Correção de hash de senha calculada e descartada; contagem de estoque com stub que retorna zero; corrida na reserva de capacidade (conta-então-insere); fluxo de doação interrompido sem o artefato principal; exame sem aplicar resultado ao estado; reserva duplicada/horário passado não impedidos |
| Médio | 2 | CI que compila sem rodar testes; campo numérico aceitando negativo |
| Baixo | 1 | Políticas de exclusão divergentes entre entidades |

A técnica de análise por trás de cada família:

- **Defeitos binários de boot** — leitura da query + conhecimento do comportamento do framework na validação de repositórios: qualquer execução com banco real reproduz. O CI não pega porque não roda testes — registrado como achado próprio.
- **Corrida de concorrência** — inspeção do padrão conta-então-insere contra o requisito da issue ("proteção em transação e no banco"); o versionamento otimista de registros individuais não protege a capacidade compartilhada.
- **Correção ineficaz** — a correção chegou (dependência certa, código novo), mas o resultado calculado é descartado três linhas antes do uso: classe de defeito que só aparece quando se lê a correção, não o relatório do corrector.
- **Fluxo de estado incompleto** — comparação do critério de aceite da issue com o que o fluxo realmente produz; a regra de intervalo entre doações é inerte porque nada no sistema produz o estado que ela consulta.

## Técnica em destaque: sentinela → prova do contrato

Sentinela é teste que **documenta um defeito vivo** com asserção no comportamento atual errado — para que a suíte fique verde sem esconder o problema, e o defeito nunca se perca de vista. Quando a correção chega, a sentinela inverte o papel: vira a **prova do contrato corrigido**. Padrão genérico:

```kotlin
// ANTES — sentinela do defeito: a rota não declarava o segmento {id}
mockMvc.perform(put("/api/recurso/{id}", id))
    .andExpect(status().isMethodNotAllowed)   // 405: evidência executável do defeito

// DEPOIS — prova do contrato corrigido, critério de aceite da issue de testes
val corpo = RecursoRequest(nome = "exemplo", email = "exemplo@teste.local")
whenever(service.atualizar(id, corpo)).thenReturn(resposta())

mockMvc.perform(
    put("/api/recurso/{id}", id)
        .contentType(MediaType.APPLICATION_JSON)
        .content("""{"nome":"exemplo","email":"exemplo@teste.local"}""")
)
    .andExpect(status().isOk)
    .andExpect(jsonPath("$.campo").value("valor"))
```

Aprendizado de Kotlin+Mockito no caminho (custou dois erros vermelhos): matchers `eq()`/`any()` devolvem `null` placeholder em runtime — parâmetros não-nulos de funções Kotlin lançam NPE e vazam estado para o teste seguinte (`InvalidUseOfMatchers`). Stubbing com **argumento concreto** resolve quando o DTO é data class: a igualdade por valor casa com a instância que o controller desserializa.

## Entrega e rastreabilidade

| Item | Referência |
|---|---|
| Issue de testes que guiou a conversão das sentinelas | #15 (marco M3 de testes) |
| Pull request da rodada | [#28](https://github.com/marcelopinotti/hemonorte/pull/28) — CI build verde, aguardando code owner |
| PR anterior (#27) | Fechado automaticamente ao renomear a branch de origem — lição abaixo |

## Lições de processo

1. **Renomear branch com PR aberto mata o PR** — a API de rename não faz retargeting (só a interface faz). Renomear antes de abrir o PR, ou aceitar PR novo.
2. **Rulesets mudaram o fluxo de entrega de QA** — push direto na develop acabou; o fluxo permanente agora é branch → PR → CI → code owner. O QA documentou o ruleset inteiro para o time.
3. **CI verde ≠ qualidade** — o check obrigatório só compila (`-DskipTests`): o achado crítico de boot passaria direto. Registrado como defeito próprio do pipeline.
4. **Ambiente restrito exige agente pré-carregado** — onde o attach de agente JVM é negado, pré-carregar o agente do Mockito via `JAVA_TOOL_OPTIONS` mantém a suíte executável sem tocar na configuração do projeto (detalhe e armadilhas no relatório de métricas desta rodada).
