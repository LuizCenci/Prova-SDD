# Contrato: loans API (v1)

Rede interna. Toda rota exige `Authorization: Bearer <token>`. O tomador do empréstimo é sempre o `sub` do token; o corpo não aceita `borrower_id`.

| Método | Rota | Perfil | Descrição | Respostas |
|---|---|---|---|---|
| POST | `/v1/loans` | qualquer logado | Corpo `{equipment_id, due_on}`; confirma no catalog que o equipamento existe e está ativo | 201 `{id, equipment_id, borrower_id, loaned_on, due_on}`; 400 `due_on` ausente ou anterior a hoje; 403 usuário inativo (confirmado no identity); 404 equipamento inexistente ou inativo; 409 já emprestado |
| POST | `/v1/loans/{id}/return` | admin | Registra a devolução na data de hoje | 200; 404; 409 empréstimo já devolvido |
| GET | `/v1/loans` | qualquer logado | Histórico; admin vê todos, usuário vê os seus; filtros `equipment_id`, `open` | 200 |
| GET | `/v1/equipment-status` | qualquer logado | Situação dos equipamentos; filtro `status=available\|loaned\|overdue`, opcionalmente `ids=...`. Cada item: `{equipment_id, status, loan?: {id, borrower_id, loaned_on, due_on, days_overdue}}` | 200 |
| GET | `/health` | público | Verificação de saúde | 200 |

Observação: `status=available` considera apenas equipamentos sem empréstimo em aberto; o `web` cruza esse resultado com a lista do catalog (equipamentos ativos) para montar a tela.
