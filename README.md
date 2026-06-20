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

## Requisitos

- [opencode](https://opencode.ai)
- [pnpm](https://pnpm.io)

---

**Autor**: Alexis J Perdomo D — Backend Engineer (Go, Python, PostgreSQL)
