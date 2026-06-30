# PoC ETL com Apache Hop

Este repositório contém uma prova de conceito para uso do Apache Hop como motor de ingestão de dados e para avaliação de alternativas de gestão e orquestração operacional dos jobs ETL.

O objetivo da PoC não é apenas validar a construção de pipelines e workflows no Hop. A proposta é avaliar uma solução executável, versionável e operável para agendamento, parametrização, reexecução, logs, auditoria, controle de concorrência, isolamento de ambiente e suporte à operação diária das cargas.

## Objetivos

- Validar o Apache Hop como motor de execução de pipelines e workflows de ingestão.
- Definir um modelo reproduzível de execução por CLI ou container, preferencialmente com `hop-run`.
- Separar a lógica de ingestão e transformação da camada de orquestração operacional.
- Comparar ferramentas candidatas para gestão de jobs: Dagu, pyinfra e Rundeck.
- Documentar critérios técnicos e operacionais para apoiar uma decisão arquitetural.
- Manter a PoC simples, versionável e executável em ambiente local sempre que possível.

## Escopo da PoC

O escopo inicial contempla:

- Um projeto Apache Hop versionado no repositório.
- Um workflow mínimo, real ou simulado, que represente uma carga de ingestão.
- Parametrização por ambiente e por execução.
- Execução por `hop-run` ou por container do Apache Hop.
- Persistência e consulta de logs de execução.
- Testes equivalentes do mesmo cenário mínimo nas ferramentas candidatas.
- Registro dos comandos necessários para reprodução da PoC.

Ficam fora do escopo inicial:

- Definição da plataforma corporativa definitiva de ETL.
- Implantação produtiva.
- Modelagem completa de dados.
- Gestão avançada de segredos.
- Integrações corporativas profundas, como RBAC, SSO e ferramentas de ITSM, exceto quando necessárias para comparação conceitual.

## Arquitetura Conceitual

A direção arquitetural inicial é tratar o Apache Hop como motor de execução e manter a orquestração em uma camada separada.

```text
Fonte de dados
    |
    v
Apache Hop
    - pipelines
    - workflows
    - environments
    - run configurations
    - parametros
    |
    v
Camada de orquestracao
    - agendamento
    - dependencias
    - retries
    - logs
    - historico
    - controle de concorrencia
    |
    v
Area de destino / camada de persistencia
```

O modelo desejado é que os workflows do Hop possam ser executados de forma explícita, parametrizada e reproduzível:

```bash
hop-run \
  --project poc-hop \
  --environment local \
  --file workflows/main.hwf \
  --runconfig local \
  --level Basic
```

Essa separação reduz acoplamento entre os pipelines de dados e a ferramenta de agenda escolhida, permitindo comparar alternativas sem reescrever a lógica ETL.

## Opções de Orquestração

### Dagu

O Dagu é a hipótese inicial mais alinhada quando o objetivo principal é orquestrar comandos, containers e dependências de jobs com baixo overhead operacional. A abordagem declarativa em YAML, a Web UI, o histórico, os logs e recursos como retry e agendamento tendem a favorecer uma PoC enxuta com Apache Hop.

Resumo detalhado: [resumos/dagu.md](resumos/dagu.md)

### pyinfra

O pyinfra é mais adequado como ferramenta de automação de infraestrutura e provisionamento do que como orquestrador principal de pipelines ETL. Pode apoiar a preparação de servidores, instalação do Hop, configuração de serviços, publicação de artefatos e automação de ambientes.

Resumo detalhado: [resumos/pyinfra.md](resumos/pyinfra.md)

### Rundeck

O Rundeck tende a ser mais adequado quando a prioridade inclui governança operacional, RBAC, autosserviço, runbooks, integrações corporativas e execução controlada por times de operação. Pode ser uma opção forte em ambientes com requisitos mais altos de controle e rastreabilidade operacional.

Resumo detalhado: [resumos/rundeck.md](resumos/rundeck.md)

## Critérios de Avaliação

As ferramentas candidatas devem ser comparadas pelos seguintes critérios:

- Facilidade de acionar `hop-run` localmente ou em container.
- Suporte a parametrização por ambiente e por execução.
- Agendamento nativo e controle de timezone.
- Controle de dependências entre jobs.
- Retry, timeout, cancelamento e reexecução parcial.
- Logs centralizados por execução.
- Histórico, auditoria e rastreabilidade.
- Controle de concorrência para evitar sobreposição de cargas.
- Gestão de segredos e integração com variáveis de ambiente.
- Facilidade de empacotar a PoC com Docker Compose.
- Curva de aprendizado para equipes de dados e operação.
- Aderência a ambientes corporativos restritos.
- Licença, maturidade, manutenção e risco de adoção.

## Estrutura Sugerida do Repositório

```text
.
├── docs/
│   ├── decisao-orquestrador.md
│   ├── criterios-avaliacao.md
│   └── execucao-hop.md
├── hop/
│   ├── project/
│   └── config/
├── orchestrators/
│   ├── dagu/
│   ├── pyinfra/
│   └── rundeck/
├── resumos/
│   ├── dagu.md
│   ├── pyinfra.md
│   └── rundeck.md
├── scripts/
├── docker/
├── AGENTS.md
└── README.md
```

Descrição esperada:

- `docs/`: decisões, critérios de avaliação e guias operacionais.
- `hop/`: projeto, workflows, pipelines e configurações do Apache Hop.
- `orchestrators/`: implementações comparáveis para Dagu, pyinfra e Rundeck.
- `resumos/`: análise objetiva de cada ferramenta candidata.
- `scripts/`: comandos auxiliares finos para execução local.
- `docker/`: arquivos de empacotamento, composição e runtime.

## Como Executar Futuramente

Os comandos finais serão definidos conforme os artefatos forem adicionados à PoC. A direção inicial é suportar pelo menos dois modos de execução.

Execução direta com Apache Hop:

```bash
hop-run \
  --project poc-hop \
  --environment local \
  --file workflows/main.hwf \
  --runconfig local \
  --level Basic
```

Execução via orquestrador:

```bash
# Exemplo futuro com Dagu
dagu start ingestao-hop
```

Execução via container:

```bash
# Exemplo futuro com Docker Compose
docker compose up
```

Também deve ser criado um `.env.example` com as variáveis esperadas para execução local, sem inclusão de segredos reais no repositório.

## Próximos Passos

1. Criar a estrutura inicial de diretórios da PoC.
2. Adicionar um projeto Apache Hop mínimo e executável.
3. Definir um workflow de ingestão simples, parametrizado e idempotente.
4. Documentar a execução local com `hop-run`.
5. Criar um cenário mínimo equivalente em Dagu.
6. Avaliar o papel do pyinfra para provisionamento e publicação da PoC.
7. Criar um cenário mínimo em Rundeck, caso a avaliação de governança operacional seja necessária.
8. Registrar evidências de execução, logs, limitações e esforço operacional.
9. Consolidar a decisão em `docs/decisao-orquestrador.md`.

## Status Atual

Status: fase inicial de estruturação.

Até o momento, o repositório contém o contexto arquitetural e operacional da PoC em `AGENTS.md`. Os artefatos de Hop, Docker, scripts e configurações dos orquestradores ainda precisam ser criados.

## Referências

- Apache Hop: https://hop.apache.org/
- Dagu: https://dagu.sh/pt
- pyinfra: https://pyinfra.com/
- Rundeck: https://www.rundeck.com/
