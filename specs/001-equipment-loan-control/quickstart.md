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

## Critérios
Os cenários 4 a 9 e 13 cobrem SC-001 a SC-003; o 8 cobre SC-004; 11, 12 e 14 cobrem os Princípios I e II da constituição (14 inclui eventos de segurança; o acesso à aplicação só funciona por HTTPS).
