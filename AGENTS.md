# Regole per gli agenti

Qualsiasi agente (Claude Code, subagenti, altri strumenti) che lavora in questo repository deve
seguire le regole nella cartella `docs/`. Sono vincolanti, non suggerimenti. Se una richiesta
è in conflitto con una regola, fermati e segnalalo invece di aggirarla.

## Documenti da rispettare

- `docs/architecture.md`: architettura generale (frontend, backend, ambienti). Leggilo prima di
  qualsiasi modifica strutturale.
- `docs/backend-guidelines.md`: da leggere prima di toccare `backend/`.
- `docs/frontend-guidelines.md`: da leggere prima di toccare `frontend/`.
- `docs/versions.md`: versioni dei pacchetti; se ne cambi una, aggiorna il file nella stessa modifica.
- Ogni altro file in `docs/` vale allo stesso modo.

Costituzione del progetto: `.specify/memory/constitution.md`.

## Come lavorare

1. Prima di modificare codice, leggi i documenti pertinenti all'area che tocchi.
2. Le regole sono etichettate [test], [CI], [review], [obiettivo]. Rispetta anche quelle
   [review]; quelle [obiettivo] non sono ancora applicate: non darle per scontate.
3. Se una modifica rende obsoleto un documento in `docs/`, aggiornalo nella stessa modifica.
4. Se i documenti sono ambigui o in contrasto tra loro, chiedi invece di scegliere da solo.
