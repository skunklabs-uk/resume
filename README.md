# Resume And Job Search Workspace

Questo repository e il luogo di lavoro per CV, posizionamento professionale e
strategia di ricerca lavoro di Ignazio Ingenito.

## Organizzazione

```text
cv/
  Ignazio.Ingenito.docx
  Ignazio.Ingenito.pdf

profile/
  .gitkeep
  sources/
    cv-analysis/
      Ignazio.Ingenito.1.pdf
      Ignazio.Ingenito.2.pdf

job-search/
  .gitkeep

automations/
  .gitkeep
```

## Responsabilita Delle Cartelle

`cv/` contiene solo il CV corrente. Il file principale modificabile e
`cv/Ignazio.Ingenito.docx`; il PDF principale e `cv/Ignazio.Ingenito.pdf`.

`profile/` conterra il posizionamento professionale: sintesi, narrativa,
target role, punti di forza, criteri di candidatura e materiali riusabili per
cover letter o messaggi. I materiali storici usati per analisi, ma non correnti,
stanno in `profile/sources/`.

`job-search/` conterra la strategia di ricerca lavoro: osservatorio di mercato,
query LinkedIn/Indeed, fonti italiane, role family, scoring model e note sulle
candidature.

`automations/` conterra documentazione e riferimenti alle automazioni che
supportano la ricerca lavoro.

## Separazione Da `n8n-workflows`

Questo repository e la source of truth per la strategia personale di ricerca
lavoro: profilo, CV, mercato target, query, criteri di ranking e decisioni sulle
candidature.

Il repository `/home/iingenito/projects/personal/n8n-workflows` resta invece la
source of truth tecnica per workflow n8n importabili nel cluster: JSON dei
workflow, handoff GitOps, note di import, credenziali da collegare nella UI e
attivazione manuale.

In pratica:

- decisioni di carriera e mercato: qui;
- automazioni n8n eseguibili: `n8n-workflows`;
- i workflow n8n possono implementare o automatizzare specifiche definite qui,
  ma non devono diventare il posto dove ragionare sul posizionamento personale.

## Prossimi Documenti Previsti

- `profile/positioning.md`
- `profile/target-roles.md`
- `job-search/market-observatory-spec.md`
- `job-search/linkedin-query-seeds.md`
- `job-search/italy-market-sources.md`
- `job-search/scoring-model.md`
- `automations/n8n-workflows.md`
