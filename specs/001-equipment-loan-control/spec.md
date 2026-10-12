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
- Q: Quais perfis de usuário existem? → A: Dois perfis. O administrador cadastra equipamentos e pessoas e registra devoluções; o usuário comum registra o próprio empréstimo e consulta a situação dos equipamentos.

### Session 2026-10-12

- Q: Como um equipamento inativo aparece na lista de equipamentos? → A: Não aparece; fica oculto do catálogo para todos os perfis.
- Q: O usuário comum pode consultar o histórico de empréstimos? → A: Sim, apenas o próprio histórico; o administrador vê o histórico de todos.
- Q: Somente o administrador cria contas? → A: Não. Qualquer pessoa pode criar a própria conta de usuário comum, por conta própria, sem aprovação do administrador. O administrador continua podendo cadastrar pessoas, e só ele cria administradores. Usuário desativado continua perdendo o acesso imediatamente.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Cadastrar equipamentos e pessoas (Priority: P1)

O administrador cadastra os equipamentos que podem ser emprestados e as pessoas que podem recebê-los, mantendo essas listas atualizadas.

**Why this priority**: Sem equipamentos e pessoas cadastrados não existe empréstimo; é a base de todo o sistema.

**Independent Test**: Cadastrar um equipamento e uma pessoa e verificar que ambos aparecem nas respectivas listas.

**Acceptance Scenarios**:

1. **Given** a lista de equipamentos, **When** o administrador cadastra um equipamento com nome e identificação, **Then** o equipamento aparece na lista com situação "disponível".
2. **Given** a lista de pessoas, **When** o administrador cadastra uma pessoa com nome, e-mail, contato, perfil e senha inicial, **Then** a pessoa aparece na lista, ativa, e passa a poder entrar no sistema e registrar empréstimos em seu próprio nome.
3. **Given** um equipamento com a identificação "PAT-001" já cadastrado, **When** o administrador tenta cadastrar outro equipamento com a mesma identificação, **Then** o sistema recusa e informa o motivo.
4. **Given** uma pessoa com o e-mail "ana@exemplo.com" já cadastrada, **When** o administrador tenta cadastrar outra pessoa com o mesmo e-mail, **Then** o sistema recusa e informa o motivo.
5. **Given** um equipamento ou uma pessoa cadastrados, **When** o administrador altera o nome ou o contato, **Then** a lista mostra os dados novos e o histórico de empréstimos continua associado ao mesmo registro.
6. **Given** um equipamento disponível, **When** o administrador o desativa, **Then** ele deixa de aparecer na lista de equipamentos (fica oculto) e não pode ser emprestado.
7. **Given** uma pessoa ativa, **When** o administrador a desativa, **Then** ela não consegue mais entrar no sistema nem registrar empréstimos, e seus empréstimos em aberto continuam visíveis e podem ser devolvidos.
8. **Given** um usuário comum logado, **When** ele tenta cadastrar, editar ou desativar equipamentos ou pessoas, **Then** o sistema recusa por falta de permissão.

---

### User Story 2 - Registrar empréstimo com data prevista de devolução (Priority: P1)

O usuário logado seleciona um equipamento disponível, define a data prevista de devolução e registra o empréstimo em seu próprio nome.

**Why this priority**: É a função central do sistema e o motivo de sua existência.

**Independent Test**: Com um equipamento e uma pessoa cadastrados, entrar como essa pessoa, registrar um empréstimo e verificar que o equipamento passa a "emprestado" e não pode ser emprestado novamente.

**Acceptance Scenarios**:

1. **Given** um equipamento disponível e um usuário logado, **When** o usuário registra o empréstimo com uma data prevista igual ou posterior à data atual, **Then** o equipamento passa a "emprestado" e o empréstimo mostra o próprio usuário como tomador, a data de saída (data atual) e a data prevista.
2. **Given** um equipamento já emprestado, **When** um usuário tenta emprestá-lo novamente, **Then** o sistema impede a operação e informa que o equipamento está emprestado.
3. **Given** o formulário de empréstimo, **When** a data prevista de devolução é anterior à data atual, **Then** o sistema recusa e pede uma data válida.
4. **Given** um usuário logado, administrador ou comum, **When** registra um empréstimo, **Then** o tomador é sempre o próprio usuário; o sistema não oferece a opção de emprestar em nome de outra pessoa.
5. **Given** um usuário que foi desativado enquanto estava logado, **When** tenta registrar um empréstimo, **Then** o sistema recusa com a mensagem "Usuário inativo não pode pegar equipamentos emprestados."

---

### User Story 3 - Registrar devolução (Priority: P1)

Quando a pessoa devolve o equipamento, o administrador registra a devolução e o equipamento volta a ficar disponível.

**Why this priority**: Sem a devolução, o equipamento nunca voltaria a ser emprestável.

**Independent Test**: Devolver um equipamento emprestado e verificar que ele volta a "disponível" e pode ser emprestado de novo.

**Acceptance Scenarios**:

1. **Given** um equipamento emprestado, **When** o administrador registra a devolução, **Then** o empréstimo é encerrado com a data real de devolução e o equipamento volta a "disponível".
2. **Given** um equipamento disponível, **When** o administrador tenta registrar sua devolução, **Then** o sistema informa que não há empréstimo em aberto.
3. **Given** um equipamento emprestado, **When** um usuário comum tenta registrar a devolução, **Then** o sistema recusa por falta de permissão e o empréstimo continua em aberto.
4. **Given** um equipamento atrasado, **When** o administrador registra a devolução, **Then** o empréstimo é encerrado normalmente e o histórico mostra a data prevista e a data real.

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

### User Story 5 - Entrar no sistema (Priority: P1)

A pessoa cria a própria conta de usuário comum, ou recebe uma do administrador, e entra com e-mail e senha para usar o sistema; sem login, nenhuma função fica acessível além do login e do autocadastro.

**Why this priority**: Todas as outras histórias dependem de saber quem está operando, para aplicar o perfil e registrar a responsabilidade sobre o item.

**Independent Test**: Criar uma conta pela tela de autocadastro, entrar com as credenciais corretas, sair, e verificar que, sem login, as telas do sistema não abrem.

**Acceptance Scenarios**:

1. **Given** uma pessoa ativa cadastrada, **When** ela informa e-mail e senha corretos, **Then** entra no sistema e vê as funções do seu perfil.
2. **Given** a tela de login, **When** o e-mail ou a senha estão errados, **Then** o sistema recusa com a mensagem "E-mail ou senha inválidos.", sem indicar qual dos dois está errado.
3. **Given** uma pessoa desativada, **When** ela informa e-mail e senha corretos, **Then** o sistema recusa o acesso.
4. **Given** um visitante sem login, **When** tenta abrir qualquer tela do sistema que não seja o login ou o autocadastro, **Then** é levado para a tela de login.
5. **Given** um usuário logado, **When** ele sai do sistema, **Then** precisa entrar de novo para acessar qualquer tela.
6. **Given** um visitante sem login, **When** cria a própria conta informando nome, e-mail, contato e senha, **Then** a conta é criada ativa, com perfil usuário comum, sem aprovação do administrador, e a pessoa passa a poder entrar no sistema e registrar empréstimos em seu próprio nome.
7. **Given** uma pessoa com o e-mail "ana@exemplo.com" já cadastrada, **When** um visitante tenta criar uma conta com o mesmo e-mail, **Then** o sistema recusa com a mensagem "Já existe uma pessoa com este e-mail."
8. **Given** a tela de autocadastro, **When** o visitante cria a conta, **Then** não há opção de escolher o perfil: a conta criada por conta própria é sempre de usuário comum.

---

### Edge Cases

- Devolução registrada exatamente na data prevista: não é atraso.
- Prazo vencido durante a noite: o equipamento deve aparecer como atrasado no dia seguinte, sem ação manual.
- Tentativa simultânea de emprestar o mesmo equipamento a duas pessoas: apenas um empréstimo deve ser concluído.
- Tentativa de registrar empréstimo sem equipamento ou sem data prevista: o sistema recusa e indica o campo faltante.
- Data prevista igual à data do empréstimo (devolução no mesmo dia): permitido; o equipamento só fica atrasado a partir do dia seguinte.
- Pessoa com vários equipamentos emprestados ao mesmo tempo: permitido.
- Equipamento desativado enquanto está emprestado: o empréstimo continua em aberto, aparece como emprestado ou atrasado e pode ser devolvido; após a devolução, o equipamento fica oculto da lista e não volta a "disponível" enquanto estiver inativo.
- Pessoa desativada com empréstimos em aberto: os empréstimos continuam visíveis e podem ser devolvidos pelo administrador.
- Tentativa de excluir equipamento ou pessoa: não há exclusão; para retirá-los de uso, o administrador os desativa.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema MUST permitir cadastrar equipamentos com nome e identificação única.
- **FR-002**: O sistema MUST permitir que o administrador cadastre pessoas com nome, e-mail único (usado no login), contato, perfil (administrador ou usuário comum) e senha inicial. Somente o administrador cria contas de administrador.
- **FR-003**: O sistema MUST recusar equipamentos com identificação duplicada.
- **FR-004**: O sistema MUST permitir que o usuário autenticado registre o empréstimo de um equipamento disponível em seu próprio nome, com data prevista de devolução obrigatória; a data do empréstimo é a data atual.
- **FR-005**: O sistema MUST impedir o empréstimo de um equipamento que já possui empréstimo em aberto.
- **FR-006**: O sistema MUST recusar data prevista de devolução anterior à data do empréstimo; a mesma data do empréstimo é aceita.
- **FR-007**: O sistema MUST permitir registrar a devolução de um empréstimo em aberto, guardando a data real.
- **FR-008**: O sistema MUST classificar cada equipamento como "disponível", "emprestado" ou "atrasado" com base nos empréstimos em aberto e na data atual.
- **FR-009**: O sistema MUST permitir listar equipamentos filtrando por cada uma das três situações.
- **FR-010**: O sistema MUST exibir, para equipamentos emprestados ou atrasados, a pessoa, a data do empréstimo e a data prevista de devolução.
- **FR-011**: O sistema MUST manter o histórico de empréstimos já devolvidos, associados ao equipamento e à pessoa.
- **FR-012**: O sistema MUST exibir mensagens claras quando uma operação for recusada.
- **FR-013**: O sistema MUST exigir login para qualquer operação, exceto entrar no sistema e criar a própria conta (FR-018); usuários não autenticados não acessam nenhuma outra função.
- **FR-014**: O sistema MUST registrar, em cada empréstimo e devolução, o usuário autenticado que realizou a operação, para garantir responsabilidade sobre o item.
- **FR-015**: O sistema MUST oferecer dois perfis: administrador (cadastra equipamentos e pessoas, registra devoluções e consulta tudo) e usuário comum (registra empréstimos em seu próprio nome, consulta a situação dos equipamentos e consulta apenas o próprio histórico de empréstimos).
- **FR-016**: O sistema MUST impedir que um usuário desativado faça login ou pegue equipamentos emprestados, mesmo que ainda possua uma sessão ativa.
- **FR-017**: O sistema MUST permitir que o administrador edite os dados e desative equipamentos e pessoas. Não há exclusão: equipamento inativo não pode ser emprestado e fica oculto da lista de equipamentos (exceto enquanto tiver empréstimo em aberto, para que possa ser devolvido); pessoa inativa segue a regra do FR-016. O histórico de empréstimos é preservado.
- **FR-018**: O sistema MUST permitir que qualquer pessoa, sem login, crie a própria conta informando nome, e-mail único (usado no login), contato e senha, sem aprovação do administrador. A conta criada por autocadastro é sempre de usuário comum e nasce ativa.

### Key Entities

- **Equipamento**: Item emprestável. Atributos: nome, identificação única, ativo/inativo, situação (derivada: disponível, emprestado, atrasado).
- **Pessoa**: Quem recebe equipamentos. Atributos: nome, e-mail, contato, perfil, situação ativo/inativo. Possui acesso por login (senha definida pela própria pessoa no autocadastro, ou senha inicial definida pelo administrador) e responde pelos itens que pega ou cede emprestado.
- **Empréstimo**: Liga um equipamento a uma pessoa. Atributos: data do empréstimo, data prevista de devolução, data real de devolução (vazia enquanto em aberto).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Um usuário registra um empréstimo completo (equipamento e data prevista) em menos de 1 minuto.
- **SC-002**: O administrador registra uma devolução em menos de 30 segundos.
- **SC-003**: Em 100% das tentativas, um equipamento já emprestado não pode ser emprestado novamente.
- **SC-004**: A situação de qualquer equipamento (disponível, emprestado, atrasado) é identificada em até 10 segundos a partir da tela inicial.
- **SC-005**: Sem treinamento, 90% dos novos usuários comuns concluem um empréstimo na primeira tentativa, e 90% dos novos administradores concluem uma devolução na primeira tentativa.

## Out of Scope

Não fazem parte do MVP e não devem ser planejados nem implementados:

- Reserva antecipada de equipamentos (questão legal).
- Renovação ou prorrogação do prazo de devolução (questão legal).
- Multas ou penalidades por atraso (questão legal).
- Notificações automáticas de atraso por e-mail, SMS ou outros canais (questão legal).
- Controle de quantidade em estoque (itens fungíveis): cada equipamento é uma unidade.
- Empréstimo registrado em nome de outra pessoa.
- Recuperação de senha por e-mail.
- Login por conta institucional (SSO).
- Exclusão de equipamentos e pessoas (substituída pela desativação).
- Aplicativo móvel nativo.

## Assumptions

- O primeiro administrador é criado na instalação do sistema, já que o autocadastro só cria usuários comuns.
- Cada equipamento é uma unidade individual; uma pessoa pode ter vários equipamentos emprestados ao mesmo tempo.
- Atraso é calculado por dia: o equipamento está atrasado quando a data atual é posterior à data prevista.
- Se um usuário esquecer a senha, o administrador define uma nova senha ao editar a pessoa.
- O sistema é acessado por navegador em computador; uso em celular é desejável, mas não obrigatório.
