# Migração para as contas da Alra (25/11/2026)

- Código: `alra-marketing/sistema-alra` (GitHub da Alra), deploy automático no Vercel time `alramarketing1`, projeto `sistema-alra`.
- Banco: Supabase org **Alra Marketing**, projeto `sistema-alra` (`gzkosfkozotmbqcrutbg`, São Paulo, plano Free).
  Variáveis do Supabase sincronizadas no Vercel pela integração oficial (não editar à mão).
- Backup privado: `alra-marketing/sistema-alra-backup`.
- Pendente no Vercel: cadastrar `CRON_SECRET` (protege `/api/cron-backup` e `/api/cron-limpeza`).
