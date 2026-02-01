# servicenowteste_role — Ansible Role para ServiceNow ITSM

Role Ansible que abstrai e simplifica sistematicamente as funcionalidades do módulo **servicenow.itsm** (v2.13.2), permitindo gerir incidentes, problemas, change requests, service catalog e configuration items do ServiceNow através de uma interface unificada baseada em `snow_entity` + `snow_action`.

---

## Estrutura da Role

```
servicenowteste_role/
├── collections/
│   └── requirements.yml              # Dependência: servicenow.itsm 2.13.2
├── defaults/
│   └── main.yml                      # Timeout e validate_certs padrão
├── vars/
│   └── main.yml                      # Bloco de conexão (snow_connection_args)
├── tasks/
│   ├── main.yml                      # Entry point: validação + dispatcher
│   ├── dispatcher/
│   │   └── main.yml                  # Roteamento dinâmico: modules/{entity}/{action}.yml
│   └── modules/
│       ├── incident/                 # Gestão de Incidentes
│       │   ├── create.yml            #   → servicenow.itsm.incident (state: new)
│       │   ├── info.yml              #   → servicenow.itsm.incident_info
│       │   └── update.yml            #   → servicenow.itsm.incident (update/resolve/close)
│       ├── change/                   # Gestão de Change Requests
│       │   ├── create.yml            #   → servicenow.itsm.change_request (state: new)
│       │   ├── info.yml              #   → servicenow.itsm.change_request_info
│       │   └── update.yml            #   → servicenow.itsm.change_request (update/close)
│       ├── change_task/              # Gestão de Change Request Tasks
│       │   ├── create.yml            #   → servicenow.itsm.change_request_task (create)
│       │   ├── info.yml              #   → servicenow.itsm.change_request_task_info
│       │   └── update.yml            #   → servicenow.itsm.change_request_task (update/close)
│       ├── problem/                  # Gestão de Problems
│       │   ├── create.yml            #   → servicenow.itsm.problem (state: new)
│       │   ├── info.yml              #   → servicenow.itsm.problem_info
│       │   └── update.yml            #   → servicenow.itsm.problem (update/resolve)
│       ├── problem_task/             # Gestão de Problem Tasks
│       │   ├── create.yml            #   → servicenow.itsm.problem_task (create)
│       │   ├── info.yml              #   → servicenow.itsm.problem_task_info
│       │   └── update.yml            #   → servicenow.itsm.problem_task (update/close)
│       ├── req/                      # Gestão de Service Catalog Requests (REQ)
│       │   ├── info.yml              #   → servicenow.itsm.catalog_request_info
│       │   └── update.yml            #   → servicenow.itsm.catalog_request
│       ├── ritm/                     # Gestão de Service Catalog Request Items (RITM)
│       │   ├── info.yml              #   → servicenow.itsm.api_info (sc_req_item)
│       │   └── update.yml            #   → servicenow.itsm.api PATCH (sc_req_item)
│       ├── sc_task/                  # Gestão de Service Catalog Tasks (SCTASK)
│       │   ├── info.yml              #   → servicenow.itsm.catalog_request_task_info
│       │   └── update.yml            #   → servicenow.itsm.catalog_request_task
│       ├── ci/                       # Gestão de Configuration Items (CMDB)
│       │   ├── create.yml            #   → servicenow.itsm.configuration_item (idempotente)
│       │   └── info.yml              #   → servicenow.itsm.configuration_item_info
│       ├── api_info/                 # API Genérica — Leitura
│       │   └── get.yml               #   → servicenow.itsm.api_info (GET)
│       └── api/                      # API Genérica — Escrita
│           ├── post.yml              #   → servicenow.itsm.api (POST)
│           ├── patch.yml             #   → servicenow.itsm.api (PATCH)
│           └── delete.yml            #   → servicenow.itsm.api (DELETE)
├── examples/                         # Playbooks de exemplo com todos os cenários
│   ├── incident_create.yml
│   ├── incident_info.yml
│   ├── incident_update.yml
│   ├── change_full.yml
│   ├── change_task_full.yml
│   ├── problem_full.yml
│   ├── problem_task_full.yml
│   ├── ritm_update.yml
│   ├── sc_task_info_update.yml
│   ├── ci_full.yml
│   └── api_generic_full.yml
└── meta/
    └── main.yml
```

---

## Pré-Requisitos

```bash
# Instalar a coleção (versão fixada)
ansible-galaxy collection install servicenow.itsm==2.13.2
# ou usar o ficheiro requirements.yml da role:
ansible-galaxy collection install -r collections/requirements.yml
```

---

## Variáveis de Entrada

### Obrigatórias (sempre)

| Variável | Descrição |
|---|---|
| `snow_instance` | URL da instância ServiceNow (ex: `https://dev12345.service-now.com`) |
| `snow_username` | Utilizador de autenticação |
| `snow_password` | Password (usar vault em produção) |
| `snow_entity` | Entidade alvo: `incident`, `change`, `change_task`, `problem`, `problem_task`, `req`, `ritm`, `sc_task`, `ci`, `api_info`, `api` |
| `snow_action` | Ação: `create`, `info`, `update`, `get`, `post`, `patch`, `delete` (dependente da entidade) |
| `snow_data` | Dicionário com os dados específicos da operação |

### Opcionais (autenticação OAuth)

| Variável | Descrição |
|---|---|
| `snow_client_id` | Client ID OAuth |
| `snow_client_secret` | Client Secret OAuth |

### Configuração padrão (`defaults/main.yml`)

| Variável | Valor Padrão | Descrição |
|---|---|---|
| `servicenow_default_timeout` | `30` | Timeout da conexão em segundos |
| `servicenow_default_validate_certs` | `true` | Validação de certificados SSL |

---

## Mapeamento: Entidade + Ação → Ficheiro

| `snow_entity` | `snow_action` | Ficheiro executado | Módulo servicenow.itsm |
|---|---|---|---|
| `incident` | `create` | `incident/create.yml` | `servicenow.itsm.incident` |
| `incident` | `info` | `incident/info.yml` | `servicenow.itsm.incident_info` |
| `incident` | `update` | `incident/update.yml` | `servicenow.itsm.incident` |
| `change` | `create` | `change/create.yml` | `servicenow.itsm.change_request` |
| `change` | `info` | `change/info.yml` | `servicenow.itsm.change_request_info` |
| `change` | `update` | `change/update.yml` | `servicenow.itsm.change_request` |
| `change_task` | `create` | `change_task/create.yml` | `servicenow.itsm.change_request_task` |
| `change_task` | `info` | `change_task/info.yml` | `servicenow.itsm.change_request_task_info` |
| `change_task` | `update` | `change_task/update.yml` | `servicenow.itsm.change_request_task` |
| `problem` | `create` | `problem/create.yml` | `servicenow.itsm.problem` |
| `problem` | `info` | `problem/info.yml` | `servicenow.itsm.problem_info` |
| `problem` | `update` | `problem/update.yml` | `servicenow.itsm.problem` |
| `problem_task` | `create` | `problem_task/create.yml` | `servicenow.itsm.problem_task` |
| `problem_task` | `info` | `problem_task/info.yml` | `servicenow.itsm.problem_task_info` |
| `problem_task` | `update` | `problem_task/update.yml` | `servicenow.itsm.problem_task` |
| `req` | `info` | `req/info.yml` | `servicenow.itsm.catalog_request_info` |
| `req` | `update` | `req/update.yml` | `servicenow.itsm.catalog_request` |
| `ritm` | `info` | `ritm/info.yml` | `servicenow.itsm.api_info` |
| `ritm` | `update` | `ritm/update.yml` | `servicenow.itsm.api` (PATCH) |
| `sc_task` | `info` | `sc_task/info.yml` | `servicenow.itsm.catalog_request_task_info` |
| `sc_task` | `update` | `sc_task/update.yml` | `servicenow.itsm.catalog_request_task` |
| `ci` | `create` | `ci/create.yml` | `servicenow.itsm.configuration_item` |
| `ci` | `info` | `ci/info.yml` | `servicenow.itsm.configuration_item_info` |
| `api_info` | `get` | `api_info/get.yml` | `servicenow.itsm.api_info` |
| `api` | `post` | `api/post.yml` | `servicenow.itsm.api` |
| `api` | `patch` | `api/patch.yml` | `servicenow.itsm.api` |
| `api` | `delete` | `api/delete.yml` | `servicenow.itsm.api` |

---

## Exemplos de Uso

### Criar Incidente

```yaml
- name: "Criar Incidente"
  ansible.builtin.include_role:
    name: servicenowteste_role
  vars:
    snow_instance: "https://dev12345.service-now.com"
    snow_username: "ansible_user"
    snow_password: "{{ vault_snow_password }}"
    snow_entity: incident
    snow_action: create
    snow_data:
      caller_id: "joseflavio@3coracoes.com.br"
      business_service: "SAP ERP"
      service_offering: "SAP Production"
      category: "software"
      short_description: "Erro ao acessar transação Z123"
      assignment_group: "Service Desk"
      impact: "2"
      urgency: "2"
      description: "Usuário relata timeout após clicar em salvar."
      work_notes: "Incidente aberto via automação Ansible."
```

### Criar Change Request

```yaml
- name: "Criar Change Request"
  ansible.builtin.include_role:
    name: servicenowteste_role
  vars:
    snow_entity: change
    snow_action: create
    snow_data:
      short_description: "Atualização do firmware do Switch Core"
      type: "normal"
      priority: "moderate"
      risk: "low"
      impact: "low"
      category: "network"
      requested_by: "joseflavio@3coracoes.com.br"
      assignment_group: "DEVOPS"
```

### Criar Problem

```yaml
- name: "Criar Problem"
  ansible.builtin.include_role:
    name: servicenowteste_role
  vars:
    snow_entity: problem
    snow_action: create
    snow_data:
      short_description: "Lentidão recorrente no portal"
      impact: "high"
      urgency: "high"
      assigned_to: "joseflavio@3coracoes.com.br"
      assignment_group: "DEVOPS"
      cmdb_ci: "APP-PORTAL-01"
```

### Registar CI no CMDB

```yaml
- name: "Registar VM no CMDB"
  ansible.builtin.include_role:
    name: servicenowteste_role
  vars:
    snow_entity: ci
    snow_action: create
    snow_data:
      name: "SRV-APP-PORTAL-01"
      sys_class_name: "cmdb_ci_vm_instance"
      category: "Servers"
      environment: "production"
      operational_status: "Operational"
      ip_address: "10.0.1.55"
      fqdn: "srv-app-portal-01.3coracoes.com.br"
```

### API Genérica (POST para tabela customizada)

```yaml
- name: "Criar registo em tabela customizada"
  ansible.builtin.include_role:
    name: servicenowteste_role
  vars:
    snow_entity: api
    snow_action: post
    snow_data:
      resource: "u_automation_log"
      data:
        u_timestamp: "2024-03-15 10:00:00"
        u_source: "Ansible"
        u_status: "success"
        u_message: "Deploy concluído."
```

---

## Tradução de Estados

Os módulos `change/update`, `change_task/update`, `problem/update` e `problem_task/update` suportam tradução automática de estados numéricos para string, permitindo usar payloads genéricos (ex: vindos do EDA).

| Entidade | Numérico | String |
|---|---|---|
| **change** | 1 | new |
| | 2 | assess |
| | 3 | authorize |
| | 4 | scheduled |
| | 5 | implement |
| | 6 | review |
| | 7 | closed |
| | 8 | canceled |
| **change_task** | 1 | open |
| | 2 | in_progress |
| | 3 | pending |
| | 4 | closed |
| | 6 | canceled |
| **problem** | 1 | new |
| | 2 | assess |
| | 3 | under_investigation |
| | 4 | resolved |
| | 5 | closed |
| **problem_task** | 1 | new |
| | 2 | assess |
| | 3 | work_in_progress |
| | 4 | closed |
| **sc_task** | 1 | open |
| | 2 | work_in_progress |
| | 3 | closed_complete |
| | 4 | closed_incomplete |
| | 7 | closed_skipped |

---

## Variáveis de Resultado (register)

Cada módulo expõe o resultado numa variável específica para uso em tarefas subsequentes:

| Entidade + Ação | Variável de Resultado |
|---|---|
| incident create | `result_incident_create` |
| incident info | `result_incident_info` |
| incident update | `result_incident_update` |
| change create | `result_change_create` |
| change info | `result_change_info` |
| change update | `result_change_update` |
| change_task create | `result_ctask_create` |
| change_task info | `result_ctask_info` |
| change_task update | `result_ctask_update` |
| problem create | `result_problem_create` |
| problem info | `result_problem_info` |
| problem update | `result_problem_update` |
| problem_task create | `result_ptask_create` |
| problem_task info | `result_ptask_info` |
| problem_task update | `result_ptask_update` |
| req info | `result_req_info` |
| req update | `result_req_update` |
| ritm info | `result_ritm_info` |
| ritm update | `result_ritm_update` |
| sc_task info | `result_sctask_info` |
| sc_task update | `result_sctask_update` |
| ci create | `result_ci_create` |
| ci info | `result_ci_info` |
| api_info get | `result_api_info` |
| api post | `result_api_post` |
| api patch | `result_api_patch` |
| api delete | `result_api_delete` |

---

## Licença

MIT — Autor: Flávio Mattos | Empresa: 3 Corações
