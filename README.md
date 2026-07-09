# Planner Bosi

Web app di pianificazione settimanale operatori/cantieri. Due profili:
- **Manager** (`docs/manager.html`) — CRUD operatori, festività, note del giorno, fasce orarie, calendario settimanale. Login richiesto.
- **Viewer** (`docs/index.html`) — vista sola lettura a 2 giorni (Oggi + domani), per wall screen 40"/50". **Nessun login richiesto**: si apre direttamente con l'URL.

Backend: Supabase (Postgres + Auth + Realtime). Frontend: HTML/CSS/JS puro (nessun build step), pubblicato con GitHub Pages.

> ⚠️ **Nota di sicurezza**: il Viewer legge i dati senza autenticazione, usando la chiave pubblica del progetto. Chiunque conosca l'URL della pagina può vedere operatori, cantieri e note (in sola lettura, non può modificarli). Non condividere l'URL pubblicamente se i contenuti sono sensibili.

---

## 1. Setup Supabase

1. Crea un nuovo progetto su [supabase.com](https://supabase.com).
2. Vai su **SQL Editor**, incolla ed esegui il contenuto di `supabase/schema.sql`.
3. Vai su **Authentication → Users → Add user** e crea 2 utenti Manager (email + password a scelta):
   - `manager1@bosi.it`
   - `manager2@bosi.it`
   
   (Non serve più creare un'utenza per il Viewer.)
4. Per ciascun manager creato, copia lo **User UID** (visibile nella lista utenti) ed esegui nel SQL Editor:
   ```sql
   insert into profiles (id, role, full_name) values
     ('UUID-MANAGER-1', 'manager', 'Manager 1'),
     ('UUID-MANAGER-2', 'manager', 'Manager 2');
   ```
5. Vai su **Project Settings → API** e copia:
   - `Project URL` → da incollare in `SUPABASE_URL`
   - `anon public` key (o `Publishable key`) → da incollare in `SUPABASE_ANON_KEY`

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
4. Branch: `main`, cartella: **/ (root)** oppure **/docs** a seconda di dove hai caricato i file. Salva.
5. Dopo 1-2 minuti l'app sarà live su `https://<tuo-utente>.github.io/<nome-repo>/manager.html` e `.../index.html`.

## 5. Reset password (solo Manager)

Nella pagina Manager, il link "Password dimenticata?" invia un'email di reset tramite Supabase Auth. Il link nell'email riporta l'utente su `reset-password.html`, dove può impostare la nuova password.

**Importante**: su Supabase, vai su **Authentication → URL Configuration** e aggiungi l'URL della tua pagina `reset-password.html` pubblicata alla lista **Redirect URLs**, altrimenti il link di reset verrà rifiutato.

## 6. Struttura dati

Vedi `supabase/schema.sql` per lo schema completo. Tabelle principali:
- `squads` — operatori (nome, colore, attivo/disattivo, ordinamento)
- `holidays` — festività/chiusure (con opzione ricorrenza annuale)
- `tasks` — attività assegnate (operatore, data, titolo = cantiere/cliente, note, fascia oraria opzionale)
- `day_notes` — una nota libera per ciascuna data, visibile sia da Manager che da Viewer accanto al giorno
- `settings` — riga singola con colore sfondo, dimensioni font e le 4 fasce orarie configurabili

> Il cantiere/cliente **non** è una tabella a parte: è semplicemente il campo "Titolo" del task, scritto liberamente dal manager.

**Migrazioni disponibili** (da lanciare una sola volta nel SQL Editor, solo se il progetto Supabase è già esistente):
- `supabase/migrazione-rimuovi-cantieri-materiali.sql` — rimuove le vecchie tabelle cantieri/materiali.
- `supabase/migrazione-note-fasce-viewer-pubblico.sql` — aggiunge note del giorno, fasce orarie, e rende il Viewer accessibile senza login.

Se stai partendo da zero, `supabase/schema.sql` contiene già tutto: non serve lanciare le migrazioni.

Permessi (Row Level Security): la **lettura** è pubblica su tutte le tabelle applicative (squads, holidays, tasks, settings, day_notes) — necessaria per il Viewer senza login. La **scrittura** resta riservata ai soli utenti con ruolo Manager autenticato.

## 7. Fasce orarie

Le 4 fasce orarie (es. 07:00, 08:00, 13:00, 14:00) si configurano in **Impostazioni** lato Manager. Sono personalizzabili in qualsiasi momento; i task già creati mantengono l'orario assegnato anche se in seguito una fascia viene rinominata. Nel modulo di creazione/modifica task, il manager clicca sul pulsante corrispondente per assegnare (o rimuovere) la fascia oraria al task; l'orario compare come badge sia lato Manager sia lato Viewer.

## 8. Note del giorno

Accanto a ciascuna data, lato Manager compare un'icona ✎ per inserire/modificare una nota libera per quel giorno specifico (es. "consegna materiali ore 6"). La nota è sempre visibile in alto, accanto alla data, sia lato Manager sia lato Viewer.

## 9. Cosa manca / da rifinire con l'uso reale

- Il logo va aggiunto manualmente (`docs/logo.png`).
- Le impostazioni (colore sfondo, font size, fasce orarie) sono al momento **globali** per tutta l'app, condivise tra i due manager.
- La settimana manager copre Lunedì–Sabato (nessuna Domenica), come da riferimento grafico fornito.
- Consigliata la verifica su un monitor reale 40"/50" per tarare eventuali dimensioni testo via Impostazioni.
