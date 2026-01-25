🔧 Sistema de Roteamento (Dispatcher)
tasks/dispatcher/main.yml
Propósito: Sistema de roteamento centralizado que direciona as requisições para os módulos e operações específicas.

Funcionalidades:

Validação de Módulos: Verifica se o módulo solicitado existe e é suportado

Validação de Operações: Confirma se a operação é válida para o módulo

Roteamento Dinâmico: Usa include_tasks para carregar o arquivo YAML específico

Passagem de Parâmetros: Transfere variáveis específicas do módulo

Fluxo:
main.yml → dispatcher/main.yml → modules/[módulo]/[operação].yml

1. 📋 CATALOG TASK (SCTASK)
   Visão Geral
   As Catalog Tasks são tarefas específicas geradas a partir de solicitações do catálogo de serviços (RITMs). Representam atividades técnicas que precisam ser executadas para cumprir uma solicitação.

Arquivos e Funcionalidades

tasks/modules/sc_task/create.yml
Objetivo: Criar uma nova Catalog Task
Parâmetros Obrigatórios:

sc_task_short_description: Descrição curta da tarefa
Campos Principais:

state: Estado inicial (open, work_in_progress, etc.)

assignment_group: Grupo responsável

request_item: Relacionamento com RITM
Uso: Criar tarefas técnicas para implementação de serviços

tasks/modules/sc_task/update.yml
Objetivo: Atualizar uma Catalog Task existente
Parâmetro Chave: sc_task_sys_id (identificador único)
Campos Atualizáveis: Todos os campos da tabela sc_task
Uso: Atualizar progresso, atribuições ou informações da tarefa

tasks/modules/sc_task/assign_user.yml
Objetivo: Atribuir a tarefa a um usuário específico
Parâmetros:

sc_task_sys_id: ID da tarefa

sc_task_assigned_to: Usuário de destino
Ação: Atualiza assigned_to e limpa assignment_group

tasks/modules/sc_task/assign_group.yml
Objetivo: Atribuir a tarefa a um grupo
Parâmetros:

sc_task_sys_id: ID da tarefa

sc_task_assignment_group: Grupo de destino
Ação: Atualiza assignment_group e limpa assigned_to

tasks/modules/sc_task/change_status.yml
Objetivo: Mudar o estado da tarefa
Estados Suportados:

open (Aberta)

work_in_progress (Em progresso)

closed_complete (Fechada completa)

closed_incomplete (Fechada incompleta)

pending (Pendente)
Campos Específicos: close_code, close_notes para estados fechados

tasks/modules/sc_task/add_comment.yml
Objetivo: Adicionar comentário visível ao solicitante
Campo: comments
Uso: Comunicação com o usuário final

tasks/modules/sc_task/add_work_note.yml
Objetivo: Adicionar nota de trabalho interna
Campo: work_notes
Uso: Anotações internas da equipe

tasks/modules/sc_task/attach_file.yml
Objetivo: Anexar arquivo à tarefa
Integração: Usa módulo servicenow.itsm.attachment
Parâmetros:

table_name: "sc_task"

table_sys_id: ID da tarefa

path: Caminho do arquivo local

tasks/modules/sc_task/relate_to_ritm.yml
Objetivo: Estabelecer relação com RITM
Campo: request_item
Uso: Vincular tarefa à solicitação que a originou

tasks/modules/sc_task/close_technical.yml
Objetivo: Encerrar tarefa técnica com documentação completa
Campos Obrigatórios:

close_code: Código de fechamento (solved, not_solved, etc.)

close_notes: Descrição da resolução
Ação: Muda estado para closed_complete

tasks/modules/sc_task/other_fields.yml
Objetivo: Atualizar campos personalizados ou adicionais
Mecanismo: Recebe dicionário sc_task_other_fields com campos dinâmicos
Uso: Para campos específicos da implementação do cliente

2. 🛒 REQUESTED ITEM (RITM)
   Visão Geral
   Requested Items são itens solicitados através do Catálogo de Serviços. Cada RITM representa uma solicitação específica de um item do catálogo e pode gerar múltiplas Catalog Tasks.

Arquivos e Funcionalidades
tasks/modules/ritm/create.yml
Objetivo: Criar um item solicitado via Catálogo de Serviços
Parâmetro Chave: ritm_cat_item (item do catálogo)
Características Especiais:

variables: Dicionário com variáveis do catálogo

quantity: Quantidade solicitada

request: Relacionamento com Request (REQ)
Uso: Automatizar solicitações de catálogo

tasks/modules/ritm/update.yml
Objetivo: Atualizar um RITM existente
Campo Especial: stage (estágio no fluxo de aprovação)

tasks/modules/ritm/view_catalog_vars.yml
Objetivo: Visualizar variáveis preenchidas no catálogo
Tecnologia: Consulta à API com sysparm_display_value: true
Saída: Exibe todas as variáveis e seus valores

tasks/modules/ritm/generate_sc_task.yml
Objetivo: Gerar Catalog Tasks a partir do RITM
Mecanismo: Usa with_sequence para múltiplas tasks
Herança: Copia informações do RITM para as tasks
Uso: Automatizar criação de fluxos de trabalho

tasks/modules/ritm/relate_to_req.yml
Objetivo: Relacionar RITM com Request
Campo: request
Uso: Consolidar múltiplos RITMs em um Request

3. 📦 REQUEST (REQ)
   Visão Geral
   Requests são pedidos de serviço que agregam múltiplos RITMs. Representam uma solicitação completa do usuário que pode conter vários itens do catálogo.

Arquivos e Funcionalidades
tasks/modules/req/create.yml
Objetivo: Criar um novo Request
Parâmetro Chave: req_requested_for (usuário solicitante)
Características:

requested_by: Quem está fazendo a solicitação

category/subcategory: Categorização do pedido
Uso: Criar solicitações de serviço complexas

tasks/modules/req/view_status.yml
Objetivo: Visualizar status consolidado do Request
Funcionalidades:

Consulta informações básicas do Request

Lista todos os RITMs associados

Mostra status individual de cada RITM
Saída: Relatório consolidado do pedido

tasks/modules/req/aggregate_ritms.yml
Objetivo: Agregar múltiplos RITMs ao Request
Mecanismo: Loop sobre lista de ritm_sys_ids
Uso: Consolidar solicitações dispersas

tasks/modules/req/close_with_ritms.yml
Objetivo: Fechar Request automaticamente baseado no status dos RITMs
Lógica:

Verifica status de todos os RITMs associados

Fecha Request apenas se TODOS os RITMs estiverem fechados

Adiciona notas explicativas
Uso: Encerramento automático de pedidos

tasks/modules/req/view_history.yml
Objetivo: Visualizar histórico completo do Request
Consultas:

sys_history_line: Histórico de alterações de campos

sys_journal_field: Histórico de comentários
Saída: Linha do tempo completa do pedido

4. 🚨 INCIDENT (INC)
   Visão Geral
   Incidentes são interrupções não planejadas ou degradações de serviço. Seguem o processo de gerenciamento de incidentes do ITIL.

Arquivos e Funcionalidades
tasks/modules/incident/create.yml
Objetivo: Criar um novo incidente
Parâmetro Chave: incident_short_description
Campos de Classificação:

severity: Severidade (1=Crítico, 2=Alto, 3=Médio, 4=Baixo)

urgency: Urgência (1=Alta, 2=Média, 3=Baixa)

impact: Impacto (1=Alto, 2=Médio, 3=Baixo)

priority: Calculado automaticamente baseado em urgência/impacto
Uso: Abertura automática de incidentes via monitoramento

tasks/modules/incident/change_status.yml
Objetivo: Mudar estado do incidente
Mapeamento de Estados:

text
1: New (Novo)
2: In Progress (Em progresso)
3: On Hold (Em espera)
4: Resolved (Resolvido)
5: Closed (Fechado)
6: Canceled (Cancelado)
7: Approved (Aprovado)
Campos Específicos:

close_code: Código de fechamento

close_notes: Notas de fechamento

tasks/modules/incident/set_impact_urgency.yml
Objetivo: Definir/atualizar classificação do incidente
Cálculo de Prioridade: Baseado em matriz urgência×impacto
Uso: Reclassificação durante triagem

tasks/modules/incident/associate_ci.yml
Objetivo: Associar Configuration Item (CMDB)
Campo: cmdb_ci
Ação: Relaciona incidente ao item de configuração afetado
Consulta: Busca informações do CI na CMDB

tasks/modules/incident/resolve_close.yml
Objetivo: Fluxo completo de resolução e fechamento
Dois Passos:

Resolução (state: 4)

Fechamento (state: 5) - opcional com incident_auto_close
Campos Obrigatórios: close_code, close_notes

tasks/modules/incident/relate_to_problem.yml
Objetivo: Relacionar incidente a um Problem
Campo: problem_id
Uso: Vincular incidentes sintomas a problemas causais

tasks/modules/incident/user_communication.yml
Objetivo: Registrar comunicação com usuário
Diferença:

comments: Visível ao usuário

work_notes: Interno da equipe
Uso: Documentar interações de suporte

5. 🔍 PROBLEM (PRB)
   Visão Geral
   Problems representam a causa raiz de um ou mais incidentes. Focam na identificação e resolução permanente da causa subjacente.

Arquivos e Funcionalidades
tasks/modules/problem/create.yml
Objetivo: Criar um novo Problem
Diferenciação: Foco em investigação, não em restauração
Campos Especiais: known_error (indicador de erro conhecido)

tasks/modules/problem/change_status.yml
Objetivo: Mudar estado do Problem
Estados do Fluxo Problem:

new: Novo

assess: Em avaliação

root_cause_analysis: Análise de causa raiz

fix_in_progress: Correção em andamento

resolved: Resolvido

closed: Fechado

known_error: Erro conhecido
Campos Específicos: fix_notes, cause_notes

tasks/modules/problem/relate_incidents.yml
Objetivo: Relacionar múltiplos incidentes ao Problem
Mecanismo: Loop sobre incident_sys_ids
Uso: Agrupar incidentes relacionados à mesma causa

tasks/modules/problem/record_root_cause.yml
Objetivo: Registrar causa raiz identificada
Campo: root_cause
Ação: Muda estado para root_cause_analysis
Uso: Documentar resultado da investigação

tasks/modules/problem/create_known_error.yml
Objetivo: Criar registro de Known Error
Ação:

Marca known_error: true

Muda estado para known_error

Preenche known_error_date
Uso: Documentar workarounds e soluções temporárias

tasks/modules/problem/generate_change.yml
Objetivo: Gerar Change Request para corrigir causa raiz
Herança: Copia informações do Problem para a Change
Relacionamento: problem_id na Change
Uso: Automatizar criação de mudanças corretivas

tasks/modules/problem/close_problem.yml
Objetivo: Encerrar Problem após correção
Ações Adicionais:

Atualiza incidentes relacionados

Registra data de fechamento

Preenche fix_notes com solução permanente

6. 🔄 CHANGE (CHG)
   Visão Geral
   Changes representam mudanças planejadas no ambiente de TI. Seguem processos de controle de mudanças para minimizar riscos.

Arquivos e Funcionalidades
tasks/modules/change/create.yml
Objetivo: Criar uma nova Change Request
Campos de Classificação:

type: Tipo (normal, standard, emergency)

risk: Risco (low, moderate, high)

impact: Impacto (low, medium, high)
Campos de Planejamento:

start_date: Data de início planejada

end_date: Data de término planejada

tasks/modules/change/change_status.yml
Objetivo: Mudar estado da Change
Estados do Fluxo Change:

new: Nova

assess: Em avaliação

authorize: Em autorização

scheduled: Agendada

implement: Em implementação

review: Em revisão

closed: Fechada
Campos Específicos por Estado:

implementation_plan: Plano de implementação

review_comments: Comentários da revisão

tasks/modules/change/set_change_type.yml
Objetivo: Definir tipo de change com configurações automáticas
Lógica por Tipo:

Normal: Risco médio, aprovação padrão

Standard: Risco baixo, aprovação simplificada

Emergency: Risco alto, aprovação acelerada
Configurações Automáticas: Categoria, risco, urgência

tasks/modules/change/assess_risk_impact.yml
Objetivo: Avaliar formalmente risco e impacto
Campos de Análise:

risk_impact_analysis: Análise detalhada de risco

impact_analysis: Análise de impacto
Uso: Documentação para aprovação

tasks/modules/change/manage_approvals.yml
Objetivo: Gerenciar processo de aprovação
Campos:

approval: Status (requested, approved, rejected)

approval_set: Conjunto de aprovadores
Ação: Muda estado para authorize

tasks/modules/change/plan_dates.yml
Objetivo: Planejar datas de implementação
Validações:

Formato YYYY-MM-DD

Data início < Data fim

Cálculo automático de duração
Campos Relacionados: maintenance_schedule

tasks/modules/change/execute_implementation.yml
Objetivo: Executar implementação da change
Ações:

Muda estado para implement

Registra implemented_at

Cria Catalog Task de implementação (opcional)
Uso: Iniciar janela de implementação

tasks/modules/change/post_implementation_review.yml
Objetivo: Registrar revisão pós-implementação
Métricas de Sucesso:

on_time: Dentro do prazo?

on_budget: Dentro do orçamento?

success_criteria_met: Critérios atendidos?
Ação: Muda estado para review

tasks/modules/change/close_change.yml
Objetivo: Encerrar change após revisão bem-sucedida
Ações Adicionais:

Atualiza Problems relacionados

Atualiza CI na CMDB se necessário

Registra closed_at

🔗 Integrações entre Módulos
Relações Hierárquicas
text
REQ (Request)
└── RITM (Requested Item)
└── SCTASK (Catalog Task)
Relações de Causa-Efeito
text
PRB (Problem) → CHG (Change) → SCTASK (Catalog Task)
↑
INC (Incident)
Cross-Referencing
INC → PRB: problem_id no incidente

PRB → INC: Múltiplos incidentes relacionados

PRB → CHG: problem_id na change

RITM → SCTASK: request_item na task

🛡️ Características de Segurança
Proteção de Credenciais
yaml
no_log: true # Presente em TODAS as tasks com credenciais
Validação de Parâmetros
Validação de tipos

Validação de valores permitidos

Verificação de obrigatoriedade

Tratamento de Erros
Mensagens descritivas

Fail early principle

Rollback automático onde aplicável

📊 Idempotência e Controle de Estado
Mecanismos de Idempotência
state: present em todos os módulos

Verificação de mudanças (is changed)

Registro de diferenças (diff.before/after)

Controle de Transições de Estado
Validação de transições permitidas

Campos obrigatórios por estado

Datas automáticas (resolved_at, closed_at)

🔄 Padrões de Implementação Comuns
Estrutura Padrão de Tasks
yaml

- name: "Validar parâmetros"
  assert: ...

- name: "Executar operação"
  servicenow.itsm.[module]:

  # Conexão

  instance: "{{ servicenow_creds... }}"

  # Identificação

  sys_id: "{{ module_sys_id }}"

  # Campos dinâmicos

  "{{ module_other_fields | default({}) }}"
  register: result
  no_log: true

- name: "Exibir resultado"
  debug:
  msg: ...
  changed_when: result is changed
  Tratamento de Campos Dinâmicos
  yaml

# Permite atualizar qualquer campo via dicionário

"{{ module_other_fields | default({}) }}"
Registro e Logging
register: operation_result em todas as tasks

Debug apenas quando changed

Mensagens estruturadas para parsing

🚀 Otimizações de Performance
Consulta Eficiente
sysparm_fields para trazer apenas campos necessários

sysparm_limit para limitar resultados

sysparm_query otimizado

Processamento em Lote
with_sequence para múltiplas criações

loop sobre listas para operações repetitivas

run_once: true em handlers

🔧 Configuração de Conexão
Estrutura de Credenciais (vars/credentials.yml)
yaml
servicenow_connection:
instance: "{{ servicenow_instance_vars.host }}"
username: "{{ servicenow_instance_vars.username }}"
password: "{{ servicenow_instance_vars.password }}"
timeout: 10
validate_certs: true
Suporte a Múltiplos Métodos de Autenticação
Basic Authentication (usuário/senha)

OAuth 2.0 (configurável)

Proxy (configurável)

📈 Métricas e Monitoramento
Variáveis de Controle
servicenow_enable_debug: Habilitar logging detalhado

servicenow_log_level: Nível de logging

notification_webhook_url: Webhook para notificações

Handlers de Notificação
Webhooks para sistemas externos

Email via templates Jinja2

Logging estruturado

🔄 Fluxos de Trabalho Completos
Fluxo: Solicitação de Serviço
text

1. req/create.yml → Cria Request
2. ritm/create.yml → Adiciona RITMs ao Request
3. ritm/generate_sc_task.yml → Gera Catalog Tasks
4. sc_task/assign_group.yml → Atribui tarefas
5. sc_task/close_technical.yml → Encerra tarefas
6. req/close_with_ritms.yml → Fecha Request automaticamente
   Fluxo: Gerenciamento de Problemas
   text
7. incident/create.yml → Cria incidente
8. problem/create.yml → Cria Problem relacionado
9. problem/relate_incidents.yml → Agrupa incidentes
10. problem/record_root_cause.yml → Identifica causa
11. problem/generate_change.yml → Gera Change corretiva
12. change/execute_implementation.yml → Implementa correção
13. problem/close_problem.yml → Encerra Problem
14. incident/resolve_close.yml → Resolve incidentes
    🎯 Melhores Práticas Implementadas
15. Validação Defensiva
    Assert antes de executar

Valores padrão sensatos

Fallback para omit

2. Documentação Incorporada
   Comentários explicativos em YAML

Exemplos de uso em README

Mensagens de erro claras

3. Modularidade
   Uma funcionalidade por arquivo

Reutilização de padrões

Facilidade de extensão

4. Manutenibilidade
   Variáveis centralizadas

Configuração externalizada

Logging consistente

🔍 Troubleshooting
Problemas Comuns e Soluções
Erro: "Connection refused"
yaml

# Verificar:

servicenow_instance_vars:
host: "https://instance.service-now.com" # HTTPS obrigatório
timeout: 30 # Aumentar timeout
Erro: "Invalid field value"
Verificar mapeamento em vars/module_parameters/

Usar other_fields.yml para campos personalizados

Erro: "Module not found"
bash
ansible-galaxy collection install servicenow.itsm
📚 Referências
Documentação Oficial
ServiceNow REST API

Ansible Collection servicenow.itsm

Padrões ITIL Implementados
Gerenciamento de Incidentes

Gerenciamento de Problemas

Gerenciamento de Mudanças

Catálogo de Serviços

Mapeamento de Campos
Cada módulo inclui mapeamento completo de campos em vars/module_parameters/
