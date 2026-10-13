# EcoSenseMonitor Android — projeto de distribuição (TWA)

Este repositório contém a aplicação web responsiva/PWA e a infraestrutura para hospedar um APK **na própria UFMA**. O APK **não está incluído**: deve ser gerado em uma estação de desenvolvimento com Android SDK e Java, depois assinado pelo responsável institucional. Não existe APK funcional sem URL pública de produção e configuração de assinatura.

## Método recomendado: Trusted Web Activity (TWA)

Com o domínio em HTTPS e o site no ar:

1. Instale Android Studio/JDK/Android SDK e Node.js em uma máquina de desenvolvimento. Mantenha o arquivo `.jks` fora do repositório, com cópias protegidas.
2. Confirme que `https://SEU_DOMINIO/manifest.webmanifest` responde com JSON, que o site funciona em Android e que as páginas de login Google/Microsoft estão autorizadas.
3. Instale a ferramenta oficial de geração TWA: `npm install -g @bubblewrap/cli`.
4. No diretório escolhido para o projeto Android, execute `bubblewrap init --manifest=https://SEU_DOMINIO/manifest.webmanifest`. Informe package ID, nome, cores, ícones e caminhos de assinatura solicitados pela ferramenta.
5. Execute `bubblewrap build` e teste o APK em dispositivo Android **real**.
6. Confira e anote o certificado de assinatura: `keytool -list -v -keystore CAMINHO-DA-CHAVE.jks -alias ALIAS` (se for certificado SHA-256).
7. Na pasta do EcoSenseMonitor, execute `python scripts/GERAR_ASSETLINKS.py br.ufma.ecosensemonitor AA:BB:...` com o identificador e fingerprint reais.
8. Copie o APK assinado validado para `releases/EcoSenseMonitor.apk`. A rota pública `/downloads/EcoSenseMonitor.apk` já serve esse arquivo, e `/android` mostra o botão quando existir.
9. Verifique `https://SEU_DOMINIO/.well-known/assetlinks.json`. Use o Android Asset Links verifier e teste a TWA (sem barra de URL) em aparelho real.

**Importante:** a TWA carrega o mesmo site pela internet e não é um app que executa o servidor localmente. Sem rede, partes dependentes da API e do mapa não estarão disponíveis. Google e Microsoft autenticam pelo navegador sem o aplicativo receber suas senhas. A compilação precisa ser refeita para trocar domínio, package ID ou assinatura.

A distribuição de APK fora das lojas depende das políticas de segurança do Android e da UFMA. O usuário pode receber um aviso para permitir instalações por essa origem. Nunca instrua usuários a desativar o antivírus ou contornar verificações de assinatura.

## Entrega à TI

- APK de produção **assinado e testado**
- versão / checksum SHA-256 do arquivo
- certificado SHA-256 / Digital Asset Links
- package ID, licenças e política de atualização
- arquivo de chave sob custódia institucional (NUNCA público)

Referência: https://developer.android.com/develop/ui/views/layout/webapps/guide-trusted-web-activities