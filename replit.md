# VemAprenderComigo — Plataforma Educacional Brasileira

## Overview

pnpm workspace monorepo usando TypeScript. Plataforma educacional com landing page pública, academia interna, tutor IA (Wilson), modelo freemium e materiais por aluna.

## Stack

- **Monorepo tool**: pnpm workspaces
- **Node.js version**: 24
- **Package manager**: pnpm
- **TypeScript version**: 5.9
- **API framework**: Express 5
- **Database**: PostgreSQL + Drizzle ORM
- **Validation**: Zod (`zod/v4`), `drizzle-zod`
- **API codegen**: Orval (from OpenAPI spec)
- **Build**: esbuild (CJS bundle)
- **Frontend**: React + Vite + TailwindCSS + shadcn/ui
- **AI**: OpenAI GPT via Replit integration (streaming SSE)

## Key Commands

- `pnpm run typecheck` — full typecheck
- `pnpm run build` — typecheck + build all
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks/Zod from openapi.yaml
- `pnpm --filter @workspace/db run push` — push DB schema changes
- `pnpm --filter @workspace/api-server run dev` — run API server locally

## User Accounts

| Usuário | E-mail / login | Senha | Role | Plan | Cursos |
|---------|---------------|-------|------|------|--------|
| Professora Sandra | `voceaprendecomjoaquina@gmail.com` | `JpSg2026` | admin | paid | — |
| Lorena | `lorenamariaalunaum@gmail.com` | `Aprendelorena01` | student | paid | Digitação Online |
| Bruna Isabela (id=3) | `isabelaalunadois@gmail.com` | `Aprendebella02` | student | paid | Inglês (Kultivi), Digitação Online, TalkPal |
| Miguel (id=4) | `lmiguelaluno03@gmail.com` | `Aprendemiguel03` | student | paid | Digitação Online, Bullying |
| Ana Maria (pai da Lorena) | `ana.maria.pai@gmail.com` | `Pais2026` | parent | paid | — |
| Carlos Souza (pai da Bruna Isabela) | `carlos.pai@gmail.com` | `Pais2026` | parent | paid | — |
| Maria Silva (pai do Miguel) | `maria.miguel.pai@gmail.com` | `Pais2026` | parent | paid | — |

## Freemium Model

- **Plano Gratuito (R$0)**: Acesso apenas à landing page com catálogo de 13 cursos
- **Plano Premium (R$75/mês)**: Acesso completo à academia (Mapa do Tesouro, Diário, Ranking, Wilson IA, Materiais, Dashboard)
- Novos usuários se cadastram em `/cadastro?plan=free` ou `/cadastro?plan=paid`
- Usuários free que tentam acessar `/academy` são redirecionados para `/assinar`
- Admin (role=admin) sempre tem acesso total, independente do plan

## Tutor IA Wilson

- Nome: **Wilson** (não Sofia)
- Sistema de "starts" personalizados por aluna:
  - **Lorena** (Matemática): Greeting com QAP, menção à Professora Sandra e Mapa do Tesouro; sugestões de frações, quiz de multiplicação, porcentagem
  - **Isabelle** (Inglês): Greeting com QAP, foco em Inglês; sugestões de present perfect, verbos irregulares, since vs for
  - **Admin**: Greeting focado em criação de exercícios e acompanhamento
- Trigger especial `<<GREET>>` dispara boas-vindas personalizadas ao criar nova conversa
- O trigger não aparece na UI (filtrado no frontend e não enviado ao histórico da API)
- Sistema prompt inclui: nome da aluna, matéria, menção à Professora Sandra
- Streaming SSE em tempo real
- Conversas persistidas no PostgreSQL

## Rotas Frontend

- `/` — Landing page pública (13 cursos)
- `/login` — Login (com abas "Acesso Estudante" e "Acesso Pais")
- `/assinar` — Página de planos freemium (R$0 vs R$75/mês)
- `/cadastro?plan=free|paid` — Cadastro novo usuário
- `/pais` — Portal dos Pais (role=parent)
- `/academy` — Dashboard academia (paidOnly)
- `/academy/tutor` — Wilson IA (paidOnly)
- `/academy/mapa/:studentId` — Mapa do Tesouro (paidOnly)
- `/academy/diario` — Diário de Bordo (paidOnly)
- `/academy/ranking` — Ranking (paidOnly)
- `/academy/dashboard/:studentId` — Evolução (paidOnly)

## DB Schema

Tabelas: `users` (campos: `plan` free|paid, `avatar` varchar(200), `role` admin|student|parent), `courses`, `subjects`, `diary_entries`, `conversations`, `messages`, `materials`, `user_courses`, `parent_student_links`, `parent_logbook`, `parent_questions`, `contact_submissions`, `postits`, `postit_replies`

### Portal dos Pais
- `parent_student_links` — vincula parentId ao studentId
- `parent_logbook` — entradas do diário da professora para os pais (com campo de resposta do pai)
- `parent_questions` — perguntas dos pais com campo de resposta da professora

## API Routes

- `POST /api/auth/login` — Login
- `POST /api/auth/logout` — Logout
- `GET /api/auth/me` — Usuário atual (retorna `plan`)
- `POST /api/auth/register` — Cadastro
- `PATCH /api/users/:id/plan` — Atualizar plano (admin only)
- `PATCH /api/users/:id/avatar` — Salvar avatar (DiceBear seed, próprio usuário)
- `POST /api/contact` — Submissão de formulário de contato (público, sem auth)
- `GET /api/contact` — Listar mensagens de contato (admin only)
- `PATCH /api/contact/:id/read` — Marcar mensagem como lida (admin only)
- `GET /api/postits` — Listar Post-its visíveis ao usuário (auth)
- `POST /api/postits` — Criar Post-it (admin only)
- `DELETE /api/postits/:id` — Excluir Post-it (admin only)
- `POST /api/postits/:id/replies` — Responder Post-it (auth)
- `GET/POST /api/openai/conversations` — Conversas com Wilson
- `POST /api/openai/conversations/:id/messages` — Mensagem para Wilson (streaming SSE)
- `/api/storage/*` — Object storage (materiais)
- `/api/materials` — CRUD materiais por aluna

### Portal dos Pais
- `GET /api/parent/dashboard` — Dashboard com dados do filho (role=parent)
- `GET /api/parent/logbook` — Entradas do diário da professora para o pai (role=parent)
- `POST /api/parent/logbook/:id/reply` — Responder uma entrada do diário (role=parent)
- `GET /api/parent/questions` — Listar perguntas do pai (role=parent)
- `POST /api/parent/questions` — Criar pergunta para a professora (role=parent)
- `GET /api/admin/parents` — Listar todos os pais com vínculo ao aluno (role=admin)
- `GET /api/admin/parent-logbook` — Listar todas as entradas do diário dos pais (role=admin)
- `POST /api/admin/parent-logbook` — Criar entrada no diário para um pai específico (role=admin)
- `GET /api/admin/parent-questions` — Listar perguntas dos pais (role=admin)
- `POST /api/admin/parent-questions/:id/reply` — Responder pergunta de um pai (role=admin)
