# Italy Market Sources

## Uso Nel Sistema

Queste fonti servono a tre scopi distinti:

1. Orientare le query e la lettura del mercato italiano, usando segnali istituzionali e settoriali.
2. Normalizzare titoli, professioni, competenze e famiglie professionali in modo coerente con `profile/target-roles.md` e `job-search/market-observatory-spec.md`.
3. Rafforzare o indebolire una lettura di fit solo quando piu fonti convergono sullo stesso segnale.

Regola pratica:

- le fonti di domanda vanno usate per pesare priorita e ricorrenza;
- le fonti di classificazione vanno usate per mappare titolo, professione e competenze;
- le fonti settoriali e di ricerca vanno usate come contesto, non come verita sul singolo annuncio;
- nessuna di queste fonti sostituisce gli annunci osservati o il feedback manuale.

## Excelsior Unioncamere

URL: https://excelsior.unioncamere.net/

Cosa misura:

- prospettive occupazionali e fabbisogni professionali e formativi espressi dalle imprese italiane;
- andamento della domanda di lavoro per settore, professione, competenze e territorio;
- lettura periodica tramite dashboard, banca dati e pubblicazioni.

Granularita attesa:

- buona granularita su macro-territori, settori, professioni e competenze;
- utile sia per viste mensili sia trimestrali;
- non e una fonte di singolo annuncio, ma di domanda aggregata.

Uso nello scoring:

- fonte primaria per il demand signal italiano;
- utile per aumentare o ridurre il peso di una role family quando la domanda aggregata e coerente con gli annunci osservati;
- utile per confrontare ricorrenza di titoli, competenze e aree geografiche;
- utile per dare priorita alle family che mostrano domanda ripetuta e linguaggio compatibile con il profilo.

Limiti:

- parla di fabbisogni aggregati, non di vacancy specifiche;
- puo riflettere un orizzonte temporale diverso da quello degli annunci online;
- richiede interpretazione prudente quando la terminologia e molto generale.

## ISTAT Professioni

URL: https://professioni.istat.it/

Cosa misura:

- la Nomenclatura e classificazione delle Unita Professionali, edizione 2021;
- la struttura gerarchica delle professioni italiane, con navigazione per livelli e rami.

Granularita attesa:

- molto buona sul piano tassonomico e gerarchico;
- alta utilita per distinguere famiglie, sottofamiglie e sinonimi istituzionali;
- bassa utilita come segnale di domanda diretta.

Uso nello scoring:

- fonte di normalizzazione, non di ranking mercato;
- utile per tradurre un titolo libero in una famiglia coerente;
- utile per evitare di mescolare ruoli diversi sotto lo stesso alias;
- utile per costruire dizionari di sinonimi e per allineare le etichette tra fonti diverse.

Limiti:

- non misura domanda attiva di mercato;
- puo essere piu rigida del linguaggio usato negli annunci reali;
- va usata come riferimento lessicale e gerarchico, non come proxy della disponibilita di ruoli.

## INAPP Atlante Lavoro

URL: https://www.inapp.gov.it/atlantelavoro/

Cosa misura:

- sistema classificatorio e informativo del lavoro;
- 24 settori economico-professionali;
- repertorio delle qualificazioni e linguaggio di competenze;
- sul sito e indicata la licenza CC BY-NC 4.0.

Granularita attesa:

- buona granularita settoriale e sulle qualificazioni;
- utile per collegare competenze, profili e ambiti produttivi;
- non e un feed di vacancy.

Uso nello scoring:

- fonte di mapping tra competenze osservate e famiglie di ruolo;
- utile per rafforzare il lessico di competenza nella lettura degli annunci italiani;
- utile per separare segnali di settore da segnali di ruolo;
- utile come supporto alla normalizzazione di titoli e skill.

Limiti:

- non misura direttamente la domanda di assunzione;
- il valore e soprattutto classificatorio e descrittivo;
- l'uso deve rispettare la licenza indicata dal sito;
- non va usata per dedurre trend quantitativi da sola.

## Cedefop Skills-OVATE

URL: https://www.cedefop.europa.eu/en/tools/skills-online-vacancies

Cosa misura:

- skills e contenuti estratti da online vacancies a livello europeo;
- distribuzione di competenze, titoli e requisiti nei testi degli annunci;
- lettura comparativa utile per osservare pattern cross-country e cross-role.

Granularita attesa:

- media o alta sul testo degli annunci e sulle skill osservate;
- piu utile per pattern che per decisioni su un singolo mercato locale;
- adatta a confronti tra famiglie di ruolo e richieste ricorrenti.

Uso nello scoring:

- fonte secondaria per validare che un lessico di skill sia davvero ricorrente negli annunci online;
- utile per confrontare il profilo italiano con il linguaggio europeo;
- utile per rafforzare o indebolire segnali di skill quando coincidono con Excelsior e con gli annunci reali;
- utile come supporto alla query refinement.

Limiti:

- e una vista europea, non una fotografia puntuale dell'Italia;
- non sostituisce i segnali locali;
- le skill estratte possono essere piu forti del contesto di ruolo reale;
- va pesata meno delle fonti che osservano direttamente il mercato italiano.

## EURES

URL: https://eures.europa.eu/index_en

Cosa misura:

- ricerca lavoro, informazioni sul mercato del lavoro europeo e vacancy insights;
- indicatori su shortages, surpluses e dinamiche transfrontaliere;
- contesto utile per mobilita e confronto tra mercati.

Granularita attesa:

- buona a livello europeo e per alcuni tagli nazionali o settoriali;
- utile per segnali su shortage/surplus e per disponibilita di annunci;
- meno precisa delle fonti locali quando serve una lettura strettamente italiana.

Uso nello scoring:

- fonte di contesto per capire se una role family e sostenuta anche fuori Italia;
- utile per distinguere segnali locali da segnali paneuropei;
- utile come seconda opinione quando il profilo e compatibile con mobilita internazionale o cross-border;
- utile per confermare ricorrenza di certe famiglie quando Excelsior e gli annunci pubblici convergono.

Limiti:

- non e una fonte primaria italiana;
- il livello di dettaglio puo variare molto da paese a paese;
- shortage/surplus non equivalgono a opportunita reale per il singolo profilo;
- non va usata come peso dominante sul mercato italiano.

## Anitec-Assinform

URL: https://www.anitec-assinform.it/

Cosa misura:

- visione dell'associazione italiana ICT;
- rapporti, studi, policy paper e materiali su Digitale in Italia;
- segnali su competenze, cultura digitale, filiere ICT e priorita di settore.

Granularita attesa:

- medio-alta sul settore ICT e sulle sue sotto-tematiche;
- buona per interpretare priorita industriali, non per singola vacancy;
- utile per leggere il vocabolario del mercato digitale italiano.

Uso nello scoring:

- fonte settoriale per capire quali aree ICT sono piu centrali nel dibattito italiano;
- utile per calibrare il peso di cloud, piattaforme, sicurezza, dati e trasformazione digitale;
- utile per rafforzare il contesto quando gli annunci usano linguaggio ICT ampio;
- utile per distinguere segnali di filiera da segnali di domanda concreta.

Limiti:

- associazione di settore, quindi inevitabilmente prospettica e non neutra;
- non e una misura diretta della domanda di lavoro;
- puo essere molto utile per contesto, ma non per ranking finale di un annuncio.

## Osservatori Digital Innovation Politecnico di Milano

URL: https://www.osservatori.net/

Cosa misura:

- ricerche e osservatori su innovazione digitale, trasformazione e adozione tecnologica in Italia;
- analisi su mercati, processi, modelli organizzativi e casi d'uso;
- segnali di maturita e direzione dei trend digitali.

Granularita attesa:

- buona sul piano tematico e di mercato;
- spesso orientata a fenomeni, segmenti o aree applicative piu che a vacancy;
- utile per letture verticali su cloud, AI, data, security, piattaforme e trasformazione.

Uso nello scoring:

- fonte di contesto per capire dove si concentra la trasformazione digitale italiana;
- utile per interpretare la rilevanza relativa di temi tecnici nei target role;
- utile per dare un peso leggermente maggiore a famiglie che incrociano trend consolidati e non solo buzzword;
- utile per affinare narrative e keyword, non per sostituire gli annunci.

Limiti:

- non e una fonte di domanda diretta;
- puo privilegiare temi di ricerca o analisi piu che bisogni immediati di hiring;
- va trattata come segnale di contesto e non come evidenza di vacancy.

## Fonti Da Non Sovrappesare

Queste fonti sono utili, ma non devono dominare lo scoring:

- ISTAT Professioni: serve soprattutto per tassonomia e mapping, non per la domanda.
- INAPP Atlante Lavoro: ottimo per classificazione e competenze, non per vacancy.
- Cedefop Skills-OVATE: utile per pattern europei, ma non per una lettura italiana primaria.
- EURES: importante come confronto europeo, ma non come proxy principale del mercato italiano.
- Anitec-Assinform: fonte settoriale e prospettica, non feed di domanda.
- Osservatori Digital Innovation Politecnico di Milano: contesto di mercato e trend, non domanda puntuale.

Regola di peso:

- se una di queste fonti contraddice gli annunci o il feedback manuale, va usata come ipotesi di contesto da verificare, non come correzione automatica del record;
- se queste fonti concordano tra loro ma non con gli annunci, il sistema deve abbassare il peso del segnale, non forzare il risultato.

## Priorita Di Integrazione

Ordine consigliato per integrare queste fonti nel sistema:

1. Excelsior Unioncamere
2. ISTAT Professioni
3. INAPP Atlante Lavoro
4. EURES
5. Cedefop Skills-OVATE
6. Anitec-Assinform
7. Osservatori Digital Innovation Politecnico di Milano

Motivo pratico:

- prima si integra la fonte piu vicina alla domanda italiana;
- poi si integrano le fonti di classificazione per tenere puliti titoli e competenze;
- poi si aggiungono i confronti europei e i segnali settoriali, con peso piu basso;
- infine si usano le fonti di ricerca e di contesto per rifinire la lettura, non per guidarla.

## Note Di Manutenzione

- Verificare periodicamente che gli URL pubblici restino corretti e che le sezioni citate siano ancora accessibili.
- Se una fonte cambia struttura, aggiornare prima il mapping concettuale e solo dopo eventuali automazioni.
- Non introdurre numeri di mercato o stime quantitative in questo file senza una fonte esplicita e verificabile.
- Se una fonte viene usata per un nuovo scopo, aggiornare anche `job-search/market-observatory-spec.md` e, se necessario, `job-search/scoring-model.md`.
- Tenere separati i ruoli delle fonti: domanda, classificazione, contesto e calibrazione.
- Per INAPP, rispettare sempre la licenza indicata dal sito nella fase di riuso o redistribuzione dei contenuti.
