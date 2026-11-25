# Sistema Alra

Sistema interno da Alra Marketing para a gestão da operação da agência: equipe, clientes, conteúdo, captações, comercial e financeiro de mídia.

- **Endereço:** https://sistema-alra.vercel.app
- **Código:** `alra-marketing/sistema-alra`, branch `main`
- **Hospedagem:** Vercel (time `alramarketing1`, projeto `sistema-alra`), com deploy automático a cada atualização do `main`
- **Banco, login e arquivos:** Supabase (organização *Alra Marketing*, projeto `sistema-alra`, região São Paulo, plano Free)
- **Quem usa:** a equipe da agência, com login. Os clientes não têm conta: usam links próprios de aprovação e de onboarding.

---

## Sumário

1. [História](#história)
2. [O que o sistema faz](#o-que-o-sistema-faz)
3. [Sistema Alra e Pode Postar](#sistema-alra-e-pode-postar)
4. [Estrutura do repositório](#estrutura-do-repositório)
5. [Infraestrutura](#infraestrutura)
6. [Automações](#automações)
7. [Acesso e permissões](#acesso-e-permissões)
8. [Backup e restauração](#backup-e-restauração)
9. [Como colocar no ar do zero](#como-colocar-no-ar-do-zero)
10. [Pendências](#pendências)
11. [Glossário](#glossário)

---

## História

| Quando | O que aconteceu |
|---|---|
| Antes de 2026 | A operação era controlada no **Notion**. |
| 2026 | Primeira versão própria, o *Gestão Alra*, em Cloudflare Workers + banco D1 (`gestao.alramarketing.workers.dev`, repositório `alra-gest-o`). Os dados do Notion foram migrados e a conexão com o Notion foi desligada. |
| 2026 | O sistema foi refeito como **Sistema Alra**, em Vercel + Supabase, nas contas do Pablo. |
| Out/2026 | Reorganização: times A/B/C, menus por time, novos campos e modo demonstração. A versão anterior do código ficou no branch `v1-estavel`, e uma cópia do banco foi guardada na tabela `docs_backup_20261023`. |
| 25/11/2026 | **Migração para as contas da Alra:** o código passou para o GitHub `alra-marketing`, o deploy para o time `alramarketing1` da Vercel e o banco para o projeto Supabase da organização *Alra Marketing*. Um backup privado foi guardado em `alra-marketing/sistema-alra-backup`, com 2.143 registros e 11 logins. Os detalhes estão em [MIGRACAO.md](MIGRACAO.md). |
| 25/11/2026 | Decisão: o conteúdo (planejamento, criação, aprovação e publicação) vai para o **Pode Postar**. Por enquanto, o Sistema Alra continua funcionando completo, sem mudanças na estrutura. |

---

## O que o sistema faz

O menu varia conforme o time da pessoa (veja [Acesso e permissões](#acesso-e-permissões)).

**Trabalho**
- **Início:** visão geral do dia, com a fila da pessoa, os atrasos e o que está com o cliente.
- **Pauta da equipe:** grade semana × pessoa × dia, com o que cada um tem para fazer.
- **Kanban:** pautas e tarefas internas por etapa, com cartões arrastáveis.
- **Calendário:** planejamento de conteúdo em visão mensal ou semanal.
- **Aprovações:** pacotes por cliente e período, com o envio em lote para o cliente.
- **Publicações:** o que já foi aprovado e precisa ser publicado (times A e B).
- **Captações:** gravações e sessões de fotos/vídeos com os clientes.

**Gestão**
- **Clientes:** visão geral, cadastro, orientações (onboarding), materiais, pautas, captações, alinhamentos, acessos e feedbacks.
- **Mapa de processos:** fluxos e matriz de responsabilidades da agência.
- **Comercial:** pipeline de vendas (time A).
- **Financeiro de mídia:** contas de anúncio (Meta, Google, LinkedIn) e cobranças.
- **Dashboards:** indicadores, desempenho individual e relatórios (times A e B).
- **Configurações:** usuários, unidades, integrações, prazos, segurança e backup (time A).

**Links públicos, sem login**
- `/c/<token>`: portal de aprovação do cliente. Mostra só as peças daquele cliente e registra a aprovação ou o pedido de ajuste.
- `/o/<token>`: onboarding do cliente. O cliente responde o checklist e as respostas vão para *Orientações*.

**Modo demonstração:** `/?demo=1` abre o sistema com dados de exemplo, sem usar o banco.

---

## Sistema Alra e Pode Postar

A agência passa a usar o **Pode Postar** para planejamento, criação, aprovação e publicação de conteúdo.

| Fica no Sistema Alra | Vai para o Pode Postar (quando a migração for feita) |
|---|---|
| Início, Pauta da equipe, Captações | Calendário |
| Clientes (ficha, squad, onboarding, alinhamentos) | Kanban de planejamento/criação |
| Mapa de processos, Comercial, Financeiro de mídia | Aprovações (incluindo o portal do cliente) |
| Dashboards da operação, Configurações e backup | Publicações |

**Situação atual:** nada foi removido. O sistema segue funcionando como sempre. Quando a mudança for feita, os dados antigos ficam guardados para consulta e não são apagados.

---

## Estrutura do repositório

```
index.html                  Todo o app (telas, regras e estilos) em uma página única
favicon.png
vercel.json                 Rotas dos links públicos, cabeçalhos de segurança e agendamentos (cron)
api/                        Funções do servidor (Vercel Functions)
  _supa.js                  Acesso ao Supabase pelo servidor, com a chave de serviço (nunca vai ao navegador)
  config.js                 Entrega ao navegador o endereço do Supabase e a chave pública
  portal.js                 Portal de aprovação do cliente (/c/<token>)
  briefing.js               Onboarding do cliente (/o/<token>)
  usuarios.js               Logins da equipe: liberar, redefinir senha, trocar e-mail, excluir
  cron-backup.js            Backup do banco para o armazenamento privado
  cron-limpeza.js           Limpeza diária das imagens antigas das pautas
supabase/                   Scripts SQL do banco (estrutura e regras de acesso)
  schema.sql                Estrutura inicial: tabelas, regras (RLS), tempo real e armazenamento
  schema-um-comando.sql     O mesmo, em um comando só
  acesso-diretoria.sql      Só a Diretoria libera acessos e o papel Diretoria
  equipe-diretoria.sql      Equipe gerida só pela Diretoria
  senha-padrao.sql          Troca obrigatória da senha no primeiro acesso
MIGRACAO.md                 Anotações da migração de 25/11/2026
```

### Como os dados são guardados

- Quase tudo fica em uma tabela só, `docs`, com as colunas `col` (coleção), `id` e `data` (JSON).
- **Coleções:** `unidades`, `pessoas`, `clientes`, `pecas` (pautas e tarefas), `arquivo` (pautas arquivadas), `captacoes`, `materiais`, `acessos`, `feedbacks`, `pacotes`, `notifs`, `relatorios`, `comercial`, `midiaContas`, `midiaCobs`, `alinhamentos`, `mapa` e `cfg` (configurações).
- `admins`: administradores. Não pode ser lida pelo navegador.
- `docs_backup_AAAAMMDD`: cópias da tabela `docs` feitas antes de mudanças grandes.
- **Armazenamento:** o bucket `arquivos` guarda artes, fotos e logos (leitura por link). O bucket `backups` é privado.
- **Tempo real:** a tabela `docs` está no *Realtime* do Supabase, então quem está com o sistema aberto vê as mudanças na hora.

---

## Infraestrutura

| Peça | Onde | Observações |
|---|---|---|
| Site e funções | Vercel, time `alramarketing1`, projeto `sistema-alra` | Deploy automático a cada push no `main`. Framework: *Other*. |
| Banco, login e arquivos | Supabase, org *Alra Marketing*, projeto `sistema-alra` (São Paulo, Free) | O plano Free tem 500 MB de banco, 1 GB de arquivos e 5 GB de tráfego. |
| Código | GitHub `alra-marketing/sistema-alra` | Branches: `main` (no ar), `v1-estavel`, `reorganizacao`, `visual-dark-halloween`. |
| Backup externo | GitHub `alra-marketing/sistema-alra-backup` (privado) | Não restaurar sem autorização da direção. |

### Variáveis de ambiente na Vercel

| Variável | Para quê |
|---|---|
| `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY` | Criadas pela integração oficial Supabase ↔ Vercel. **Não edite à mão.** |
| `CRON_SECRET` | Protege `/api/cron-backup` e `/api/cron-limpeza`. Cadastrada em 25/11/2026. |
| `SENHA_PADRAO` | (opcional) Senha inicial dos novos logins. |

Depois de criar ou mudar uma variável, é preciso fazer um novo deploy (*Deployments → Redeploy*) para ela valer.

---

## Automações

| Automação | Quando (Brasília) | O que faz |
|---|---|---|
| Backup semanal | Domingo, 21h47 | Salva o banco inteiro em JSON no bucket privado `backups` e registra um relatório no sistema. |
| Limpeza de imagens | Todo dia, 3h17 | Apaga do armazenamento as imagens de pautas com mais de 7 dias, para caber no plano Free. Não toca em fotos de perfil, logos, materiais nem comprovantes. Os links do Drive e os dados das pautas ficam. |
| Arquivamento | Sob demanda | A função `arquivar_publicadas` do banco move para a coleção `arquivo` as pautas publicadas há 24h ou mais. Hoje nenhum agendamento a chama. |
| Backup manual | Quando a Diretoria quiser | *Configurações → Segurança e backup → Fazer backup agora*. |

Os horários estão em UTC no `vercel.json`: `47 0 * * 1` e `17 6 * * *`.

---

## Acesso e permissões

- **Login:** e-mail e senha pelo Supabase Auth. O login nasce com uma senha provisória, e o sistema obriga a troca no primeiro acesso.
- **Quem libera:** só a Diretoria cria logins, redefine senhas e define cargos e papéis (*Configurações → Usuários*). As regras ficam no banco (RLS e gatilho `guard_pessoas`) e no servidor (`api/usuarios.js`).
- **E-mails:** a equipe entra com e-mail `@alramarketing.com.br`. Quem está na tabela `admins` entra mesmo sem cadastro em Equipe.
- **Times:**

| Time | Quem | Vê |
|---|---|---|
| A | Diretoria | Tudo do B, mais Comercial e Configurações |
| B | Head | Tudo do C, mais Publicações e Dashboards |
| C | Demais | Início, Pauta da equipe, Kanban, Calendário, Aprovações, Captações, Clientes, Mapa de processos |
| D | Clientes | Só os links públicos de aprovação e onboarding |

O time vem do papel da pessoa e pode ser fixado em *Configurações → Usuários* (campo `time`).

- **Cadastros excluídos:** quando alguém sai, o login é apagado, e as demandas da pessoa ficam no sistema marcadas como EXCLUÍDO.

---

## Backup e restauração

Há três camadas:

1. **Semanal automático** no bucket privado `backups` do Supabase.
2. **Cópias no próprio banco** (`docs_backup_AAAAMMDD`), feitas antes de mudanças grandes.
3. **Externo** no repositório privado `alra-marketing/sistema-alra-backup`.

A restauração só é feita com autorização da direção. Antes de restaurar, faça um backup do estado atual.

---

## Como colocar no ar do zero

Só é preciso se o projeto tiver que ser recriado.

1. **Banco:** no Supabase, abra o *SQL Editor* e rode `supabase/schema.sql`. Depois rode `acesso-diretoria.sql`, `equipe-diretoria.sql` e `senha-padrao.sql`. Os dados vêm do backup e não ficam neste repositório.
2. **Vercel:** *Add New → Project*, importe este repositório (framework *Other*) e faça o deploy.
3. **Ligar o banco:** em *Storage* (ou *Integrations*), conecte o projeto Supabase ao projeto da Vercel. As chaves entram sozinhas.
4. **Cron:** crie `CRON_SECRET` em *Settings → Environment Variables* e faça um novo deploy.
5. **Endereço do login:** no Supabase, em *Authentication → URL Configuration*, coloque o endereço da Vercel em *Site URL* e em *Redirect URLs*.
6. **Primeiro administrador:** inclua o e-mail na tabela `admins` e crie o login em *Authentication → Users → Add user*.

---

## Pendências

- [ ] Conferir se o deploy foi refeito depois do cadastro do `CRON_SECRET`.
- [ ] Dar acesso de administrador a mais uma pessoa da direção.
- [ ] Definir a data e a forma de tirar do sistema Calendário, Kanban de conteúdo, Aprovações e Publicações, que vão para o Pode Postar, guardando os dados para consulta.
- [ ] Revisar a área *Acessos* dos clientes.
- [ ] Revisar a visibilidade do repositório e as recomendações de segurança do Supabase (*Advisors*).

---

## Glossário

| Termo | Significado |
|---|---|
| **Pauta / peça** | Um conteúdo (post, story, vídeo, anúncio, newsletter etc.) ou uma tarefa interna. |
| **Pacote** | O conjunto de peças que o cliente aprova de uma vez (o mês ou a semana). |
| **Squad** | A equipe de um cliente: planejamento, produção, social media, tráfego e atendimento. |
| **Captação** | Gravação ou sessão de fotos/vídeos com o cliente. |
| **Onboarding / Orientações** | Checklist que o cliente responde pelo link `/o/<token>`. |
| **Pode Postar** | Plataforma que vai assumir o fluxo de conteúdo. |
| **Time A/B/C/D** | Nível de acesso: Diretoria, Head, demais da equipe e clientes. |
