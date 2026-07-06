# Project Setup — hoe bouwen we een QBlogics-project

De vaste handoff voor elk nieuw project. Volg dit en het project past in de
org-standaarden, de server-infra en de deploy-flow.

---

## 0. Start van een template

Nieuw project? Begin **niet** blanco. Gebruik een template:

- **Flask** (marketing, portfolio, content-sites): [`qblogics/template-web-flask`](https://github.com/qblogics/template-web-flask)
- **Django** (auth, database, admin, secure forms): [`qblogics/template-web-django`](https://github.com/qblogics/template-web-django)

Op GitHub: **Use this template → Create new repository** onder de `qblogics` org.

```bash
git clone git@github.com:qblogics/<nieuwe-repo>.git
cd <nieuwe-repo>
# vervang de placeholder overal
grep -rl PROJECT_NAME_PLACEHOLDER . | xargs sed -i 's/PROJECT_NAME_PLACEHOLDER/<nieuwe-repo>/g'
```

Bestaande app (PrestaShop, ander framework)? Zie **§6 Niet-template projecten**.

---

## 1. Repo-hygiëne (eenmalig)

Elke repo hoort te hebben:

- [ ] `LICENSE` — proprietary (`Copyright (c) 2026 QBlogics. All rights reserved.`)
- [ ] `.gitignore` met `.env`, `venv/`, `__pycache__/`, build-output
- [ ] `.env.example` met alle vereiste keys (secrets als placeholder)
- [ ] `README.md`: wat / setup / run / test / deploy
- [ ] `.github/dependabot.yml` (erft anders van de org)
- [ ] `.github/CODEOWNERS` → `* @qblogics/engineers`
- [ ] Squash-only merge + delete-branch-on-merge (repo settings)
- [ ] Labels via het org-schema (`type:*`, `priority:*`, `status:*`)

De templates hebben dit al. Bij een bestaande repo: kopieer uit een template.

---

## 2. Secrets

- **Nooit** secrets in git. `.env` staat in `.gitignore`.
- `.env.example` documenteert welke keys nodig zijn (met placeholders).
- Productie-secrets staan als `.env` op de server (chmod 600) of als
  GitHub Actions secret.
- Genereer sterke waarden: `python3 -c "import secrets; print(secrets.token_urlsafe(48))"`

---

## 3. Dependencies

- Python: **uv** (`uv sync`, `uv add <pkg>`). Lockfile in git, `.python-version` pinnen.
- Dependabot draait wekelijks (pip) + maandelijks (actions/docker).

---

## 4. PR-gates (kwaliteit)

Twee lagen:

**Lokaal (pre-commit)** — de echte gate op Free-plan:
```bash
pip install pre-commit
pre-commit install
```
Draait bij elke commit: ruff format/check, pytest -x, detect-secrets,
conventional-commit check. Config: `.pre-commit-config.yaml` (uit template).

**GitHub Actions (CI)** — adviserend:
- `.github/workflows/ci.yml` draait pytest + ruff op elke PR.
- Free-plan heeft geen branch-protection, dus CI **blokkeert** merge niet hard.
  Behandel rood als "niet mergen". Wie op Team-plan zit kan het afdwingen.

---

## 5. Deploy naar de server (dev / test / prod)

De server draait Traefik als reverse-proxy. Zie [`qblogics/infra`](https://github.com/qblogics/infra)
voor de architectuur. Kort:

1. **Container op het `traefik-web` netwerk** (compose: extern netwerk `traefik-web`).
2. **Route-file** `~/infra/traefik/dynamic/<project>.yml` op de server:
   - prod-router: `Host(<project>.qblogics.com)`, publiek
   - test/dev-router: `Host(<project>-test.qblogics.com)`, `middlewares: [qb-auth@file]`
3. Klaar — de tunnel-wildcard `*.qblogics.com` maakt het extern bereikbaar,
   geen tunnel-edit nodig.

**Naamgeving** (enkel-niveau, gratis SSL):

| Env | URL | Auth |
|---|---|---|
| prod | `<project>.qblogics.com` | publiek |
| test | `<project>-test.qblogics.com` | login |
| dev | `<project>-dev.qblogics.com` | login |
| preview | `<project>-<branch>.qblogics.com` | login |

**Poorten**: elk project luistert intern op zijn eigen poort (Flask 5000,
Django 8000). Traefik routeert op hostname — geen host-poort per project nodig.

---

## 6. Niet-template projecten (bestaande apps)

Voor apps die niet uit een template komen (PrestaShop, Node, etc.):

- Zorg voor een `docker-compose.yml` met secrets via `${VAR}` uit `.env`.
- `.env` gitignored, `.env.example` erbij.
- Hang de web-container op `traefik-web`, maak een route-file.
- Voeg alsnog `LICENSE`, `README`, `.gitignore` toe.

Voorbeeld in de praktijk: [`qblogics/braber-shop`](https://github.com/qblogics/braber-shop) (PrestaShop).

---

## 7. Handoff-checklist (voor je "klaar" zegt)

- [ ] Repo van template of met alle hygiëne-bestanden
- [ ] `.env.example` compleet, echte `.env` gitignored
- [ ] `pre-commit install` gedraaid
- [ ] CI groen op de PR
- [ ] Container draait op `traefik-web` + route-file toegevoegd
- [ ] prod publiek, test/dev achter `qb-auth@file`
- [ ] README uitgelegd: wat / setup / run / test / deploy
- [ ] `CLAUDE.md` aanwezig (context voor AI-sessies)

Zie ook [CONTRIBUTING.md](CONTRIBUTING.md) voor commits, branches en merges.
