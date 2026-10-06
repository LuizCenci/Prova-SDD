# Contrato: rotas do serviço web

Único serviço exposto ao navegador (HTTPS). Páginas renderizadas no servidor. Toda rota exige login, exceto `/login`. Formulários `POST` levam token CSRF. O token da sessão do usuário é repassado aos serviços internos. Falhas de regra voltam à página com mensagem e status 4xx.

| Método | Rota | Perfil | Chama | Resultados |
|---|---|---|---|---|
| GET/POST | `/login` | público | identity `POST /v1/auth/login` | 302 para `/equipment`; 401 com mensagem se inválido |
| POST | `/logout` | logado | (descarta a sessão) | 302 para `/login` |
| GET | `/equipment` | logado | catalog + loans `equipment-status` + identity `lookup` | 200; filtro `?status=available\|loaned\|overdue`; exibe pessoa, data do empréstimo, prazo e dias de atraso (FR-009, FR-010) |
| GET/POST | `/equipment/new` | admin | catalog `POST /v1/equipment` | 302; 400; 409 código duplicado |
| GET/POST | `/equipment/{id}/edit` | admin | catalog `PATCH` | 302; 400; 409 |
| GET/POST | `/people`, `/people/new` | admin | identity `/v1/users` | 200; 302; 400; 409 e-mail duplicado |
| GET/POST | `/people/{id}/edit` | admin | identity `PATCH` | 302; 400 |
| GET/POST | `/equipment/{id}/loan` | logado | loans `POST /v1/loans` | 302; 400 prazo inválido; 404; 409 já emprestado |
| POST | `/loans/{id}/return` | admin | loans `POST /v1/loans/{id}/return` | 302; 409 já devolvido |
| GET | `/loans` | logado | loans `GET /v1/loans` + identity `lookup` | 200 |

Sem login: 302 para `/login`. Sem permissão: 403. Usuário desativado: 403 com a mensagem "Usuário inativo não pode pegar equipamentos emprestados." (e sem acesso após o login). Se um serviço interno estiver indisponível: página de erro amigável com status 503.

## Mensagens esperadas (FR-012)
- "Este equipamento já está emprestado."
- "Já existe um equipamento com este código."
- "A data prevista de devolução não pode ser anterior a hoje."
- "Não há empréstimo em aberto para este equipamento."
- "Serviço temporariamente indisponível. Tente novamente em instantes."
