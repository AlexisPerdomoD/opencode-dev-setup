---
name: github-issues
description: Use when creating, viewing, updating, or searching GitHub issues. Also use when analyzing a repository to plan issues or epics with subtasks. Trigger: github, issue, epic, subtask, label, milestone, assign.
---

# GitHub Issues

## Herramientas disponibles

| Tool | Uso |
|------|-----|
| `github_issue_write` `method:"create"` | Crear issue nuevo |
| `github_issue_write` `method:"update"` | Editar issue existente (estado, labels, body, etc.) |
| `github_issue_read` `method:"get"` | Ver detalles de un issue |
| `github_list_issues` | Listar issues (filtros: state, labels, assignee) |
| `github_issue_read` `method:"get_comments"` | Ver comentarios de un issue |

## Crear sub-issues (GraphQL)

**Siempre vía GraphQL**, no REST. La REST API no soporta `parent`.

```python
# Endpoint: https://api.github.com/graphql
mutation {
  createIssue(input: {
    repositoryId: "<REPO_NODE_ID>",
    title: "...",
    body: "...",
    parentIssueId: "<PARENT_NODE_ID>"
  }) {
    issue { number title parent { number } }
  }
}
```

Obtener `node_id` de un issue:
```bash
curl -s "https://api.github.com/repos/OWNER/REPO/issues/NUMBER" \
  -H "Authorization: Bearer $GITHUB_TOKEN" | python3 -c "import sys,json; print(json.load(sys.stdin)['node_id'])"
```

## Prefijos de título

| Prefijo | Uso |
|---------|-----|
| `feat:` | Nueva funcionalidad |
| `fix:` | Corrección de bug |
| `refactor:` | Cambio de estructura |
| `style:` | Lint, formato |
| `chore:` | Dependencias, tooling |
| `docs:` | Documentación |
| `test:` | Tests |
| `revert:` | Reversión |

Sub-issues de proyecto: `[F1.1]`, `[FE.2]` — sin números hardcodeados en títulos.

## Labels

- `epic` — issue padre que agrupa subtasks.
- `blocked` — esperando dependencia externa.
- `in-progress` — en desarrollo activo.

Tamaño opcional: `xs`, `s`, `m`, `xl`.

## Commits

`tipo: descripción breve`. Commits atómicos y pequeños. `ruff check` / lint pasa antes de commit.

## Reglas importantes

1. **Esperar aprobación explícita** antes de crear issues (el usuario dice "dale", "approved", etc.).
2. **Frontend tasks → repo de frontend**, no en el repo de backend.
3. **No asumir contexto** de sesiones anteriores — preguntar si no estás seguro.
