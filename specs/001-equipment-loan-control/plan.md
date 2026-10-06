# Implementation Plan: Controle de Empréstimos de Equipamentos (Loanner)

**Branch**: `001-equipment-loan-control` | **Date**: 2026-10-05 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `/specs/001-equipment-loan-control/spec.md`, mais a diretriz: "a aplicação deve trabalhar com microsserviços, para facilitar a manutenção e desenvolvimento".

## Summary

O Loanner é entregue como quatro serviços independentes: **identity** (usuários, login e perfis), **catalog** (equipamentos), **loans** (empréstimos, devoluções e situação) e **web** (interface do navegador, que orquestra os demais). Cada serviço dono de dados tem seu próprio banco; a regra "um equipamento emprestado não pode ser emprestado de novo" fica no serviço **loans**, protegida por índice único parcial. A situação "atrasado" é calculada por dia, sem ser gravada. Os serviços se comunicam por contratos HTTP versionados, sempre sobre TLS (borda, serviços e bancos), e confiam em um token assinado emitido pelo **identity**. Decisões em [research.md](research.md).

## Technical Context

**Language/Version**: Python 3.12 em todos os serviços (uma só linguagem facilita a manutenção por um time pequeno)

**Primary Dependencies**: FastAPI e Pydantic (APIs dos serviços de dados), SQLAlchemy 2 e Alembic (acesso e migrações), httpx (chamadas entre serviços), PyJWT e Argon2 (token e hash de senha), Jinja2 (páginas do serviço web)

**Storage**: PostgreSQL 16, um **banco separado por serviço** (identity, catalog, loans); o serviço web não tem banco

**Testing**: pytest por serviço (unitário e integração com banco real em contêiner) e testes de contrato entre serviços; teste de concorrência para o índice único de empréstimos

**Target Platform**: Contêineres Docker orquestrados por Docker Compose (desenvolvimento e MVP); acesso por navegador desktop com layout responsivo

**Project Type**: microsserviços (3 serviços de dados + 1 serviço web)

**Performance Goals**: Páginas em menos de 1 segundo com até alguns milhares de equipamentos; atende SC-001 a SC-004

**Constraints**: Todo tráfego criptografado com TLS (proxy reverso na borda; CA interna entre serviços e bancos); eventos de segurança registrados (login, 401, 403); CI obrigatório para merge; sem acesso de um serviço ao banco de outro; toda chamada entre serviços é autenticada; configuração só por variáveis de ambiente; cada serviço expõe `/health` e logs estruturados

**Scale/Scope**: Dezenas de usuários, até alguns milhares de equipamentos e empréstimos; cerca de 8 telas

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Princípio | Avaliação (antes do design) | Avaliação (depois do design) |
|---|---|---|
| I. Microsserviços | Passa: serviços por capacidade, bancos separados | Passa: contratos em `contracts/`, sem acesso cruzado a banco |
| II. Segurança por padrão | Passa: login, perfis, hash, CSRF, registro de quem opera | Passa: JWT assinado verificado em cada serviço; TLS na borda, entre serviços e nos bancos (T013, T014); eventos de segurança registrados; usuário inativo recusado (FR-016); `loaned_by` e `returned_by` gravados |
| III. Stack web | Passa: interface web responsiva | Passa: serviço web em Python/Jinja, mesma linguagem dos demais |
| IV. Clean code | Passa (regra verificada na implementação) | Passa: regras em `domain/`, separadas de HTTP e banco |
| V. Código documentado e teste primeiro | Passa (verificado nas tarefas) | Passa: testes de contrato e concorrência previstos |
| VI. Documentação para onboarding | Passa (verificado nas tarefas) | Passa: README por serviço (inclusive o do web, T048), ADRs e glossário previstos na estrutura |
| Fluxo e gates | Passa | Passa: pipeline de CI previsto (T016) para linter, formatador e testes antes do merge |

Sem violações. Complexity Tracking vazio.

## Project Structure

### Documentation (this feature)

```text
specs/001-equipment-loan-control/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   ├── identity-api.md
│   ├── catalog-api.md
│   ├── loans-api.md
│   └── web-routes.md
├── checklists/
│   └── requirements.md
└── tasks.md             # Fase 2 (/speckit-tasks, não criado aqui)
```

### Source Code (repository root)

```text
services/
├── identity/
│   ├── README.md
│   ├── src/identity/{api,domain,infra}/
│   ├── migrations/
│   └── tests/{unit,integration,contract}/
├── catalog/
│   ├── README.md
│   ├── src/catalog/{api,domain,infra}/
│   ├── migrations/
│   └── tests/{unit,integration,contract}/
├── loans/
│   ├── README.md
│   ├── src/loans/{api,domain,infra}/
│   ├── migrations/
│   └── tests/{unit,integration,contract}/
└── web/
    ├── README.md
    ├── src/web/{routes,clients,templates,static}/
    └── tests/{unit,integration}/

docs/
├── architecture.md        # visão geral e diagrama dos serviços
├── adr/                   # um registro por decisão relevante
└── glossary.md            # termos do domínio

deploy/
├── proxy/Caddyfile        # proxy reverso com HTTPS
└── certs/                 # CA interna e certificados (não versionado)
scripts/
├── generate_certs.py
└── check_isolation.py
.github/workflows/ci.yml   # linter, formatador e testes
docker-compose.yml
README.md                  # propósito, arquitetura, como rodar e testar
```

**Structure Decision**: um diretório por serviço, cada um com `api` (HTTP), `domain` (regras de negócio puras) e `infra` (banco e clientes externos), conforme o princípio de clean code. O serviço `web` fala com os demais apenas pelos contratos.

## Complexity Tracking

Sem violações a justificar. Observação: microsserviços aumentam a complexidade operacional em relação a uma aplicação única; isso é uma exigência da constituição (Princípio I) e da diretriz do usuário, e está mitigado por usar uma só linguagem, Docker Compose e contratos simples.
