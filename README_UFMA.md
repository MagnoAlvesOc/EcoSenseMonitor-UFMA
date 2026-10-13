# EcoSenseMonitor — Docker na UFMA

**Comece por [`docs/IMPLANTACAO_UFMA_DOCKER.md`](docs/IMPLANTACAO_UFMA_DOCKER.md).**

```bash
cp .env.ufma.example .env
python3 scripts/GERAR_SEGREDOS.py   # coloque segredos gerados no .env
# Configure DOMAIN/DNS/HTTPS e chaves OAuth em .env
docker compose --env-file .env -f compose.ufma.yaml up -d --build
```

- Portal, backend Python e contas: mesmo host HTTPS.
- Dados: PostgreSQL no volume Docker persistente (não exposto à Internet).
- Google/Microsoft login OIDC opcional quando credenciais configuradas.
- Android: `android/COMPILAR_APK.md`; o servidor hospeda o arquivo `releases/EcoSenseMonitor.apk` **quando for criado, assinado e colocado lá**.
- Fonte ESP32: POST `/api/ingest` com header `X-Station-Key`; Apps Script legado é opcional.
- O site funciona mesmo sem nenhuma estação cadastrada; leituras ausentes não são fabricadas.

**Não entregue os segredos Google/Microsoft nem o `.env` preenchido junto ao ZIP.**

Para publicar no GitHub: [`docs/PUBLICAR_GITHUB.md`](docs/PUBLICAR_GITHUB.md). O proxy HTTPS encaminha para `web:8000`, correspondente ao serviço do Docker Compose.