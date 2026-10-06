# Data Model: Controle de Empréstimos de Equipamentos (Loanner)

Cada serviço é dono de suas tabelas. Relações entre serviços são apenas por id, sem chave estrangeira entre bancos.

## Serviço identity: `User`
| Campo | Tipo | Regras |
|---|---|---|
| id | UUID | chave primária |
| name | texto | obrigatório |
| email | texto | obrigatório, único (login) |
| contact | texto | opcional |
| password_hash | texto | Argon2; nunca exposto |
| role | `admin` \| `user` | obrigatório; padrão `user` |
| active | booleano | padrão verdadeiro; inativo não faz login nem pega itens emprestados (o `loans` confirma a situação no identity a cada empréstimo, FR-016) |


Usuário com histórico não é excluído, apenas desativado.

## Serviço catalog: `Equipment`
| Campo | Tipo | Regras |
|---|---|---|
| id | UUID | chave primária |
| name | texto | obrigatório |
| code | texto | obrigatório, **único** (FR-003) |
| active | booleano | padrão verdadeiro; inativo não pode ser emprestado |

A situação não é gravada aqui. Equipamento com histórico não é excluído.

## Serviço loans: `Loan`
| Campo | Tipo | Regras |
|---|---|---|
| id | UUID | chave primária |
| equipment_id | UUID | obrigatório; referência ao catalog |
| borrower_id | UUID | obrigatório; referência ao identity; quem pegou o item |
| loaned_on | data | obrigatório |
| due_on | data | obrigatório; `due_on >= loaned_on` (FR-006) |
| returned_on | data | vazio enquanto em aberto; `>= loaned_on` |
| loaned_by_id | UUID | quem registrou o empréstimo (FR-014) |
| returned_by_id | UUID | quem registrou a devolução; vazio enquanto em aberto |

**Índice único parcial**: `UNIQUE(equipment_id) WHERE returned_on IS NULL` (FR-005, SC-003).

## Situação derivada (calculada pelo loans, FR-008)
- **disponível**: sem empréstimo em aberto (e equipamento ativo, informação do catalog).
- **emprestado**: em aberto com `due_on >= hoje`.
- **atrasado**: em aberto com `due_on < hoje`; dias de atraso = `hoje − due_on`.

## Transições
```text
disponível --emprestar--> emprestado --(passa due_on)--> atrasado
emprestado/atrasado --devolver--> disponível
```

## Permissões (FR-015)
| Ação | admin | user |
|---|---|---|
| Cadastrar/editar equipamentos e usuários | sim | não |
| Registrar empréstimo (sempre em seu nome) | sim | sim |
| Registrar devolução | sim | não |
| Consultar situação | sim | sim |
| Ver histórico | todos | apenas os seus |
