# Market Observatory Spec

## Obiettivo
L'osservatorio di mercato deve aiutare Ignazio a capire:

- quali role family hanno piu probabilita di fit con il suo profilo;
- quali annunci meritano attenzione immediata, monitoraggio o scarto;
- quali segnali di mercato sono ricorrenti e quali invece sono rumore;
- come aggiornare il posizionamento e le query senza inseguire un singolo annuncio.

Il sistema non deve produrre "verita" sul mercato, ma una lettura operativa, ripetibile e spiegabile. Ogni valutazione deve separare:

- fatto osservato nell'annuncio o nella fonte;
- inferenza sulla role family o sulla seniority;
- segnale debole, come salary guide o trend generico;
- feedback manuale successivo all'applicazione.

## Non Obiettivi
L'osservatorio non deve:

- fare scraping di LinkedIn loggato come percorso di default;
- raccogliere credenziali, token, password o dati privati;
- inventare dati di mercato, salary benchmark o disponibilita di ruoli;
- sostituire il giudizio manuale sulle candidature;
- trasformarsi in un tracker generico di annunci senza legame con il posizionamento;
- decidere da solo il titolo target finale di Ignazio senza feedback umano.

## Fonti

### Saved-search / job-alert email
Fonte primaria per osservare alert da LinkedIn, Indeed-like e prodotti analoghi via email o notifica salvata.

- Cosa prova: quali annunci sono stati effettivamente mostrati dal motore di ricerca o dall'alert.
- Cosa non prova: qualita del database completo, copertura totale del mercato o ranking reale dell'algoritmo.
- Regola: usare preferibilmente inbox o alert esportati, non login scraping.

### Public job APIs o public job feeds
Job board pubbliche, feed RSS, endpoint pubblici, aggregatori con accesso aperto e condizioni d'uso compatibili.

- Cosa prova: disponibilita pubblica dell'annuncio e metadati leggibili.
- Cosa non prova: che l'annuncio sia realmente attivo in tutte le geografie o che il ruolo sia ancora aperto.
- Regola: privilegiare fonti pubbliche e stabili, con source URL persistente quando possibile.

### Fonti italiane
Fonti rilevanti per il mercato italiano, il posizionamento locale e i segnali di domanda per area, settore o famiglia professionale.

- Cosa prova: contesto di mercato, lessico del ruolo, diffusione settoriale, segnali istituzionali o di filiera.
- Cosa non prova: fit immediato di un singolo annuncio.
- Regola: usarle per orientare query, priorita e lettura del mercato, non come sostituto degli annunci.

### Salary guide come segnali deboli
Guide salariali, survey di compensation, report di recruiting e benchmark pubblici.

- Cosa prova: una possibile forchetta o narrativa di mercato, se molto coerente con ruolo e geografia.
- Cosa non prova: il valore reale di una specifica offerta o la disponibilita di budget in un'azienda.
- Regola: trattarle come segnale debole e secondario, utile solo per calibrare aspettative o individuare outlier.

### Feedback manuale
Esito delle candidature, reazioni dei recruiter, note qualitative su colloqui, shortlist, scarti e motivazioni.

- Cosa prova: il fit reale percepito dal mercato e l'efficacia delle query.
- Cosa non prova: la validita universale di un titolo o di una fonte.
- Regola: e il segnale piu importante per calibrare il sistema.

## Record Normalizzato
Ogni annuncio o segnale osservato deve essere rappresentato con un record normalizzato. I campi del piano sono obbligatori a livello logico; alcuni potranno essere vuoti quando la fonte non li fornisce.

- `source`: origine del record, per esempio alert email, feed pubblico, fonte italiana o feedback manuale.
- `capturedAt`: timestamp di acquisizione, non della pubblicazione.
- `title`: titolo cosi come appare nella fonte.
- `company`: nome azienda o ente, cosi come osservato.
- `location`: localita dichiarata, inclusa eventuale area geografica o paese.
- `remotePolicy`: politica remota osservata o inferita dal testo, per esempio onsite, hybrid, remote, flexible.
- `url`: link canonico all'annuncio o alla fonte.
- `description`: testo utile dell'annuncio o estratto normalizzato.
- `roleFamily`: famiglia di ruolo assegnata secondo `profile/target-roles.md`.
- `outOfScope`: booleano che indica se il record e stato escluso prima della classificazione perche fuori perimetro.
- `exclusionReason`: motivo sintetico dell'esclusione, per esempio `commercial_role`, `admin_role`, `content_role`, `junior_entry_level`, `location_mismatch`, `non_technical_operations`, `low_signal_public_feed`.
- Per calibrazione corrente, usare anche `non_permanent_engagement` per freelance/contractor/talent network/marketplace incompatibili con una ricerca permanent, e `standalone_ic_engineering` per Product Engineer, Full-stack Engineer o Software Engineer senza ownership ampia.
- `exclusionSignals`: segnali osservati che giustificano l'esclusione.
- `seniority`: seniority osservata o inferita, con etichetta coerente e prudente.
- `companySizeAssumption`: stima o ipotesi sulla dimensione aziendale solo se supportata da segnali pubblici; altrimenti vuoto.
- `marketDemandSignals`: segnali di domanda osservati, per esempio ricorrenza del titolo, frequenza, geografia, settore, stack o numero di fonti convergenti.
- `profileFitSignals`: segnali di fit con il profilo di Ignazio, per esempio ownership tecnica, cloud, platform, AI, leadership o contesto regolato.
- `candidateAdvantageSignals`: segnali che creano vantaggio competitivo specifico, per esempio esperienza su banking, startup, Kubernetes, AWS, DevOps, governance o delivery end-to-end.
- `dataPoor`: booleano che indica che il record ha troppo poco testo descrittivo per uno scoring affidabile oltre a titolo, azienda, localita e link.
- `enrichmentStatus`: stato dell'arricchimento pubblico, per esempio `not_attempted`, `found_public_source`, `not_found`, `blocked_login` o `failed`.
- `enrichmentSource`: URL pubblico usato per testo aggiuntivo, quando disponibile.
- `recommendedAction`: azione consigliata, per esempio apply, monitor, ignore, refine query, inspect manually.
- `explanation`: spiegazione breve e tracciabile del perche il record ha ricevuto quella lettura.

Regole del record:

- i campi devono contenere solo informazioni osservabili o inferenze dichiarate;
- ogni inferenza deve essere distinguibile nel testo dell'`explanation`;
- `companySizeAssumption` non e un fatto, ma un'ipotesi da verificare;
- se `outOfScope` e `true`, `roleFamily` deve restare vuoto o `out_of_scope`: il sistema non deve forzare una famiglia target su un record escluso;
- se mancano dati, il campo deve restare vuoto o marcato come unknown, non riempito a intuito.

## Pipeline Logica
1. Ingestione.
   Raccogliere record da alert email, feed pubblici, fonti italiane e feedback manuale.

2. Normalizzazione.
   Portare ogni record nello stesso schema, preservando il testo originale utile e il link fonte.

3. Gate di esclusione.
   Prima di assegnare una role family, applicare esclusioni dure sui record chiaramente fuori perimetro. Questo gate serve a impedire che parole generiche come team, startup, product, architecture, owner o business trascinino annunci non pertinenti dentro famiglie tecniche.

   Escludere prima della classificazione quando il titolo o il perimetro indicano chiaramente:
   - sales, account executive, business development, closer, head of sales o ruoli commerciali;
   - writer, copywriter, content writer, video editor, office assistant, customer support, data labeling o online data analyst;
   - internship, trainee, junior puro, entry-level o ruoli senza ownership tecnica;
   - operations amministrative, bookkeeping, calendari, data entry, customer service o supporto non tecnico;
   - freelance, contractor, independent, talent network, marketplace, staff augmentation o modelli analoghi quando il target e permanent employment;
   - Product Engineer, Full-stack Engineer o Software Engineer standalone quando il perimetro e da individual contributor e non da leadership, architettura, piattaforma o ownership tecnica ampia;
   - localita o requisiti di residenza incompatibili con il perimetro operativo noto;
   - ruolo tecnico solo esecutivo quando manca qualunque segnale di architettura, piattaforma, AI industrializzata, ownership o leadership.

   Un record escluso deve ricevere `outOfScope: true`, `recommendedAction: ignore` o `refine query`, score azzerati o non calcolati, e una `explanation` breve basata sui segnali di esclusione.

4. Classificazione.
   Assegnare `roleFamily`, `seniority` e `remotePolicy` con regole semplici e coerenti con `profile/target-roles.md`.

5. Lettura dei segnali.
   Separare in tre gruppi:
   - domanda di mercato;
   - fit con il profilo;
   - vantaggio competitivo personale.

6. Valutazione.
   Produrre `recommendedAction` e `explanation` usando regole esplicite, non intuizioni non documentate.

7. Feedback loop.
   Aggiornare il sistema con esito di candidature, risposte dei recruiter e qualita delle query.

8. Manutenzione delle query.
   Usare gli esiti per mantenere, dividere, restringere o ritirare query e alert.

## Output Attesi
L'osservatorio deve produrre output semplici da leggere e facili da usare in una fase successiva di automazione:

- una coda di annunci normalizzati e classificati;
- una vista per role family con segnalazioni ricorrenti;
- una lista di annunci ad alta priorita per candidatura o revisione;
- una lista di alert da monitorare ma non ancora usare;
- una lista di query da tenere, modificare o ritirare;
- un log di feedback manuale per calibrare il sistema.

Ogni output deve conservare il riferimento alla fonte originale e alla spiegazione della valutazione.

Regole di compattezza:

- il report operativo deve privilegiare `topMatches`, sintesi per role family e sintesi degli esclusi;
- `description` completa deve restare disponibile nel record grezzo o nel dettaglio, ma non va inclusa di default nei webhook o nei digest umani;
- per i record esclusi includere solo conteggi per `exclusionReason`, esempi di titolo/fonte e query da restringere;
- i record sotto soglia ma non esclusi possono essere inclusi come lista breve, senza HTML e senza descrizione completa;
- `matchCount` deve contare solo record non esclusi che superano la soglia di priorita operativa.

## Calibrazione
La calibrazione deve essere lenta, esplicita e basata su evidenza osservata, non su sensazioni.

- Se una query produce troppi falsi positivi, restringerla prima di aggiungerne altre.
- Se una query produce annunci buoni ma troppo pochi, allargarla con cautela o splittarla per variante.
- Se una role family mostra fit alto ma volume basso, tenerla separata invece di fonderla con una famiglia vicina.
- Se salary guide e annunci pubblici divergono, dare priorita agli annunci e trattare la salary guide come segnale secondario.
- Se il feedback manuale smentisce ripetutamente l'interpretazione automatica, correggere il mapping o la regola, non il singolo record.

Regola di calibrazione pratica:

- keep: quando il segnale e coerente e ripetibile;
- split: quando una query o una family mescola due mercati diversi;
- narrow: quando il rumore supera il valore informativo;
- retire: quando una query non produce piu segnali utili o genera quasi solo falsi positivi.

## Rischi E Limiti

- Copertura incompleta: nessuna fonte vede tutto il mercato.
- Bias di piattaforma: alert e feed riflettono il ranking della fonte, non il mercato intero.
- Ambiguita dei titoli: lo stesso titolo puo coprire perimetri molto diversi.
- Overfitting: il sistema puo leggere troppo bene un piccolo campione e troppo male il mercato generale.
- False certezze sulla seniority: un titolo non garantisce il livello reale del ruolo.
- Stima errata della company size: senza dati pubblici solidi e solo un'ipotesi.
- Salary guide troppo pesanti: possono distorcere il ragionamento se usate come verita invece che come segnale debole.
- Rischio di manutenzione: se il feedback manuale non viene registrato, il sistema degrada.
- Vincoli legali e contrattuali: alcune fonti non sono adatte a estrazione automatica aggressiva.

## Open Questions

- Quale geografia deve avere priorita reale: Italia, Svizzera, UK o remoto internazionale.
- Quanto peso assegnare ai ruoli in italiano rispetto a quelli in inglese.
- Quale frequenza di osservazione e utile: giornaliera, settimanale o mista.
- Quali fonti italiane devono entrare per prime nel perimetro operativo.
- Quanto separare CTO, Head of Engineering e Platform leadership nella lettura del mercato.
- Quanto usare salary guide per calibrare aspettative rispetto al target di Ignazio.
- Quale livello di dettaglio mantenere nel feedback manuale per non renderlo troppo oneroso.
- Quale soglia rende un annuncio "buon fit" abbastanza forte da meritare candidatura.
