# EcoSenseMonitor 8.0 — entrega para implantação em servidor da UFMA

## Escopo desta entrega

Plataforma pública responsiva, dados de estações ambientais, cadastro privado de proprietários, integração direta de dispositivos ESP32, integração Google Apps Script opcional, APIs e análises Python, banco PostgreSQL, login Google e Microsoft (OAuth/OIDC), portal jurídico, download de APK (quando assinado e publicado). A aplicação, o banco e o arquivo APK ficam hospedados no servidor da UFMA.

**Estado de validação:** a aplicação e os endpoints foram testados localmente em SQLite. O caminho PostgreSQL foi desenvolvido, mas seu funcionamento com contêineres deverá ser validado pela equipe de TI no ambiente com Docker. Login OAuth exige registro e credenciais reais; APK precisa ser compilado e assinado. Não foi realizada publicação na UFMA.

## Requisitos de infraestrutura a solicitar à STI/UFMA

- VM Linux x86_64 com Ubuntu Server 24.04 LTS ou outra distribuição mantida; inicialmente **4 vCPU, 8 GiB RAM e 100 GiB SSD** recomendados (dimensionamento a revisar conforme volume de telemetria e estações).
- Docker Engine e Docker Compose plugin atualizados; permissões administrativas restritas.
- Domínio ou subdomínio institucional (ex. `ecosense.seu-dominio.ufma.br`) apontado ao IP do servidor; DNS gerenciado pela instituição.
- Liberação **TCP 80 e 443** (e opcionalmente UDP 443 para HTTP/3); TCP 22 somente via VPN/rede administrativa. **Não expor TCP 5432 ou 8000** na internet.
- Certificado TLS público confiável; Caddy pode obter e renovar automaticamente se DNS público e políticas de rede permitirem. Se a UFMA fornecer certificado institucional, adaptar Caddy para utilizá-lo.
- Outbound HTTPS TCP 443 para autenticação Google, Microsoft, Google Apps Script (se utilizado), fontes de mapa/CDNs e validações de certificado. Para eliminar serviços cartográficos/CDN externos, os arquivos deverão ser hospedados localmente e o mapa deverá usar um provedor interno licenciado.
- IP público roteável ou publicação por proxy reverso institucional, firewall e regras de WAF/rate-limit quando disponíveis.
- Espaço de backup independente, monitoramento, atualizações, registro de mudanças e regras de retenção compatíveis com a política da UFMA.
- Antes da disponibilização para produção: homologação e aceite institucional, revisão de privacidade, política de uso de dados, responsável pelo tratamento de dados e licenças cartográficas.

## Instalação na VM da UFMA

1. Copie os arquivos para `/opt/ecosensemonitor`, ex.: transferência SFTP segura ou repositório Git institucional. Nunca inclua o `.env` com senhas.
2. No servidor execute:

```bash
cd /opt/ecosensemonitor
cp .env.ufma.example .env
python3 scripts/GERAR_SEGREDOS.py
```

3. Edite `.env` com `DOMAIN`, `SECRET_KEY`, `POSTGRES_PASSWORD` e credenciais de provedores (se configurados). O segredo do banco no exemplo é hexadecimal para não quebrar a URL de conexão.
4. Confirme DNS, portas e permissão para Caddy emitir certificado. Depois execute:

```bash
docker compose --env-file .env -f compose.ufma.yaml config
docker compose --env-file .env -f compose.ufma.yaml up -d --build
docker compose --env-file .env -f compose.ufma.yaml ps
docker compose --env-file .env -f compose.ufma.yaml logs --tail=120 web
docker compose --env-file .env -f compose.ufma.yaml logs --tail=120 caddy
```

5. Verifique as rotas:

- `https://SEU_DOMINIO/api/health` — API e DB disponíveis
- `https://SEU_DOMINIO/` — mapa inicial
- `https://SEU_DOMINIO/login` — login local, Google e Microsoft (se configurados)
- `https://SEU_DOMINIO/api/stations` — estações públicas
- `https://SEU_DOMINIO/analises` — análises
- `https://SEU_DOMINIO/juridico` — central jurídica
- `https://SEU_DOMINIO/android` — distribuição de APK (404 no download se não houver APK assinado)

6. Crie um administrador somente com a equipe de TI, seguindo `README.md` e `python -m ecosense create-admin`. No contêiner: `docker compose --env-file .env -f compose.ufma.yaml exec web python -m ecosense create-admin --username NOME`; o comando poderá solicitar senha em sessão segura.
7. Reinicie web após alterar credenciais/variáveis: `docker compose --env-file .env -f compose.ufma.yaml up -d --force-recreate web`.

## Login por Google

Registrar um **cliente OAuth Web** no Google Cloud, com escopos mínimos `openid email profile`. URI de redirecionamento autorizado EXATO:

`https://SEU_DOMINIO/auth/google/callback`

Configurar no `.env` `GOOGLE_CLIENT_ID` e `GOOGLE_CLIENT_SECRET`. Registrar domínios autorizados e tela de consentimento. Para público geral, o app poderá necessitar publicação e verificação de marca por Google. O site nunca solicita nem armazena senhas Google.

## Login por Microsoft

Registrar um aplicativo em **Microsoft Entra (App registrations)**. Configure público-alvo do aplicativo como desejado (contas institucionais ou também contas pessoais) e URI do tipo **Web**:

`https://SEU_DOMINIO/auth/microsoft/callback`

Gerar segredo de cliente e configurar `MICROSOFT_CLIENT_ID`, `MICROSOFT_CLIENT_SECRET` e `MICROSOFT_TENANT_ID`. `common` permite contas pessoais e organizações se o registro permitir; `organizations` só organizações; GUID de tenant limita ao diretório informado. A TI institucional pode precisar consentir com o registro, dependendo das políticas Entra.

Os usuários são identificados por `sub` Google ou par `tenant + oid/sub` Microsoft. E-mails iguais **não são automaticamente vinculados** entre diferentes credenciais; um usuário existente deve entrar e usar os botões **Vincular Google/Microsoft** em Meu espaço. Isso evita que uma conta nova controle estações de outra pessoa por correspondência de e-mail.

## Dados persistentes, integridade e cópias de segurança

O Docker Compose usa `postgres_data` para PostgreSQL, `app_data` para dados auxiliares e `caddy_data` para certificados. Dados não devem ficar somente dentro das camadas efêmeras dos contêineres. Realize `bash scripts/Gerar_Backup_Postgres_UFMA.sh` e depois mova a cópia para destino separado (ex.: storage institucional) e teste rotinas de restauração. Plano de retenção e criptografia são responsabilidade institucional.

**Migrar histórico da versão SQLite/Google Sheets:** não copie o `.db` antigo para substituir o PostgreSQL. Faça uma importação controlada, usando as APIs e scripts de migração, com validação da quantidade de leituras, usuários e vínculo de estações antes de apontar dispositivos para o servidor. Não compartilhar chaves secretas de estações publicamente.

## Android: APK hospedado pela UFMA

A rota de download já existe, mas **não há APK construído nesta entrega**. Execute os passos de `android/COMPILAR_APK.md` depois de haver HTTPS e domínio definitivo. Coloque o arquivo assinado `releases/EcoSenseMonitor.apk` no servidor; o menu `/android` passará a exibir o link. Gere `releases/assetlinks.json` a partir do fingerprint real, publicado em `/.well-known/assetlinks.json`.

## Riscos e limites a conferir antes de produção

- Faça teste de PostgreSQL real, login OAuth com cada provedor, integração ESP32 de ponta a ponta e download Android em dispositivo físico. O teste de autenticação sem credenciais reais não prova a integração com provedores.
- O site ainda carrega imagens de mapa, bibliotecas e fontes de terceiros. A hospedagem da aplicação e do APK é própria, mas não completamente offline/independente desses recursos. Avalie um servidor de tiles autorizado e copie bibliotecas para static caso exija independência integral.
- Faça teste de carga e implemente rate-limiting distribuído/WAF para novos usuários e API de ingestão. A limitação atual da aplicação é apenas por processo para rotas de autenticação.
- Revisar documentos jurídicos (minutas) com o setor jurídico e encarregado LGPD da instituição. Estabelecer planos de resposta a incidentes, segurança, proteção de segredos e backups, gestão de credenciais.
- Google/Microsoft são provedores externos de identidade: se ficarem indisponíveis, a autenticação social falhará, mas login local continuará disponível.