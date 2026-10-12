# Quickstart: validação ponta a ponta (Loanner)

Modelo em [data-model.md](data-model.md); contratos em [contracts/](contracts/).

## Pré-requisitos
- Docker e Docker Compose
- Python 3.12 (para gerar os certificados com `scripts/generate_certs.py` e rodar testes fora dos contêineres)

## Subir o sistema
0. Gerar a CA interna e os certificados com `scripts/generate_certs.py`.
1. Copiar o arquivo de exemplo de variáveis de ambiente para `.env` e preencher os valores (segredos nunca são versionados).
2. Subir os serviços com Docker Compose (identity, catalog, loans, web e os três bancos).
3. Criar o primeiro administrador pelo comando de linha do serviço identity.
4. Abrir a página de login do serviço web no navegador.

## Rodar os testes
Em cada serviço (`services/<nome>`), rodar `pytest`. Esperado: todos passam, incluindo os testes de contrato e o de concorrência do empréstimo duplicado no `loans`.

## Verificação de saúde
Cada serviço responde em `/health` com 200 (dentro da rede do Compose).

## Cenários de validação manual
1. **Login obrigatório**: abrir `/equipment` sem login. Esperado: redireciona para `/login`.
2. **Cadastro (admin)**: cadastrar "Notebook 01" (código `PAT-001`) e um usuário comum. Esperado: equipamento *disponível*.
3. **Código duplicado**: cadastrar outro `PAT-001`. Esperado: recusado com mensagem.
4. **Empréstimo (usuário comum)**: emprestar `PAT-001` com prazo futuro. Esperado: *emprestado*, com pessoa e prazo.
5. **Duplicidade**: com outro usuário, emprestar `PAT-001`. Esperado: recusado ("já está emprestado").
6. **Prazo inválido**: emprestar outro item com prazo no passado. Esperado: recusado.
7. **Permissão**: como usuário comum, tentar devolver ou cadastrar. Esperado: 403.
8. **Atraso**: usar o relógio simulado dos testes (ou ajustar o prazo para ontem em ambiente de teste). Esperado: filtro *atrasados* mostra o item com 1 dia de atraso.
9. **Devolução (admin)**: devolver `PAT-001`. Esperado: volta a *disponível*; histórico guarda a data e quem registrou; pode ser emprestado de novo.
10. **Devolução repetida**: devolver de novo. Esperado: recusado.
11. **Isolamento**: acessar `catalog` ou `loans` diretamente de fora da rede do Compose, ou sem token. Esperado: inacessível ou 401.
12. **Falha parcial**: parar o serviço `catalog` e abrir `/equipment`. Esperado: página de erro amigável (503), sem falha dos demais serviços.
13. **Usuário desativado**: como admin, desativar um usuário comum que já está logado; ele tenta emprestar um item. Esperado: recusado (403) e novo login negado.
14. **Eventos de segurança**: provocar um login falho, um 401 e um 403. Esperado: cada um aparece nos logs de segurança do serviço, sem senha nem token.
15. **Login inválido e saída (US5)**: entrar com senha errada. Esperado: "E-mail ou senha inválidos.", sem dizer qual campo errou. Entrar corretamente, sair e abrir `/equipment`. Esperado: redireciona para `/login`.
16. **E-mail duplicado**: cadastrar outra pessoa com o e-mail de uma já existente. Esperado: recusado com mensagem.
17. **Edição (admin)**: alterar o nome de um equipamento com histórico. Esperado: a lista mostra o nome novo e o histórico continua no mesmo registro.
18. **Desativar equipamento**: desativar um equipamento disponível. Esperado: some da lista de equipamentos (em qualquer filtro) e não pode ser emprestado. Desativar outro que está emprestado. Esperado: continua na lista como *emprestado* até a devolução; depois de devolvido, fica oculto.
19. **Prazo no mesmo dia**: emprestar com data prevista igual a hoje. Esperado: aceito; aparece como *emprestado*, não atrasado.
20. **Devolução de item atrasado**: devolver o item do cenário 8. Esperado: empréstimo encerrado; histórico mostra data prevista e data real.
21. **Autocadastro (FR-018)**: sem login, abrir `/signup` e criar uma conta. Esperado: o formulário não oferece perfil; a conta nasce ativa como usuário comum e consegue entrar e emprestar. Repetir com o mesmo e-mail. Esperado: "Já existe uma pessoa com este e-mail."

## Critérios
Os cenários 4 a 9 e 13 cobrem SC-001 a SC-003; o 8 cobre SC-004; 1, 15 e 21 cobrem a US5 e o FR-018; 2, 3, 16 a 18 cobrem a US1 e o FR-017; 19 e 20 cobrem casos de borda da US2 e da US3; 11, 12 e 14 cobrem os Princípios I e II da constituição (14 inclui eventos de segurança; o acesso à aplicação só funciona por HTTPS).
