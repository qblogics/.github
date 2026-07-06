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

- `main` = altijd deployable
- Feature branches: `feat/<korte-naam>`, `fix/<korte-naam>`, `chore/<korte-naam>`
- PR title = titel van de squash-commit (Conventional Commit format)

## Merge

- **Squash-only.** Merge commits en rebase-merges staan uit op alle repos.
- Delete branch on merge = aan.
- Auto-merge mag na CI-groen + 1 approval.

## Code style

- **Python**: ruff format + ruff check (config in `pyproject.toml`)
- **Type hints** op publieke API's
- **Tests**: pytest, minimaal 80% coverage nagestreefd
- **Docs**: README bijwerken bij feature-changes, DESIGN.md/PRODUCT.md
  bijwerken bij visuele of strategische shifts

## Dependencies

- Python: **uv** voor lockfile + venv (`uv sync`, `uv add <pkg>`)
- Dependabot draait wekelijks, patch/minor auto-merge waar CI groen is

## Security

Zie [SECURITY.md](SECURITY.md).

## Naming

- Own product / marketing: `website`, `blog`
- Client-werk: `client-<naam>` (bijv. `client-acme`)
- Interne tools: `tool-<naam>`
- Experimenten: `lab-<naam>`

## Rol-scheiding

Beide members (`@BartHolterman`, `@Melonkon`) zijn admin op alle repos via
team `@qblogics/engineers`. Externe contributors krijgen `write` op de
enkele repo waar ze aan werken, geen org-admin.
