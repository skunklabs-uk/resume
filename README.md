# Resume And Job Search Workspace

Questo repository e il luogo di lavoro per CV, posizionamento professionale e
strategia di ricerca lavoro di Ignazio Ingenito.

## Organizzazione

```text
cv/
  Ignazio.Ingenito.docx
  Ignazio.Ingenito.pdf

profile/
  positioning.md
  target-roles.md
  sources/
    cv-analysis/
      Ignazio.Ingenito.1.pdf
      Ignazio.Ingenito.2.pdf

job-search/
  market-observatory-spec.md
  linkedin-query-seeds.md
  italy-market-sources.md
  scoring-model.md

automations/
  n8n-workflows.md

docs/
  plans/
    2026-06-11-job-search-observatory-plan.md
```

## Responsabilita Delle Cartelle

`cv/` contiene solo il CV corrente. Il file principale modificabile e
`cv/Ignazio.Ingenito.docx`; il PDF principale e `cv/Ignazio.Ingenito.pdf`.

`profile/` contiene il posizionamento professionale: sintesi, narrativa,
target role, punti di forza, criteri di candidatura e materiali riusabili per
cover letter o messaggi. I materiali storici usati per analisi, ma non correnti,
stanno in `profile/sources/`.

`job-search/` contiene la strategia di ricerca lavoro: osservatorio di mercato,
query LinkedIn/Indeed, fonti italiane, role family, scoring model e note sulle
candidature.

`automations/` contiene documentazione e riferimenti alle automazioni che
supportano la ricerca lavoro.

`docs/plans/` contiene piani operativi versionati per modifiche multi-step.

## Separazione Da `n8n-workflows`

Questo repository e la source of truth per la strategia personale di ricerca
lavoro: profilo, CV, mercato target, query, criteri di ranking e decisioni sulle
candidature.

Il repository `skunklabs-uk/n8n-workflows` resta invece la
source of truth tecnica per workflow n8n importabili nel cluster: JSON dei
workflow, handoff GitOps, note di import, credenziali da collegare nella UI e
attivazione manuale.

In pratica:

- decisioni di carriera e mercato: qui;
- automazioni n8n eseguibili: `n8n-workflows`;
- i workflow n8n possono implementare o automatizzare specifiche definite qui,
  ma non devono diventare il posto dove ragionare sul posizionamento personale.

## Collegamento seriale al Developer Workspace

Un incarico autorizzato viene eseguito da un consumer seriale in un checkout
isolato, vincolato a repository e thread, branch, head e prompt.

Per l’adozione documentale di [Homelab #1265](https://github.com/skunklabs-uk/homelab/issues/1265),
il child prepara un report senza modificare file. Il coordinatore lo verifica
e ne registra l’accettazione con RETURN. L’applicazione della proposta e il
merge avvengono separatamente, tramite una normale PR discendente da main;
lo snapshot senza parent non viene integrato.

L’incarico riguarda questa nota tecnica. CV, profilo, strategia di ricerca
lavoro, candidature e corpus personali restano fuori scope. Le decisioni
personali appartengono a questo repository; i workflow n8n eseguibili al
repository dedicato. La preview HTTP non si applica a questa modifica
documentale, che non attesta il funzionamento delle automazioni.

Le procedure correnti sono nel [runbook Developer Workspace](https://github.com/skunklabs-uk/developer-workspace/blob/main/docs/WORKSPACE-HANDOFF.md)
e nel [README del deployment Homelab](https://github.com/skunklabs-uk/homelab/blob/main/gitops/apps/developer-workspace/README.md).

## Documenti Principali

- `profile/positioning.md`
- `profile/target-roles.md`
- `job-search/market-observatory-spec.md`
- `job-search/linkedin-query-seeds.md`
- `job-search/italy-market-sources.md`
- `job-search/scoring-model.md`
- `automations/n8n-workflows.md`
