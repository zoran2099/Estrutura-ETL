# Dagu para PoC de ETL com Apache Hop

## Visão geral

Dagu é um orquestrador de workflows leve, autocontido e orientado a DAGs declarativas em YAML. Ele se posiciona como alternativa simples a cron e a orquestradores mais pesados, com execução de comandos, containers Docker, jobs Kubernetes, comandos remotos via SSH, Web UI, histórico de execuções e logs por workflow.

Para esta PoC, o ponto mais relevante é que o Dagu pode atuar como camada operacional externa ao Apache Hop: o Hop continua sendo o motor de execução dos pipelines e workflows, enquanto o Dagu agenda, parametriza, encadeia, registra e reexecuta chamadas ao `hop-run` ou a containers com Hop.

## Capacidades relevantes para jobs Apache Hop

- Definição de workflows em YAML, versionável junto ao repositório.
- Execução de etapas por comando shell usando `run`.
- Execução de etapas em container Docker, permitindo isolar a versão do Apache Hop e dependências.
- Dependências explícitas entre etapas por `depends`.
- Parâmetros declarativos por workflow, com tipos, valores padrão, obrigatoriedade e listas permitidas.
- Resolução de valores como `${params.nome}`, `${env.NOME}` e saídas de etapas anteriores.
- Política de retry por etapa com limite e intervalo.
- Web UI para acompanhar execuções, status, histórico e logs.
- Scheduler para execuções recorrentes.
- Possibilidade de compor workflows maiores por sub-DAGs, usando uma DAG para chamar outra.

Essas capacidades cobrem bem o cenário inicial da PoC: executar cargas batch do Hop de forma reprodutível, com parâmetros explícitos, logs consultáveis e controle simples de dependências.

## Pontos fortes

- Baixo overhead operacional: a proposta de ferramenta autocontida reduz a quantidade de componentes necessários para iniciar a PoC.
- Boa aderência a CLI: o modelo de execução por comandos combina diretamente com `hop-run`.
- Declarativo e versionável: os DAGs em YAML facilitam revisão, reprodução e evolução incremental.
- Bom encaixe com containers: a execução de etapas em Docker favorece isolamento de ambiente e padronização da versão do Hop.
- Curva de aprendizado menor que orquestradores corporativos mais completos.
- Web UI e histórico ajudam a operação sem exigir que toda investigação seja feita apenas por terminal.
- Permite começar pequeno, com um workflow mínimo, e evoluir para dependências, retries, parâmetros e sub-DAGs.

## Limitações e riscos

- Governança corporativa pode ser limitada quando comparada a plataformas como Rundeck, especialmente para RBAC avançado, autosserviço operacional, integrações corporativas e trilhas formais de aprovação.
- O modelo leve e local exige atenção ao desenho de persistência, backup, retenção de logs e histórico.
- Deve-se validar com cuidado controle de concorrência, cancelamento, reexecução parcial e comportamento em falhas do host ou do container.
- Para ambientes restritos, é necessário confirmar políticas de instalação, execução de Docker, rede, armazenamento e gestão de segredos.
- Não substitui controles de idempotência no próprio desenho ETL. Retries e reexecuções podem causar duplicidade se os pipelines Hop não forem preparados para isso.
- A maturidade operacional precisa ser avaliada na prática com cenários reais de falha, carga longa, concorrência e reprocessamento.

## Como poderia executar `hop-run` ou container

Um DAG simples pode acionar diretamente o `hop-run` instalado no host:

```yaml
params:
  - name: environment
    type: string
    default: local
  - name: workflow
    type: string
    default: workflows/main.hwf

steps:
  - name: executar_hop
    run: >
      hop-run
      --project poc-hop
      --environment ${params.environment}
      --file ${params.workflow}
      --runconfig local
      --level Basic
    retry_policy:
      limit: 2
      interval_sec: 60
```

Também é possível executar em container para reduzir dependências no host:

```yaml
params:
  - name: environment
    type: string
    default: local

steps:
  - name: executar_hop_container
    container:
      image: apache/hop:latest
    run: >
      hop-run
      --project poc-hop
      --environment ${params.environment}
      --file workflows/main.hwf
      --runconfig local
      --level Basic
```

Na PoC, a abordagem com container tende a ser preferível se o objetivo for reprodutibilidade. A abordagem com `hop-run` local pode ser útil para desenvolvimento rápido, desde que a instalação do Hop, variáveis de ambiente e caminhos sejam bem documentados.

## Aderência à PoC

Dagu é bem aderente ao objetivo inicial da PoC porque separa claramente a orquestração operacional da lógica de transformação. O Hop permanece responsável pelos pipelines e workflows de dados; o Dagu fica responsável por agenda, dependências, parâmetros, retries, logs e histórico.

Essa separação permite testar a operação real de cargas batch sem introduzir cedo demais uma plataforma corporativa pesada. Também favorece comparação objetiva com Rundeck e pyinfra, pois o mesmo workflow Hop pode ser chamado por cada ferramenta candidata.

## Critérios de avaliação específicos

- Facilidade para executar `hop-run` com projeto, environment, run configuration e parâmetros explícitos.
- Facilidade para executar Hop em container com volumes, variáveis de ambiente e logs persistidos.
- Clareza do YAML para equipe de dados e operação.
- Suporte a agendamento recorrente e controle de timezone.
- Capacidade de impedir sobreposição de cargas do mesmo pipeline.
- Comportamento de retry, timeout, cancelamento e reexecução após falha.
- Qualidade dos logs por execução e facilidade de diagnóstico.
- Retenção e consulta de histórico para auditoria operacional.
- Suporte a parâmetros por ambiente e por execução manual.
- Integração com variáveis de ambiente e estratégia segura para segredos.
- Esforço para empacotar com Docker Compose.
- Comportamento em falhas do host, reinício do serviço e execuções longas.
- Maturidade, manutenção do projeto, documentação e risco de adoção.

## Recomendação preliminar

Dagu deve ser tratado como candidato principal para a primeira iteração da PoC. Ele parece bem alinhado ao problema imediato: orquestrar comandos e containers do Apache Hop com baixo overhead, configuração versionável, UI operacional, logs, histórico, dependências e retries.

A recomendação é implementar um cenário mínimo com Dagu antes de decidir: um workflow Hop parametrizado, execução manual, execução agendada, retry controlado, simulação de falha, reexecução e prevenção de concorrência. Se esses pontos funcionarem com baixa complexidade e boa observabilidade, Dagu tende a ser uma escolha pragmaticamente adequada para a PoC.

Para produção corporativa, a decisão deve permanecer condicionada à validação de segurança, RBAC, auditoria, retenção, backup, operação em alta disponibilidade e integração com os padrões internos da organização.
