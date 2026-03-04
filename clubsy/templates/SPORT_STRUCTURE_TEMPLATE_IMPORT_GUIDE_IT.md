# Import della struttura sportiva tramite modello CSV

## Obiettivo
Consentire il caricamento massivo di **categorie**, **leghe** e **squadre** nella stagione attiva del club usando un modello CSV.

Questa guida documenta il **comportamento attualmente implementato** nella schermata della struttura sportiva.

## Dove viene usato
- Schermata: `SportStructureScreen`
- Pulsante: **Importa modello CSV**
- Requisito: utente con permessi di gestione del club e modalità modifica attiva.

## File di riferimento
- Modello base: `docs/templates/sport_structure_import_template.csv`
- Esempio valido: `docs/templates/sport_structure_import_example_valid_en.csv`
- Esempio non valido: `docs/templates/sport_structure_import_example_invalid_en.csv`

## Formato CSV richiesto
Intestazioni richieste (ordine consigliato):

```csv
entity,name,category,league
```

### Colonne
- `entity`: tipo di riga. Valori consentiti:
  - `category`
  - `league`
  - `team`
- `name`: nome principale dell’entità.
- `category`: obbligatorio solo quando `entity=team`.
- `league`: obbligatorio solo quando `entity=team`.

## Regole di validazione
La validazione viene eseguita prima degli inserimenti:

1. Il file deve contenere intestazione + almeno una riga dati.
2. Le colonne `entity` e `name` devono esistere.
3. Se `entity=team`, `category` e `league` sono obbligatori.
4. Qualsiasi valore `entity` diverso da `category|league|team` è invalido.
5. Per le squadre, categoria/lega referenziata deve esistere:
   - già nel DB della stagione attiva, oppure
   - creata nello stesso CSV tramite righe `category`/`league`.

Se ci sono errori di validazione, **l’import non viene eseguito** e viene mostrato un riepilogo errori.

## Comportamento di inserimento
- Nuove categorie: inserite se assenti nella stagione attiva (confronto nome normalizzato).
- Nuove leghe: stesso comportamento.
- Nuove squadre: inserite se la tupla `name+category+league` non esiste.
- I dati esistenti non vengono eliminati.
- I nomi esistenti non vengono aggiornati (comportamento orientato all’aggiunta).

## Normalizzazione del testo
Prima di confronto/inserimento:
- viene applicato `trim`
- gli spazi multipli vengono ridotti a uno

Esempio:
- `"  Senior   A  "` → `"Senior A"`

## Esempio valido
```csv
entity,name,category,league
category,U18,,
category,U16,,
league,Premier,,
league,Regional,,
team,U18 A,U18,Premier
team,U18 B,U18,Regional
team,U16 A,U16,Regional
```

Risultato atteso:
- Categorie create: 2 (se mancanti)
- Leghe create: 2 (se mancanti)
- Squadre create: 3 (se mancanti)

## Esempio non valido
```csv
entity,name,category,league
team,Squadra senza lega,U18,
foo,Tipo riga sconosciuto,,
team,,U18,Premier
team,Squadra con riferimento mancante,U14,Premier
```

Errori attesi:
- riga `team` senza `league`;
- `entity` non valido (`foo`);
- `name` vuoto;
- riferimento a categoria inesistente (`U14`).

## Flusso operativo consigliato
1. Scarica il modello base.
2. Compila prima categorie e leghe.
3. Aggiungi le squadre usando i nomi esatti di categoria/lega.
4. Importa in ambiente di test.
5. Controlla il riepilogo creazioni.
6. Ripeti in produzione.

## Buone pratiche
- Mantieni nomi coerenti (es. `U18 A`, `U18 B`).
- Evita varianti tipografiche per la stessa entità.
- Importa per blocchi (prima struttura base, poi estensioni incrementali).

## Ambito attuale
Questo import attualmente copre:
- categorie
- leghe
- squadre

Non ancora incluso:
- assegnazione massiva membri
- caricamento massivo di `player_profile`
- aggiornamento/eliminazione massiva

## Risoluzione problemi
### "CSV requires columns: entity,name,category,league"
L’intestazione è errata. Verificare la prima riga.

### "Template has no data"
Il file contiene solo intestazione oppure è vuoto.

### "team requires category and league"
La riga squadra è incompleta.

### "category/league does not exist"
Aggiungi righe di creazione nello stesso CSV o crea prima tali record nell’app.
