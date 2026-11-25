# Linee guida del backend

Come si scrive il backend (`backend/`): FastAPI sincrono, SQLAlchemy 2, PostgreSQL 16 con RLS.
Le versioni stanno nei lockfile (`backend/uv.lock`);


Ogni regola ha un'etichetta che dice **chi la fa rispettare**:

- **[test]**: la verifica un test in `backend/tests/` (quindi `make check` e la CI);
- **[CI]**: la verifica uno strumento in `make lint` / CI (ruff, mypy, import-linter, Bandit,
  pip-audit);
- **[review]**: oggi si controlla solo in revisione;
- **[obiettivo]**: non è ancora applicata (elenco in fondo, sezione «Da fare»). Non darla per
  scontata.

## Regole del backend
 Qui sotto le regole che non si aggirano.

- Nuovo modulo: crea `app/modules/<nome>/` con `public.py`, aggiungilo al contratto
  import-linter in `backend/pyproject.toml` e i suoi models in `app/db_models.py`
  (i test lo verificano).
- Da un altro modulo importa **solo** `app.modules.<altro>.public`.
- Endpoint di dominio: usa `Depends(tenant_db)` e `Depends(current_context)` /
  `require_role(...)`. Non usare `system_session()` fuori da tenancy e identity.
- Tabella con dati di un cliente: nella migrazione usa `tenant_id_column()` e chiama
  `enable_tenant_rls("<tabella>")`. `--autogenerate` non lo fa da solo: controlla sempre
  la migrazione generata.
- Non modificare una migrazione già applicata in staging o produzione (cioè presente su
  `develop` o `main`): scrivine una nuova.

Se una regola [test] o [CI] fallisce, correggi il codice, non il controllo (vedi `AGENTS.md`).

---

## 1. Principi

1. **Il contratto API è la fonte di verità.** Lo schema OpenAPI generato dal backend è ciò su cui
   compila il frontend. Dopo ogni modifica agli endpoint o ai modelli: `make api-types`. **[CI]**
   (`make api-contract` fallisce se il client è vecchio).
2. **Validare ai confini.** Tutto ciò che entra (body, query, header, ambiente) passa da un modello
   Pydantic o da un tipo FastAPI. Il dominio lavora con dati già validi.
3. **Errori uniformi.** Un solo formato (Problem Details, §5), mai stack trace al client.
4. **Autorizzazione su ogni risorsa.** Essere loggati non basta: l'isolamento tra clienti sta nel
   database (RLS, §3).
5. **Le migrazioni sono codice**: versionate, lette a mano, mai modificate dopo l'applicazione.
6. **Osservabile**: ogni richiesta ha un request id (§9).
7. **Semplicità prima dell'astrazione.** Niente pattern senza un problema concreto.
8. **La configurazione vive nell'ambiente**: lo stesso artefatto gira in locale, staging e
   produzione (§8).

---



**Regole**
- Da un altro modulo si importa **solo** `app.modules.<altro>.public`. **[CI]** (import-linter) e
  **[test]** (`test_architecture.py`: ogni modulo è nel contratto, ha `public.py` ed è registrato
  per Alembic).
- `app/shared/` non importa i moduli, né `app.main`, né `app.db_models`. **[CI]**
- Nuovo modulo: `public.py`, voce nel contratto import-linter di `pyproject.toml`, models in
  `app/db_models.py`. **[test]**
- I modelli ORM non escono dall'API: si convertono sempre in schemi di output (`_out(...)`).
  **[review]**
- La logica di dominio che cresce va in `service.py`, non nel router: il router traduce HTTP ↔
  servizio, il servizio non conosce `Request`. Il modulo di esempio `projects` è abbastanza
  piccolo da tenere le query nel router; `identity` ha già un `service.py`. **[review]**

---



**Regole**

Le regole d'uso (`tenant_db` negli endpoint di dominio, `system_session()` solo in `tenancy` e
`identity`, `tenant_id_column()` + `enable_tenant_rls()` nelle migrazioni, un test di isolamento
per ogni dato di un cliente) stanno in `AGENTS.md` e in `docs/constitution.md` (II). Qui come si
verificano:

- Ogni tabella con `tenant_id` ha la RLS attiva. **[test]** (`test_rls_coverage.py`)
- Una risorsa di un altro tenant risponde **404**, non 403: non si rivela che esiste. **[test]**
  (`test_tenant_isolation.py`)
- Gli endpoint di dominio usano `tenant_db` e non usano `system_session()`. **[review]**

---

## 4. Tipizzazione e stile

- **Type hints ovunque**, `mypy --strict` con il plugin Pydantic (`uv run mypy app`). **[CI]**
  I test e le migrazioni sono esclusi da mypy.
- Niente `Any` implicito. **[CI]**
- Sintassi moderna: `list[int]`, `X | None`. **[CI]** (regole `UP`)
- Date e ora **timezone-aware**: colonne `DateTime(timezone=True)`; mai `datetime.now()` o
  `utcnow()` senza fuso. **[CI]** (regole `DTZ`)
- Importi monetari con `Decimal` (mai `float`), serializzati come stringa o in centesimi.
  **[review]**
- Il codice delle dipendenze FastAPI usa `Depends(...)` negli argomenti (`B008` è ignorato apposta).

Ruff (`pyproject.toml`): `E F W I N UP B SIM S ASYNC DTZ PT RUF TID`. **[CI]**

---

## 5. Design delle API

### 5.1 Convenzioni

| Operazione | Metodo | Path | Successo |
|---|---|---|---|
| Lista | GET | `/projects` | 200 |
| Dettaglio | GET | `/projects/{project_id}` | 200 / 404 |
| Creazione | POST | `/projects` | 201 |
| Modifica parziale | PATCH | `/projects/{project_id}` | 200 |
| Cancellazione | DELETE | `/projects/{project_id}` | 204 |

- Risorse al plurale, `kebab-case` nei path. **[review]**
- **JSON in `snake_case`** (`created_at`, `tenant_name`): è la convenzione del progetto e il client
  TypeScript la riflette. L'unica eccezione sono i Problem Details (`requestId`, RFC 9457).
  **[review]**
- `/docs` e `/openapi.json` sono spenti in staging e produzione. **[test]**

### 5.2 Schemi Pydantic

- Input e output sono **modelli separati** (`ProjectIn`, `ProjectOut`): `id` e `created_at` non si
  inviano. **[review]**
- Gli input hanno `model_config = ConfigDict(extra="forbid")`: un campo sconosciuto è un 422, non un
  dato ignorato in silenzio. **[test]** per `ProjectIn`; per i nuovi input **[review]**.
- I vincoli stanno nello schema (`min_length`, `max_length`, `ge`, `le`): finiscono in OpenAPI.
  **[review]**
- Stati e tipi chiusi con `StrEnum`. **[review]**

### 5.3 Liste e paginazione

- Ogni lista è paginata **a cursore**, con un limite massimo imposto dal server. **[test]**
- Parametri `limit` (1–100, default 50) e `cursor`; risposta `{"items": [...], "next_cursor":
  null}`.
- L'helper è `app/shared/pagination.py` (keyset su `created_at` e `id` decrescenti): il cursore è
  opaco, uno malformato dà 400. Si interroga `limit + 1` righe per sapere se c'è un'altra pagina.
- Filtri e ordinamenti con parametri espliciti e una whitelist di campi. **[review]**







### 5.4 Contratto con il frontend

- `frontend/src/api/openapi.json` e `schema.d.ts` sono versionati e generati dal backend
  (`app/export_openapi.py`). Il frontend chiama il backend solo tramite
  `frontend/src/api/client.ts`. **[CI]**

---

## 6. Errori: un formato, sempre

Tutte le risposte d'errore sono **Problem Details** (RFC 9457, `application/problem+json`),
prodotte da `app/shared/errors.py`:

```json
{
  "type": "about:blank",
  "title": "Unauthorized",
  "status": 401,
  "detail": "Autenticazione richiesta",
  "instance": "/projects",
  "requestId": "3f2a…"
}
```

**Regole**
- Nel codice si solleva `HTTPException(status, "messaggio")`: l'handler lo converte in Problem
  Details (anche i 403 del middleware di sicurezza). Non costruire `JSONResponse` d'errore a mano:
  usa `problem(request, status, ...)`. **[test]**
- I 422 indicano il **campo** (`errors: [{field, message}]`) e non rimandano indietro il valore
  ricevuto, che potrebbe contenere dati personali. **[test]**
- I 5xx sono generici per il client («Errore interno»); il dettaglio sta nei log con lo stesso
  `requestId`. **[test]**
- `401` = non autenticato, `403` = autenticato ma senza permessi, `404` anche per risorse esistenti
  ma non accessibili. **[test]**
- Le risposte 401, 403, 422 e 500 sono dichiarate una volta su `FastAPI(responses=COMMON_RESPONSES)`
  e finiscono in OpenAPI con il modello `Problem`. Altre risposte (404, 409, …) si aggiungono con
  `responses=` sull'endpoint. **[review]**

---

## 7. Database

- **Sessioni sincrone** (`Session`, driver psycopg 3). Una richiesta = una sessione = una
  transazione, aperta da `tenant_db` (o `system_session` dove ammesso) e confermata a fine blocco.
  **[review]**
- Niente chiamate HTTP esterne dentro una transazione aperta. **[review]**
- Evita le query N+1: carica le relazioni necessarie (`selectinload` / `joinedload`). **[review]**
- Vincoli nel database (`NOT NULL`, `UNIQUE`, `FOREIGN KEY`, `CHECK`) e indici per le colonne
  usate in filtri e ordinamenti. **[review]**
- Mai SQL costruito con concatenazione di stringhe con input utente. **[CI]** (Bandit, regole `S`).
  L'unica eccezione, con soli identificatori costanti, è `app/shared/rls.py`.

### Migrazioni (Alembic)

Le regole (una migrazione per ogni modifica allo schema, mai modificare una migrazione già su
`develop` o `main`, compatibilità con il codice precedente) stanno in `AGENTS.md` e in
`docs/constitution.md` (VI). In più:

- Le migrazioni autogenerate si **leggono e correggono sempre** a mano. **[review]**
- Mai `create_all()` fuori dai test. **[review]**
- Le migrazioni si applicano su un database vuoto a ogni esecuzione dei test, e il controllo RLS
  verifica lo schema risultante. **[test]**

---

## 8. Sicurezza e configurazione

- **CORS** limitato alle origini in `ALLOWED_ORIGINS`, mai `*`. **[review]** In cloud le origini
  devono essere `https`. **[test]**
- **CSRF**: una richiesta che modifica dati e porta il cookie di sessione deve avere un `Origin`
  ammesso. **[test]**

- **Sessione**: cookie firmato, `HttpOnly`, con prefisso `__Host-` in cloud (Secure, Path=/, niente
  Domain; il nome è verificato da `test_staging.py`). 
  **[review]**
- **Header di sicurezza** su tutte le risposte (`nosniff`, `X-Frame-Options`, `Referrer-Policy`; in
  cloud anche CSP e HSTS); `Cache-Control: no-store` su `/auth/*`. Verificato solo `nosniff` e
  `X-Frame-Options`; il resto è **[review]**.
- **Segreti** solo da variabili d'ambiente: mai nel repo, mai nei log; `.env` mai committato, solo
  `.env.example`. **[review]**
- **Dipendenze**: `pip-audit` e Bandit in CI. **[CI]**
- Nessun dato personale nei log oltre il necessario (email, token): maschera o ometti. **[review]**
- Non disattivare un controllo di sicurezza per far funzionare qualcosa: se blocca, chiedi
  (`AGENTS.md`).

---

## 9. Osservabilità

- Ogni richiesta ha un **request id** (`app/shared/request_id.py`): si riusa `X-Request-ID` in
  arrivo se è valido (8–64 caratteri `A-Za-z0-9_-`), altrimenti si genera; è restituito nell'header
  `X-Request-ID` e nel `requestId` dei Problem Details. **[test]**
- Una riga di log per richiesta, con `request_id`, metodo, path, status e `duration_ms` (logger
  `app.request`). Gli errori non gestiti si registrano con `logging.exception` e lo stesso id.
  **[test]**
- Livelli: `ERROR` = richiede intervento, `WARNING` = anomalia gestita, `INFO` = eventi di business.
  **[review]**
- `/health` controlla il database (usato dall'healthcheck di Railway, senza il segreto
  Cloudflare). **[test]**

---

## 10. Job in background

- I job periodici stanno in `app/worker.py` (lista `JOBS`): stesso codice dell'API, processo
  separato. Un job fallito non ferma il worker (si registra con `log.exception`). **[review]**
- Le operazioni lente (oltre 1–2 secondi: export, invii massivi) non vanno nella richiesta: job nel
  worker e risposta `202` con un endpoint di stato. Quando servirà una coda vera si aggiunge qui,
  senza nuovi servizi finché non è necessario. **[review]**

---

## 11. Test

I test girano su un **Postgres vero**, in un database creato e cancellato apposta
(`tests/conftest.py`); mai SQLite. Comandi: `make test-backend`, o un singolo file/test come in
`CLAUDE.md`.


---

## 12. Comandi

I comandi di tutti i giorni (`make check`, `make test-backend`, `make api-types`,
`make migration`) sono in `AGENTS.md`; `make lint` esegue ruff, mypy, import-linter e Bandit.
Senza Docker, dalla cartella `backend/` (serve un Postgres raggiungibile tramite
`TEST_DATABASE_URL`):

```bash
uv run ruff format --check . && uv run ruff check . && uv run mypy app \
  && uv run lint-imports && uv run pytest
```

Le dipendenze di sviluppo (come mypy) vivono in un volume `.venv` del container: dopo aver cambiato
`pyproject.toml` rigenera l'ambiente con `docker compose up -d --build -V api worker`.

---



**Contratto e API**
- Prefisso di versione (`/v1`) o versionamento per header.
- `summary`, descrizione e `operationId` leggibili su ogni endpoint (`generate_unique_id_function`);
  header `Location` sulle creazioni (201).
- Rilevamento automatico delle rotture del contratto OpenAPI, oltre al controllo «client
  aggiornato».
- Idempotenza dei `POST` critici (`Idempotency-Key`) e concorrenza ottimistica (`version` o
  `ETag` / `If-Match` → 412), appena esiste un'operazione che li richiede (pagamenti, ordini).

**Affidabilità**
- Timeout espliciti sul database (`statement_timeout`, timeout di connessione) e su ogni nuovachiamata esterna .
- Pool di connessioni dimensionato in base ai worker: `worker × (pool_size + max_overflow)` non deve
  superare le connessioni consentite dal database (oggi si usano i valori predefiniti).
- Limite alla dimensione del body e degli upload, con verifica del tipo reale dei file.
- Rate limiting condiviso tra repliche (oggi in memoria per processo).

**Osservabilità**
- Log strutturati in JSON con campi fissi (oggi testo con `key=value`).
- `/healthz` (liveness) separato da `/readyz` (readiness): oggi `/health` fa entrambe le cose.
- OpenTelemetry per tracce e metriche; alert su 5xx e latenza p95.
- Obiettivi di latenza per gli endpoint critici (indicativi: p95 < 300 ms in lettura, < 800 ms in
  scrittura semplice).

**Qualità**
- Un `service.py` per ogni modulo di dominio con logica non banale (oggi l'esempio `projects` ha le
  query nel router).
- Pre-commit con ruff.
- Test automatico che ogni endpoint abbia i casi 401 e accesso ad altro tenant.

---




## 15. Standard di riferimento

| Area | Riferimento |
|---|---|
| Errori API | RFC 9457 Problem Details |
| Semantica HTTP | RFC 9110 |
| Contratto | OpenAPI Specification |
| Date e ora | RFC 3339 / ISO 8601 |
| Sicurezza | OWASP API Security Top 10, OWASP ASVS |
| Configurazione | The Twelve-Factor App |
| Privacy | GDPR (minimizzazione, retention, cancellazione) |
