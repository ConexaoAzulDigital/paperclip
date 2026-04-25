# Rollback — Paperclip Conexão Azul

## Rollback Rápido (< 5 min)
```bash
cd /docker/paperclip
docker compose down
git log --oneline -10   # ver commits
git checkout <commit-anterior>
docker compose up -d --build
```

## Rollback com Restore de Dados
```bash
docker compose down
git checkout <commit-anterior>
# Restaurar banco
docker compose up -d paperclip-db
sleep 10
docker exec -i paperclip-db psql -U paperclip paperclip < backups/paperclip-YYYYMMDD-HHMM.sql
docker compose up -d
```

## Remoção Completa (nuclear)
```bash
docker compose down -v   # REMOVE VOLUMES — dados perdidos!
docker rmi paperclip-paperclip 2>/dev/null
rm -rf /docker/paperclip/data
```
⚠️ Irreversível. Só executar com aprovação explícita.

## Impacto do Rollback
- Paperclip: sem impacto nos outros serviços (isolado)
- Outros serviços: zero impacto
- Dados no DB: preservados se não usar -v
