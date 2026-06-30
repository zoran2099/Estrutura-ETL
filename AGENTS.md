# AGENTS.md

## Contexto do Projeto

Este projeto é uma prova de conceito para uso do Apache Hop como solução de ingestão de dados.

O foco não é apenas construir pipelines e workflows no Hop, mas validar uma forma operacional de gerenciar a execução desses jobs: agendamento, parametrização, reexecução, logs, auditoria, controle de concorrência, isolamento de ambiente e facilidade de operação.

Atue como especialista em ETL, integração de dados e operação de pipelines batch.

## Direção Arquitetural Inicial

- Tratar o Apache Hop como o motor de execução dos pipelines e workflows.
- Preferir execução reproduzível por CLI ou container, usando `hop-run`, projetos, environments, run configurations e parâmetros explícitos.
- Separar a orquestração operacional da lógica de transformação/ingestão.
- Versionar workflows, pipelines, configurações e exemplos de execução.
- Evitar acoplamento desnecessário entre a PoC e uma ferramenta de agenda específica até que os critérios estejam claros.

## Opções Iniciais Para Gestão de Jobs

As opções iniciais em avaliação são:

- Dagu: https://dagu.sh/pt
- pyinfra: https://pyinfra.com/
- Rundeck: https://www.rundeck.com/

### Hipótese Inicial

Para uma PoC de ingestão com Apache Hop, Dagu tende a ser a opção mais alinhada quando o objetivo principal é orquestrar comandos, containers e dependências de jobs com baixo overhead operacional, YAML declarativo, Web UI, logs, retries e histórico.

Rundeck tende a ser mais adequado se o objetivo principal incluir governança operacional, RBAC, autosserviço, runbooks, integrações corporativas e execução por times de operação.

pyinfra tende a ser mais adequado como ferramenta de automação de infraestrutura e provisionamento, não como orquestrador principal de pipelines ETL. Pode ser útil para preparar servidores, instalar Hop, configurar serviços, publicar artefatos e automatizar ambientes.

## Critérios de Avaliação

Ao comparar as ferramentas, usar estes critérios:

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
- Curva de aprendizado para equipe de dados e operação.
- Aderência a ambientes corporativos restritos.
- Licença, maturidade, manutenção e risco de adoção.

## Modelo de Execução Desejado Para Hop

Preferir exemplos que sigam este padrão:

- Projeto Hop versionado no repositório.
- Environment por ambiente alvo, por exemplo `local`, `dev`, `homolog` e `prod`.
- Run configuration explícita, por exemplo `local`.
- Parâmetros de execução documentados.
- Execução por `hop-run` ou container oficial do Apache Hop.
- Logs persistidos fora do container quando aplicável.
- Scripts finos de entrada apenas quando simplificarem a operação.

Exemplo conceitual:

```bash
hop-run \
  --project poc-hop \
  --environment local \
  --file workflows/main.hwf \
  --runconfig local \
  --level Basic
```

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
├── scripts/
└── docker/
```

## Boas Práticas Para Implementação

- Começar pequeno: um workflow Hop real ou simulado, parametrizado e executável.
- Criar o mesmo cenário mínimo nas ferramentas candidatas antes de decidir.
- Medir esforço operacional, não apenas capacidade técnica.
- Documentar comandos exatos para reprodução.
- Preferir arquivos declarativos e versionáveis.
- Não guardar segredos no repositório.
- Usar `.env.example` para variáveis esperadas.
- Manter a PoC executável em máquina local sempre que possível.

## Instruções de Trabalho

- Antes de alterar arquitetura ou dependências, inspecione o estado atual do repositório.
- Use Context7 MCP para documentação atual quando houver dúvidas sobre Apache Hop, Dagu, pyinfra, Rundeck, Docker, Java, Python ou outras ferramentas, SDKs, CLIs e frameworks.
- Para fontes específicas informadas por URL, verifique a documentação oficial quando a decisão depender de capacidades atuais.
- Ao propor decisões, explicite tradeoffs técnicos e operacionais.
- Para revisões, priorize riscos de operação ETL: perda/duplicidade de dados, idempotência, reprocessamento, observabilidade, recuperação de falhas e controle de concorrência.

