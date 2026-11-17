# Opencode Dev Setup

Configuración reproducible de [opencode](https://opencode.ai) como herramienta de desarrollo.

## Setup

```bash
git clone https://github.com/AlexisPerdomoD/opencode-dev-setup.git
cd opencode-dev-setup
pnpm install
```

## Estructura

```
├── opencode.json          # Configuración principal de opencode
├── package.json          # Dependencias
├── pnpm-lock.yaml        # Lockfile (pnpm)
├── .gitignore
└── skills/
    ├── github-issues/     # Workflow para gestión de issues
    └── github-workflow/   # Convenciones y contexto para PRs/reviews
```

## Skills Incluidos

- **github-issues**: Workflow para creación y gestión de issues
- **github-workflow**: Convenciones de commit, PR y code review

## MCP

Los servidores MCP viven en `opencode.json` bajo `mcp.servers`. En OpenCode V2 se usa `disabled: true` para dejar un servidor declarado sin conectarlo (no existe `enabled`).

| Servidor   | Tipo   | Estado   | Requisitos                                    |
| ---------- | ------ | -------- | --------------------------------------------- |
| `github`   | remote | disabled | `AGENTS_GITHUB_TOKEN`                         |
| `linkedin` | local  | enabled  | `uv`/`uvx` y sesión de LinkedIn               |
| `slack`    | remote | disabled | App de Slack con OAuth y aprobación del admin |

### LinkedIn

Requiere `uv`/`uvx` en el PATH. En Arch:

```bash
sudo pacman -S uv
```

Si `uvx` no está en el PATH con el que se lanza OpenCode, instala `uv` con pacman o usa la ruta absoluta en `command`. La sesión se guarda en `~/.linkedin-mcp/`:

```bash
uvx mcp-server-linkedin@latest --status
uvx mcp-server-linkedin@latest --login
```

Este servidor usa el navegador y la sesión del usuario, no la API oficial de LinkedIn. Revisa el riesgo de scraping antes de usarlo con una cuenta corporativa.

### Slack (oficial)

1. Crear una app en <https://api.slack.com/apps> (interna o publicada en el Marketplace).
2. Activar **Agents → Slack MCP Server**.
3. Agregar los user scopes de lectura en **OAuth & Permissions**. La lista exacta está en `opencode.json` (`oauth.scope`).
4. Registrar el redirect `http://localhost:3118/callback`.
5. Activar **PKCE** (es irreversible).
6. Exportar las credenciales y autenticar:

```bash
export SLACK_MCP_CLIENT_ID="..."
export SLACK_MCP_CLIENT_SECRET="..."
opencode mcp auth slack
```

El servidor queda en `disabled: true` hasta que la app esté aprobada por el admin del workspace. Para activarlo, cambia a `false` o elimina la propiedad.

La alternativa comunitaria (`npx -y slack-mcp-server@latest`) funciona con tokens de navegador y sin aprobación de admin, pero usa APIs no oficiales; no se recomienda para workspaces corporativos.

## Requisitos

- [opencode](https://opencode.ai)
- [pnpm](https://pnpm.io)
- [uv](https://docs.astral.sh/uv/) — opcional, para el MCP de LinkedIn
- Node.js — opcional, para MCPs vía `npx`

---

**Autor**: Alexis J Perdomo D — Backend Engineer (Go, Python, PostgreSQL)
