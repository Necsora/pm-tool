# Linee guida del frontend

Come si scrive l'app (`frontend/`): una SPA React 19 + Vite + TypeScript, su Cloudflare Workers,
che parla con il backend tramite un client generato dallo schema OpenAPI. Le versioni stanno in
`frontend/package.json` e nel lockfile; il quadro generale è in `docs/architecture.md`.

Ogni regola ha un'etichetta che dice **chi la fa rispettare**:

- **[test]**: la verifica un test (`npm --prefix frontend test`, quindi `make check` e la CI);
- **[CI]**: la verifica uno strumento (`tsc`, oxlint, build, `make api-contract`);
- **[review]**: oggi si controlla solo in revisione;
- **[obiettivo]**: non è ancora applicata (elenco in fondo). Non darla per scontata.

Le regole generali del frontend (chiamate al backend solo da `src/api/client.ts`, niente script o
stili inline) stanno in `AGENTS.md` e in `docs/constitution.md`: qui non si ripetono.

---

## 1. Principi di UX

1. **Ogni azione ha un feedback entro 100 ms**: pulsante disabilitato, stato pressed, update
   ottimistico. **[review]**
2. **Ogni schermata con dati ha quattro stati**: *loading*, *empty*, *error*, *success* (§7).
   Se ne manca uno, la feature non è finita. **[review]**
3. **Non perdere mai il lavoro dell'utente.** Un errore di rete non svuota un form. **[review]**
4. **Gli errori dicono cosa è successo e cosa fare**, con un'azione (*Riprova*). **[review]**
5. **Niente salti di layout** (CLS < 0.1): riserva lo spazio a ciò che arriva in modo asincrono.
   **[review]**
6. **La tastiera è di prima classe**, con focus sempre visibile (§9). **[review]**
7. **L'URL è stato**: filtri, tab e paginazione sopravvivono a un ricarico. **[obiettivo]** (oggi
   la sezione attiva sta nello stato del componente e un ricarico torna alla «Dashboard»).
8. **Le azioni distruttive sono reversibili o confermate.** **[review]**
9. **Coerenza prima della creatività**: stesso componente, stesso comportamento. **[review]**




**Regole**
- `openapi.json` e `schema.d.ts` non si modificano a mano: `make api-types`. **[CI]**
  (`make api-contract` fallisce se sono vecchi.)
- I test stanno accanto al codice che testano. **[review]**
- Quando l'app cresce, si organizza **per feature** (`src/features/<nome>/` con componenti, hook e
  test) con i componenti comuni in `src/components/`; una feature non importa i file interni di
  un'altra. **[obiettivo]**
- Un componente fa una cosa: oltre ~150 righe chiediti cosa estrarre. **[review]** (`App.tsx` è
  ancora piccolo, ma `Workspace` e `ClerkLogin` sono i primi candidati a un file proprio.)

---

## 3. TypeScript

- `strict` è attivo (è il default di TypeScript 6), più `noUnusedLocals`, `noUnusedParameters`,
  `verbatimModuleSyntax` ed `erasableSyntaxOnly` (niente `enum` e `namespace`). **[CI]**
  (`npm run typecheck`, parte di `npm run build`.)
- Vietato `any`: usa `unknown` e restringi. **[review]** Niente `as` per zittire il compilatore:
  se serve, c'è un problema di modellazione.
- Gli stati si modellano con **union discriminate**, non con booleani multipli. `AuthState` in
  `App.tsx` è l'esempio da imitare:

```ts
type AuthState = { status: 'loading' } | { status: 'anonymous' } | { status: 'signed-in'; me: Me }
```

- I tipi di dominio derivano dallo schema OpenAPI (`components['schemas'][...]`), non si riscrivono
  a mano. **[review]**
- `import type` per i soli tipi. **[CI]** (`verbatimModuleSyntax`)
- `noUncheckedIndexedAccess` e `exactOptionalPropertyTypes` non sono attivi. **[obiettivo]**

---

## 4. Il client API

`src/api/client.ts` crea **un solo** client con `openapi-fetch`:

- `baseUrl` è `VITE_API_URL`, oppure `/api` in locale (il proxy di Vite lo gira a FastAPI: stessa
  origine per il browser, quindi niente CORS);
- `credentials: 'include'` invia il cookie di sessione `HttpOnly`: **non esistono token in
  `localStorage` né header `Authorization`** verso il nostro backend. L'unico `Bearer` è il token di
  Clerk, mandato a `POST /auth/session` (ADR-005) e, dopo la verifica di una nuova email, a
  `POST /auth/me/email`: il backend legge l'email da quel token, mai dal corpo. **[review]**

### 4.1 Errori

Il backend risponde sempre con **Problem Details** (`application/problem+json`, vedi
`docs/backend-guidelines.md` §6): `title`, `status`, `detail`, `requestId` e, per i 422,
`errors: [{field, message}]`. Il tipo `Problem` è nello schema generato.

- La UI decide il messaggio dallo **status**, non dal testo del backend; non mostra mai `detail`
  tecnici o stack trace. `ClerkLogin` in `App.tsx` lo fa così per 400, 409 e gli altri. **[review]**
- Gli errori 422 si mappano sui campi del form tramite `errors[].field`. **[obiettivo]** (oggi non
  c'è ancora un form con validazione lato server.)
- In un errore 5xx mostra `requestId` o lo include nel messaggio di supporto, così si trova nei
  log. **[obiettivo]**
- Un tipo `AppError` unico (`network`, `unauthorized`, `forbidden`, `notFound`, `validation`,
  `server`) per decidere i messaggi in un solo punto, appena gli errori da gestire sono più di
  due o tre. **[obiettivo]**


### 4.2 Liste

Le liste del backend sono paginate a cursore: `GET /projects?limit=&cursor=` risponde
`{ items, next_cursor }`. Il frontend accumula gli `items` e mostra «Mostra altri» finché
`next_cursor` non è `null` (`Workspace` in `App.tsx`). Un elenco che crescerà molto va
virtualizzato (§10). **[review]**

### 4.3 Richieste e race condition

- Ogni effect che avvia una richiesta ha un cleanup: oggi con un flag `active` che scarta la
  risposta se il componente è stato smontato; per le richieste che dipendono da parametri, passa un
  `AbortSignal` (openapi-fetch lo accetta) in modo che "vinca l'ultima risposta". **[review]**

---

## 5. Stato e data fetching

| Tipo di stato | Dove |
|---|---|
| Dati dal server | Stato del componente che li mostra, aggiornato con una nuova richiesta dopo una scrittura (oggi `version` che si incrementa); mai copiati in più posti |
| Stato della vista condivisibile (filtri, pagina, tab) | URL (`searchParams`) **[obiettivo]** |
| Stato UI locale | `useState` nel componente |
| Preferenze della persona (lingua, tema) | Context + `localStorage` (mai token o dati) |
| Stato globale reale (utente) | Context con valore stabile, pochi provider |
| Valori derivati | Calcolati in render, **non** in `useEffect` + `useState` |

- `useEffect` serve a sincronizzarsi con sistemi esterni (rete, DOM, timer). Una logica scatenata
  da un click va nell'handler, non in un effect che osserva lo stato. **[review]**
- Le regole degli hook sono controllate da oxlint (`react/rules-of-hooks`). **[CI]**
- Quando i dati serviranno in più viste, valuta una libreria di cache (TanStack Query o simili)
  prima di scrivere a mano invalidazione e deduplicazione. **[obiettivo]**
- Ricerca server-side: debounce di 250–400 ms, `useDeferredValue` o `useTransition`, risultati
  precedenti attenuati durante il caricamento. **[review]**

---

## 6. React 19

- **Form**: preferisci le Actions (`<form action>`, `useActionState`, `useFormStatus`). Dopo
  un'action riuscita React svuota i campi non controllati: in caso di errore restituisci i valori
  inseriti nello stato, così l'utente non perde nulla. Oggi `Workspace` usa un form con `onSubmit`
  e stato controllato, che va bene per un campo solo. **[review]**
- **Aggiornamenti ottimistici** (`useOptimistic`) per azioni ad alta probabilità di successo; gli
  elementi in sospeso sono distinguibili e, se il server fallisce, lo stato torna indietro con un
  messaggio e la possibilità di riprovare. **[review]**
- **Testi**: ogni testo visibile sta in `src/i18n.ts` (italiano e inglese, stessa chiave) e si legge con
  `useI18n()`; niente stringhe nei componenti. Restano fuori, per ora, le schermate prima del login (i
  componenti di Clerk hanno la loro localizzazione). **[review]**
- `ref` si passa come prop: niente `forwardRef` nei componenti nuovi.
- `<title>` e `<meta>` dentro il componente vanno nell'`<head>`: ogni vista ha un titolo
  significativo. **[obiettivo]**
- `use()` legge una Promise dentro `<Suspense>`: la Promise va creata **fuori** dal render. Avvia
  le richieste il prima possibile e in parallelo, per evitare i *waterfall*. **[review]**
- Niente `useMemo`/`useCallback` sparsi: solo dove misuri un problema o serve stabilità
  referenziale (come `refresh` in `Session`, che è una dipendenza di un effect).
- `key` stabili dai dati (mai l'indice in liste riordinabili). **[review]**
- Ogni vista andrebbe avvolta in un **Error Boundary** con un fallback utile e il log
  dell'errore. **[obiettivo]**

---

## 7. Gli stati della UI

**Loading**
- Sotto ~300 ms non mostrare nulla (niente flash di spinner); tra 300 ms e 2 s uno skeleton della
  forma del contenuto; oltre 2 s skeleton più un messaggio di avanzamento. Gli spinner vanno bene
  dentro i pulsanti, non per intere pagine. **[review]** (Oggi c'è solo il testo «Caricamento…».)

**Empty**
- Spiega *perché* è vuoto e *cosa fare*; distingui «vuoto perché nuovo» da «vuoto per i filtri».
  **[review]** (Esempio: «Nessuna segnalazione aperta.»)

**Error**
- Messaggio umano più un'azione (Riprova, Torna indietro); un errore locale non abbatte la pagina.
  **[review]** Un errore di rete oggi non ha una gestione: `Workspace` ignora il caso `data`
  assente. **[obiettivo]**

**Success**
- Conferma breve e non bloccante (toast di 4–6 s, con *Annulla* se applicabile). **[obiettivo]**

---

## 8. Form

- **Etichetta visibile** per ogni campo (il placeholder non è un'etichetta; `aria-label` solo dove
  l'etichetta visibile non c'è, come nel campo di domanda a Valeria). **[review]**
- Tipi e attributi giusti: `type="email"`, `inputMode`, `autoComplete`, e i limiti del backend
  (`maxLength={500}` come `AskIn.question`). **[review]**
- Validazione al blur, non a ogni tasto; dopo il primo errore, ricontrolla mentre si corregge.
- Errore accanto al campo, collegato con `aria-describedby`, campo con `aria-invalid`; al submit
  fallito il focus va al primo campo in errore.
- Pulsante di invio disabilitato durante l'invio: niente doppi invii.
- Mantieni i valori inseriti in caso di errore del server. Sempre.

---

## 9. Accessibilità (WCAG 2.2 AA come minimo)

- **HTML semantico prima di ARIA**: `<button>` per le azioni, `<a href>` per la navigazione,
  `<main>`, `<header>`, titoli in ordine, un solo `<h1>` per vista. **[review]**
- **Focus visibile** sempre (`:focus-visible`); mai `outline: none` senza alternativa. Al cambio di
  vista il focus va al titolo o al `<main>`. Le modali intrappolano il focus e lo restituiscono
  (`<dialog>` nativo dove possibile); `Esc` le chiude. **[review]**
- Aggiornamenti asincroni rilevanti in una regione `aria-live="polite"`, gli errori urgenti con
  `role="alert"` (come in `ClerkLogin`). **[review]**
- Contrasto 4.5:1 (testo) e 3:1 (testo grande e componenti), **anche nel tema scuro**. **[review]**
- Target di tocco almeno 24×24 px, meglio 44×44 su mobile. Il colore non è mai l'unico veicolo
  d'informazione. `alt` descrittivo sulle immagini, `alt=""` se decorative.
- Rispetta `prefers-reduced-motion` e `prefers-color-scheme` (quest'ultimo è già gestito in
  `index.css`). **[review]**
- Regole di accessibilità nel lint (`jsx-a11y`). **[obiettivo]**

---

## 10. Performance

| Metrica (75° percentile utenti reali) | Obiettivo |
|---|---|
| LCP | < 2.5 s |
| INP | < 200 ms |
| CLS | < 0.1 |
| JS iniziale (gzip) | budget di progetto; oggi ~97 KB |

Le soglie sono obiettivi, non controlli: oggi nessuno strumento le misura. **[obiettivo]**

- Code splitting per vista con `lazy()` + `<Suspense>` quando ci saranno più viste. **[obiettivo]**
- Importa solo ciò che usi e controlla la dimensione del bundle a ogni release. **[review]**
- Immagini con `width`/`height`, formati moderni, `loading="lazy"` sotto la piega. Font: pochi
  pesi e `font-display: swap`; nessun font o risorsa da domini esterni (la CSP lo vieta).
- Liste molto lunghe: paginazione (già a cursore) o virtualizzazione. **[review]**
- Variabili d'ambiente al client solo con prefisso `VITE_`, e **mai segreti**: finiscono nel
  bundle. Oggi sono pubbliche per natura: `VITE_API_URL` e la chiave pubblicabile di Clerk.
  **[review]**

---

## 11. Visual design

- Colori, tema chiaro/scuro e font vivono come variabili CSS in `src/index.css`
  (`--bg`, `--surface`, `--text`, `--muted`, `--border`, `--accent`, `--danger`): niente colori o
  valori «magici» nei componenti. **[review]**
- Quando servono spaziature, raggi e durate ricorrenti, si aggiungono come variabili nello stesso
  posto (scala fissa, multipli di 4 px). **[obiettivo]**
- Gerarchia tipografica chiara (4–5 dimensioni); righe di 60–80 caratteri per i testi lunghi.
- **Mobile first**: da 320 px in su, niente scroll orizzontale. **[review]**
- Un'azione primaria per schermata, visivamente evidente.
- Animazioni di 150–300 ms, solo `transform` e `opacity`, che comunicano qualcosa; con
  `prefers-reduced-motion: reduce` si riducono o spengono. **[review]**
- Niente stili inline: la CSP li vieta (unica deroga per Clerk, vedi §14). **[review]**

---

## 12. Test

Vitest + Testing Library su jsdom. Comando: `npm --prefix frontend test` (un file:
`npm --prefix frontend test -- src/App.test.tsx`). I test sono `src/App.test.tsx` e
`scripts/finalize-headers.test.mjs`.

**Regole**
- **Testa ciò che l'utente vede e fa**, non l'implementazione. Query nell'ordine `getByRole` →
  `getByLabelText` → `getByText` → (ultima spiaggia) `getByTestId`. **[review]**
- Per l'asincrono usa `findBy…` / `waitFor`, mai `setTimeout`. **[review]**
- **Mock al confine di rete.** Il client accetta una `fetch` risolta a ogni chiamata
  (`globalThis.fetch`), quindi i test sostituiscono `fetch` con `vi.stubGlobal` e non mockano mai
  i moduli interni:

```ts
vi.stubGlobal('fetch', vi.fn(async (req: Request) =>
  json(req.url.endsWith('/auth/me') ? me : { items: [], next_cursor: null }),
))
```

- Il provider esterno si finge: `@clerk/clerk-react` è mockato in `App.test.tsx`, e la chiave
  pubblicabile è finta e costruita al volo (nessuna chiave vera nel repository). **[review]**
- I test non leggono il `.env.local` di chi li lancia: `vite.config.ts` fissa le variabili.
- Ogni feature ha almeno il percorso felice, lo stato di errore e lo stato vuoto; ogni bug corretto
  porta un test che lo riproduce. **[review]**
- Per interazioni realistiche valuta `@testing-library/user-event` (non è ancora tra le
  dipendenze; oggi si usa `fireEvent`). **[obiettivo]**

---

## 13. Lint e tipi

- `npm run lint` (oxlint) e `npm run typecheck` (`tsc -b`) girano in `make lint` e in CI. **[CI]**
- La configurazione (`.oxlintrc.json`) oggi attiva i plugin `react`, `typescript` e `oxc` e due
  regole esplicite: `react/rules-of-hooks` (errore) e `react/only-export-components` (avviso).
- I file generati (`src/api/schema.d.ts`) non si modificano a mano e non si correggono per il lint.
- Da aggiungere: `exhaustive-deps` come errore, `jsx-a11y`, `--deny-warnings`, niente `console.log`
  in produzione. **[obiettivo]**

---

## 14. Sicurezza, build e deploy

- **Header e CSP** stanno in `public/_headers` con dei segnaposto; `scripts/finalize-headers.mjs`
  li completa in build (origine del backend da `VITE_API_URL`; con la chiave di Clerk aggiunge ciò
  che serve ai suoi componenti, compreso `'unsafe-inline'` per i soli **stili**; con
  `DEPLOY_ENV=staging` aggiunge `X-Robots-Tag: noindex`). Gli script non hanno mai `'unsafe-inline'`
  né `'unsafe-eval'`. **[test]** (`finalize-headers.test.mjs`)
- Non modificare i segnaposto a mano. Ammettere un nuovo dominio nella CSP è un controllo di
  sicurezza: **chiedi prima** (`AGENTS.md`).
- `vite.config.ts` imposta `assetsInlineLimit: 0` perché niente finisca inline in `index.html`.
  **[review]**
- **Cloudflare Workers** (`wrangler.jsonc`): gli asset di `dist/` con `not_found_handling:
  "single-page-application"`, quindi ricaricare una rotta client non dà 404. Due ambienti, staging
  e produzione, pubblicati dal workflow Deploy dopo la CI; il deploy a mano è vietato.
- **Cache**: `/assets/*` è immutabile (nome con hash); `index.html` non si cachea a lungo.
  **[review]** (è in `public/_headers`)

