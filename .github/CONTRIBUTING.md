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

Elke website-repo krijgt vier environments. **Enkel-niveau subdomeinen**
(`<project>-<env>.qblogics.com`), omdat Cloudflare's gratis Universal SSL
alleen `*.qblogics.com` (één niveau) dekt — `test.braber.qblogics.com`
(twee niveaus) zou een TLS-fout geven.

| Env | URL-patroon | Trigger | Auth |
|---|---|---|---|
| **dev** | `<project>-dev.qblogics.com` | lokaal / handmatig | BasicAuth |
| **preview** | `<project>-<branch>.qblogics.com` | PR met label `deploy-preview` | BasicAuth |
| **test** | `<project>-test.qblogics.com` | merge naar `main` | BasicAuth |
| **prod** | `<project>.qblogics.com` | git tag `v<x.y.z>` | publiek |

### Voorbeelden

```
qblogics.com                → prod van de marketing-site (apex)
website-test.qblogics.com   → test na merge naar main       (login)
website-dev.qblogics.com    → dev                            (login)

braber.qblogics.com         → prod deploy voor Braber
braber-test.qblogics.com    → test-branch van client-werk    (login)
braber-variant-e.qblogics.com → preview per variant          (login)
```

### Onder de motorkap

Op de server draait **Traefik** als reverse proxy op poort **:5000** (host).
Cloudflared routeert `qblogics.com` + `*.qblogics.com` → `localhost:5000`.
Traefik routeert daarna op hostname naar de juiste container.

**Provider**: op deze server gebruikt Traefik de **file-provider** (route-configs
in `~/infra/traefik/dynamic/*.yml`), NIET docker-labels. Reden: de snap
Docker-daemon (min API 1.40) en Traefik's docker-client (pingt 1.24)
onderhandelen niet. Elk project krijgt dus een `dynamic/<project>.yml` met
z'n routes; de docker-compose labels in de templates zijn hier decoratief.

Een nieuw project bereikbaar maken:
1. Container op netwerk `traefik-web` hangen.
2. `dynamic/<project>.yml` toevoegen (prod-router publiek, dev/test/preview
   met `middlewares: [qb-auth@file]`).
3. Geen tunnel-edit nodig — de wildcard `*.qblogics.com` vangt alles op.

### Auth op dev/test/preview

Alle niet-prod omgevingen zitten achter een Traefik BasicAuth-middleware
(`qb-auth@file`). Één gedeelde credential (`qblogics` + wachtwoord in
password manager). Prod-routers laten de middleware weg.

### Poort-allocatie

Op de server draait alles via Traefik op hostname — **geen host-poort per
project meer nodig**. Elke container luistert intern op zijn eigen poort
(Flask 5000, Django 8000) en Traefik routeert ernaartoe over het
`traefik-web` netwerk. Poort-boekhouding verdwijnt.

Alleen bij **lokaal dev buiten Traefik** (los `make dev` zonder proxy) is een
unieke host-poort handig; die staat per template in `docker-compose.dev.yml`.

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
