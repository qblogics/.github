# Contributing

Standaarden die gelden voor elke QBlogics-repo, tenzij het projekt-README
expliciet iets anders zegt.

## Commits

Conventional Commits. Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`,
`perf`, `ci`, `build`, `style`.

```
feat(hero): add scroll-triggered marquee slowdown
fix(auth): honor SameSite=Lax on session cookie
docs(readme): add uv install instructions
```

Body optioneel, footer voor `BREAKING CHANGE:` of `Closes #123`.

## Branches

- `main` = altijd deployable, hoort op **prod** te draaien na tag
- Feature branches: `feat/<korte-naam>`, `fix/<korte-naam>`, `chore/<korte-naam>`
- PR title = titel van de squash-commit (Conventional Commit format)

## Merge

- **Squash-only.** Merge commits en rebase-merges staan uit op alle repos.
- Delete branch on merge = aan.
- Auto-merge mag na CI-groen + 1 approval.

## Environments & URL-conventie

Elke website-repo krijgt vier environments:

| Env | URL-patroon | Trigger | Publiek? |
|---|---|---|---|
| **dev** | `localhost:5001` per developer | lokaal opstarten (`make dev`) | Nee |
| **preview** | `<branch>.<project>.qblogics.com` | feature-branch push naar remote | Ja (klant kijkt mee per variant) |
| **test** | `test.<project>.qblogics.com` | merge naar `main` | Ja (klant finale check) |
| **prod** | `<project>.qblogics.com` | git tag `v<x.y.z>` | Ja (live) |

### Voorbeelden

```
localhost:5001                        → dev van website, lokaal
cleanup-css.website.qblogics.com      → preview van feature-branch
test.website.qblogics.com             → test na merge naar main
website.qblogics.com                  → prod na tag v1.2.3

variant-a.braber.qblogics.com         → preview per variant
test.braber.qblogics.com              → test-branch van client-werk
braber.qblogics.com                   → prod deploy voor Braber
```

### Onder de motorkap

Op de server draait **Traefik** als reverse proxy op poort 80. Cloudflared
routeert `*.qblogics.com` naar Traefik. Elke Docker-container heeft labels
die zeggen welke subdomeinen naar welke container gaan.

Nieuwe branch = nieuwe container met eigen label = automatisch bereikbaar
op de branch-subdomain zonder cloudflared config-edit.

Zie [`template-web-flask/deploy/README.md`](https://github.com/qblogics/template-web-flask/blob/main/deploy/README.md)
voor het volledige patroon.

### Poort-allocatie (per project)

Interne container-poorten volgen een schema om conflicten te vermijden:

| Project | dev-poort (lokaal) | container-poort (via Traefik) |
|---|---|---|
| website | 5001 | 5000 |
| braber | 4321 | 4300 |
| sterk-op-beeld | 4322 | 4310 |
| toekomstig | +10 per project | +10 per project |

Traefik ontkoppelt intern-poort van publiek-hostname; container-poort is dus
alleen intern.

## CI/CD gate

### Laag 1: pre-commit hooks (lokaal, per commit)

Elke repo heeft `.pre-commit-config.yaml` met ruff format/check, pytest -x,
detect-secrets, conventional-commit check. Install 1× per repo:

```bash
pip install pre-commit
pre-commit install
```

Voorkomt kapotte commits. Kan omzeild worden met `--no-verify`; dan grijpt
Laag 2 in.

### Laag 2: GitHub Actions (server-side, niet omzeilbaar)

- **PR open/sync**: pytest + ruff draaien in Actions, PR groen = mergebaar
- **Merge naar `main`**: image build → SSH deploy naar `test.<project>.qblogics.com`
- **Tag `v<x.y.z>`**: image build → SSH deploy naar prod
- **PR-labeled `deploy-preview`**: image build → deploy naar `<branch>.<project>.qblogics.com`

GitHub Actions Free tier = 2000 min/mnd. Ruime marge voor 3-6 repos.

Als jullie ooit weg willen van Actions: **Woodpecker CI** self-hosted op de
server is de switch (identieke YAML-syntax).

## Code style

- **Python**: ruff format + ruff check (config in `pyproject.toml`)
- **Type hints** op publieke API's
- **Tests**: pytest, minimaal 80% coverage nagestreefd
- **Docs**: README bijwerken bij feature-changes, `DESIGN.md`/`PRODUCT.md`
  bijwerken bij visuele of strategische shifts

## Dependencies

- Python: **uv** voor lockfile + venv (`uv sync`, `uv add <pkg>`)
- Elke repo pinnt Python-versie via `.python-version` en `pyproject.toml`
- Dependabot draait wekelijks, patch/minor auto-merge waar CI groen is

## Security

Zie [SECURITY.md](SECURITY.md).

- Geen secrets in code. `.env.*` zit in `.gitignore`.
- Elk project heeft `.env.example` als template met alle vereiste keys.
- Productie-secrets via GitHub Actions secrets of server-side env-file
  (chmod 600, owned by service-user).

## Naming

- Own product / marketing: `website`, `blog`
- Client-werk: `<client>` als losstaand woord of `<client>-<project>`
  (bijv. `braber`, `sterk-op-beeld`)
- Interne tools: `tool-<naam>`
- Experimenten: `lab-<naam>`

## Rol-scheiding

Beide members (`@BartHolterman`, `@Melonkon`) zijn admin op alle repos via
team `@qblogics/engineers`. Externe contributors krijgen `write` op de
enkele repo waar ze aan werken, geen org-admin.
