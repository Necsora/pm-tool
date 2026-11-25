# Versioni
Le versioni esatte dei pacchetti stanno nei lockfile (`backend/uv.lock`,
`frontend/package-lock.json`, `landing/package-lock.json`): sono la fonte di verità. Questo
file serve a leggerle in un posto solo. Quando cambi una versione, aggiornalo nella stessa
modifica.

## GitHub Actions

Riferite per versione maggiore nei workflow di `.github/workflows/`.

| Action | Versione |
|---|---|
| actions/checkout | v5 |
| actions/setup-node | v5 |
| astral-sh/setup-uv | v6 |
| gitleaks/gitleaks-action | v2 |

