# Runbook — Paperclip Conexão Azul

## Iniciar
```bash
cd /docker/paperclip
docker compose up -d
docker compose logs -f paperclip
```

## Parar
```bash
docker compose down
```

## Reiniciar só app (mantém DB)
```bash
docker compose restart paperclip
```

## Ver logs
```bash
docker compose logs -f paperclip
docker compose logs -f paperclip-db
```

## Backup manual
```bash
cd /docker/paperclip
docker exec paperclip-db pg_dump -U paperclip paperclip > backups/paperclip-$(date +%Y%m%d-%H%M).sql
```

## Restaurar backup
```bash
docker exec -i paperclip-db psql -U paperclip paperclip < backups/paperclip-YYYYMMDD-HHMM.sql
```

## Rebuild após update do fork
```bash
cd /docker/paperclip
git pull origin main
docker compose up -d --build
```

## Verificar saúde
```bash
docker compose ps
curl -sf http://localhost:3100/api/health && echo "OK"
```

## Ver uso de disco
```bash
docker system df
du -sh /docker/paperclip/data /docker/paperclip/backups
```
