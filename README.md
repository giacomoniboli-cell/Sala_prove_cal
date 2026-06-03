# Calendario presenze sala prove

Pagina HTML statica pubblicabile su GitHub Pages per segnare le presenze nei giovedi dell'anno.

## File

- `calendario_presenze_giovedi.html`: pagina pronta da pubblicare.

## Pubblicazione su GitHub Pages

1. Caricare il file in un repository GitHub.
2. In GitHub aprire `Settings` > `Pages`.
3. Selezionare il branch principale e la cartella `/root`.
4. Aprire l'indirizzo GitHub Pages generato.

## Salvataggio condiviso online

GitHub Pages non salva dati direttamente: serve un archivio esterno. La pagina e gia predisposta per Supabase.

### 1. Creare progetto Supabase

1. Aprire <https://supabase.com>.
2. Creare un nuovo progetto.
3. Aprire `SQL Editor`.
4. Eseguire questo script:

```sql
create table if not exists public.attendance_state (
  id text primary key,
  data jsonb not null default '{}'::jsonb,
  updated_at timestamptz not null default now()
);

alter table public.attendance_state enable row level security;

create policy "public read attendance"
on public.attendance_state
for select
to anon
using (id like 'sala-prove-%');

create policy "public insert attendance"
on public.attendance_state
for insert
to anon
with check (id like 'sala-prove-%');

create policy "public update attendance"
on public.attendance_state
for update
to anon
using (id like 'sala-prove-%')
with check (id like 'sala-prove-%');
```

### 2. Configurare la pagina

In Supabase aprire `Project Settings` > `API` e copiare:

- `Project URL`
- `anon public` key

Nel file `calendario_presenze_giovedi.html`, compilare:

```js
const CLOUD_CONFIG = {
  supabaseUrl: 'INCOLLARE_PROJECT_URL',
  supabaseAnonKey: 'INCOLLARE_ANON_PUBLIC_KEY',
  tableName: 'attendance_state'
};
```

La chiave `anon public` e pensata per stare in una pagina pubblica. Le regole RLS dello script limitano l'accesso alla tabella del calendario, ma la configurazione attuale consente a chiunque abbia il link di modificare i dati del calendario.

## Sicurezza modifiche

Configurazione attuale:

- lettura pubblica tramite chiave `anon public`;
- inserimento e aggiornamento pubblici sulla sola riga `sala-prove-*`;
- nessun login e nessuna password;
- chiunque abbia il link puo modificare presenze, band, note e giorni visibili.

Questa impostazione e comoda per un gruppo piccolo e informale, ma non protegge da modifiche indesiderate se il link viene condiviso troppo.

Evoluzione consigliata, se servira proteggere le modifiche senza introdurre utenti e password:

- lasciare pubblica la lettura;
- bloccare `insert` e `update` diretti al ruolo `anon`;
- salvare le modifiche tramite una Supabase Edge Function;
- proteggere la funzione con un codice condiviso salvato nei secret Supabase.

In questo modo il calendario resta facile da consultare, ma solo chi conosce il codice puo modificarlo.

## Funzionamento

- La pagina mostra i giovedi futuri dell'anno selezionato, includendo oggi se oggi e giovedi.
- Si possono aggiungere anche giorni extra tramite calendario.
- I giorni possono essere nascosti dalla visualizzazione e ripristinati dal pannello `Giorni extra`.
- Per ogni giorno si puo indicare la band che provera, lasciando vuoto se non ancora definita.
- L'elenco band include opzioni predefinite e puo essere ampliato dalla pagina.
- In alto sono mostrate le prossime 5 prove con la band selezionata per ciascuna data.
- Le persone possono essere nascoste dalla vista principale senza eliminarle.
- Ogni modifica viene salvata nel browser.
- Se Supabase e configurato, ogni modifica viene salvata anche online.
- Il pulsante `Ricarica online` recupera l'ultima situazione condivisa.
- Ogni scheda giorno mostra il riepilogo complessivo di chi ha indicato `Presente`.
- `Esporta CSV` produce un file apribile con Excel.

## Supabase

Non servono nuove tabelle per giorni extra, giorni nascosti, persone nascoste o band: questi dati sono salvati nel campo JSON `data` della tabella `attendance_state`.
