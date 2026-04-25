# Portal dos Pais com Diário de Bordo

## What & Why
Criar um portal completo para os pais dos alunos, com acesso separado do acesso dos estudantes. Na tela de login, adicionar abas "Acesso Estudante" e "Acesso Pais". Quando os pais acessam, veem um dashboard com a evolução do filho, o Diário de Bordo (feedbacks da professora para os pais), podendo responder aos feedbacks e criar perguntas para a professora.

## Done looks like
- A tela de login tem duas abas visuais: **Acesso Estudante** e **Acesso Pais**, com o mesmo formulário de login mas contexto visual diferente
- Usuários com role `parent` são redirecionados para `/pais` ao fazer login
- O dashboard dos pais mostra:
  - Resumo da evolução do filho: pontos, streak, % de progresso no mapa, assuntos concluídos
  - Seção **Diário de Bordo**: entradas criadas pela professora para os pais (não as entradas do aluno), com opção de resposta do pai/mãe
  - Seção **Minhas Perguntas**: pais podem criar perguntas para a professora e ver as respostas
- A professora (admin) vê no seu painel:
  - Seção "Diário dos Pais" para escrever novas entradas do diário destinadas a um pai específico
  - Seção "Perguntas dos Pais" com as perguntas recebidas e campo para responder
- Banco de dados atualizado com as novas tabelas necessárias
- Dados de exemplo (seed) com usuários pais vinculados aos alunos existentes

## Out of scope
- Portal dos pais no app mobile (apenas web por agora)
- Notificações por email ou push
- Cadastro de pais pelo próprio usuário (criado pela professora/admin)

## Tasks

1. **Schema e migrations do banco** — Adicionar a tabela `parent_student_links` (vincula parentId ao studentId), a tabela `parent_logbook` (entradas do diário da professora para os pais, com campo de resposta do pai e timestamp) e a tabela `parent_questions` (perguntas dos pais com campo de resposta da professora). Atualizar o seed com 2-3 usuários pai de exemplo (role=`parent`, plan=`paid`) vinculados aos alunos existentes.

2. **Rotas de API para o portal dos pais** — Criar endpoints em `/api/parent/*`: buscar o dashboard do filho vinculado (progresso, streak, pontos), listar/criar entradas do diário dos pais (`/parent/logbook`), responder a uma entrada do diário (`/parent/logbook/:id/reply`), listar e criar perguntas para a professora (`/parent/questions`). Criar endpoints em `/api/admin/*` para a professora: criar entradas do diário destinadas a um pai (`/admin/parent-logbook`), listar perguntas dos pais e responder a uma pergunta (`/admin/parent-questions/:id/reply`).

3. **Tela de login com abas** — Modificar o `LoginPage.tsx` para exibir duas abas no topo do formulário: "Acesso Estudante" (com ícone de estudante) e "Acesso Pais" (com ícone de família/coração). A aba selecionada muda apenas o visual e o texto de apoio — o formulário de login é o mesmo. Após login bem-sucedido, usuários com role `parent` são redirecionados para `/pais` em vez de `/academy`.

4. **Portal dos pais — dashboard e diário** — Criar a página `PaisPage.tsx` com layout próprio (sidebar/nav adaptado para pais). O dashboard exibe: cards com pontos, streak e progresso do filho; seção Diário de Bordo com as entradas da professora em ordem cronológica, cada uma com botão "Responder" que abre um campo de texto inline; seção "Minhas Perguntas" com lista de perguntas enviadas (com resposta da professora quando disponível) e botão "Nova Pergunta" que abre um formulário. Registrar a rota `/pais` no `App.tsx` com proteção para role `parent`.

5. **Painel da professora — gestão dos pais** — Atualizar o painel admin para incluir: botão/seção "Diário dos Pais" na navegação, onde a professora escolhe o pai (select com lista de pais cadastrados) e escreve uma nova entrada; e seção "Perguntas dos Pais" que lista as perguntas recebidas com nome do pai, data e campo de resposta em linha.

## Relevant files
- `artifacts/ensina/src/pages/LoginPage.tsx`
- `artifacts/ensina/src/App.tsx`
- `artifacts/ensina/src/components/AcademyLayout.tsx`
- `artifacts/ensina/src/pages/AcademyPage.tsx`
- `artifacts/ensina/src/contexts/AuthContext.tsx`
- `artifacts/api-server/src/routes/auth.ts`
- `artifacts/api-server/src/routes/index.ts`
- `artifacts/api-server/src/routes/dashboard.ts`
- `lib/db/src/schema/index.ts`
- `artifacts/api-server/src/seed.ts`
