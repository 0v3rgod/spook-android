# SPOOK Android (PentestLink)

App Android (Kotlin + Jetpack Compose) do ecossistema SPOOK. Comunica-se com o **Oracle Relay** (chat, tasks, IA, media, relatórios, pad, finance, reader) e com o **Arch Server** (comandos, métricas, terminal, teclado, scans).

## Versão atual

- **versionName**: 1.16
- **versionCode**: 16
- **minSdk**: 29 (Android 10)
- **targetSdk**: 36
- **Package**: `com.taylor.pentestlink`

## Funcionalidades

| Aba | Descrição | Backend |
|-----|-----------|---------|
| **HOME** | Dashboard com status de conexão, bateria do celular, ações rápidas | Relay + Arch |
| **CHAT** | Mensagens sincronizadas via Socket.IO + Drive (um arquivo por msg) | Relay |
| **IA** | Assistente (Groq `openai/gpt-oss-20b`) com tool-call para scans/comandos | Relay `/ia` |
| **READER** | Leitor de e-books cross-device (ficha por livro = sha256 do conteúdo) | Relay HTTP + Socket |
| **PAD** | Notas + Tasks com sync HTTP + Socket (merge 3 vias, calendário) | Relay |
| **SCANS** | Galeria de relatórios (nmap/nikto/gobuster...) + Termux RUN_COMMAND | Relay (media/relatorios) |
| **FINANCE** | Despesas e parcelas (Room local + sync relay LWW) | Relay HTTP |
| **CONFIG** | Pareamento, URLs, Drive (SAF), logs | — |

## Arquitetura de rede

```
App (Poco X7, Tailscale <CELULAR_TAILSCALE_IP>)
    │
    ├── Relay (Oracle VM, <RELAY_TAILSCALE_IP>:5000) ──► Google Drive (gdrive:SPOOK)
    │     ├── Socket.IO: chat, tasks, pad, IA, notificações, leitura
    │     └── HTTP: /livros, /media, /relatorios, /finance, /pad, /upload
    │
    └── Arch Server (<ARCH_TAILSCALE_IP>:5000) ──► Google Drive (mesmo gdrive:SPOOK)
          ├── Socket.IO: comandos, métricas, terminal, teclado, scans IA
          └── Bridge IA: executa tool-calls do relay (nmap, comandos, info)
```

## Build

```bash
# Requisitos: JDK 17+ (jdk-openjdk no Arch), Android SDK 36
cd AndroidStudioProjects/PentestLink
./gradlew assembleDebug
# APK em: app/build/outputs/apk/debug/app-debug.apk
```

## Instalação no Poco X7

```bash
adb -s <SERIAL_DO_CELULAR> install -r app/build/outputs/apk/debug/app-debug.apk
```

## Configuração (prefs `pentestlink_config`)

| Chave | Padrão | Descrição |
|-------|--------|-----------|
| `servidor_url` | `http://<ARCH_TAILSCALE_IP>:5000` | Arch (comandos/métricas/terminal) |
| `chat_url` | `http://<RELAY_TAILSCALE_IP>:5000` | Relay (chat/tasks/IA/media/relatorios/pad/finance/reader) |
| `ia_url` | `http://<RELAY_TAILSCALE_IP>:5000/ia` | Namespace IA do relay |
| `relatorios_url` | `http://<RELAY_TAILSCALE_IP>:5000` | Relatórios via relay |
| `media_url` | `http://<RELAY_TAILSCALE_IP>:5000` | Mídia via relay |
| `pad_url` | `http://<RELAY_TAILSCALE_IP>:5000` | Pad via relay |

**Drive (SAF)**: pasta `SPOOK` selecionada manualmente uma vez (tree URI `acc=5` no Poco X7). Persistido em `spook_drive`/`tree_uri`.

## Pareamento

1. App abre tela de pareamento → gera chave pública Ed25519
2. Envia pro Arch (`/parear`) → Arch salva `chave_publica_celular.pem`
3. Arch gera token de sessão → salva em `~/.pentestlink/token_sessao.txt`
4. App recebe token → usa em `Authorization: Bearer <token>` (HTTP) e `auth={token}` (Socket.IO)

## Termux (scans no celular)

- Permissão `com.termux.permission.RUN_COMMAND` no manifest
- `TermuxLauncher.executarComandoScan` inicia `com.termux.RUN_COMMAND`
- Script `~/.local/bin/pentestlink-scan` no Termux roda o scan, faz `tee` no log e `curl -F` pro relay (`/upload` com `pasta=relatorios`)
- Requer aprovação única: Settings → Termux → "Allow external apps"

## Segurança

- Token **nunca** hardcoded (lido de `Pareamento.obterToken()` → `~/.pentestlink/token_sessao.txt` no Arch/relay)
- `network_security_config.xml` permite cleartext **apenas** para os IPs Tailscale conhecidos
- Permissões mínimas: `INTERNET`, `RUN_COMMAND`, `VIBRATE`, `POST_NOTIFICATIONS`, `FOREGROUND_SERVICE`, `FOREGROUND_SERVICE_SPECIAL_USE`, `BLUETOOTH*`, `LOCATION*`, `RECORD_AUDIO`, `CAMERA`

## Estrutura principal

```
app/src/main/java/com/taylor/pentestlink/
├── finance/           # Room entities, DAO, Store, Client, UI (Expenses/Installments)
├── ui/theme/          # SpookTheme, ícones custom (ImageVector), tipografia Mono
├── ConexaoChat.kt     # Socket.IO chat/tasks/pad/IA
├── ConexaoNotebook.kt # Socket.IO comandos/métricas/terminal (Arch)
├── HubStore.kt        # Modelo local (Task, Expense, Livro, Leitura)
├── MainActivity.kt    # BottomNav (7 abas), navegação Compose
├── NotificacaoPcServico.kt # Foreground service (notificações do relay)
├── Pareamento.kt      # Chaves Ed25519, token de sessão
├── TelaDashboard.kt   # Home (status, bateria, ações)
├── TelaPad.kt         # Abas NOTES / TASKS (calendário)
├── TelaPadTarefas.kt  # Tasks com due_date, cartão por dia
└── TelaScans.kt       # Galeria relatórios + Termux launcher
```

## Licença

MIT — uso pessoal/estudo. Código público para portfólio.
