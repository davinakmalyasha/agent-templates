# Agent Template Library

Reusable boilerplate for autonomous development. Every template is a folder with:

```
templates/<name>/
├── template.json   ← metadata: description, category, variables, files, notes, uses
└── files/          ← the Jinja2 boilerplate files (written by apply_template)
```

The engine never loads templates into context automatically — agents discover them
via `template_list()` / `template_source()` and instantiate with `apply_template()`.
Boilerplate generation moves from LLM output tokens to deterministic tool execution.

## The 5 levels

```
STACK      complete recipe for a greenfield project — composes existing templates,
           pre-connected (auth + crud + ci + deploy in ONE call). No new code.
FEATURE    a full vertical slice — multi-file, wired together (model+schema+router+
           service+tests). May reference components/snippets.
COMPONENT  one cohesive module — single file, multi-function (exporter, scheduler,
           UI table). Imported and wired once.
SNIPPET    one tiny helper — single function/decorator (retry, slugify). ~30 tokens.
PROJECT    minimal runnable skeleton — the foundation everything mounts onto.
```

## Category map

| category  | level    | templates |
|-----------|----------|-----------|
| stack     | L5       | stack-webapp, stack-webapp-full |
| feature   | L3       | feature-crud, feature-auth, feature-upload, feature-email, feature-errors, feature-config, pytest-suite, test-smoke |
| component | L4       | comp-csv-exporter, comp-excel-exporter, comp-scheduler, comp-webhook-sender, comp-search-filter, comp-zip-archiver, comp-pdf-report, comp-queue-worker, comp-paginated-table, comp-charts, comp-form, comp-modal, comp-toast |
| snippet   | L2       | util-retry, util-pagination, util-logger, util-api-response, util-rate-limiter, util-memoize, util-slugify, util-date, util-env, util-validate, util-file, util-strings |
| project   | L1       | project-fastapi, project-frontend, project-bot, project-cli |
| config    | —        | github-ci-python, dockerfile-python, docker-compose, nginx |
| routine   | —        | gitignore-python, gitignore-node, requirements, pyproject, readme, license-mit |
| spec      | —        | spec-feature, spec-bsa |

## How agents should choose

1. Greenfield project → a `stack-*` template (or `project-*` + features)
2. Adding one feature → `feature-*`
3. Adding a subsystem → `comp-*`
4. Need a small helper → `util-*`
5. Config/deploy → `config`/`routine`
6. Before implementation → `spec-*` (standardized input, fewer retries)

## Conventions (curation rules)

- `description` must be ONE line — it is what `template_list` shows
- Every template needs `sample_vars` — CI renders it with those; no sample_vars = not validated
- Frontend templates use `[[ ]]` delimiters (`"delimiters": ["[[", "]]", "<%", "%>"]`) so
  they never collide with JS/HTML template syntax
- Stacks declare required vars once; nested templates inherit them, and use-level
  overrides may reference parent vars: `"variables": {"package_dir": "{{ project_name }}"}`
- `apply_template` is cycle-safe and dedupes repeated composition

## Tooling

| tool            | purpose |
|-----------------|---------|
| `apply_template`| render a template (or whole stack) into the project |
| `template_list` | list templates, filter by category |
| `template_source` | variables + recipe + conventions for one template |
| `template_pull` | sync hub -> local (`github.com/davinakmalyasha/agent-templates`) |
| `template_push` | publish local -> hub (creates the repo if missing) |

CI (`scripts/validate_templates.py`) re-renders every template with its sample vars on
every push — a broken template fails the build in seconds.
