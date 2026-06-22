# Scoring Model

## Narrative Layer
Prima dei punteggi tradizionali, il sistema deve leggere ogni record con un layer di narrative professionali. Questo layer non sostituisce le query di raccolta, ma guida interpretazione, reporting e manutenzione degli alert.

Narrative decise:

- `Platform Modernization & Reliability` -> primaria
- `Technical Leadership for Delivery, Scale and Execution` -> primaria
- `Applied AI / Document Intelligence in Production` -> secondaria
- `Complex / Regulated Contexts` -> filtro trasversale

Regole:

- ogni job deve avere una `primaryNarrative` quando esistono segnali sufficienti;
- puo avere una `secondaryNarrative` opzionale, ma mai piu di una;
- `Complex / Regulated Contexts` non e una narrativa autonoma: aumenta il fit quando rafforza una narrativa primaria o secondaria;
- se un titolo forte non entra in nessuna narrativa primaria, il record va abbassato nettamente anche se il titolo sembra prestigioso;
- il digest umano deve mostrare prima la narrativa rilevata e poi il titolo del ruolo.

Segnali minimi per narrativa:

- `Platform Modernization & Reliability`: modernization, migration, platform engineering, cloud transformation, Kubernetes, AWS, DevOps, SRE, IaC, observability, reliability, incident reduction, developer platform;
- `Technical Leadership for Delivery, Scale and Execution`: ownership di team, delivery, execution, architecture governance, roadmap, hiring, operating model, scaling team o processi tecnici;
- `Applied AI / Document Intelligence in Production`: AI solo quando compaiono segnali di produzione o industrializzazione, per esempio deployment, operationalization, workflow automation, OCR, document processing, document intelligence, production AI platform, governance o integration.

Segnali del filtro trasversale `Complex / Regulated Contexts`:

- banking, fintech, insurance, private banking, payments;
- auditability, compliance, security, risk, enterprise governance;
- integrazione con sistemi legacy critici o contesti multi-stakeholder ad alta complessita.

## Principi
Questo modello serve a classificare annunci e segnali in modo operativo, non a produrre una verita sul mercato.

- Ogni score e una ipotesi iniziale da calibrare su evidenza osservata, non una costante del sistema.
- Ogni punteggio deve essere spiegabile a partire da segnali leggibili nel record normalizzato.
- I pesi servono per ordinare, non per sostituire il giudizio umano.
- I segnali di salary guide restano deboli e secondari: possono aiutare a contestualizzare, ma non devono dominare lo score.
- Se un dato manca, il sistema deve abbassare la confidenza, non inventare il segnale.
- I punteggi vanno letti insieme a `recommendedAction`, non da soli.

Scala comune:
- ogni score e espresso su 0-100;
- 0 significa assenza di segnale utile;
- 100 significa segnale molto forte e coerente rispetto al profilo e alla role family.

## Score
Il modello usa quattro score principali:

- `marketDemandScore`: quanto il mercato mostra domanda osservabile per la role family o per un alias vicino.
- `profileFitScore`: quanto il ruolo e coerente con il posizionamento, le responsabilita e il contesto di Ignazio.
- `candidateAdvantageScore`: quanto esiste un vantaggio competitivo specifico rispetto ad altri candidati plausibili.
- `applicationPriorityScore`: quanto vale investire tempo sulla candidatura adesso.

## Gate Prima Dello Scoring
Prima di calcolare gli score, il sistema deve applicare un gate di esclusione coerente con `profile/target-roles.md` e `job-search/market-observatory-spec.md`.

Regola: un record chiaramente fuori perimetro non deve essere classificato dentro una role family target solo perche contiene parole comuni come team, startup, product, owner, platform, business, quality o architecture.

Esclusioni dure:

- ruoli commerciali: sales, account executive, business development, closer, head of sales, sales specialist;
- ruoli content o creativi non tecnici: writer, copywriter, content writer, editor, video editor;
- ruoli amministrativi o operations non tecniche: office assistant, bookkeeping, data entry, customer service, customer operations non tecnica;
- ruoli di labeling, online data analyst, map evaluator, survey, microtask, part-time assessment o task-based work;
- internship, trainee, junior puro, entry-level o ruoli con formazione come componente principale;
- ruoli freelance, contractor, independent, talent network, marketplace o staff-augmentation quando l'obiettivo operativo e un lavoro permanent;
- Product Engineer, Full-stack Engineer o Software Engineer standalone quando il perimetro e principalmente individual contributor e non mostra ownership tecnica ampia, architettura, leadership o responsabilita di piattaforma;
- annunci con requisiti geografici o di residenza incompatibili con il perimetro operativo noto, salvo eccezione manuale;
- ruoli tecnici puramente esecutivi senza ownership, architettura, delivery, AI industrializzata, piattaforma o leadership.

Output del gate:

- `outOfScope: true`;
- `recommendedAction: ignore` quando il problema e il ruolo;
- `recommendedAction: refine query` quando il record indica che una query sta catturando rumore sistematico;
- score a `0` o non calcolati, ma mai priority sopra soglia;
- `roleFamily` vuoto o `out_of_scope`;
- `explanation` breve con `exclusionReason` e segnali osservati.

Esempi di record da escludere prima dello scoring:

- `iOS Developer` senza ownership tecnica ampia o leadership;
- `High-Ticket Financial Sales Specialist`, `Inside Sales Contractor`, `Head of Sales`;
- `Freelance Writer`, `Copywriter`, `Office Assistant`;
- `Online Data Analyst`, `Data Labeling Specialists`;
- `Customer Operations & Writing Specialist` se il perimetro e customer support/writing e non trasformazione tecnica.
- `Product Engineer` standalone quando descrive un IC full-stack/prodotto senza ownership di architettura, team o piattaforma;
- `Senior Independent AI Engineer / Architect` o simili quando il modello e freelance/contractor/talent network e non permanent employment.

Regola di composizione:

- `marketDemandScore` misura il mercato;
- `profileFitScore` misura la compatibilita;
- `candidateAdvantageScore` misura il vantaggio relativo;
- `applicationPriorityScore` combina i tre score e applica i vincoli pratici.

Formula iniziale consigliata per `applicationPriorityScore`:

`applicationPriorityScore = 0.25 * marketDemandScore + 0.45 * profileFitScore + 0.30 * candidateAdvantageScore`

Questa e una base di partenza, non una verita. Il demand score aiuta soprattutto a capire dove guardare e quali query mantenere; la priorita di candidatura deve dipendere di piu da fit e vantaggio competitivo. Se il feedback manuale mostra che il mercato privilegia un fattore diverso, i pesi vanno ricalibrati.

## marketDemandScore
Questo score misura la domanda osservata di mercato per una role family o per un cluster di alias.

Formula iniziale consigliata:

- `observedAlertVolume` = 45%
- `externalJobCount` = 25%
- `trendVelocity` = 15%
- `macroTrendSupport` = 10%
- `searchInterest` = 5%

Definizione dei componenti:

- `observedAlertVolume`: quantita di annunci effettivamente osservati negli alert o nei feed usati dal sistema.
- `externalJobCount`: volume di annunci o occorrenze da fonti esterne pubbliche e confrontabili.
- `trendVelocity`: variazione recente del volume o della ricorrenza rispetto alla finestra precedente.
- `macroTrendSupport`: coerenza con fonti di contesto come osservatori, report settoriali, associazioni di filiera o fonti istituzionali.
- `searchInterest`: interesse di ricerca o ricorrenza di termini, da trattare come segnale debole.

Regole:

- gli alert osservati pesano piu dei trend generici;
- un titolo che ricorre spesso ma solo in una piattaforma va trattato con prudenza;
- il volume assoluto conta meno della ricorrenza coerente tra fonti diverse;
- salary guide e compensazioni pubbliche, se usate, entrano solo come parte del contesto macro e non devono spostare da sole il punteggio;
- un aumento di volume senza coerenza di titolo o di ruolo non deve essere interpretato come domanda robusta.

Segnali che alzano il punteggio:

- titolo ricorrente in piu fonti;
- stessa family osservata in piu contesti o aziende;
- crescita recente del numero di osservazioni;
- allineamento con fonti italiane o settoriali pertinenti;
- linguaggio del ruolo compatibile con il posizionamento target.

Segnali che lo abbassano:

- rumore da titoli troppo generici;
- alert duplicati o quasi duplicati;
- apparente volume alto ma poca coerenza di responsabilita;
- segnali basati solo su salary guide o trend narrativi;
- mismatch sistematico con i target role definiti.

## profileFitScore
Questo score misura la compatibilita tra il ruolo osservato e il profilo professionale di Ignazio.

Formula iniziale consigliata:

- `primaryNarrativeAlignment` = 30%
- `roleFamilyAlignment` = 20%
- `responsibilityMatch` = 25%
- `domainMatch` = 20%
- `contextMatch` = 5%

Definizione dei componenti:

- `primaryNarrativeAlignment`: corrispondenza con una delle narrative primarie decise e con i relativi segnali osservabili.
- `roleFamilyAlignment`: corrispondenza con una delle role family target o con un alias chiaramente vicino.
- `responsibilityMatch`: presenza di ownership su architettura, roadmap, delivery, team, piattaforme o decisioni tecniche reali.
- `domainMatch`: coerenza con cloud, platform, DevOps, AI/document intelligence, banking/fintech, startup o trasformazione tecnica.
- `contextMatch`: coerenza con il tipo di azienda e di contesto in cui il profilo porta piu valore, per esempio startup, scaleup, enterprise regolata o iniziativa di trasformazione.

Regole:

- la narrativa primaria conta piu del titolo;
- il titolo da solo non basta: conta il perimetro reale descritto nell'annuncio;
- ruoli solo esecutivi o solo amministrativi non devono ricevere punteggi alti;
- una buona aderenza tecnica senza ownership di ruolo non basta per uno score alto;
- un contesto molto distante dal posizionamento target deve abbassare il punteggio anche se il titolo sembra vicino.
- un titolo executive senza narrativa primaria credibile non deve ricevere punteggio alto.

Segnali che alzano il punteggio:

- chiara appartenenza a una delle narrative primarie;
- ownership tecnica o di piattaforma;
- leadership di team o di delivery;
- contesto regolato, complesso o cross-funzione dove il profilo ha esperienza utile;
- forte aderenza con una role family primaria.

Segnali che lo abbassano:

- assenza di narrativa primaria coerente;
- ruolo da senior developer puro;
- coordinamento senza decisione tecnica;
- attivita solo PMO, recruiting, ticketing o vendor management;
- titolo attraente ma perimetro debole o incoerente.

## candidateAdvantageScore
Questo score misura quanto il profilo di Ignazio crea un vantaggio rispetto a candidati plausibili con skill simili ma senza la stessa combinazione di esperienza.

Formula iniziale consigliata:

- `bankingOrRegulatedExperience` = 25%
- `startupAndProductProof` = 25%
- `platformAndCloudDepth` = 20%
- `leadershipAndDeliveryProof` = 15%
- `crossDomainTranslation` = 15%

Definizione dei componenti:

- `bankingOrRegulatedExperience`: vantaggio dato dall'esperienza in contesti regolati, complessi o con vincoli forti.
- `startupAndProductProof`: vantaggio dato dall'aver portato iniziative da idea a esercizio o da prototipo a mercato.
- `platformAndCloudDepth`: vantaggio dato da cloud, Kubernetes, DevOps, GitOps, AWS o piattaforme moderne.
- `leadershipAndDeliveryProof`: vantaggio dato da coordinamento di team, delivery end-to-end, governance e capacita di esecuzione.
- `crossDomainTranslation`: vantaggio dato dalla capacita di collegare business, tecnologia, architettura e operations.

Regole:

- questo score deve premiare combinazioni rare, non semplici liste di skill;
- un singolo keyword match non basta;
- il valore cresce quando il ruolo richiede esattamente il tipo di sintesi che il profilo offre;
- se il ruolo non valorizza leadership, piattaforme o trasformazione, il vantaggio competitivo scende anche se il titolo sembra interessante.

Segnali che alzano il punteggio:

- banking, fintech o contesti regolati con bisogno di modernizzazione;
- startup o scaleup che cercano ownership tecnica reale;
- piattaforme cloud, DevOps o AI da industrializzare;
- bisogno di tradurre problemi di business in soluzioni operative.

Segnali che lo abbassano:

- profilo indistinto rispetto al mercato;
- ruolo standardizzato, facilmente sostituibile o troppo verticale;
- assenza di problemi dove l'esperienza trasversale porta vantaggio.

## applicationPriorityScore
Questo score serve a decidere dove investire tempo adesso.

Formula iniziale consigliata:

- base: combinazione pesata di `marketDemandScore`, `profileFitScore` e `candidateAdvantageScore`;
- correzioni: penalita per ambiguita, mancanza di dati, perimetro troppo vago o segnali contraddittori;
- boost: solo quando esiste una convergenza chiara tra domanda, fit e vantaggio competitivo.

Regola operativa iniziale:

- 80-100: priorita alta, candidatura o revisione immediata;
- 60-79: priorita media, monitoraggio o candidatura selettiva;
- 40-59: priorita bassa, da tenere solo se il mercato e scarso o il ruolo e strategico;
- sotto 40: normalmente non prioritario.

Questo intervallo e una ipotesi iniziale da calibrare. Se il feedback manuale mostra che le soglie sono troppo permissive o troppo rigide, vanno modificate.

Decision rules:

- se `outOfScope` e `true`, `applicationPriorityScore` deve restare sotto soglia e il record non entra in `topMatches`;
- se `marketDemandScore` e alto ma `profileFitScore` e basso, il ruolo va monitorato ma non trattato come priorita;
- se `profileFitScore` e alto ma `marketDemandScore` e basso, il ruolo puo restare in osservazione se il vantaggio competitivo e forte;
- se `candidateAdvantageScore` e alto ma domanda e fit sono deboli, la candidatura va considerata solo in casi selezionati;
- se tutti e tre gli score sono medi, il ruolo non merita automatismi forti.

## Data-Poor Alert Fallback

Saved-search email alerts, especially LinkedIn digests, may expose title,
company, location and link but not the full job description. In this case the
system must not infer detailed fit or candidate advantage.

If a record is from a job-alert email, has a strong target title, has no hard
exclusion signal, and lacks enough description to score responsibilities or
domain fit, mark it as `dataPoor: true`.

Decision rule:

- strong target title + no hard exclusion + data poor -> `inspect manually`;
- weak or ambiguous title + data poor -> `ignore` or `monitor`;
- hard exclusion + data poor -> `ignore`;
- enriched public description available -> score normally.

This fallback is a triage rule, not a boost to `apply`.

## Geographic Eligibility Gate

I ruoli in UK/Regno Unito richiedono un gate dedicato perche, fuori dall'UE,
possono richiedere visto o sponsorship. Il sistema deve trattarli come fuori
perimetro operativo quando non c'e un'indicazione esplicita di sponsorship
disponibile.

Regola:

- UK/Regno Unito/Londra senza visa sponsorship esplicita -> `ignore`;
- UK con `visa sponsorship`, `skilled worker visa`, `sponsorship available` o segnale equivalente -> scoring normale;
- titolo forte, salary alta o fit narrativo non devono superare il gate se la sponsorship manca;
- se il record e data-poor e UK con sponsorship, va comunque a `inspect manually`, non ad `apply`.

## LinkedIn Recommended Jobs Gate

Le email LinkedIn di recommended jobs possono essere usate come fonte
opportunistica, ma non sono query salvate e non devono contaminare `queryHealth`.

Regola:

- suggerimento con titolo target esplicito -> scoring normale;
- suggerimento senza titolo target ma con famiglia ruolo target riconoscibile -> scoring normale prudente;
- suggerimento senza titolo target e senza famiglia ruolo target riconoscibile -> `outOfScope: true`, `exclusionReason: low_signal_public_feed`, `recommendedAction: ignore`;
- i record esclusi da questo gate non devono comparire nelle sezioni operative del digest.

Regole aggiuntive per il layer narrativo:

- se il record e data-poor ma il titolo e coerente con una narrativa primaria, il sistema puo assegnare una `tentativePrimaryNarrative` a bassa confidenza e mandare a `inspect manually`;
- se il record e data-poor e non mostra nessuna narrativa primaria leggibile, il titolo non basta per alzare la priorita;
- per la narrativa AI, in assenza di segnali di produzione o industrializzazione, il record non deve essere classificato come `Applied AI / Document Intelligence in Production`.

Regole operative per `recommendedAction`:

- `apply`: fit e vantaggio sono entrambi forti, i vincoli pratici sono compatibili e il ruolo e abbastanza specifico da meritare lavoro immediato;
- `monitor`: fit alto o vantaggio alto, ma esiste un vincolo da verificare come geografia, seniority, modello freelance, stack non centrale o priorita bassa di mercato;
- `inspect manually`: segnali forti ma ambigui, oppure dati mancanti che possono cambiare la decisione;
- `refine query`: record fuori perimetro ma utile per capire che una query o una fonte sta generando rumore ripetuto;
- `ignore`: ruolo escluso, fit basso, vantaggio basso o vincoli incompatibili.

Soglia iniziale per `topMatches`:

- includere solo record non esclusi con `applicationPriorityScore >= 40`;
- se il report diventa rumoroso, alzare la soglia a 50 o richiedere almeno `profileFitScore >= 50`;
- un record con `profileFitScore = 0` non deve entrare in `topMatches` anche se il titolo sembra vicino.

## Alert Quality Metrics
Le query LinkedIn non vanno valutate solo per volume. L'unita di controllo principale e la query specifica, non il gruppo canonico.

Per ogni query misurare almeno:

- `jobsObserved`
- `%PrimaryNarrativeFit`: quota di job che entrano in una narrativa primaria
- `%ManualInspection`: quota di job che finiscono in `inspect manually`
- `%OutOfNarrativeNoise`: quota di job che non entrano in nessuna narrativa primaria
- `%OutOfScope`: quota di job esclusi dal gate

Regole operative:

- la lettura primaria e per `singolo alert/query`;
- aggregazioni per gruppo canonico servono solo come vista secondaria;
- non prendere decisioni `keep / split / narrow / retire` su un singolo digest rumoroso;
- osservare una query per `5 cicli` prima di decidere se tenerla, restringerla, splittarla o ritirarla, salvo rumore evidentemente patologico.

## Variante Italia
La variante italiana serve quando il sistema legge soprattutto il mercato italiano o fonti Italia-centriche.

Formula iniziale consigliata per il demand layer italiano:

- `observedLocalAlertVolume` = 35%
- `excelsiorProfessionSectorDemand` = 25%
- `difficultyToFindSignal` = 15%
- `sectorGrowthSignal` = 15%
- `titleAliasConfidence` = 10%

Definizione dei componenti:

- `observedLocalAlertVolume`: volume reale osservato da alert LinkedIn, Indeed-like o fonti analoghe accessibili.
- `excelsiorProfessionSectorDemand`: segnale di domanda da Excelsior, per professione, settore o territorio.
- `difficultyToFindSignal`: segnale che indica scarsita o difficolta a trovare certi profili, in particolare su skill STEM o digitali.
- `sectorGrowthSignal`: segnale di crescita o priorita di settore da fonti come Anitec-Assinform o Osservatori Politecnico di Milano.
- `titleAliasConfidence`: confidenza che titolo, alias e traduzione italiana descrivano davvero la stessa family.

Regole specifiche per l'Italia:

- il volume locale reale conta piu delle guide generiche;
- Excelsior e le fonti italiane servono a dare contesto e a rafforzare la lettura, non a inventare vacancy;
- la difficolta di reperimento e un segnale utile solo se agganciata a una family e a skill riconoscibili;
- la crescita settoriale aiuta a interpretare la direzione del mercato, ma non sostituisce gli annunci;
- la confidenza sugli alias e critica: titoli italiani e inglesi possono coprire perimetri diversi.

Regola di peso:

- se gli annunci osservati e le fonti italiane convergono, la variante Italia puo alzare la priorita;
- se le fonti italiane parlano in astratto ma gli annunci non convergono, il sistema deve restare prudente;
- se la salary guide diverge dagli annunci, gli annunci hanno priorita e la salary guide resta un segnale debole.

## Regole Di Spiegazione
Ogni explanation deve permettere di ricostruire perche il record ha ricevuto quel punteggio.

Formato minimo consigliato:

- che cosa e stato osservato;
- quale inferenza e stata fatta;
- quale score e stato mosso;
- quale segnale e stato considerato debole;
- quale azione deriva da quella lettura.

Regole di scrittura:

- distinguere sempre fatto e inferenza;
- usare termini semplici e stabili, senza gergo interno inutile;
- citare i segnali dominanti, non l'intero calcolo;
- se un componente e debole o mancante, dichiararlo;
- non scrivere explanation che suonino come una sentenza definitiva.

Esempio di struttura:

- osservazione: titolo, azienda, fonte o contesto;
- inferenza: role family, seniority o perimetro;
- motivo del punteggio: domanda, fit o vantaggio competitivo;
- limite: dati mancanti, salary guide debole o alias incerto;
- azione: apply, monitor, inspect manually, refine query o ignore.

## Calibrazione
La calibrazione deve essere lenta e guidata dal feedback reale.

- Se gli annunci ad alta priorita non producono risposte, il sistema sta sovrastimando fit o vantaggio.
- Se gli annunci scartati diventano spesso match buoni, il sistema sta sottostimando qualche componente.
- Se un titolo produce molti falsi positivi, il problema e nella query o nel mapping, non solo nello score.
- Se una role family mostra fit alto ma domanda bassa, va tenuta separata e non forzata dentro una famiglia piu ampia.
- Se le fonti italiane aiutano a classificare ma non a decidere, il loro peso va tenuto di supporto, non dominante.

Regola pratica di calibrazione:

- keep: il punteggio e coerente con l'esito manuale;
- split: il segnale mescola mercati o perimetri diversi;
- narrow: il punteggio lascia passare troppo rumore;
- retire: il segnale non distingue piu nulla di utile.

Politica di revisione:

- cambiare un peso solo dopo aver visto un pattern ripetuto;
- non cambiare piu variabili insieme se non si capisce quale ha mosso l'esito;
- se la differenza tra annunci buoni e cattivi e nel linguaggio della fonte, correggere prima la normalizzazione;
- se il problema e strutturale, rivedere la formula, non solo la soglia.

## Anti-pattern
Questi errori vanno evitati.

- Trattare i pesi come verita fisse.
- Usare la salary guide come driver principale dello score.
- Premiare il volume senza verificare la qualita del titolo e del perimetro.
- Confondere una role family con una singola query o con un singolo annuncio.
- Assegnare punteggi alti a ruoli solo perche contengono parole chiave familiari.
- Nascondere l'incertezza dentro explanation troppo assertive.
- Unire domanda di mercato, fit e vantaggio competitivo in un solo numero senza capire cosa sta davvero funzionando.
- Ricalibrare troppo spesso sulla base di pochi record.
- Lasciare che le fonti di contesto sovrastino gli annunci osservati.
- Dare priorita automatica a un annuncio solo perche sembra interessante, senza verificare la convergenza tra segnali.
