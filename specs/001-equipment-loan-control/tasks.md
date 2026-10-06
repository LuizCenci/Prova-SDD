---

description: "Lista de tarefas do Loanner: controle de empréstimos de equipamentos"
---

# Tasks: Controle de Empréstimos de Equipamentos (Loanner)

**Input**: documentos em `/specs/001-equipment-loan-control/` (plan.md, spec.md, research.md, data-model.md, contracts/, quickstart.md)

**Prerequisites**: plan.md e spec.md (obrigatórios). Constituição em `.specify/memory/constitution.md`.

**Tests**: INCLUÍDOS. A constituição (Princípio V) exige teste antes da implementação, visto falhando primeiro.

**Organization**: tarefas agrupadas por história de usuário, para implementação e teste independentes.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: pode rodar em paralelo (arquivos diferentes, sem dependência pendente)
- **[Story]**: história de usuário a que a tarefa pertence (US1 a US4)
- Caminhos relativos à raiz do repositório
- Todo código público MUST ter docstring (Princípio V) e todo comentário explica o porquê

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: estrutura do repositório, ferramentas e contêineres

- [ ] T001 Criar a estrutura de diretórios do plano: `services/identity/src/identity/{api,domain,infra}/`, `services/catalog/src/catalog/{api,domain,infra}/`, `services/loans/src/loans/{api,domain,infra}/`, `services/web/src/web/{routes,clients,templates,static}/`, `tests/{unit,integration,contract}/` em cada serviço de dados, `services/web/tests/{unit,integration}/`, `docs/adr/`
- [ ] T002 [P] Criar `services/identity/pyproject.toml` (Python 3.12; FastAPI, Pydantic, SQLAlchemy 2, Alembic, PyJWT, argon2-cffi, pytest; versões fixadas)
- [ ] T003 [P] Criar `services/catalog/pyproject.toml` (Python 3.12; FastAPI, Pydantic, SQLAlchemy 2, Alembic, PyJWT, httpx, pytest; versões fixadas)
- [ ] T004 [P] Criar `services/loans/pyproject.toml` (Python 3.12; FastAPI, Pydantic, SQLAlchemy 2, Alembic, PyJWT, httpx, pytest; versões fixadas)
- [ ] T005 [P] Criar `services/web/pyproject.toml` (Python 3.12; FastAPI, Jinja2, httpx, itsdangerous, pytest; versões fixadas)
- [ ] T006 [P] Configurar linter e formatador (ruff, com regras de docstring) em `ruff.toml` e `.editorconfig` na raiz
- [ ] T007 [P] Criar `.env.example` com todas as variáveis de ambiente (URLs dos bancos, chave de assinatura, URLs dos serviços, chave de sessão) e `.gitignore` que exclui `.env` e chaves
- [ ] T008 [P] Criar `services/identity/Dockerfile`
- [ ] T009 [P] Criar `services/catalog/Dockerfile`
- [ ] T010 [P] Criar `services/loans/Dockerfile`
- [ ] T011 [P] Criar `services/web/Dockerfile`
- [ ] T012 Criar `docker-compose.yml` com 3 bancos PostgreSQL 16 (um por serviço, usuários distintos), os 4 serviços, rede interna, e apenas a porta do proxy publicada (o TLS é ligado em T014)
- [ ] T013 [P] Criar `scripts/generate_certs.py`, que gera uma autoridade certificadora interna e um certificado por serviço, banco e proxy em `deploy/certs/` (diretório ignorado pelo Git, junto de qualquer chave privada), e documentar o uso no `README.md` raiz
- [ ] T014 Criar o proxy reverso `deploy/proxy/Caddyfile` e ligar o TLS no `docker-compose.yml`: o proxy é o único serviço publicado, redireciona HTTP para HTTPS e fala com o `web` por TLS; identity, catalog, loans e web aceitam apenas conexões TLS na rede interna; os bancos exigem TLS (`sslmode=verify-full`) (Princípio II: todo tráfego criptografado)
- [ ] T015 [P] Criar `README.md` raiz (propósito, visão da arquitetura, como rodar e testar), `docs/architecture.md` (esboço) e `docs/glossary.md` (termos: equipamento, empréstimo, devolução, atrasado, administrador, usuário)
- [ ] T016 [P] Criar o pipeline de CI em `.github/workflows/ci.yml`: para cada serviço roda `ruff check`, `ruff format --check` e `pytest` (com PostgreSQL), e é exigido para o merge (constituição: nenhum merge com testes, linter ou formatador falhando)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: login, verificação de token e base de cada serviço. Nenhuma história começa antes disto.

**⚠️ CRITICAL**: sem login funcionando, nenhuma história de usuário pode ser usada ou testada

- [ ] T017 [P] Escrever teste unitário de hash de senha (Argon2; senha nunca devolvida) em `services/identity/tests/unit/test_passwords.py`
- [ ] T018 [P] Escrever teste de contrato de `POST /v1/auth/login` (200 com `access_token` e `expires_in`; 401 para credencial inválida ou usuário inativo) e `GET /v1/.well-known/jwks.json`, e que cada login falho gera um evento de segurança registrado (sem senha) em `services/identity/tests/contract/test_auth_contract.py`
- [ ] T019 Criar configuração por variáveis de ambiente em `services/identity/src/identity/config.py`
- [ ] T020 Configurar sessão de banco e Alembic em `services/identity/src/identity/infra/database.py` e `services/identity/migrations/env.py`
- [ ] T021 Criar modelo `User` e migração inicial em `services/identity/src/identity/infra/models.py` e `services/identity/migrations/versions/0001_users.py` com os campos: id UUID chave primária; name texto obrigatório; email texto obrigatório, único; contact texto opcional; password_hash texto (Argon2, nunca exposto); role `admin` | `user` obrigatório, padrão `user`; active booleano, padrão verdadeiro
- [ ] T022 Implementar hash e verificação de senha em `services/identity/src/identity/domain/passwords.py` (faz T017 passar)
- [ ] T023 Implementar emissão de JWT RS256 (claims `sub`, `role`, `exp`; validade de 30 minutos) e JWKS em `services/identity/src/identity/infra/tokens.py`
- [ ] T024 Criar a aplicação, a rota de login e `/health` com logs JSON que nunca registram senha nem token, e registrar eventos de segurança (login com sucesso, login falho, usuário inativo) em `services/identity/src/identity/infra/security_log.py` em `services/identity/src/identity/main.py` e `services/identity/src/identity/api/auth.py` (faz T018 passar)
- [ ] T025 Criar o comando de linha `create-admin` e seu teste de integração em `services/identity/src/identity/cli.py` e `services/identity/tests/integration/test_cli.py`
- [ ] T026 [P] Escrever teste de verificação de token (assinatura inválida, expirado, sem token → 401; perfil insuficiente → 403), e que cada 401 e 403 gera um evento de segurança registrado em `services/catalog/tests/unit/test_auth.py`
- [ ] T027 [P] Escrever o mesmo teste de verificação de token e de evento de segurança em `services/loans/tests/unit/test_auth.py`
- [ ] T028 [P] Criar base do catalog: `config.py`, `infra/database.py`, `migrations/env.py`, `infra/auth.py` (busca a chave pública no JWKS do identity e oferece `require_role`), logs JSON, registro de eventos de segurança (401 e 403) em `infra/security_log.py`, `main.py` e `/health`, com servidor TLS em `services/catalog/src/catalog/` (faz T026 passar)
- [ ] T029 [P] Criar base do loans com os mesmos itens de T028 (inclusive eventos de segurança e TLS) em `services/loans/src/loans/` (faz T027 passar)
- [ ] T030 [P] Criar base do web em `services/web/src/web/`: `config.py`, sessão com cookie HttpOnly, proteção CSRF, logs JSON, registro de eventos de segurança (login falho e 403), `clients/base.py` (repassa o token e um id de correlação, e valida o certificado dos serviços com a CA interna), `templates/base.html` responsivo, páginas de erro 403 e 503 com a mensagem "Serviço temporariamente indisponível. Tente novamente em instantes."
- [ ] T031 [P] Escrever teste de integração de login e logout (sem login, `/equipment` redireciona para `/login`; credencial inválida mostra mensagem; login falho gera evento de segurança) em `services/web/tests/integration/test_login.py`
- [ ] T032 Implementar rotas `/login` e `/logout` e o template em `services/web/src/web/routes/auth.py` e `services/web/src/web/templates/login.html` (faz T031 passar)

**Checkpoint**: usuário consegue entrar; todos os serviços validam o token

---

## Phase 3: User Story 1 - Cadastrar equipamentos e pessoas (Priority: P1) 🎯 MVP

**Goal**: o administrador cadastra e edita equipamentos e usuários (pessoas), e qualquer logado lista equipamentos.

**Independent Test**: como administrador, cadastrar um equipamento e um usuário e vê-los nas listas; tentar repetir o código do equipamento e ser recusado.

### Tests for User Story 1 ⚠️ (escrever primeiro e ver falhar)

- [ ] T033 [P] [US1] Teste de contrato da catalog API (`POST` 201 / 400 campo faltante / 409 código duplicado; `PATCH`; `GET` lista e detalhe; 401 sem token; 403 para perfil `user` em escrita; não existe rota de exclusão) em `services/catalog/tests/contract/test_equipment_api.py`
- [ ] T034 [P] [US1] Teste unitário das regras de equipamento (nome e código obrigatórios) em `services/catalog/tests/unit/test_equipment_rules.py`
- [ ] T035 [P] [US1] Teste de contrato da identity API de usuários (`POST` 201 / 400 / 409 e-mail duplicado; `PATCH` desativar; `GET`; `lookup` devolve só `{id, name}`; apenas admin cria e edita) em `services/identity/tests/contract/test_users_api.py`
- [ ] T036 [P] [US1] Teste unitário das regras de usuário (e-mail único, perfil válido, usuário inativo não faz login) em `services/identity/tests/unit/test_user_rules.py`
- [ ] T037 [P] [US1] Teste de integração das telas de cadastro de equipamentos e pessoas, incluindo código duplicado e acesso negado ao perfil `user` (403), em `services/web/tests/integration/test_registration_pages.py`

### Implementation for User Story 1

- [ ] T038 [P] [US1] Criar modelo `Equipment` e migração em `services/catalog/src/catalog/infra/models.py` e `services/catalog/migrations/versions/0001_equipment.py` com os campos: id UUID chave primária; name texto obrigatório; code texto obrigatório, **único**; active booleano, padrão verdadeiro
- [ ] T039 [US1] Implementar regras de equipamento em `services/catalog/src/catalog/domain/equipment.py` (faz T034 passar)
- [ ] T040 [US1] Implementar repositório de equipamentos (traduz violação de unicidade em erro de domínio) em `services/catalog/src/catalog/infra/repository.py`
- [ ] T041 [US1] Implementar endpoints de equipamentos conforme `contracts/catalog-api.md` em `services/catalog/src/catalog/api/equipment.py` (faz T033 passar)
- [ ] T042 [P] [US1] Implementar regras de usuário em `services/identity/src/identity/domain/users.py` (faz T036 passar)
- [ ] T043 [US1] Implementar repositório de usuários em `services/identity/src/identity/infra/repository.py`
- [ ] T044 [US1] Implementar endpoints `/v1/users`, `/v1/users/{id}` e `/v1/users/lookup` conforme `contracts/identity-api.md` em `services/identity/src/identity/api/users.py` (faz T035 passar)
- [ ] T045 [US1] Criar clientes HTTP do catalog e do identity em `services/web/src/web/clients/catalog.py` e `services/web/src/web/clients/identity.py`
- [ ] T046 [US1] Implementar rotas e templates de equipamentos (lista, `/equipment/new`, `/equipment/{id}/edit`) com as mensagens de `contracts/web-routes.md` em `services/web/src/web/routes/equipment.py` e `services/web/src/web/templates/equipment_*.html`
- [ ] T047 [US1] Implementar rotas e templates de pessoas (`/people`, `/people/new`, `/people/{id}/edit`) em `services/web/src/web/routes/people.py` e `services/web/src/web/templates/people_*.html` (faz T037 passar)
- [ ] T048 [P] [US1] Escrever `services/catalog/README.md`, `services/identity/README.md` e o esboço de `services/web/README.md` (responsabilidade, contratos, dados que possui, configuração, como testar)

**Checkpoint**: cadastro de equipamentos e pessoas funciona sozinho (MVP de dados)

---

## Phase 4: User Story 2 - Registrar empréstimo com data prevista (Priority: P1)

**Goal**: o usuário logado empresta um equipamento disponível em seu nome, com data prevista; o mesmo equipamento não pode ser emprestado duas vezes.

**Independent Test**: emprestar um equipamento e ver "emprestado"; uma segunda tentativa é recusada, inclusive duas tentativas simultâneas.

### Tests for User Story 2 ⚠️

- [ ] T049 [P] [US2] Teste de contrato de `POST /v1/loans` (201; 400 `due_on` ausente ou anterior a hoje; 404 equipamento inexistente ou inativo; 409 já emprestado; corpo com `borrower_id` é ignorado e o tomador é o `sub` do token; usuário inativo → 403 "Usuário inativo não pode pegar equipamentos emprestados.") em `services/loans/tests/contract/test_create_loan.py`
- [ ] T050 [P] [US2] Teste unitário das regras de empréstimo (`due_on >= loaned_on`, data obrigatória) em `services/loans/tests/unit/test_loan_rules.py`
- [ ] T051 [P] [US2] Teste de integração de concorrência: duas requisições simultâneas para o mesmo equipamento resultam em exatamente um 201 e um 409, contra PostgreSQL real, em `services/loans/tests/integration/test_concurrent_loan.py`
- [ ] T052 [P] [US2] Teste de integração da tela de empréstimo (sucesso, já emprestado, prazo inválido) em `services/web/tests/integration/test_loan_page.py`

### Implementation for User Story 2

- [ ] T053 [US2] Criar modelo `Loan` e migração em `services/loans/src/loans/infra/models.py` e `services/loans/migrations/versions/0001_loans.py` com os campos: id UUID chave primária; equipment_id UUID obrigatório; borrower_id UUID obrigatório; loaned_on data obrigatória; due_on data obrigatória, `due_on >= loaned_on`; returned_on data vazia enquanto em aberto, `>= loaned_on`; loaned_by_id UUID; returned_by_id UUID vazio enquanto em aberto; e **índice único parcial** `UNIQUE(equipment_id) WHERE returned_on IS NULL`
- [ ] T054 [US2] Implementar regras de empréstimo em `services/loans/src/loans/domain/loan_rules.py` (faz T050 passar)
- [ ] T055 [US2] Implementar cliente do catalog (confirma que o equipamento existe e está ativo) em `services/loans/src/loans/infra/catalog_client.py` e cliente do identity (consulta `GET /v1/users/{sub}` com o token do chamador e confirma que o tomador está ativo, mesmo que o token ainda não tenha expirado) em `services/loans/src/loans/infra/identity_client.py`
- [ ] T056 [US2] Implementar repositório e serviço `create_loan`, traduzindo a violação do índice único em conflito 409, em `services/loans/src/loans/infra/repository.py` e `services/loans/src/loans/domain/service.py`
- [ ] T057 [US2] Implementar `POST /v1/loans` conforme `contracts/loans-api.md`, gravando `loaned_by_id` e `borrower_id` a partir do token, em `services/loans/src/loans/api/loans.py` (faz T049 e T051 passarem)
- [ ] T058 [US2] Implementar cliente do loans e a rota `/equipment/{id}/loan` com formulário e template em `services/web/src/web/clients/loans.py`, `services/web/src/web/routes/loans.py` e `services/web/src/web/templates/loan_form.html` (faz T052 passar)
- [ ] T059 [P] [US2] Escrever `services/loans/README.md` (responsabilidade, contrato, índice único, configuração, como testar)

**Checkpoint**: empréstimo funciona e é seguro contra duplicidade

---

## Phase 5: User Story 3 - Registrar devolução (Priority: P1)

**Goal**: o administrador registra a devolução e o equipamento volta a ficar disponível.

**Independent Test**: devolver um equipamento emprestado e emprestá-lo de novo; devolver outra vez é recusado.

### Tests for User Story 3 ⚠️

- [ ] T060 [P] [US3] Teste de contrato de `POST /v1/loans/{id}/return` (200; 404; 409 já devolvido; 403 para perfil `user`) em `services/loans/tests/contract/test_return_loan.py`
- [ ] T061 [P] [US3] Teste unitário das regras de devolução (data de devolução não anterior ao empréstimo; empréstimo já devolvido) em `services/loans/tests/unit/test_return_rules.py`
- [ ] T062 [P] [US3] Teste de integração da tela de devolução (sucesso, repetida, acesso negado ao usuário comum) em `services/web/tests/integration/test_return_page.py`

### Implementation for User Story 3

- [ ] T063 [US3] Implementar regras de devolução em `services/loans/src/loans/domain/return_rules.py` (faz T061 passar)
- [ ] T064 [US3] Implementar `return_loan` no serviço, gravando `returned_on` e `returned_by_id`, em `services/loans/src/loans/domain/service.py`
- [ ] T065 [US3] Implementar `POST /v1/loans/{id}/return` restrito a `admin` em `services/loans/src/loans/api/loans.py` (faz T060 passar)
- [ ] T066 [US3] Implementar a rota `POST /loans/{id}/return` e o botão de devolução em `services/web/src/web/routes/loans.py` e `services/web/src/web/templates/` (faz T062 passar)

**Checkpoint**: ciclo completo de empréstimo e devolução

---

## Phase 6: User Story 4 - Consultar situação dos equipamentos (Priority: P2)

**Goal**: qualquer logado vê os equipamentos disponíveis, emprestados e atrasados, com pessoa, datas e dias de atraso.

**Independent Test**: com equipamentos nos três estados, cada filtro mostra só os corretos; um prazo de ontem aparece com 1 dia de atraso.

### Tests for User Story 4 ⚠️

- [ ] T067 [P] [US4] Teste de contrato de `GET /v1/equipment-status` (filtros `available|loaned|overdue`, `ids`) e `GET /v1/loans` (admin vê todos, usuário vê só os seus) em `services/loans/tests/contract/test_status_and_history.py`
- [ ] T068 [P] [US4] Teste unitário da derivação de situação com relógio injetável: prazo hoje não é atraso; prazo ontem = 1 dia de atraso; sem empréstimo em aberto = disponível, em `services/loans/tests/unit/test_status.py`
- [ ] T069 [P] [US4] Teste de integração da lista com filtros, dados da pessoa e histórico em `services/web/tests/integration/test_status_pages.py`

### Implementation for User Story 4

- [ ] T070 [US4] Implementar a derivação de situação com relógio injetável (datas civis, dias de atraso = hoje − `due_on`) em `services/loans/src/loans/domain/status.py` (faz T068 passar)
- [ ] T071 [US4] Implementar consultas de empréstimos em aberto e de histórico em `services/loans/src/loans/infra/repository.py`
- [ ] T072 [US4] Implementar `GET /v1/equipment-status` e `GET /v1/loans` em `services/loans/src/loans/api/loans.py` (faz T067 passar)
- [ ] T073 [US4] Compor a lista de `/equipment` no web (catalog + `equipment-status` + `lookup` de nomes), com filtro `?status=` e exibição de pessoa, data do empréstimo, prazo e dias de atraso, em `services/web/src/web/routes/equipment.py` e `services/web/src/web/templates/equipment_list.html`
- [ ] T074 [US4] Implementar a página `/loans` (histórico) em `services/web/src/web/routes/loans.py` e `services/web/src/web/templates/loans_history.html` (faz T069 passar)

**Checkpoint**: todas as histórias funcionam de forma independente

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: documentação, segurança e verificações finais exigidas pela constituição

- [ ] T075 [P] Finalizar `docs/architecture.md` com diagrama dos quatro serviços, fluxo de login e fluxo de empréstimo
- [ ] T076 [P] Escrever registros de decisão em `docs/adr/0001-microservices.md`, `docs/adr/0002-jwt-rs256.md` e `docs/adr/0003-partial-unique-index.md` e `docs/adr/0004-tls-everywhere.md` (TLS na borda, entre serviços e nos bancos, com CA interna)
- [ ] T077 [P] Revisar `README.md` raiz, o `services/web/README.md` (esboço criado em T048) e `docs/glossary.md` para que uma pessoa nova rode o sistema e faça uma primeira alteração sem ajuda
- [ ] T078 [P] Garantir docstring em todo módulo, classe e função pública dos quatro serviços e zerar avisos do linter e do formatador
- [ ] T079 [P] Teste de contrato entre serviços: os clientes do `web` e do `loans` contra os contratos em `specs/001-equipment-loan-control/contracts/` em `services/web/tests/integration/test_client_contracts.py` e `services/loans/tests/integration/test_catalog_client_contract.py`
- [ ] T080 Reforço de segurança: cabeçalhos de segurança e cookie `Secure`/`HttpOnly`/`SameSite` no web, teste de que senhas e tokens não aparecem nos logs (eventos de segurança incluídos), em `services/web/src/web/main.py` e `services/*/tests/unit/test_log_redaction.py`
- [ ] T081 Script e teste de isolamento: apenas o `web` é alcançável de fora da rede do Compose e os demais serviços retornam 401 sem token, em `scripts/check_isolation.py`
- [ ] T082 Teste de falha parcial: com o `catalog` parado, `/equipment` mostra a página 503 amigável e os demais serviços seguem de pé, em `services/web/tests/integration/test_partial_failure.py`
- [ ] T083 Executar a validação de `quickstart.md` (os 14 cenários) e registrar o resultado

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: sem dependências
- **Foundational (Phase 2)**: depende do Setup; bloqueia todas as histórias
- **US1 (Phase 3)**: depende da Foundational
- **US2 (Phase 4)**: depende da Foundational; precisa de equipamentos e usuários (US1) para ser usada, mas os testes de contrato podem criar os dados por conta própria
- **US3 (Phase 5)**: depende de US2 (precisa de um empréstimo em aberto)
- **US4 (Phase 6)**: depende da Foundational; os dados vêm de US1 e US2
- **Polish (Phase 7)**: depende das histórias desejadas

### Within Each User Story

- Testes primeiro e vistos falhando
- Modelo antes de regras; regras antes de repositório; repositório antes de endpoints; serviço de dados antes do web

### Parallel Opportunities

- Setup: T002 a T011, T013, T015 e T016 em paralelo (T014 depende de T012 e T013)
- Foundational: os testes T017, T018, T026, T027, T031 em paralelo; as bases de catalog, loans e web (T028, T029, T030) em paralelo com o identity
- US1: o trabalho de catalog (T033, T034, T038 a T041) e o de identity (T035, T036, T042 a T044) em paralelo, por serem serviços separados
- US2 e US4 podem ser trabalhados em paralelo por pessoas diferentes após a Foundational

---

## Parallel Example: User Story 1

```text
# Testes de US1 juntos:
Task: "Teste de contrato da catalog API em services/catalog/tests/contract/test_equipment_api.py"
Task: "Teste de contrato da identity API em services/identity/tests/contract/test_users_api.py"
Task: "Teste de integração das telas em services/web/tests/integration/test_registration_pages.py"

# Implementação em serviços separados:
Task: "Modelo Equipment em services/catalog/src/catalog/infra/models.py"
Task: "Regras de usuário em services/identity/src/identity/domain/users.py"
```

---

## Implementation Strategy

### MVP First

1. Fase 1 (Setup) e Fase 2 (Foundational)
2. Fase 3 (US1), Fase 4 (US2) e Fase 5 (US3): as três histórias P1 formam o MVP funcional (cadastro, empréstimo e devolução)
3. **PARAR e VALIDAR** com os cenários 1 a 10 do `quickstart.md`
4. Adicionar US4 (consulta com atraso) e a Fase 7

### Incremental Delivery

1. Setup + Foundational → login funcionando
2. US1 → cadastro funcionando
3. US2 → empréstimo seguro contra duplicidade
4. US3 → ciclo completo
5. US4 → visão de disponíveis, emprestados e atrasados
6. Polish → documentação e segurança finais

---

## Notes

- [P] = arquivos diferentes, sem dependência pendente
- Cada serviço de dados é testado de forma isolada, sem depender de outro serviço no ar (usar dublê do catalog nos testes do loans)
- Não compartilhar código entre serviços; a verificação de token é implementada em cada um
- Commit a cada tarefa ou grupo lógico; PR só com testes, linter e formatador passando
