# pyinfra na PoC de ETL com Apache Hop

## Visão geral

pyinfra é uma ferramenta de automação de infraestrutura e deploy escrita em Python. Seu modelo é baseado em inventários, conectores, fatos do ambiente e operações declaradas em código Python, que são convertidas em comandos executados em alvos locais, servidores via SSH ou containers Docker.

No contexto desta PoC com Apache Hop, pyinfra deve ser avaliado principalmente como ferramenta de apoio à preparação e padronização do ambiente operacional, não como orquestrador principal de pipelines ETL. Ele pode ajudar a instalar dependências, distribuir artefatos, configurar diretórios, validar pré-requisitos e executar comandos controlados, mas não oferece nativamente o conjunto típico de gestão de jobs batch esperado de um scheduler/orquestrador.

## Capacidades relevantes para Apache Hop

- Execução de operações em máquina local, servidores remotos via SSH e containers Docker.
- Inventários versionáveis para representar ambientes como `local`, `dev`, `homolog` e `prod`.
- Operações declarativas para instalação de pacotes, criação de usuários, diretórios, arquivos, permissões e serviços.
- Coleta de fatos do sistema para adaptar a configuração ao ambiente alvo.
- Execução de comandos shell, o que permite acionar `hop-run` quando necessário.
- Escrita de automações em Python, facilitando reaproveitamento de lógica e composição com validações.
- Possibilidade de uso em pipelines de CI/CD para preparar servidores ou publicar versões do projeto Hop.

## Pontos fortes

- Baixo acoplamento com uma plataforma operacional específica.
- Boa aderência a provisionamento reproduzível e versionado.
- Uso de Python, linguagem familiar para muitas equipes de dados e automação.
- Suporte prático a SSH, execução local e Docker, útil para validar diferentes topologias da PoC.
- Modelo adequado para garantir estado desejado de arquivos, pacotes, diretórios e configurações.
- Pode reduzir passos manuais na instalação do Hop, preparação de Java, publicação de projetos e criação de estrutura de logs.

## Limitações e riscos como orquestrador ETL

- Não é, por natureza, um scheduler de jobs ETL.
- Não fornece Web UI nativa voltada a operação diária de pipelines, reexecução, acompanhamento de histórico e navegação por logs de cargas.
- Não entrega, de forma central, dependências entre jobs, retries operacionais, calendário, timezone, SLA, filas e controle de concorrência de cargas.
- A execução de `hop-run` por pyinfra é possível, mas tende a misturar provisionamento com operação recorrente de dados.
- A rastreabilidade de execuções ETL dependeria de convenções adicionais de log, scripts, armazenamento externo ou integração com outra ferramenta.
- Pode induzir a uma solução customizada de orquestração se for usado além do seu papel natural, aumentando manutenção e risco operacional.

## Como poderia apoiar instalação, configuração e execução do `hop-run`

pyinfra pode apoiar a PoC em tarefas como:

- Instalar Java e dependências de sistema exigidas pelo Apache Hop.
- Criar usuário técnico para execução de cargas.
- Criar diretórios padronizados para instalação, projetos, configurações, dados temporários e logs.
- Distribuir ou atualizar o projeto Hop versionado no servidor alvo.
- Publicar arquivos de environment, run configurations e parâmetros por ambiente.
- Validar pré-requisitos antes da execução, como versão do Java, presença do binário `hop-run`, permissões e variáveis de ambiente.
- Executar comandos pontuais de smoke test, por exemplo:

```bash
hop-run \
  --project poc-hop \
  --environment local \
  --file workflows/main.hwf \
  --runconfig local \
  --level Basic
```

O uso mais adequado é tratar essa execução como validação de deploy ou tarefa administrativa. Para execução recorrente de pipelines, o comando deveria ser disparado por um orquestrador ou scheduler mais apropriado.

## Aderência à PoC

pyinfra tem boa aderência à parte de provisionamento e padronização operacional da PoC. Ele ajuda a tornar reproduzível a preparação dos ambientes onde o Apache Hop será executado, principalmente se houver servidores Linux, containers ou ambientes segregados por estágio.

Sua aderência é baixa como solução principal para gestão de jobs ETL. A PoC busca avaliar agendamento, parametrização por execução, reexecução, logs, auditoria, concorrência e facilidade de operação. Esses pontos estão mais próximos de ferramentas como Dagu ou Rundeck do que de pyinfra.

Assim, pyinfra deve ser considerado uma ferramenta complementar. Ele pode preparar o terreno para que Dagu, Rundeck ou outro orquestrador execute o `hop-run` de forma controlada.

## Critérios de avaliação específicos

- Clareza para representar ambientes `local`, `dev`, `homolog` e `prod` em inventários.
- Facilidade de instalar e atualizar Apache Hop, Java e dependências.
- Capacidade de publicar projeto Hop, environments, run configurations e arquivos auxiliares sem passos manuais.
- Segurança no tratamento de variáveis de ambiente e segredos, evitando conteúdo sensível versionado.
- Idempotência das operações de instalação e configuração.
- Simplicidade para executar validações pós-deploy com `hop-run`.
- Integração com Docker Compose ou CI/CD para preparar ambientes locais e remotos.
- Esforço de manutenção dos scripts Python de automação.
- Separação clara entre automação de infraestrutura e operação recorrente dos jobs ETL.
- Compatibilidade com restrições corporativas de acesso SSH, privilégios, proxy, instalação de pacotes e auditoria.

## Recomendação preliminar

Recomenda-se manter pyinfra na PoC como ferramenta candidata para automação de infraestrutura, provisionamento e publicação de artefatos do Apache Hop, não como orquestrador ETL principal.

Para a gestão diária dos jobs, Dagu tende a ser mais alinhado quando a prioridade for baixo overhead, YAML declarativo, execução de comandos, dependências, logs, retries e histórico. Rundeck deve ser considerado quando governança operacional, RBAC, autosserviço e runbooks forem requisitos centrais.

O melhor papel preliminar para pyinfra é complementar: preparar servidores, validar ambientes, instalar dependências, distribuir configurações e executar testes controlados de `hop-run`. Usá-lo como scheduler principal exigiria construir capacidades operacionais que não são seu foco, o que aumentaria risco e custo de manutenção da solução.
