# 🧠 Reservatório de Dopamina

Aplicativo web (PWA) de autorregulação que ajuda o usuário a visualizar o equilíbrio entre atividades que **recarregam** sua energia e motivação (exercício, sono, leitura, tempo ao ar livre) e atividades que a **drenam** (rolagem infinita, jogos compulsivos, excesso de açúcar), usando a metáfora de um reservatório que enche e esvazia ao longo do dia.

> ⚠️ **Aviso:** o "nível do reservatório" é uma metáfora de hábitos e **não** uma medição médica ou clínica de dopamina. O app não substitui acompanhamento profissional.

![status](https://img.shields.io/badge/status-em%20desenvolvimento-yellow)
![license](https://img.shields.io/badge/license-MIT-blue)
![node](https://img.shields.io/badge/node-%3E%3D20-green)

---

## 📑 Sumário

- [Visão geral](#-visão-geral)
- [Funcionalidades](#-funcionalidades)
- [Tecnologias](#-tecnologias)
- [Arquitetura](#-arquitetura)
- [Estrutura do repositório](#-estrutura-do-repositório)
- [Pré-requisitos](#-pré-requisitos)
- [Instalação](#-instalação)
- [Configuração](#-configuração-variáveis-de-ambiente)
- [Execução](#-execução)
- [Banco de dados](#-banco-de-dados)
- [API](#-api)
- [Testes](#-testes)
- [Deploy](#-deploy)
- [Contribuindo](#-contribuindo)
- [Licença](#-licença)

Documentação completa (objetivo, escopo, requisitos): [`docs/DOCUMENTACAO.md`](docs/DOCUMENTACAO.md)

---

## 🎯 Visão geral

| Item | Descrição |
|------|-----------|
| **Objetivo** | Dar ao usuário consciência visual e imediata do efeito dos seus hábitos diários sobre sua energia e foco. |
| **Problema** | Muitas pessoas perdem horas em atividades de recompensa rápida sem perceber o impacto acumulado, e não têm uma forma simples e não punitiva de acompanhar isso. |
| **Solução** | Registro rápido de atividades, pontuação por tipo, nível do reservatório diário, metas e resumos semanais. |

---

## ✨ Funcionalidades

- Cadastro e login de usuários
- Registro rápido de atividades (1 toque)
- Catálogo de atividades padrão + atividades personalizadas
- Nível do reservatório (0–100) atualizado em tempo real
- Histórico diário e semanal com gráficos
- Metas diárias (ex.: "3 recargas por dia", "máx. 60 min de dreno")
- Sequência (streak) de dias equilibrados
- Funcionamento offline básico (PWA)

---

## 🛠 Tecnologias

| Camada | Tecnologia |
|--------|-----------|
| Frontend | React 18, TypeScript, Vite, React Router, TanStack Query, Recharts |
| Backend | Node.js 20, Express, TypeScript, Zod (validação) |
| Banco de dados | PostgreSQL 16 + Prisma ORM |
| Autenticação | JWT (access + refresh token), bcrypt |
| Testes | Vitest, Supertest, React Testing Library, Playwright (E2E) |
| DevOps | Docker, Docker Compose, GitHub Actions |
| Deploy sugerido | Render (API + banco) e Vercel (frontend) |

---

## 🏗 Arquitetura

Arquitetura cliente-servidor em camadas, com API REST.

```mermaid
flowchart LR
    U[Usuário] --> W[Frontend React PWA]
    W -->|HTTPS / JSON| A[API Express]
    A --> R[Routes]
    R --> C[Controllers]
    C --> S[Services - regras de negócio]
    S --> P[Prisma ORM]
    P --> D[(PostgreSQL)]
```

**Cálculo do nível do reservatório:** o nível parte de 50 no início do dia e soma os pontos das atividades registradas (recarga = pontos positivos, dreno = negativos), limitado ao intervalo 0–100.

---

## 📁 Estrutura do repositório

```
reservatorio-de-dopamina/
├── apps/
│   ├── api/
│   │   ├── prisma/            # schema.prisma e migrations
│   │   ├── src/
│   │   │   ├── routes/
│   │   │   ├── controllers/
│   │   │   ├── services/      # regras de negócio (ex.: reservoir.service.ts)
│   │   │   ├── middlewares/   # auth, validação, erros
│   │   │   └── server.ts
│   │   └── tests/
│   └── web/
│       ├── src/
│       │   ├── components/    # ReservoirGauge, ActivityButton, Charts...
│       │   ├── pages/         # Dashboard, History, Goals, Settings
│       │   ├── hooks/
│       │   ├── services/      # cliente da API
│       │   └── main.tsx
│       └── tests/
├── docs/
│   └── DOCUMENTACAO.md
├── docker-compose.yml
├── .github/workflows/ci.yml
├── .env.example
└── README.md
```

---

## ✅ Pré-requisitos

- [Node.js](https://nodejs.org/) **20 ou superior**
- [npm](https://www.npmjs.com/) 10+ (ou pnpm/yarn)
- [PostgreSQL](https://www.postgresql.org/) 16+ **ou** [Docker](https://www.docker.com/) + Docker Compose
- [Git](https://git-scm.com/)

---

## 📦 Instalação

```bash
# 1. Clone o repositório
git clone https://github.com/SEU-USUARIO/reservatorio-de-dopamina.git
cd reservatorio-de-dopamina

# 2. Instale as dependências (API e Web)
npm install --prefix apps/api
npm install --prefix apps/web
```

---

## ⚙️ Configuração (variáveis de ambiente)

Copie o arquivo de exemplo e ajuste os valores:

```bash
cp .env.example apps/api/.env
cp apps/web/.env.example apps/web/.env
```

### API (`apps/api/.env`)

| Variável | Obrigatória | Exemplo | Descrição |
|----------|:-----------:|---------|-----------|
| `NODE_ENV` | não | `development` | Ambiente de execução |
| `PORT` | não | `3333` | Porta da API |
| `DATABASE_URL` | sim | `postgresql://postgres:postgres@localhost:5432/reservatorio` | String de conexão do PostgreSQL |
| `JWT_SECRET` | sim | `troque-por-um-segredo-forte` | Segredo do access token |
| `JWT_EXPIRES_IN` | não | `15m` | Validade do access token |
| `JWT_REFRESH_SECRET` | sim | `outro-segredo-forte` | Segredo do refresh token |
| `JWT_REFRESH_EXPIRES_IN` | não | `7d` | Validade do refresh token |
| `CORS_ORIGIN` | não | `http://localhost:5173` | Origem permitida no CORS |

### Web (`apps/web/.env`)

| Variável | Obrigatória | Exemplo | Descrição |
|----------|:-----------:|---------|-----------|
| `VITE_API_URL` | sim | `http://localhost:3333/api` | URL base da API |

> 🔒 Nunca faça commit de arquivos `.env`. Eles já estão no `.gitignore`.

---

## ▶️ Execução

### Opção A: com Docker (recomendado)

```bash
docker compose up --build
```

- Frontend: http://localhost:5173
- API: http://localhost:3333/api
- PostgreSQL: `localhost:5432`

### Opção B: manualmente

```bash
# Terminal 1: banco de dados (se não usar Docker para o Postgres)
# garanta que o PostgreSQL está rodando e o DATABASE_URL está correto

# Terminal 2: API
cd apps/api
npx prisma migrate dev     # cria as tabelas
npx prisma db seed         # carrega atividades padrão
npm run dev

# Terminal 3: Web
cd apps/web
npm run dev
```

### Build de produção

```bash
npm run build --prefix apps/api
npm run build --prefix apps/web
npm run start --prefix apps/api
```

---

## 🗄 Banco de dados

PostgreSQL gerenciado via **Prisma**. Principais entidades:

```mermaid
erDiagram
    USER ||--o{ ACTIVITY_TYPE : cria
    USER ||--o{ ACTIVITY_LOG : registra
    USER ||--o{ GOAL : define
    ACTIVITY_TYPE ||--o{ ACTIVITY_LOG : classifica

    USER {
        uuid id PK
        string name
        string email UK
        string passwordHash
        datetime createdAt
    }
    ACTIVITY_TYPE {
        uuid id PK
        uuid userId FK "null = padrão do sistema"
        string name
        enum kind "RECHARGE | DRAIN"
        int points
        string icon
    }
    ACTIVITY_LOG {
        uuid id PK
        uuid userId FK
        uuid activityTypeId FK
        int durationMin
        string note
        datetime occurredAt
    }
    GOAL {
        uuid id PK
        uuid userId FK
        enum metric "RECHARGE_COUNT | DRAIN_MINUTES | MIN_LEVEL"
        int target
        boolean active
    }
```

Comandos úteis:

```bash
npx prisma migrate dev --name nome_da_migracao   # nova migration
npx prisma migrate deploy                        # aplicar em produção
npx prisma studio                                # interface visual
npx prisma db seed                               # dados iniciais
```

---

## 🔌 API

Base URL: `http://localhost:3333/api` · Formato: JSON · Autenticação: `Authorization: Bearer <accessToken>` (exceto rotas de auth).

| Método | Rota | Descrição | Auth |
|--------|------|-----------|:----:|
| POST | `/auth/register` | Cria conta | não |
| POST | `/auth/login` | Retorna access e refresh token | não |
| POST | `/auth/refresh` | Renova o access token | não |
| GET | `/me` | Dados do usuário logado | sim |
| GET | `/activity-types` | Lista atividades (padrão + personalizadas) | sim |
| POST | `/activity-types` | Cria atividade personalizada | sim |
| PATCH | `/activity-types/:id` | Edita atividade personalizada | sim |
| DELETE | `/activity-types/:id` | Remove atividade personalizada | sim |
| POST | `/logs` | Registra uma atividade | sim |
| GET | `/logs?from=&to=` | Lista registros por período | sim |
| DELETE | `/logs/:id` | Remove um registro | sim |
| GET | `/reservoir/today` | Nível atual do reservatório | sim |
| GET | `/reservoir/history?days=30` | Histórico de níveis | sim |
| GET | `/goals` | Lista metas | sim |
| POST | `/goals` | Cria meta | sim |
| GET | `/insights/weekly` | Resumo semanal | sim |
| GET | `/health` | Health check | não |

### Exemplos

**Login**

```http
POST /api/auth/login
Content-Type: application/json

{ "email": "ana@email.com", "password": "senhaSegura123" }
```

```json
{
  "accessToken": "eyJhbGciOi...",
  "refreshToken": "eyJhbGciOi...",
  "user": { "id": "a1b2c3", "name": "Ana", "email": "ana@email.com" }
}
```

**Registrar atividade**

```http
POST /api/logs
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "activityTypeId": "7f3e...",
  "durationMin": 30,
  "note": "Caminhada no parque",
  "occurredAt": "2026-11-16T08:30:00Z"
}
```

**Nível do reservatório**

```json
{
  "date": "2026-11-16",
  "level": 72,
  "recharges": 4,
  "drains": 1,
  "status": "HEALTHY"
}
```

**Erros** seguem o padrão:

```json
{ "error": { "code": "VALIDATION_ERROR", "message": "Campo 'email' inválido", "details": [] } }
```

Códigos usados: `200`, `201`, `204`, `400`, `401`, `403`, `404`, `409`, `422`, `500`.

---

## 🧪 Testes

```bash
# Testes unitários e de integração da API
npm run test --prefix apps/api

# Testes de componentes do frontend
npm run test --prefix apps/web

# Testes E2E (requer a aplicação rodando)
npm run e2e --prefix apps/web

# Cobertura
npm run test:coverage --prefix apps/api
```

- **Unitários:** regras de cálculo do reservatório, validações.
- **Integração:** rotas da API com banco de testes (`DATABASE_URL` apontando para um banco separado).
- **E2E:** fluxo cadastro → login → registrar atividade → ver reservatório.

Meta de cobertura: **≥ 80%** nos serviços de regra de negócio. O pipeline do GitHub Actions executa lint, testes e build a cada push/PR.

---

## 🚀 Deploy

### Pipeline (CI/CD)

O arquivo `.github/workflows/ci.yml` executa: instalação → lint → testes → build. Merges na branch `main` disparam o deploy automático.

### API + banco (Render)

1. Crie um **PostgreSQL** no Render e copie a `Internal Database URL`.
2. Crie um **Web Service** apontando para `apps/api`.
3. Build command: `npm install && npx prisma generate && npm run build`
4. Start command: `npx prisma migrate deploy && npm run start`
5. Defina as variáveis de ambiente da [tabela acima](#api-appsapienv).

### Frontend (Vercel)

1. Importe o repositório e defina o **Root Directory** como `apps/web`.
2. Defina `VITE_API_URL` com a URL pública da API.
3. Build command: `npm run build` · Output: `dist`.

### Checklist de produção

- [ ] `JWT_SECRET` e `JWT_REFRESH_SECRET` fortes e diferentes dos de desenvolvimento
- [ ] `CORS_ORIGIN` restrito ao domínio do frontend
- [ ] HTTPS ativo
- [ ] Backups automáticos do banco
- [ ] Migrations aplicadas (`prisma migrate deploy`)

---

## 🤝 Contribuindo

1. Faça um fork do projeto
2. Crie uma branch: `git checkout -b feat/minha-feature`
3. Commit seguindo [Conventional Commits](https://www.conventionalcommits.org/): `git commit -m "feat: adiciona meta semanal"`
4. Push: `git push origin feat/minha-feature`
5. Abra um Pull Request

---

## 📄 Licença

Distribuído sob a licença **MIT**. Veja o arquivo `LICENSE` para mais informações.

## 👤 Autor

**Seu Nome** · [GitHub](https://github.com/SEU-USUARIO) · [LinkedIn](https://linkedin.com/in/SEU-PERFIL)
