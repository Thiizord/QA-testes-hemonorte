# QA testes hemonorte

Portfólio das atividades de **Quality Assurance** no projeto **Hemonorte** — sistema de gestão de doação de sangue (hemocentro) — e repositório dos artefatos de QA produzidos no exercício da função: análises, gestão de defeitos, métricas e aprendizados.

> **Importante:** este repositório é independente do repositório do sistema e **não contém código-fonte, credenciais, dados pessoais nem informações confidenciais** do projeto Hemonorte. Todo artefato passa por um filtro de sanitização antes de entrar — detalhes em [docs/escopo-de-qa.md](docs/escopo-de-qa.md).

## Contexto

O Hemonorte é um backend Kotlin + Spring Boot (com PostgreSQL, Flyway e Testcontainers) que suporta a jornada de doação de sangue: cadastro de doadores e funcionários, agendamento, triagem clínica, coleta, exames sorológicos, estoque e expiração de bolsas. O QA atua junto aos desenvolvedores em fluxo Git com pull requests, revisão obrigatória de code owner e CI no GitHub.

## O que este repositório documenta

| Categoria | Conteúdo |
|---|---|
| Escopo e estratégia | O que QA faz no projeto, o filtro de confidencialidade e o ritual de atualização pós-atividade |
| Requisitos e rastreabilidade | Matrizes issues↔testes, análise de lacunas e vereditos de implementação dos marcos de teste |
| Gestão de defeitos | Resumos sanitizados das rodadas: achados por severidade, técnica aplicada e rastreabilidade |
| Métricas e relatórios | Resultados de execução da suíte, cobertura JaCoCo real e a leitura de cada número |
| Aprendizados | Lições de técnica e de processo, registradas conforme surgem |

## Metodologias e ferramentas efetivamente utilizadas

- **Testes unitários de domínio** — regras de negócio isoladas, sem infraestrutura (JUnit Jupiter + kotlin.test)
- **Testes de contrato de API** — camada HTTP com MockMvc standalone e serviços mockados (Mockito)
- **Testes de service** — orquestração e regras de aplicação com repositórios mockados e fixtures reais
- **Bean Validation** — testes diretos no validador, usados como evidência executável de defeitos de anotação
- **Sentinelas** — testes que documentam defeitos vivos pelo comportamento atual e viram as provas do contrato quando a correção chega
- **Cobertura** — JaCoCo medido a cada rodada, lido pacote a pacote e comparado com a rodada anterior
- **Integração (planejada)** — Testcontainers/PostgreSQL para migrações, unicidade e concorrência reais
- **Gestão de defeitos** — relatório vivo no repositório do sistema, com identificação estável (QA-XXX), severidade, evidência e correção sugerida
- **Colaboração** — issues e milestones como roteiro; branch de QA → pull request → CI → aprovação de code owner → merge

## Práticas de rastreabilidade

Cada rodada referencia: as **issues** que a guiaram, os **pull requests** de entrega e os **IDs estáveis de defeito** (QA-XXX) — mantendo o elo entre o que foi pedido, o que foi testado e o que foi encontrado.

## Confidencialidade

- Nenhum arquivo é copiado verbatim do repositório do sistema.
- Artefatos de teste entram apenas se independentes do código do sistema; técnicas são demonstradas com exemplos genéricos.
- Métricas são reais e sempre acompanhadas do contexto de medição.
- Identificadores internos, segredos e dados sensíveis não entram; na dúvida, sanitiza ou não publica.

## Estrutura

```
├── README.md
├── docs/                         escopo, estratégia e processos
├── requisitos-e-rastreabilidade/ matrizes issues↔testes e vereditos de marcos
├── gestao-de-defeitos/           rodadas sanitizadas: achados, técnicas, rastreabilidade
└── relatorios-e-metricas/        execuções, cobertura e evolução
```

Pastas novas nascem apenas quando existir conteúdo real para elas — sem pastas vazias e sem duplicação.
