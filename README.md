# Planner Bosi

Web app di pianificazione settimanale operatori/cantieri. Due profili:
- **Manager** (`docs/manager.html`) — CRUD operatori, cantieri, festività, impostazioni, calendario settimanale.
- **Viewer** (`docs/index.html`) — vista sola lettura a 2 giorni (Oggi + domani), per wall screen 40"/50".

Backend: Supabase (Postgres + Auth + Realtime). Frontend: HTML/CSS/JS puro (nessun build step), pubblicato con GitHub Pages.

---

## 1. Setup Supabase

1. Crea un nuovo progetto su [supabase.com](https://supabase.com).
2. Vai su **SQL Editor**, incolla ed esegui il contenuto di `supabase/schema.sql`.
3. Vai su **Authentication → Users → Add user** e crea 3 utenti (email + password a scelta):
   - `manager1@bosi.it`
   - `manager2@bosi.it`
   - `wallscreen@bosi.it` (utenza Viewer per il monitor)
4. Per ciascun utente creato, copia lo **User UID** (visibile nella lista utenti) ed esegui nel SQL Editor:
   ```sql
   insert into profiles (id, role, full_name) values
     ('UUID-MANAGER-1', 'manager', 'Manager 1'),
     ('UUID-MANAGER-2', 'manager', 'Manager 2'),
     ('UUID-VIEWER',    'viewer',  'Wall Screen');
   ```
5. Vai su **Project Settings → API** e copia:
   - `Project URL` → da incollare in `SUPABASE_URL`
   - `anon public` key → da incollare in `SUPABASE_ANON_KEY`

## 2. Configurazione frontend

Apri questi 3 file e sostituisci le due costanti in cima allo script con i valori copiati al punto 1.5:
- `docs/manager.html`
- `docs/index.html`
- `docs/reset-password.html`

```js
const SUPABASE_URL      = 'https://xxxxx.supabase.co';
const SUPABASE_ANON_KEY = 'eyJ...';
```

## 3. Logo

Metti il file del logo aziendale in `docs/logo.png` (stesso nome file usato come placeholder). Se il file manca, il logo semplicemente non viene mostrato (nessun errore bloccante).

## 4. Pubblicazione su GitHub Pages

1. Crea un repository su GitHub e carica tutto il contenuto di questa cartella.
2. Vai su **Settings → Pages** del repository.
3. In **Build and deployment**, seleziona **Deploy from a branch**.
4. Branch: `main`, cartella: **/docs**. Salva.
5. Dopo 1-2 minuti l'app sarà live su `https://<tuo-utente>.github.io/<nome-repo>/manager.html` e `.../index.html`.

> Nota: essendo un sito statico senza build, puoi anche caricare la cartella `docs` direttamente da interfaccia GitHub (drag & drop dei file) senza bisogno di Git da riga di comando.

## 5. Reset password

Nella pagina Manager, il link "Password dimenticata?" invia un'email di reset tramite Supabase Auth. Il link nell'email riporta l'utente su `reset-password.html`, dove può impostare la nuova password.

**Importante**: su Supabase, vai su **Authentication → URL Configuration** e aggiungi l'URL della tua pagina `reset-password.html` pubblicata (es. `https://<tuo-utente>.github.io/<nome-repo>/reset-password.html`) alla lista **Redirect URLs**, altrimenti il link di reset verrà rifiutato.

## 6. Struttura dati

Vedi `supabase/schema.sql` per lo schema completo. Tabelle principali:
- `squads` — operatori (nome, colore, attivo/disattivo, ordinamento)
- `holidays` — festività/chiusure (con opzione ricorrenza annuale)
- `tasks` — attività assegnate (operatore, data, titolo = cantiere/cliente, note)
- `settings` — riga singola con colore sfondo e dimensioni font

> Il cantiere/cliente **non** è una tabella a parte: è semplicemente il campo "Titolo" del task, scritto liberamente dal manager (es. "Installazione presso Pippo Pluto"). Non ci sono materiali/tag associati ai task.

Se avevi già eseguito lo schema originale su Supabase (con `sites` e `materials`), lancia una sola volta lo script `supabase/migrazione-rimuovi-cantieri-materiali.sql` nel SQL Editor per allinearlo.

Tutte le tabelle sono protette da Row Level Security: i Manager hanno accesso completo, il Viewer solo in lettura.

## 7. Cosa manca / da rifinire con l'uso reale

- Il logo va aggiunto manualmente (`docs/logo.png`).
- Le impostazioni (colore sfondo, font size) sono al momento **globali** per tutta l'app, condivise tra i due manager.
- La settimana manager copre Lunedì–Sabato (nessuna Domenica), come da riferimento grafico fornito.
- Consigliata la verifica su un monitor reale 40"/50" per tarare eventuali dimensioni testo via Impostazioni.
