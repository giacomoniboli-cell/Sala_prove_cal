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

La chiave `anon public` e pensata per stare in una pagina pubblica. Le regole RLS dello script limitano l'accesso alla tabella del calendario, ma chiunque abbia il link potra modificare le presenze.

## Funzionamento

- La pagina mostra i giovedi futuri dell'anno selezionato, includendo oggi se oggi e giovedi.
- Ogni modifica viene salvata nel browser.
- Se Supabase e configurato, ogni modifica viene salvata anche online.
- Il pulsante `Ricarica online` recupera l'ultima situazione condivisa.
- La colonna `Disponibili` mostra il riepilogo complessivo di chi ha indicato `Presente` per ogni giovedi.
- `Esporta CSV` produce un file apribile con Excel.
