# Feature Specification: Controle de Empréstimos de Equipamentos

**Feature Branch**: `001-equipment-loan-control`

**Created**: 2026-10-05

**Status**: Draft

**Input**: User description: "O MVP será um sistema web simples para controle de empréstimos de equipamentos, voltado para uma pequena empresa, laboratório ou universidade. A aplicação permitirá cadastrar equipamentos e pessoas, registrar o empréstimo de um equipamento para uma pessoa e definir uma data prevista para devolução. Também será possível registrar a devolução e consultar quais equipamentos estão disponíveis, emprestados ou com a devolução atrasada. O sistema deverá impedir, por exemplo, que um equipamento já emprestado seja emprestado novamente."

## Clarifications

### Session 2026-10-05

- Q: Autenticação é necessária? → A: Sim. O usuário deve fazer login para fornecer suas informações e assumir responsabilidade pelo item que pegar ou ceder emprestado.
- Q: Equipamentos são unidades individuais? → A: Sim, a maioria dos itens é única, mas uma mesma pessoa pode ter mais de um item emprestado ao mesmo tempo.
- Q: Reserva, renovação de prazo, multas e notificações automáticas ficam fora do MVP? → A: Sim, por serem questões legais.
- Q: Como o atraso é contado? → A: Por dia.
- Q: Como as contas são criadas? → A: Somente o administrador cria contas, com e-mail e senha inicial; usuário desativado perde o acesso imediatamente.
- Q: Quais perfis de usuário existem? → A: Dois perfis. O administrador cadastra equipamentos e pessoas e registra devoluções; o usuário comum registra o próprio empréstimo e consulta a situação dos equipamentos.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Cadastrar equipamentos e pessoas (Priority: P1)

O administrador cadastra os equipamentos que podem ser emprestados e as pessoas que podem recebê-los, mantendo essas listas atualizadas.

**Why this priority**: Sem equipamentos e pessoas cadastrados não existe empréstimo; é a base de todo o sistema.

**Independent Test**: Cadastrar um equipamento e uma pessoa e verificar que ambos aparecem nas respectivas listas.

**Acceptance Scenarios**:

1. **Given** a lista de equipamentos, **When** o administrador cadastra um equipamento com nome e identificação, **Then** o equipamento aparece na lista com situação "disponível".
2. **Given** a lista de pessoas, **When** o administrador cadastra uma pessoa com nome, e-mail, contato, perfil e senha inicial, **Then** a pessoa aparece na lista e pode ser escolhida em um empréstimo.
3. **Given** um equipamento com a identificação "PAT-001" já cadastrado, **When** o administrador tenta cadastrar outro equipamento com a mesma identificação, **Then** o sistema recusa e informa o motivo.

---

### User Story 2 - Registrar empréstimo com data prevista de devolução (Priority: P1)

O usuário logado seleciona um equipamento disponível, define a data prevista de devolução e registra o empréstimo em seu próprio nome.

**Why this priority**: É a função central do sistema e o motivo de sua existência.

**Independent Test**: Com um equipamento e uma pessoa cadastrados, registrar um empréstimo e verificar que o equipamento passa a "emprestado" e não pode ser emprestado novamente.

**Acceptance Scenarios**:

1. **Given** um equipamento disponível e um usuário logado, **When** o usuário registra o empréstimo com uma data prevista futura, **Then** o equipamento passa a "emprestado" e o empréstimo mostra pessoa, data de saída e data prevista.
2. **Given** um equipamento já emprestado, **When** um usuário tenta emprestá-lo novamente, **Then** o sistema impede a operação e informa que o equipamento está emprestado.
3. **Given** o formulário de empréstimo, **When** a data prevista de devolução é anterior à data atual, **Then** o sistema recusa e pede uma data válida.

---

### User Story 3 - Registrar devolução (Priority: P1)

Quando a pessoa devolve o equipamento, o administrador registra a devolução e o equipamento volta a ficar disponível.

**Why this priority**: Sem a devolução, o equipamento nunca voltaria a ser emprestável.

**Independent Test**: Devolver um equipamento emprestado e verificar que ele volta a "disponível" e pode ser emprestado de novo.

**Acceptance Scenarios**:

1. **Given** um equipamento emprestado, **When** o administrador registra a devolução, **Then** o empréstimo é encerrado com a data real de devolução e o equipamento volta a "disponível".
2. **Given** um equipamento disponível, **When** alguém tenta registrar sua devolução, **Then** o sistema informa que não há empréstimo em aberto.

---

### User Story 4 - Consultar situação dos equipamentos (Priority: P2)

Qualquer usuário logado consulta os equipamentos filtrando por situação: disponíveis, emprestados ou com devolução atrasada.

**Why this priority**: Dá visibilidade do que está disponível e do que precisa ser cobrado, mas depende das histórias anteriores.

**Independent Test**: Com equipamentos nos três estados, aplicar cada filtro e verificar que só os equipamentos correspondentes aparecem.

**Acceptance Scenarios**:

1. **Given** equipamentos em situações variadas, **When** o usuário filtra por "disponíveis", **Then** vê apenas os equipamentos sem empréstimo em aberto.
2. **Given** equipamentos emprestados no prazo, **When** filtra por "emprestados", **Then** vê cada equipamento com a pessoa e a data prevista de devolução.
3. **Given** um empréstimo em aberto cuja data prevista já passou, **When** filtra por "atrasados", **Then** o equipamento aparece com a pessoa e os dias de atraso.

---

### Edge Cases

- Devolução registrada exatamente na data prevista: não é atraso.
- Prazo vencido durante a noite: o equipamento deve aparecer como atrasado no dia seguinte, sem ação manual.
- Tentativa simultânea de emprestar o mesmo equipamento a duas pessoas: apenas um empréstimo deve ser concluído.
- Tentativa de registrar empréstimo sem equipamento, pessoa ou data: o sistema recusa e indica o campo faltante.
- Pessoa com vários equipamentos emprestados ao mesmo tempo: permitido.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema MUST permitir cadastrar equipamentos com nome e identificação única.
- **FR-002**: O sistema MUST permitir que o administrador cadastre pessoas com nome, e-mail único (usado no login), contato, perfil (administrador ou usuário comum) e senha inicial. Não há cadastro aberto: somente o administrador cria contas.
- **FR-003**: O sistema MUST recusar equipamentos com identificação duplicada.
- **FR-004**: O sistema MUST permitir registrar o empréstimo de um equipamento a uma pessoa, com data prevista de devolução obrigatória.
- **FR-005**: O sistema MUST impedir o empréstimo de um equipamento que já possui empréstimo em aberto.
- **FR-006**: O sistema MUST recusar data prevista de devolução anterior à data do empréstimo.
- **FR-007**: O sistema MUST permitir registrar a devolução de um empréstimo em aberto, guardando a data real.
- **FR-008**: O sistema MUST classificar cada equipamento como "disponível", "emprestado" ou "atrasado" com base nos empréstimos em aberto e na data atual.
- **FR-009**: O sistema MUST permitir listar equipamentos filtrando por cada uma das três situações.
- **FR-010**: O sistema MUST exibir, para equipamentos emprestados ou atrasados, a pessoa, a data do empréstimo e a data prevista de devolução.
- **FR-011**: O sistema MUST manter o histórico de empréstimos já devolvidos, associados ao equipamento e à pessoa.
- **FR-012**: O sistema MUST exibir mensagens claras quando uma operação for recusada.
- **FR-013**: O sistema MUST exigir login para qualquer operação; usuários não autenticados não acessam nenhuma função.
- **FR-014**: O sistema MUST registrar, em cada empréstimo e devolução, o usuário autenticado que realizou a operação, para garantir responsabilidade sobre o item.
- **FR-015**: O sistema MUST oferecer dois perfis: administrador (cadastra equipamentos e pessoas, registra devoluções e consulta tudo) e usuário comum (registra empréstimos em seu próprio nome e consulta a situação dos equipamentos).
- **FR-016**: O sistema MUST impedir que um usuário desativado faça login ou pegue equipamentos emprestados, mesmo que ainda possua uma sessão ativa.

### Key Entities

- **Equipamento**: Item emprestável. Atributos: nome, identificação única, situação (derivada: disponível, emprestado, atrasado).
- **Pessoa**: Quem recebe equipamentos. Atributos: nome, e-mail, contato, perfil, situação ativo/inativo. Possui acesso por login (senha inicial definida pelo administrador) e responde pelos itens que pega ou cede emprestado.
- **Empréstimo**: Liga um equipamento a uma pessoa. Atributos: data do empréstimo, data prevista de devolução, data real de devolução (vazia enquanto em aberto).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Um usuário registra um empréstimo completo (equipamento, pessoa, data prevista) em menos de 1 minuto.
- **SC-002**: Um usuário registra uma devolução em menos de 30 segundos.
- **SC-003**: Em 100% das tentativas, um equipamento já emprestado não pode ser emprestado novamente.
- **SC-004**: A situação de qualquer equipamento (disponível, emprestado, atrasado) é identificada em até 10 segundos a partir da tela inicial.
- **SC-005**: 90% dos novos usuários concluem um empréstimo e uma devolução na primeira tentativa, sem treinamento.

## Assumptions

- Cada equipamento é uma unidade individual (sem controle de quantidade em estoque); uma pessoa pode ter vários equipamentos emprestados ao mesmo tempo.
- Reserva antecipada, renovação de prazo, multas e notificações automáticas de atraso estão fora do escopo do MVP, por envolverem questões legais.
- Atraso é calculado por dia: o equipamento está atrasado quando a data atual é posterior à data prevista.
- Equipamentos e pessoas com histórico de empréstimos não podem ser excluídos.
- O sistema é acessado por navegador em computador; uso em celular é desejável, mas não obrigatório.
