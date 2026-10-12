# Contrato: catalog API (v1)

Rede interna. Toda rota exige `Authorization: Bearer <token>` válido (assinatura verificada com a chave pública do identity).

| Método | Rota | Perfil | Descrição | Respostas |
|---|---|---|---|---|
| GET | `/v1/equipment` | qualquer logado | Lista equipamentos `{id, name, code, active}` | 200 |
| GET | `/v1/equipment/{id}` | qualquer logado | Detalhe; usado pelo loans para validar o empréstimo | 200; 404 |
| POST | `/v1/equipment` | admin | Cria `{name, code}` | 201; 400 campo faltante; 409 código duplicado |
| PATCH | `/v1/equipment/{id}` | admin | Edita ou desativa | 200; 400; 404; 409 |
| GET | `/health` | público | Verificação de saúde | 200 |

Não existe rota de exclusão: equipamentos são apenas desativados (`PATCH` com `active: false`, FR-017).
