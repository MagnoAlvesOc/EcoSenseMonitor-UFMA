**Publicação inicial:** esta branch `main` contém a documentação de implantação. Os arquivos completos do aplicativo serão enviados a seguir. Não execute Docker até confirmar `ecosense/main.py`, `Dockerfile` e `compose.ufma.yaml` no repositório.

# EcoSenseMonitor — Plataforma ambiental (Docker / UFMA)

Aplicação web responsiva e API Python/FastAPI para uma rede colaborativa de estações ambientais. O site, as análises científicas, os relatórios e a API compartilham o PostgreSQL do servidor. Login local, Google e Microsoft (OIDC) mediante credenciais. Publicação HTTPS via Caddy.

> **Estado:** código preparado para homologação. Não há instância UFMA publicada, credenciais OAuth institucionais ou APK Android compilado/assinado incluídos neste repositório. O uso de mapas e scripts CDN externos ainda requer conectividade e revisão de licença/privacidade.

## Implantação na UFMA (Linux + Docker Compose)

1. Solicite à STI subdomínio, DNS, portas TCP 80/443, armazenamento persistente e autorização de deploy.
2. Copie `.env.ufma.example` para `.env` e substitua TODOS os valores `SUBSTITUIR...`, domínio e credenciais. Não comite `.env`.
3. Execute `docker compose --env-file .env -f compose.ufma.yaml config --quiet` para validar a configuração.
4. Execute `docker compose --env-file .env -f compose.ufma.yaml up -d --build`.
5. Confira `docker compose -f compose.ufma.yaml ps`, `docker compose -f compose.ufma.yaml logs --tail=100 web caddy` e `https://SEU-DOMINIO/api/health`.
6. Configure os OAuth redirects e faça teste real de login; valide ingestão ESP32, dados GPS, relatórios e backup.

> **Importante:** a versão com APK precisa de build e assinatura Android separados. O site só apresenta o download quando `releases/EcoSenseMonitor.apk` existir no host. Nunca envie chaves privadas de assinatura ao GitHub.

Documentação técnica: [implantar no servidor da UFMA](docs/IMPLANTACAO_UFMA_DOCKER.md) · [API](docs/ENDPOINTS.md) · [Android](android/COMPILAR_APK.md) · [publicação GitHub](docs/PUBLICAR_GITHUB.md).

## Desenvolvimento e testes

```bash
python -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt -r requirements-dev.txt
python -m pytest -q
uvicorn ecosense.main:app --reload
```

## Estrutura

- `ecosense/`: rotas FastAPI, ingestão IoT, OAuth, análises Pandas/NumPy e banco.
- `static/` e `templates/`: layout web responsivo, mapa e gráficos.
- `legal/`: minutas jurídicas; requerem validação antes de disponibilização institucional.
- `android/` e `releases/`: instruções e espaço para APK assinado.
- `compose.ufma.yaml`, `Dockerfile`, `Caddyfile`: publicação institucional HTTPS.
- `tests/`: testes automatizados.

## Segurança e autoria

Antes de liberar a rede publicamente, a STI deverá avaliar política de backups, responsabilidades LGPD, atualização de dependências, estratégia de rate limiting e armazenamento de imagens cartográficas. Código-fonte sem licença explícita: solicite definição do regime de licenciamento pelo responsável pelo projeto e/ou pela instituição.