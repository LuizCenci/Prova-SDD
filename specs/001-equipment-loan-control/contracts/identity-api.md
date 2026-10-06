# Contrato: identity API (v1)

Rede interna. Exceto `POST /v1/auth/login` e `GET /v1/.well-known/jwks.json`, toda rota exige `Authorization: Bearer <token>`. O contrato OpenAPI gerado pelo serviço deve coincidir com este documento.

| Método | Rota | Perfil | Descrição | Respostas |
|---|---|---|---|---|
| POST | `/v1/auth/login` | público | Corpo `{email, password}` | 200 `{access_token, expires_in}`; 401 se credencial inválida ou usuário inativo |
| GET | `/v1/.well-known/jwks.json` | público | Chave pública para validar tokens | 200 |
| GET | `/v1/users` | admin | Lista usuários | 200 |
| POST | `/v1/users` | admin | Cria usuário `{name, email, contact?, role, password}` | 201; 400 dado inválido; 409 e-mail duplicado |
| GET | `/v1/users/{id}` | admin ou o próprio | Detalhe do usuário `{id, name, email, contact, role, active}`; o `loans` o usa para confirmar que o tomador está ativo | 200; 404 |
| PATCH | `/v1/users/{id}` | admin | Edita ou desativa usuário | 200; 400; 409 |
| POST | `/v1/users/lookup` | qualquer logado | Corpo `{ids: [...]}`; devolve `{id, name}` para exibir nomes | 200 |
| GET | `/health` | público | Verificação de saúde | 200 |

**Token**: JWT RS256, validade de 30 minutos, com `sub` (id do usuário), `role` e `exp`.
