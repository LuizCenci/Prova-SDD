# Relatório: Loanner (controle de empréstimos de equipamentos)

## 1. Produto escolhido e escopo do MVP

O produto é o **Loanner**, um sistema web simples para controlar empréstimos de equipamentos em uma pequena empresa, laboratório ou universidade, com o objetivo de digitalizar esse tipo de controle.

**Entrou no MVP** (especificação em `specs/001-equipment-loan-control/spec.md`):
- Cadastro de equipamentos (nome e código único) e de pessoas (nome, e-mail, contato, perfil e senha inicial).
- Login obrigatório, com dois perfis: o administrador cadastra e registra devoluções; o usuário comum empresta em seu próprio nome e consulta.
- Registro de empréstimo com data prevista de devolução, sem permitir emprestar de novo um equipamento já emprestado.
- Registro de devolução e histórico de empréstimos.
- Consulta dos equipamentos por situação: disponível, emprestado ou atrasado (atraso contado por dia).

**Ficou fora do MVP:** reserva antecipada, renovação de prazo, multas e notificações automáticas de atraso (questões legais), controle de quantidade em estoque e login por conta institucional (SSO).

Além da especificação, o trabalho gerou a constituição do projeto (microsserviços, segurança, clean code e documentação completa), o plano com quatro serviços (identity, catalog, loans e web), os contratos entre eles e uma lista de 83 tarefas.

## 2. Agente de IA e versão do Spec Kit

- **Agente:** Claude Code (Anthropic), com as skills do Spec Kit. O trabalho usou Sonnet 5.5.
- **Spec Kit:** versão **1.0.13** (registrada em `.specify/init-options.json`), com numeração sequencial de features.
- **Comandos usados:** `/speckit-constitution`, `/speckit-specify`, `/speckit-clarify`, `/speckit-plan`, `/speckit-tasks` e seguido de ajustes manuais pedidos a partir da análise.

## 3. O que o `/speckit.clarify` perguntou e como mudou a especificação

A especificação inicial tinha uma seção de suposições. A clarificação partiu de quatro delas, e a ferramenta fez uma pergunta formal adicional.

| Ponto | Resposta | Efeito na especificação |
|---|---|---|
| Autenticação (a suposição era "perfil único, sem login") | O usuário deve fazer login para fornecer suas informações e assumir responsabilidade pelo item que pegar ou ceder | Entraram os requisitos FR-013 (login obrigatório) e FR-014 (registrar quem emprestou e devolveu); a suposição de perfil único foi removida |
| Equipamentos individuais | A maioria dos itens é única, mas uma pessoa pode ter mais de um item ao mesmo tempo | Suposição mantida e reforçada; virou caso de borda |
| Itens fora do escopo | Sim, por serem questões legais | Reserva, renovação, multas e notificações registradas como fora do MVP, com o motivo |
| Contagem do atraso | Por dia | Confirmada a regra de atraso por dia |
| **Pergunta da ferramenta:** todos os usuários logados têm as mesmas permissões, ou existe um perfil administrador? | Opção B: administrador cadastra e registra devoluções; usuário comum registra o próprio empréstimo e consulta | Novo requisito FR-015 (dois perfis); as quatro histórias de usuário passaram a dizer quem faz cada ação |

Em resumo, a clarificação transformou uma especificação de usuário único em uma com autenticação, responsabilidade individual e permissões por perfil. Essa mudança alterou o plano, o modelo de dados e as rotas.

## 4. O que eu mudaria se refizesse o trabalho

Definiria os requisitos de identidade e autorização desde o início. Como o sistema envolve empréstimos atribuídos a pessoas, autenticação, perfis de acesso, recuperação de senha, desativação de usuários e responsabilidades sobre cada operação deveriam fazer parte da definição inicial do produto. Em um ambiente real, essas características não poderiam surgir apenas durante a clarificação.

Validaria a arquitetura antes de adotar microsserviços. Para um MVP desse tamanho, microsserviços podem aumentar significativamente a complexidade operacional, principalmente em comunicação entre serviços, autenticação, observabilidade, testes e deploy. Em um projeto real, eu começaria avaliando uma arquitetura monolítica modular e só adotaria microsserviços caso existissem requisitos concretos que justificassem essa decisão.

Estabeleceria requisitos não funcionais como parte da especificação. Segurança, disponibilidade, desempenho, auditoria, tratamento de erros, backup e proteção dos dados deveriam ser definidos antes da implementação. Como o sistema registra quem emprestou e devolveu equipamentos, seria importante, por exemplo, determinar quais ações precisam ser auditadas e por quanto tempo esses registros devem ser mantidos.

Tornaria os critérios de aceitação mensuráveis e úteis para validação. Critérios como "identificar a situação em 10 segundos" ou "90% de acerto na primeira tentativa" são difíceis de validar sem definir uma metodologia de medição. Em um projeto real, eu priorizaria critérios objetivos, que pudessem ser diretamente convertidos em testes automatizados ou procedimentos de validação com usuários.

Validaria as decisões tecnológicas com base no contexto do projeto. A escolha de Python, FastAPI, PostgreSQL e demais tecnologias não deveria ser tratada como consequência automática da especificação. Em um cenário real, essas decisões seriam comparadas considerando experiência da equipe, infraestrutura disponível, custos, requisitos de segurança, suporte e facilidade de manutenção.

Daria maior atenção aos casos de borda e às regras de negócio antes de gerar as tarefas. Situações como uma pessoa possuir vários equipamentos, tentativa de empréstimo simultâneo, exclusão de equipamentos com histórico, devoluções inválidas e usuários desativados deveriam estar explicitamente descritas e vinculadas a critérios de aceitação e testes. Isso reduziria ambiguidades durante a implementação.

Separaria o que é essencial para o MVP do que é apenas uma possibilidade futura. Em um contexto real, funcionalidades como notificações, reservas e renovação poderiam continuar fora do primeiro lançamento, mas deveriam ser registradas como possíveis extensões do produto, permitindo que a arquitetura não bloqueasse sua implementação posteriormente.
