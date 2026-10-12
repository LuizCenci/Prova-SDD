# Research: Controle de Empréstimos de Equipamentos (Loanner)

Nenhum `NEEDS CLARIFICATION` ficou no plano. A pilha não foi definida pelo usuário, apenas a diretriz de microsserviços; as escolhas abaixo são propostas e podem ser trocadas antes de `/speckit-tasks`.

## 1. Divisão em serviços
- **Decision**: quatro serviços: `identity` (usuários, login, perfis), `catalog` (equipamentos), `loans` (empréstimos e situação) e `web` (interface).
- **Rationale**: cada um corresponde a uma capacidade do spec e muda por razões diferentes. A regra mais crítica (empréstimo duplicado) fica inteira no `loans`.
- **Alternatives**: aplicação única (descartada pela constituição, Princípio I); separar devolução em outro serviço (fragmentação excessiva para o MVP).

## 2. Linguagem e frameworks
- **Decision**: Python 3.12 em todos os serviços; FastAPI + Pydantic nas APIs; Jinja2 no `web`.
- **Rationale**: uma linguagem só reduz custo de manutenção e de onboarding; FastAPI gera contrato OpenAPI automaticamente.
- **Alternatives**: Node/TypeScript (equivalente, mas sem ganho claro); Flask (sem validação e documentação de contrato embutidas).

## 3. Bancos de dados
- **Decision**: PostgreSQL 16, um banco (e usuário de banco) por serviço dono de dados.
- **Rationale**: o Princípio I proíbe leitura do banco de outro serviço; PostgreSQL suporta índice único parcial e concorrência real.
- **Alternatives**: SQLite (não serve a múltiplos contêineres de forma segura); um banco compartilhado (viola o isolamento).

## 4. Impedir empréstimo duplicado (FR-005, SC-003)
- **Decision**: índice único parcial `UNIQUE(equipment_id) WHERE returned_on IS NULL` no banco do `loans`, além da checagem na regra de domínio; a violação vira resposta 409.
- **Rationale**: só o banco garante exclusão mútua com requisições simultâneas.
- **Alternatives**: checar só na aplicação (falha em concorrência); status gravado no catálogo (dois serviços divergiriam).

## 5. Consistência entre serviços
- **Decision**: `loans` guarda apenas os ids de equipamento e usuário (sem chave estrangeira entre bancos). Ao emprestar, chama o `catalog` para confirmar que o equipamento existe e está ativo. Para telas, o `web` compõe os dados chamando `catalog`, `loans` e `identity`.
- **Rationale**: mantém a propriedade dos dados; equipamentos e usuários nunca são excluídos (apenas desativados), então ids antigos continuam válidos no histórico.
- **Alternatives**: eventos assíncronos e réplicas locais (complexidade desnecessária para o MVP); chamadas do `catalog` ao `loans` (acoplamento invertido).

## 6. Situação "atrasado" (FR-008)
- **Decision**: o `loans` calcula a situação na consulta: sem empréstimo em aberto = disponível; em aberto com `due_on >= hoje` = emprestado; em aberto com `due_on < hoje` = atrasado, com dias de atraso = `hoje − due_on`.
- **Rationale**: atende "contado por dia" sem tarefas agendadas e sem estado desatualizado.
- **Alternatives**: gravar status e atualizar por job noturno.

## 7. Autenticação entre serviços (FR-013 a FR-015)
- **Decision**: `identity` faz login por e-mail e senha (hash Argon2) e emite JWT assinado com chave assimétrica (RS256) de curta duração, com `sub` (id do usuário) e `role`. Os demais serviços validam a assinatura com a chave pública do `identity` e aplicam as permissões por perfil. O `web` guarda o token na sessão do servidor (cookie HttpOnly) e o repassa nas chamadas.
- **Rationale**: nenhum serviço precisa consultar o `identity` a cada requisição; o segredo de assinatura fica só no `identity`.
- **Alternatives**: chave simétrica compartilhada (qualquer serviço poderia forjar tokens); sessão central consultada a cada chamada (acoplamento e latência); SSO institucional (depende de infraestrutura do cliente, fora do MVP).

## 8. Perfis e quem empresta
- **Decision**: perfis `admin` e `user`. O usuário empresta sempre em seu próprio nome (o `loans` usa o `sub` do token como `borrower_id`). Apenas `admin` registra devolução. O primeiro administrador é criado por comando de linha no `identity`. Qualquer pessoa pode criar a própria conta pelo `POST /v1/auth/register` (FR-018), que é público e sempre grava `role = user`, mesmo que o corpo tente enviar outro perfil; somente o admin cria administradores.
- **Rationale**: resposta B do clarify; reforça responsabilidade sobre o item.

## 9. Borda e transporte (todo tráfego criptografado)
- **Decision**: um proxy reverso é o único componente publicado ao navegador, com HTTPS e redirecionamento de HTTP. Entre serviços, e entre serviços e bancos, o tráfego também usa TLS, com certificados emitidos por uma autoridade certificadora (CA) interna gerada por script (`scripts/generate_certs.py`); os clientes validam o certificado. `identity`, `catalog` e `loans` ficam na rede interna e exigem token. Registrado em ADR 0004.
- **Rationale**: o Princípio II exige transporte criptografado em todo tráfego; a CA interna evita depender de serviços externos. Fica a cargo da operação trocar a CA interna por certificados da organização em produção.
- **Alternatives**: criptografar só na borda (viola o Princípio II); malha de serviço com mTLS (mais segura, porém pesada para o MVP e pode ser adotada depois).

## 10. Contratos e testes
- **Decision**: cada API publica contrato OpenAPI versionado em `/v1`, espelhado em `contracts/`. Testes de contrato verificam o cliente do `web` e do `loans` contra esses contratos. Datas são datas civis (sem hora), com a data local do servidor.
- **Rationale**: Princípios I e V.

## 11. Observabilidade e eventos de segurança
- **Decision**: `/health` e logs JSON em cada serviço, com id de correlação repassado entre chamadas; senhas e tokens nunca entram nos logs. Cada serviço registra eventos de segurança em um canal próprio de log: no identity, login com sucesso, login falho, tentativa de usuário inativo e conta criada por autocadastro; em todos, respostas 401 e 403, com id do usuário (quando houver), rota e horário.
- **Rationale**: Princípio II exige registrar eventos relevantes de segurança sem expor segredos nem dados pessoais além do necessário.

## 12. Usuário desativado (FR-016)
- **Decision**: ao criar um empréstimo, o `loans` consulta o `identity` (`GET /v1/users/{sub}`, com o token do chamador) e recusa com 403 se o usuário estiver inativo. O identity também recusa login de inativo.
- **Rationale**: o token vale até 30 minutos; sem essa consulta, um usuário desativado ainda poderia pegar itens nesse intervalo.
- **Alternatives**: aceitar o intervalo do token (rejeitado por enfraquecer a responsabilidade sobre o item); lista de tokens revogados (mais estado compartilhado).
