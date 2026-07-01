# Codebase Reading Notes - L09

## Comportamento Osservato

- Operatore corrente: `operators.js:1` esporta `currentOperatorId = "op-001` e un array di 1 solo operatore. `server/index.js:19-25` fa `array.find()` su quello — nessun login.
- Lista ticket: Hardoded in `server\data\tickets.js`. 3 ticket fixture con `status: "open"`. Il server filtra per `status === "open"` (`server/index.js:33`). `responseDueAt` e `urgencyLabel` assenti dai dati (non calcolati).
- Create ticket: `\server\index.js` ritorna un errore 501 quando si prova a creare un ticket `POST /api/tickets → 501 (server/index.js:44-50)`. E' possibile provare a creare Tickt vuoti. Form in `CreateTicketPanel.jsx:4-9` raccoglie solo `{title, description, requesterEmail, priority}` — nessun campo  `area` o  `customerName`. (Un operatore crea un ticket dopo aver ricevuto una richiesta da uno dei canali elencati in `sourceChannel`)

## File Chiave

| File | Perche' e' importante | Evidenza |
| --- | --- | --- |
| prisma\schema.prisma | Definisce lo schema dei dati | Campi  "volutamente non ancora gestiti", SQLite `dev.db` non presente |
| server\data\operators.js | Contiene la lista degli operatori | `operators.js:1-10` — nessun login, nessuna sessione, un solo operatore hardcoded |
| server\data\tickets.js | Lista dei ticket e array dei campi allowed per priorità/aree/canali | `allowedPriorities`, `allowedAreas`, `allowedSourceChannels` mai utilizzati per popolare il form. `responseDueAt`/`urgencyLabel` mai calcolati |
| src\components\CreateTicketPanel.jsx | Form create: raccoglie `{title, description, requesterEmail, priority}`. Mancano `area` e `sourceChannel`. | `CreateTicketPanel.jsx:46` La UI stessa dice "Area, canale e regola SLA restano il gap" (SLA: Service Level Agreement/Accordo sul Livello di Servizio) |
| src\components\TicketCard.jsx | Card per ogni ticket: mostra area, canale, e "non ancora calcolato" per campo derivato (`responseDueAt`) | TicketCard.jsx:35-37 — `<dd>non ancora calcolato</dd>` |
| server\index.js | Routing API: `GET /api/me`, `GET /api/tickets`, `GET /api/ticket-options`, `POST /api/tickets → 501` | `index.js:44-50` — 501 con messaggio esplicito |
| src\api.js | Funzioni fetch React: `fetchCurrentOperator()`, `fetchOpenTickets()`, `fetchTicketOptions()`, `createTicket()` | |
| src\runtime-app.js | Duplica tutta la logica di `api.js` + componenti in vanilla JS (nessun React/build) per il runtime servito da Express | Funzione `renderShell()`, `renderCreatePanel()`, `renderTicketCard()` ecc. |

## Gap Osservati

| Area | Gap | Evidenza |
| --- | --- | --- |
| Creazione del Ticket | `POST /api/tickets` ritorna 501; form non ha `area` né `sourceChannel`; nessuna validazione | server/index.js:44-50 (501); `CreateTicketPanel.jsx` (Form senza `area`/`sourceChannel`); nessun required o validazione payload |
| Lista dei Ticket | `responseDueAt` e `urgencyLabel` non calcolati; la card mostra "non ancora calcolato" | `prisma/schema.prisma:35-36` (campi `?`); `TicketCard.jsx:35-37` (`ticket-card__missing`); |
| Login | Operatore hardcoded, nessun auth reale, nessuna sessione | `operators.js` (un solo operatore); `server/index.js:19-25` (nessuna logica di autenticazione, solo rotta per mostrare l'operatore corrente) |

## Domanda Per L10

- Come verrà calcolato il campo derivato `responseDueAt`? Il README dice priority + area → responseDueAt ma non specifica la logica.
- Come verrà usato `urgencyLabel`?
- `area` e `sourceChannel` vanno resi campi obbligatori o possono restare opzionali come nello schema Prisma (`area?`, `sourceChannel?`)?