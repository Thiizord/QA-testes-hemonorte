# Métricas da rodada — 2026-10-19

Medição real, executada na máquina de QA na data indicada. Números sem contexto não dizem nada — cada bloco traz a leitura.

## Execução da suíte

| Bloco | Resultado |
|---|---|
| Regras de domínio (8 classes) | 75 verdes |
| Bean Validation de cadastro (1 classe) | 7 verdes — antes eram a evidência vermelha do defeito corrigido |
| Contrato de API standalone (2 classes) | 7 verdes — incluindo as 2 provas de contrato convertidas de sentinelas |
| Integração (contexto Spring + banco) | 1 não executável na sessão (exige Docker) |
| **Total** | **90 testes — 89 verdes, 1 pendente de ambiente** |

## Cobertura JaCoCo

| Pacote | Linhas | Instruções | Branches |
|---|---|---|---|
| Domínio (model) | 55,8% (173/310) | 63,1% | 70,1% |
| Enums de domínio | 100% | 100% | — |
| DTOs | 29,4% (37/126) | 35,3% | — |
| Controllers | 13,3% (13/98) | 14,6% | 0% |
| Services | 0% (0/203) | 0% | 0% |
| **Projeto** | **31,7% (240/757)** | **38,4%** | — |

### Evolução contra a medição anterior

Medição anterior: 50,0% das linhas (234/468); domínio a 60,3% de linhas e 73,2% de branches.

**A queda de 50% para 31,7% é crescimento, não perda:** as linhas efetivamente cobertas subiram (234 → 240), mas a onda de código novo entrou com 289 linhas sem teste direto — 11 services e 7 controllers. A porcentagem caiu porque o denominador cresceu 62%.

Leitura por pacote:

- **Domínio a 55,8%** — continua o dono das regras e o termômetro que importa; caiu de 60,3% porque duas entidades novas entraram sem teste (uma delas sem nenhum teste, 0/23; a outra é a entidade de auditoria, 0/17 — justamente a peça que um achado crítico aponta como nunca emitida).
- **Service a 0%** — os testes de API mockam os services por desenho; a cobertura deles chega quando os testes de service existirem. O roteiro para isso já está definido nas issues de testes do marco M3.
- **Controller a 13,3%** — subiu em valor absoluto (rotas novas testadas), caiu em porcentagem pelo mesmo motivo do geral.

**Conclusão honesta:** cobertura aqui segue sendo termômetro do domínio, não da camada nova. A meta conversável com o time é limite mínimo apenas no pacote de domínio — o único lugar onde coverage diz algo clínico sobre o produto.

## Nota metodológica: medição de cobertura em ambiente restrito

Duas armadilhas de ambiente documentadas durante a medição — úteis para quem reproduzir em sandbox corporativo:

1. **Agente de mock com attach negado** — o mock maker inline do Mockito precisa anexar um agente à JVM; onde o mecanismo é bloqueado, os testes de API falham em massa com `Could not initialize inline Byte Buddy mock maker`. Solução sem tocar no projeto: pré-carregar o agente pela variável `JAVA_TOOL_OPTIONS=-javaagent:<caminho do mockito-core.jar>`.
2. **`-DargLine` no CLI sombreia o JaCoCo** — a propriedade de linha de comando tem precedência sobre a que o `prepare-agent` define; passar qualquer argLine pelo CLI faz a cobertura sumir silenciosamente. A regra: agente de teste entra por `JAVA_TOOL_OPTIONS`; `argLine` pertence ao JaCoCo.

A cobertura só é confiável quando o `jacoco.exec` da rodada tem timestamp fresco e o relatório é regenerado a partir dele — nunca confiar em relatório de build anterior.
