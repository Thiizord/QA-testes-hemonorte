# Métricas da rodada — 2026-10-23

Medição real, executada na máquina de QA na data indicada, imediatamente antes da entrega via pull request.

## Execução da suíte

| Bloco | Resultado |
|---|---|
| Regras de domínio (8 classes) | 75 verdes |
| Bean Validation de cadastro (1 classe) | 7 verdes |
| Contratos de API standalone (doador, funcionário, agendamentos) | 16 verdes — incl. 201/400/404/409 e 3 sentinelas |
| Services (slot, aptidão, agendamento) — nova fronteira | 17 verdes — incl. sentinela da reserva duplicada |
| Integração (contexto Spring + banco) | 1 não executável na sessão (exige Docker) |
| **Total** | **116 testes — 115 verdes, 1 pendente de ambiente** |

## Cobertura JaCoCo

| Pacote | Linhas | Instruções | Branches |
|---|---|---|---|
| Domínio (model) | 56,1% (174/310) | 64,3% | 70,1% |
| Enums de domínio | 100% | 100% | — |
| DTOs | 42,9% | 48,6% | — |
| Controllers | 31,6% | 31,5% | 0% |
| Services | 28,6% | 27,5% | 21,8% |
| **Projeto** | **44,1% (334/757)** | **49,9%** | — |

## Evolução contra as medições anteriores

| Rodada | Linhas cobertas | % linhas | O que moveu |
|---|---|---|---|
| 2026-10-14 | 234/468 | 50,0% | primeira medição da história |
| 2026-10-19 | 240/757 | 31,7% | +289 linhas de código de produção sem teste (denominador explodiu) |
| 2026-10-23 | 334/757 | **44,1%** | +94 linhas cobertas com 26 testes novos — sem uma linha nova de produção |

**Leitura:** a queda de 50%→31,7% era crescimento do denominador (código novo sem teste); a recuperação para 44,1% é cobertura real, produzida só por teste. No pacote service, três classes a 100% (agendamento 26/26, aptidão 18/18, slot 10/10); as oito zeradas são justamente as travadas por defeitos de fluxo.

## Técnica: cobertura como mapa de bloqueios

Service a 28,6% não significa "falta de teste" — é inventário do que está bloqueado: coleta (0/20), triagem (0/23), estoque (0/15) e expiração (0/13) dependem de fluxos que o código não implementa (comprovante, transição auditada, contagem real de estoque, cron determinístico). Quando os fluxos fecharem, essas classes ganham teste e a cobertura salta sem esforço extra. Por isso a meta conversável com o time segue sendo limite mínimo apenas no domínio — o único pacote onde coverage diz algo sobre o produto.

## Lições técnicas da rodada

1. **Nome de teste entre crases não aceita ponto e vírgula** no Kotlin — o compilador rejeita; hífen resolve (custou um build para descobrir).
2. **Nunca aninhar stubbing** — criar mock/stub dentro do argumento `thenReturn(...)` de outro `when(...)` produz `UnfinishedStubbing`; a fixture se constrói antes da stubbing.
3. **Fixture real bate mock de entidade** — getters finais e validação no construtor tornaram o mock de slot inútil; datas controladas com o parâmetro de data de referência do serviço provaram as mesmas regras (intervalo 60/90 dias por sexo, limite inclusivo, doação mais recente vence) sem fragilidade.
4. **Matcher não-nulo para Kotlin+Mockito** — o `any()` bruto devolve null em runtime e o check de parâmetro não-nulo explode; um helper com cast unchecked registra o matcher e deixa o null passar.
