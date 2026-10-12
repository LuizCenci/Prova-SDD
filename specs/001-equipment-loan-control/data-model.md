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
| role | `admin` \| `user` | obrigatório; padrão `user`; o autocadastro sempre cria `user` (FR-018); só o admin cria `admin` |
| active | booleano | padrão verdadeiro; inativo não faz login nem pega itens emprestados (o `loans` confirma a situação no identity a cada empréstimo, FR-016) |

Não há exclusão de usuários: para retirá-los de uso, o administrador os desativa (FR-017). Empréstimos em aberto de um usuário inativo continuam visíveis e podem ser devolvidos pelo administrador.

## Serviço catalog: `Equipment`
| Campo | Tipo | Regras |
|---|---|---|
| id | UUID | chave primária |
| name | texto | obrigatório |
| code | texto | obrigatório, **único** (FR-003) |
| active | booleano | padrão verdadeiro; inativo não pode ser emprestado e fica oculto da lista de equipamentos (FR-017) |

A situação não é gravada aqui. Não há exclusão de equipamentos, apenas desativação (FR-017). Desativar um equipamento emprestado não encerra o empréstimo: ele segue visível como emprestado ou atrasado até ser devolvido e, depois disso, fica oculto.

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
emprestado/atrasado --devolver--> disponível (se o equipamento estiver ativo)
```
Equipamento inativo sem empréstimo em aberto fica oculto da lista e não pode ser emprestado.

## Permissões (FR-015)
| Ação | admin | user |
|---|---|---|
| Cadastrar, editar e desativar equipamentos e usuários | sim | não |
| Registrar empréstimo (sempre em seu nome) | sim | sim |
| Registrar devolução | sim | não |
| Consultar situação | sim | sim |
| Ver histórico | todos | apenas os seus |
