# Rundeck para PoC de ETL com Apache Hop

## Visão geral

Rundeck é uma plataforma de automação operacional e runbook automation voltada a executar, organizar e auditar procedimentos em ambientes distribuídos. Para esta PoC, ele deve ser avaliado como uma camada de operação sobre o Apache Hop, não como motor de transformação de dados.

O papel mais adequado do Rundeck seria encapsular comandos `hop-run`, scripts finos ou execuções em container como jobs operacionais, oferecendo interface web, controle de acesso, histórico, logs por execução, parâmetros, notificações e execução sob demanda ou agendada.

## Capacidades relevantes para gerenciar jobs Apache Hop

- Definição de jobs com steps sequenciais, comandos, scripts e plugins.
- Parametrização por opções de job, com valores informados na UI ou via API.
- Execução em nós locais ou remotos, usando modelo de nodes e filtros.
- Agendamento e execução sob demanda de rotinas operacionais.
- Histórico de execuções com status, duração, saída de logs e detalhes por step.
- Controle de acesso por políticas, permitindo separar quem visualiza, executa, edita ou interrompe jobs.
- Armazenamento seguro de chaves e segredos para credenciais operacionais.
- Notificações e integrações com sistemas externos.
- API para disparo de jobs por automações externas.
- Possibilidade de abortar execuções em andamento.

## Pontos fortes

- Forte aderência à operação: UI, histórico, auditoria, permissões e execução controlada são nativos do produto.
- Boa opção para ambientes corporativos que exigem governança, segregação de acesso e rastreabilidade.
- Facilita autosserviço operacional: usuários autorizados podem reexecutar cargas, informar parâmetros e acompanhar logs sem acesso direto ao servidor.
- Modelo de jobs e steps encaixa bem em workflows de ingestão batch, especialmente quando cada carga pode ser representada como comando ou script.
- Suporta execução em infraestrutura heterogênea, o que ajuda quando Hop, bancos, servidores de arquivos e sistemas legados estão distribuídos.
- Permite padronizar runbooks de operação: carga diária, reprocessamento, validação, limpeza, notificação e acionamento manual.

## Limitações e riscos

- Pode ser mais pesado que o necessário para uma PoC pequena focada apenas em orquestrar comandos locais ou containers.
- A configuração de projetos, nodes, ACLs, usuários, plugins e segredos aumenta o custo inicial de implantação.
- O modelo mental é mais orientado a operação/runbooks do que a DAGs de dados; dependências complexas entre pipelines podem ficar menos naturais que em orquestradores declarativos simples.
- Algumas capacidades avançadas podem depender da edição comercial ou de configurações adicionais, devendo ser verificadas antes de assumir uso em produção.
- Reexecução parcial, idempotência e controle de duplicidade continuam sendo responsabilidade do desenho dos workflows Hop e dos comandos executados.
- Interromper uma execução pode encerrar o processo operacional, mas não garante rollback de efeitos já aplicados em bancos, arquivos ou sistemas de destino.
- Se usado apenas como uma casca para `docker run` ou `hop-run`, há risco de concentrar lógica demais em scripts shell em vez de manter a transformação versionada no Hop.

## Como poderia executar `hop-run` ou container

### Execução direta com `hop-run`

Um job Rundeck poderia ter um step de comando chamando o Hop CLI com projeto, environment, run configuration e parâmetros explícitos:

```bash
hop-run \
  --project poc-hop \
  --environment ${option.environment} \
  --file workflows/main.hwf \
  --runconfig ${option.runconfig} \
  --level Basic
```

As opções do job poderiam incluir `environment`, `runconfig`, data de referência, identificador da carga, modo de reprocessamento e nível de log. Isso favorece execução manual controlada e também agendamento.

### Execução via container

Outra alternativa é definir um step que execute Hop em container, montando projeto, configuração e diretórios de log:

```bash
docker run --rm \
  --env HOP_ENVIRONMENT_NAME=${option.environment} \
  --volume /opt/poc-hop/hop/project:/files/project \
  --volume /opt/poc-hop/hop/config:/files/config \
  --volume /var/log/poc-hop:/files/logs \
  apache/hop:latest \
  hop-run \
    --project poc-hop \
    --environment ${option.environment} \
    --file /files/project/workflows/main.hwf \
    --runconfig ${option.runconfig} \
    --level Basic
```

Para a PoC, a execução em container tende a ser mais reproduzível, desde que a versão da imagem seja fixada e os volumes sejam padronizados. Evitar `latest` em cenários comparativos ou produtivos.

## Aderência à PoC

Rundeck é aderente quando a PoC precisa demonstrar operação corporativa dos pipelines: quem pode executar, como reexecutar, onde consultar logs, como auditar execuções e como padronizar runbooks de suporte.

Ele é menos aderente se o objetivo principal for apenas validar uma orquestração leve, declarativa e versionável em YAML com baixo overhead local. Nesse caso, Dagu tende a ser mais simples para comparar rapidamente cenários de agenda, dependências, retries e containers.

Para esta PoC, Rundeck deve ser tratado como candidato forte para o eixo "governança operacional", não necessariamente como a opção mais enxuta para o primeiro experimento.

## Critérios de avaliação específicos

- Esforço para subir Rundeck localmente via Docker Compose.
- Facilidade de versionar jobs e configurações relevantes no repositório.
- Clareza para mapear um workflow Hop para um job Rundeck.
- Suporte prático a parâmetros por ambiente e por execução.
- Qualidade dos logs capturados para falhas em `hop-run`.
- Facilidade de localizar histórico de execuções por carga, data e status.
- Controle de concorrência para evitar sobreposição da mesma ingestão.
- Facilidade de reexecutar uma carga com parâmetros diferentes.
- Modelo de segredos para senhas, tokens e conexões.
- Capacidade de restringir execução por perfil de usuário.
- Facilidade de acionar jobs por API.
- Custo operacional para manter projetos, usuários, ACLs, nodes e backups.
- Aderência a ambientes corporativos restritos, incluindo rede, autenticação e auditoria.
- Licença, edição necessária e impacto de funcionalidades comerciais.

## Recomendação preliminar

Rundeck deve permanecer na avaliação como alternativa robusta para operação de pipelines Apache Hop em ambiente corporativo. Ele é especialmente relevante se a PoC precisar demonstrar RBAC, autosserviço, auditoria, runbooks, histórico operacional e execução controlada por times de operação.

Para a primeira etapa da PoC, a recomendação é implementar um cenário mínimo no Rundeck com um job que execute `hop-run` ou container Hop, receba parâmetros por opções, grave logs persistentes e bloqueie execuções concorrentes da mesma carga. A comparação com Dagu deve medir não apenas capacidade técnica, mas também overhead de instalação, manutenção e operação diária.

Recomendação preliminar: Rundeck é uma boa escolha se governança operacional for requisito central; se a prioridade for simplicidade, portabilidade local e baixo atrito para uma PoC curta, deve ser considerado secundário em relação ao Dagu.

## Fontes oficiais consultadas

- Documentação oficial Rundeck: Jobs, options, histórico, logs e controle de acesso: https://docs.rundeck.com/docs/manual/jobs/
- Documentação oficial Rundeck via Context7: `/rundeck/docs`
