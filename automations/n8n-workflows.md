# n8n Workflows Handoff

## Responsabilita
Questo repository e la source of truth per strategia e decisioni:

- posizionamento professionale;
- role family target;
- query seed;
- osservatorio di mercato;
- scoring;
- criteri di calibrazione.

Il repository `skunklabs-uk/n8n-workflows` e invece la source of truth tecnica per:

- workflow n8n importabili in JSON;
- credenziali da collegare nella UI di n8n;
- parametri operativi per ambiente e schedulazione;
- note di import, attivazione e manutenzione;
- eventuali wrapper o flussi di orchestrazione necessari all'esecuzione.

Regola di confine:

- qui si decide cosa osservare, come classificarlo e come valutarlo;
- in `n8n-workflows` si implementa l'esecuzione operativa;
- i workflow non devono diventare il posto dove si ragiona sul posizionamento o sulle regole di mercato.

## Workflow Previsti

- Email alert ingestion: ingestione di alert da mailbox o inoltri di job alert, con estrazione di titolo, azienda, localita, URL e testo utile.
- Public source ingestion: ingestione da feed pubblici, API aperte o fonti accessibili senza login, con priorita a sorgenti stabili e link canonici.
- Normalization: conversione dei record grezzi nello schema normalizzato definito in `job-search/market-observatory-spec.md`.
- Exclusion gate: esclusione dei record fuori perimetro prima della classificazione, con `outOfScope`, `exclusionReason` ed `exclusionSignals`.
- Classification: assegnazione di `roleFamily`, `seniority` e `remotePolicy` solo ai record non esclusi, in modo coerente con `profile/target-roles.md`.
- Scoring: calcolo dei punteggi definiti in `job-search/scoring-model.md`, inclusa la variante Italia quando applicabile.
- Report generation: produzione di digest, code di priorita, viste per family e note di calibrazione per le query.
- Manual review capture: raccolta del feedback umano su candidature, risposte recruiter, scarti e osservazioni utili alla calibrazione.

## Pattern n8n

- Separare i workflow per fonte da quelli di trasformazione comune: ogni sorgente ha un ingresso proprio, ma normalizzazione, classificazione e scoring devono convergere su una catena condivisa.
- Tenere i nodi di parsing piccoli e mirati; se una sorgente cambia formato, si aggiorna solo il ramo interessato.
- Usare nodi di controllo espliciti per distinguere record validi, record incompleti e record da revisione manuale.
- Preferire sub-workflow o workflow richiamati quando la stessa logica si ripete tra inbox, feed pubblici e fonti italiane.
- Mantenere la logica di business nei documenti di questo repo e nei mapping configurabili, non in espressioni sparse o hardcoded.
- Trattare la persistenza operativa come dettaglio tecnico del repo `n8n-workflows`, non come parte della strategia.
- Per gli alert email, conservare la history operativa delle query in una Data Table n8n ispezionabile e resettabile, non in stato volatile nascosto nel workflow.

## Input

- `job-search/linkedin-query-seeds.md`: seed stretti e calibrabili per saved search e alert.
- `profile/target-roles.md`: perimetro delle role family, alias e esclusioni.
- `job-search/market-observatory-spec.md`: schema del record normalizzato, fonti ammesse e pipeline logica.
- `job-search/scoring-model.md`: score, pesi iniziali, spiegazione e regole di calibrazione.
- `job-search/italy-market-sources.md`: fonti italiane e loro ruolo come segnali di contesto o domanda.
- Feedback manuale: esito candidature, note recruiter, shortlist, scarti e correzioni di lettura.

Input operativi attesi nei workflow:

- email di alert o forwarding;
- URL pubblici di annunci o feed;
- record manuali di revisione;
- parametri di schedulazione e routing;
- configurazione di connessioni e credenziali tramite UI n8n.

## Output

- record normalizzati e classificati;
- coda prioritaria per candidatura o revisione;
- report giornalieri o settimanali per role family;
- lista di query da tenere, dividere, restringere o ritirare;
- history per query/ciclo utile a calcolare raccomandazioni su una finestra rolling di 5 cicli;
- log di errori, warning e fonti non parseabili;
- backlog di record per manual review;
- export o payload pronti per consumo esterno, se necessario.

L'output deve conservare sempre il riferimento alla fonte originale e una spiegazione sintetica della lettura.

Regole per report e webhook:

- includere `topMatches` con record non esclusi sopra soglia, link, score e spiegazione;
- non inviare di default `records` completi con HTML lunghi nei webhook umani;
- includere `familySummary` calcolata solo sui record non esclusi, oppure indicare chiaramente se include anche gli esclusi;
- aggiungere `excludedSummary` con conteggi per `exclusionReason`, esempi di titolo/fonte e query candidate da restringere;
- includere `monitorQueue` come lista breve per record non esclusi sotto soglia ma potenzialmente interessanti;
- per gli alert LinkedIn, parsare prima i blocchi strutturati dell'email e
  scartare preambolo/footer; quando rimangono solo titolo, azienda, localita e
  link, marcare il record come `dataPoor`;
- i record `dataPoor` con titolo target forte vanno a `inspect manually`, non a
  `apply`, e l'arricchimento deve usare solo fonti pubbliche non loggate;
- i ruoli in UK/Regno Unito senza sponsorship o supporto visto esplicito vanno
  esclusi prima dello scoring con `exclusionReason: uk_requires_sponsorship`;
- le email LinkedIn di recommended jobs possono entrare nello stesso parser, ma
  i suggerimenti senza titolo target o famiglia ruolo target riconoscibile vanno
  esclusi con `exclusionReason: low_signal_public_feed` e non devono alimentare
  `queryHealth`;
- mantenere `rawRecords` o descrizioni complete solo in storage/debug tecnico, non nel digest principale;
- `matchCount` deve contare soltanto i record presenti in `topMatches`.

Payload minimo consigliato:

```json
{
  "generatedAt": "...",
  "totalFetched": 0,
  "dedupedCount": 0,
  "matchCount": 0,
  "minPriorityScore": 40,
  "familySummary": {},
  "excludedSummary": {},
  "topMatches": [],
  "monitorQueue": [],
  "queryCalibrationNotes": [],
  "markdown": "..."
}
```

## Configurazione

- Le credenziali non vanno versionate in questo repository e non vanno trattate come parte della strategia.
- In `n8n-workflows` le connessioni a mailbox, feed, API o servizi esterni vanno collegate nella UI di n8n.
- Gli import JSON devono restare riusabili tra ambienti, con parametri esterni per endpoint, mailbox, schedule e routing.
- La configurazione deve distinguere ambiente, credenziali e logica di workflow.
- Le note operative devono indicare i passaggi manuali richiesti dopo l'import: collegare credenziali, verificare i trigger, attivare i workflow, testare un campione minimo.
- Non introdurre dipendenze da scraping loggato di LinkedIn; il percorso previsto resta alert, email, feed pubblici e fonti aperte compatibili.

## Error Handling

- Se una fonte fallisce nel parsing, salvare il payload grezzo o il riferimento minimo utile e mandare il record in revisione.
- Se un campo manca, lasciarlo vuoto o unknown invece di inferirlo di forza.
- Se una fonte produce duplicati, deduplicare a valle senza perdere il riferimento alla sorgente.
- Se la classificazione o lo scoring non hanno abbastanza segnali, abbassare la confidenza e inviare il record a revisione manuale.
- Se una sorgente cambia formato, isolare il guasto al ramo della fonte e non bloccare il resto della pipeline.
- Le failure transitorie devono essere retryable; le failure strutturali devono emergere come warning operativi, non come silenzio.

## Confini Con Questo Repo

- Questo repository definisce strategia, tassonomia, query e scoring.
- Il repository `n8n-workflows` definisce i workflow eseguibili, i JSON importabili, le connessioni e le note operative.
- Qui non si archiviano credenziali, token, segreti o configurazioni sensibili.
- Qui non si introduce logica di scraping di LinkedIn loggato.
- Qui non si duplicano i dettagli esecutivi dei workflow: si descrive il comportamento atteso, non l'implementazione runtime.
- Se la strategia cambia, la variazione va riflessa prima qui e poi nei workflow n8n.

## Prossimi Passi

1. Creare nel repo `n8n-workflows` gli scaffold dei workflow previsti.
2. Collegare le credenziali nella UI di n8n e validare i trigger su un campione minimo.
3. Implementare la catena ingestion -> normalization -> classification -> scoring -> reporting.
4. Aggiungere il canale per manual review capture e feedback loop.
5. Verificare che gli output rispettino lo schema e le regole di calibrazione definite qui.
6. Aggiornare questo handoff solo se cambiano strategia, confini o responsabilita.
