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
- **[Story]**: história de usuário a que a tarefa pertence (US1 a US5; o login da US5 é entregue na fase Foundational, sem rótulo, por bloquear as demais; o autocadastro da US5 tem fase própria, com rótulo [US5])
- Caminhos relativos à raiz do repositório
- Todo código público MUST ter docstring (Princípio V) e todo comentário explica o porquê
- Escopo (Princípio VII): nenhuma tarefa implementa itens da seção Out of Scope da spec (reserva, renovação, multas, notificações, estoque, empréstimo em nome de outra pessoa, recuperação de senha por e-mail, SSO, exclusão, app nativo)

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

**Purpose**: login, verificação de token e base de cada serviço. Nenhuma história começa antes disto. Esta fase entrega o login da US5 (cenários 1 a 5): T018, T024, T031 e T032. O autocadastro (cenários 6 a 8, FR-018) fica na Phase 4, pois depende das regras e do repositório de usuários da US1.

**⚠️ CRITICAL**: sem login funcionando, nenhuma história de usuário pode ser usada ou testada

- [ ] T017 [P] Escrever teste unitário de hash de senha (Argon2; senha nunca devolvida) em `services/identity/tests/unit/test_passwords.py`
- [ ] T018 [P] Escrever teste de contrato de `POST /v1/auth/login` (200 com `access_token` e `expires_in`; 401 para credencial inválida ou usuário inativo) e `GET /v1/.well-known/jwks.json`, e que cada login falho gera um evento de segurança registrado (sem senha) em `services/identity/tests/contract/test_auth_contract.py`
- [ ] T019 Criar configuração por variáveis de ambiente em `services/identity/src/identity/config.py`
- [ ] T020 Configurar sessão de banco e Alembic em `services/identity/src/identity/infra/database.py` e `services/identity/migrations/env.py`
- [ ] T021 Criar modelo `User` e migração inicial em `services/identity/src/identity/infra/models.py` e `services/identity/migrations/versions/0001_users.py` com os campos: id UUID chave primária; name texto obrigatório; email texto obrigatório, único; contact texto opcional; password_hash texto (Argon2, nunca exposto); role `admin` | `user` obrigatório, padrão `user`; active booleano, padrão verdadeiro
- [ ] T022 Implementar hash e verificação de senha em `services/identity/src/identity/domain/passwords.py` (faz T017 passar)
- [ ] T023 Implementar emissão de JWT RS256 (claims `sub`, `role`, `exp`; validade de 30 minutos) e JWKS em `services/identity/src/identity/infra/tokens.py`
- [ ] T024 Criar a aplicação, a rota de login e `/health` com logs JSON que nunca registram senha nem token, e registrar eventos de segurança (login com sucesso, login falho, usuário inativo) em `services/identity/src/identity/infra/security_log.py` em `services/identity/src/identity/main.py` e `services/identity/src/identity/api/auth.py` (faz T018 passar)
- [ ] T025 Criar o comando de linha `create-admin` (o primeiro administrador é criado na instalação, conforme Assumptions da spec; o autocadastro nunca cria administrador) e seu teste de integração em `services/identity/src/identity/cli.py` e `services/identity/tests/integration/test_cli.py`
- [ ] T026 [P] Escrever teste de verificação de token (assinatura inválida, expirado, sem token → 401; perfil insuficiente → 403), e que cada 401 e 403 gera um evento de segurança registrado em `services/catalog/tests/unit/test_auth.py`
- [ ] T027 [P] Escrever o mesmo teste de verificação de token e de evento de segurança em `services/loans/tests/unit/test_auth.py`
- [ ] T028 [P] Criar base do catalog: `config.py`, `infra/database.py`, `migrations/env.py`, `infra/auth.py` (busca a chave pública no JWKS do identity e oferece `require_role`), logs JSON, registro de eventos de segurança (401 e 403) em `infra/security_log.py`, `main.py` e `/health`, com servidor TLS em `services/catalog/src/catalog/` (faz T026 passar)
- [ ] T029 [P] Criar base do loans com os mesmos itens de T028 (inclusive eventos de segurança e TLS) em `services/loans/src/loans/` (faz T027 passar)
- [ ] T030 [P] Criar base do web em `services/web/src/web/`: `config.py`, sessão com cookie HttpOnly, proteção CSRF, logs JSON, registro de eventos de segurança (login falho e 403), `clients/base.py` (repassa o token e um id de correlação, e valida o certificado dos serviços com a CA interna), `templates/base.html` responsivo, páginas de erro 403 e 503 com a mensagem "Serviço temporariamente indisponível. Tente novamente em instantes."
- [ ] T031 [P] Escrever teste de integração de login e logout (sem login, qualquer tela redireciona para `/login`; e-mail ou senha errados mostram "E-mail ou senha inválidos." sem indicar qual campo errou; usuário desativado com credenciais corretas é recusado; após `/logout`, `/equipment` volta a redirecionar para `/login`; login falho gera evento de segurança) em `services/web/tests/integration/test_login.py`
- [ ] T032 Implementar rotas `/login` e `/logout` e o template em `services/web/src/web/routes/auth.py` e `services/web/src/web/templates/login.html` (faz T031 passar)

**Checkpoint**: usuário consegue entrar; todos os serviços validam o token

---

## Phase 3: User Story 1 - Cadastrar equipamentos e pessoas (Priority: P1) 🎯 MVP

**Goal**: o administrador cadastra, edita e desativa equipamentos e usuários (pessoas), sem exclusão (FR-017); qualquer logado lista os equipamentos ativos.

**Independent Test**: como administrador, cadastrar um equipamento e um usuário e vê-los nas listas; tentar repetir o código do equipamento ou o e-mail da pessoa e ser recusado; desativar o equipamento e vê-lo sumir da lista.

### Tests for User Story 1 ⚠️ (escrever primeiro e ver falhar)

- [ ] T033 [P] [US1] Teste de contrato da catalog API (`POST` 201 / 400 campo faltante / 409 código duplicado; `PATCH` edita nome e desativa com `active: false`, mantendo o mesmo `id`; `GET` lista e detalhe devolvem `active`; 401 sem token; 403 para perfil `user` em escrita; não existe rota de exclusão) em `services/catalog/tests/contract/test_equipment_api.py`
- [ ] T034 [P] [US1] Teste unitário das regras de equipamento (nome e código obrigatórios; equipamento inativo não pode ser emprestado) em `services/catalog/tests/unit/test_equipment_rules.py`
- [ ] T035 [P] [US1] Teste de contrato da identity API de usuários (`POST` 201 / 400 / 409 e-mail duplicado; `PATCH` com `{name?, contact?, role?, password?, active?}` edita dados, define nova senha (o login passa a aceitar só a nova) e desativa; `GET`; `lookup` devolve só `{id, name}`; apenas admin usa `/v1/users` para criar e editar) em `services/identity/tests/contract/test_users_api.py`
- [ ] T036 [P] [US1] Teste unitário das regras de usuário (e-mail único, perfil válido, usuário inativo não faz login) em `services/identity/tests/unit/test_user_rules.py`
- [ ] T037 [P] [US1] Teste de integração das telas de cadastro de equipamentos e pessoas: código duplicado ("Já existe um equipamento com este código."); e-mail duplicado ("Já existe uma pessoa com este e-mail."); edição de nome e contato refletida na lista; equipamento desativado some da lista `/equipment` (fica oculto) e sua tela de empréstimo responde 404; pessoa desativada não entra mais; perfil `user` recebe 403 ao cadastrar, editar ou desativar; em `services/web/tests/integration/test_registration_pages.py`

### Implementation for User Story 1

- [ ] T038 [P] [US1] Criar modelo `Equipment` e migração em `services/catalog/src/catalog/infra/models.py` e `services/catalog/migrations/versions/0001_equipment.py` com os campos: id UUID chave primária; name texto obrigatório; code texto obrigatório, **único**; active booleano, padrão verdadeiro
- [ ] T039 [US1] Implementar regras de equipamento em `services/catalog/src/catalog/domain/equipment.py` (faz T034 passar)
- [ ] T040 [US1] Implementar repositório de equipamentos (traduz violação de unicidade em erro de domínio) em `services/catalog/src/catalog/infra/repository.py`
- [ ] T041 [US1] Implementar endpoints de equipamentos conforme `contracts/catalog-api.md` em `services/catalog/src/catalog/api/equipment.py` (faz T033 passar)
- [ ] T042 [P] [US1] Implementar regras de usuário em `services/identity/src/identity/domain/users.py` (faz T036 passar)
- [ ] T043 [US1] Implementar repositório de usuários em `services/identity/src/identity/infra/repository.py`
- [ ] T044 [US1] Implementar endpoints `/v1/users`, `/v1/users/{id}` e `/v1/users/lookup` conforme `contracts/identity-api.md` em `services/identity/src/identity/api/users.py` (faz T035 passar)
- [ ] T045 [US1] Criar clientes HTTP do catalog e do identity em `services/web/src/web/clients/catalog.py` e `services/web/src/web/clients/identity.py`
- [ ] T046 [US1] Implementar rotas e templates de equipamentos (lista que oculta equipamentos inativos, `/equipment/new`, `/equipment/{id}/edit` com campos nome, código e ativo; sem botão ou rota de exclusão) com as mensagens de `contracts/web-routes.md` em `services/web/src/web/routes/equipment.py` e `services/web/src/web/templates/equipment_*.html`
- [ ] T047 [US1] Implementar rotas e templates de pessoas (`/people`, `/people/new` com nome, e-mail, contato, perfil e senha inicial; `/people/{id}/edit` com nome, contato, perfil, nova senha opcional e ativo; sem exclusão) em `services/web/src/web/routes/people.py` e `services/web/src/web/templates/people_*.html` (faz T037 passar)
- [ ] T048 [P] [US1] Escrever `services/catalog/README.md`, `services/identity/README.md` e o esboço de `services/web/README.md` (responsabilidade, contratos, dados que possui, configuração, como testar)

**Checkpoint**: cadastro de equipamentos e pessoas funciona sozinho (MVP de dados)

---

## Phase 4: User Story 5 - Entrar no sistema: autocadastro (Priority: P1)

**Goal**: qualquer pessoa, sem login, cria a própria conta de usuário comum, sem aprovação do administrador (FR-018; US5, cenários 6 a 8).

**Independent Test**: sem login, abrir `/signup`, criar uma conta, entrar com ela e registrar um empréstimo; repetir o e-mail e ser recusado.

### Tests for User Story 5 ⚠️

- [ ] T049 [P] [US5] Teste de contrato de `POST /v1/auth/register` (público, sem token; 201 `{id, name, email}` cria usuário com `role = user` e `active = true`; um `role: admin` enviado no corpo é ignorado e a conta nasce `user`; 400 nome, e-mail ou senha ausentes; 409 e-mail duplicado; a senha é gravada só como hash Argon2; a conta criada gera um evento de segurança sem senha; o login com a nova conta devolve 200) em `services/identity/tests/contract/test_register_contract.py`
- [ ] T050 [P] [US5] Teste de integração da tela de autocadastro (`/signup` abre sem login; o formulário tem nome, e-mail, contato e senha, sem campo de perfil, e exige token CSRF; sucesso redireciona para `/login` e a pessoa consegue entrar como usuário comum; e-mail repetido mostra "Já existe uma pessoa com este e-mail."; a tela de login tem link para `/signup`) em `services/web/tests/integration/test_signup.py`

### Implementation for User Story 5

- [ ] T051 [US5] Implementar `POST /v1/auth/register` conforme `contracts/identity-api.md`, reutilizando as regras de `services/identity/src/identity/domain/users.py` e o repositório de `services/identity/src/identity/infra/repository.py`, sempre com `role = user` e `active = true`, e registrando o evento de segurança "conta criada por autocadastro" em `services/identity/src/identity/infra/security_log.py`, em `services/identity/src/identity/api/auth.py` (faz T049 passar)
- [ ] T052 [US5] Implementar a rota `/signup` (GET/POST, pública, com CSRF) e o template, e incluir o link na tela de login, em `services/web/src/web/routes/auth.py`, `services/web/src/web/clients/identity.py`, `services/web/src/web/templates/signup.html` e `services/web/src/web/templates/login.html` (faz T050 passar)

**Checkpoint**: uma pessoa nova cria a própria conta e entra sem depender do administrador

---

## Phase 5: User Story 2 - Registrar empréstimo com data prevista (Priority: P1)

**Goal**: o usuário logado empresta um equipamento disponível em seu nome, com data prevista; o mesmo equipamento não pode ser emprestado duas vezes.

**Independent Test**: emprestar um equipamento e ver "emprestado"; uma segunda tentativa é recusada, inclusive duas tentativas simultâneas.

### Tests for User Story 2 ⚠️

- [ ] T053 [P] [US2] Teste de contrato de `POST /v1/loans` (201; 400 `due_on` ausente ou anterior a hoje; 404 equipamento inexistente ou inativo; 409 já emprestado; corpo com `borrower_id` é ignorado e o tomador é o `sub` do token; usuário inativo → 403 "Usuário inativo não pode pegar equipamentos emprestados.") em `services/loans/tests/contract/test_create_loan.py`
- [ ] T054 [P] [US2] Teste unitário das regras de empréstimo (`due_on >= loaned_on`, data obrigatória; `due_on` igual à data de hoje é aceito; `loaned_on` é sempre a data atual) em `services/loans/tests/unit/test_loan_rules.py`
- [ ] T055 [P] [US2] Teste de integração de concorrência: duas requisições simultâneas para o mesmo equipamento resultam em exatamente um 201 e um 409, contra PostgreSQL real, em `services/loans/tests/integration/test_concurrent_loan.py`
- [ ] T056 [P] [US2] Teste de integração da tela de empréstimo (sucesso mostra o próprio usuário como tomador, a data de saída e o prazo; o formulário não tem campo de tomador; data prevista ausente indica o campo faltante; já emprestado → "Este equipamento já está emprestado."; prazo anterior a hoje → "A data prevista de devolução não pode ser anterior a hoje."; usuário desativado com sessão ativa → 403 "Usuário inativo não pode pegar equipamentos emprestados."; vale para os perfis `admin` e `user`) em `services/web/tests/integration/test_loan_page.py`

### Implementation for User Story 2

- [ ] T057 [US2] Criar modelo `Loan` e migração em `services/loans/src/loans/infra/models.py` e `services/loans/migrations/versions/0001_loans.py` com os campos: id UUID chave primária; equipment_id UUID obrigatório; borrower_id UUID obrigatório; loaned_on data obrigatória; due_on data obrigatória, `due_on >= loaned_on`; returned_on data vazia enquanto em aberto, `>= loaned_on`; loaned_by_id UUID; returned_by_id UUID vazio enquanto em aberto; e **índice único parcial** `UNIQUE(equipment_id) WHERE returned_on IS NULL`
- [ ] T058 [US2] Implementar regras de empréstimo em `services/loans/src/loans/domain/loan_rules.py` (faz T054 passar)
- [ ] T059 [US2] Implementar cliente do catalog (confirma que o equipamento existe e está ativo) em `services/loans/src/loans/infra/catalog_client.py` e cliente do identity (consulta `GET /v1/users/{sub}` com o token do chamador e confirma que o tomador está ativo, mesmo que o token ainda não tenha expirado) em `services/loans/src/loans/infra/identity_client.py`
- [ ] T060 [US2] Implementar repositório e serviço `create_loan`, traduzindo a violação do índice único em conflito 409, em `services/loans/src/loans/infra/repository.py` e `services/loans/src/loans/domain/service.py`
- [ ] T061 [US2] Implementar `POST /v1/loans` conforme `contracts/loans-api.md`, gravando `loaned_by_id` e `borrower_id` a partir do token, em `services/loans/src/loans/api/loans.py` (faz T053 e T055 passarem)
- [ ] T062 [US2] Implementar cliente do loans e a rota `/equipment/{id}/loan` com formulário e template em `services/web/src/web/clients/loans.py`, `services/web/src/web/routes/loans.py` e `services/web/src/web/templates/loan_form.html` (faz T056 passar)
- [ ] T063 [P] [US2] Escrever `services/loans/README.md` (responsabilidade, contrato, índice único, configuração, como testar)

**Checkpoint**: empréstimo funciona e é seguro contra duplicidade

---

## Phase 6: User Story 3 - Registrar devolução (Priority: P1)

**Goal**: o administrador registra a devolução e o equipamento volta a ficar disponível.

**Independent Test**: devolver um equipamento emprestado e emprestá-lo de novo; devolver outra vez é recusado; um usuário comum não consegue devolver.

### Tests for User Story 3 ⚠️

- [ ] T064 [P] [US3] Teste de contrato de `POST /v1/loans/{id}/return` (200; 404; 409 já devolvido; 403 para perfil `user`, e o empréstimo continua em aberto; devolução funciona para tomador desativado e para equipamento desativado) em `services/loans/tests/contract/test_return_loan.py`
- [ ] T065 [P] [US3] Teste unitário das regras de devolução (data de devolução não anterior ao empréstimo; empréstimo já devolvido) em `services/loans/tests/unit/test_return_rules.py`
- [ ] T066 [P] [US3] Teste de integração da tela de devolução (sucesso; devolução repetida → "Não há empréstimo em aberto para este equipamento."; acesso negado ao usuário comum; devolução de item atrasado encerra o empréstimo e o histórico mostra data prevista e data real; equipamento inativo devolvido fica oculto da lista) em `services/web/tests/integration/test_return_page.py`

### Implementation for User Story 3

- [ ] T067 [US3] Implementar regras de devolução em `services/loans/src/loans/domain/return_rules.py` (faz T065 passar)
- [ ] T068 [US3] Implementar `return_loan` no serviço, gravando `returned_on` e `returned_by_id`, em `services/loans/src/loans/domain/service.py`
- [ ] T069 [US3] Implementar `POST /v1/loans/{id}/return` restrito a `admin` em `services/loans/src/loans/api/loans.py` (faz T064 passar)
- [ ] T070 [US3] Implementar a rota `POST /loans/{id}/return` (409 vira a mensagem "Não há empréstimo em aberto para este equipamento.") e o botão de devolução, visível só para `admin`, em `services/web/src/web/routes/loans.py` e `services/web/src/web/templates/` (faz T066 passar)

**Checkpoint**: ciclo completo de empréstimo e devolução

---

## Phase 7: User Story 4 - Consultar situação dos equipamentos (Priority: P2)

**Goal**: qualquer logado vê os equipamentos disponíveis, emprestados e atrasados, com pessoa, datas e dias de atraso.

**Independent Test**: com equipamentos nos três estados, cada filtro mostra só os corretos; um prazo de ontem aparece com 1 dia de atraso.

### Tests for User Story 4 ⚠️

- [ ] T071 [P] [US4] Teste de contrato de `GET /v1/equipment-status` (filtros `available|loaned|overdue`, `ids`) e `GET /v1/loans` (admin vê todos, usuário vê só os seus) em `services/loans/tests/contract/test_status_and_history.py`
- [ ] T072 [P] [US4] Teste unitário da derivação de situação com relógio injetável: prazo hoje não é atraso; prazo ontem = 1 dia de atraso; sem empréstimo em aberto = disponível, em `services/loans/tests/unit/test_status.py`
- [ ] T073 [P] [US4] Teste de integração da lista com filtros, dados da pessoa e histórico (equipamento inativo sem empréstimo em aberto não aparece em nenhum filtro; inativo com empréstimo em aberto aparece como emprestado ou atrasado até a devolução; em `/loans`, o administrador vê o histórico de todos e o usuário comum vê apenas o próprio, conforme FR-015) em `services/web/tests/integration/test_status_pages.py`

### Implementation for User Story 4

- [ ] T074 [US4] Implementar a derivação de situação com relógio injetável (datas civis, dias de atraso = hoje − `due_on`) em `services/loans/src/loans/domain/status.py` (faz T072 passar)
- [ ] T075 [US4] Implementar consultas de empréstimos em aberto e de histórico em `services/loans/src/loans/infra/repository.py`
- [ ] T076 [US4] Implementar `GET /v1/equipment-status` e `GET /v1/loans` em `services/loans/src/loans/api/loans.py` (faz T071 passar)
- [ ] T077 [US4] Compor a lista de `/equipment` no web (catalog + `equipment-status` + `lookup` de nomes), com filtro `?status=`, exibição de pessoa, data do empréstimo, prazo e dias de atraso, e ocultando equipamentos inativos sem empréstimo em aberto (FR-017), em `services/web/src/web/routes/equipment.py` e `services/web/src/web/templates/equipment_list.html`
- [ ] T078 [US4] Implementar a página `/loans` (histórico) em `services/web/src/web/routes/loans.py` e `services/web/src/web/templates/loans_history.html` (faz T073 passar)

**Checkpoint**: todas as histórias funcionam de forma independente

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: documentação, segurança e verificações finais exigidas pela constituição

- [ ] T079 [P] Finalizar `docs/architecture.md` com diagrama dos quatro serviços, fluxo de login e fluxo de empréstimo
- [ ] T080 [P] Escrever registros de decisão em `docs/adr/0001-microservices.md`, `docs/adr/0002-jwt-rs256.md` e `docs/adr/0003-partial-unique-index.md` e `docs/adr/0004-tls-everywhere.md` (TLS na borda, entre serviços e nos bancos, com CA interna)
- [ ] T081 [P] Revisar `README.md` raiz, o `services/web/README.md` (esboço criado em T048) e `docs/glossary.md` para que uma pessoa nova rode o sistema e faça uma primeira alteração sem ajuda
- [ ] T082 [P] Garantir docstring em todo módulo, classe e função pública dos quatro serviços e zerar avisos do linter e do formatador
- [ ] T083 [P] Teste de contrato entre serviços: os clientes do `web` e do `loans` contra os contratos em `specs/001-equipment-loan-control/contracts/` em `services/web/tests/integration/test_client_contracts.py` e `services/loans/tests/integration/test_catalog_client_contract.py`
- [ ] T084 Reforço de segurança: cabeçalhos de segurança e cookie `Secure`/`HttpOnly`/`SameSite` no web, teste de que senhas e tokens não aparecem nos logs (eventos de segurança incluídos), em `services/web/src/web/main.py` e `services/*/tests/unit/test_log_redaction.py`
- [ ] T085 Script e teste de isolamento: apenas o `web` é alcançável de fora da rede do Compose e os demais serviços retornam 401 sem token, em `scripts/check_isolation.py`
- [ ] T086 Teste de falha parcial: com o `catalog` parado, `/equipment` mostra a página 503 amigável e os demais serviços seguem de pé, em `services/web/tests/integration/test_partial_failure.py`
- [ ] T087 Executar a validação de `quickstart.md` (os 21 cenários) e registrar o resultado

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: sem dependências
- **Foundational (Phase 2)**: depende do Setup; bloqueia todas as histórias
- **US1 (Phase 3)**: depende da Foundational
- **US5 autocadastro (Phase 4)**: depende de US1 (regras e repositório de usuários do identity, T042 e T043, e cliente do identity no web, T045)
- **US2 (Phase 5)**: depende da Foundational; precisa de equipamentos e usuários (US1) para ser usada, mas os testes de contrato podem criar os dados por conta própria
- **US3 (Phase 6)**: depende de US2 (precisa de um empréstimo em aberto)
- **US4 (Phase 7)**: depende da Foundational; os dados vêm de US1 e US2
- **Polish (Phase 8)**: depende das histórias desejadas

### Within Each User Story

- Testes primeiro e vistos falhando
- Modelo antes de regras; regras antes de repositório; repositório antes de endpoints; serviço de dados antes do web

### Parallel Opportunities

- Setup: T002 a T011, T013, T015 e T016 em paralelo (T014 depende de T012 e T013)
- Foundational: os testes T017, T018, T026, T027, T031 em paralelo; as bases de catalog, loans e web (T028, T029, T030) em paralelo com o identity
- US1: o trabalho de catalog (T033, T034, T038 a T041) e o de identity (T035, T036, T042 a T044) em paralelo, por serem serviços separados
- US5 autocadastro: T049 e T050 em paralelo
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
2. Fase 3 (US1), Fase 4 (autocadastro da US5), Fase 5 (US2) e Fase 6 (US3): as histórias P1 formam o MVP funcional (cadastro, autocadastro, empréstimo e devolução)
3. **PARAR e VALIDAR** com os cenários 1 a 7, 9, 10, 13, 15 a 17, 19 e 21 do `quickstart.md` (os demais dependem da US4)
4. Adicionar US4 (consulta com atraso) e a Fase 8

### Incremental Delivery

1. Setup + Foundational → login funcionando
2. US1 → cadastro funcionando
3. US5 (autocadastro) → pessoas criam a própria conta
4. US2 → empréstimo seguro contra duplicidade
5. US3 → ciclo completo
6. US4 → visão de disponíveis, emprestados e atrasados
7. Polish → documentação e segurança finais

---

## Notes

- [P] = arquivos diferentes, sem dependência pendente
- Cada serviço de dados é testado de forma isolada, sem depender de outro serviço no ar (usar dublê do catalog nos testes do loans)
- Não compartilhar código entre serviços; a verificação de token é implementada em cada um
- Commit a cada tarefa ou grupo lógico; PR só com testes, linter e formatador passando
