# GitLab Flow

**ATTENZIONE: consiglio caldamente di leggere il documento dalla main branch in inglese in quanto questa versione del documento
è stata tradotta automaticamente in italiano da un LLM.
La traduzione potrebbe contenere errori di battitura, traduzione, o altro. Più avanti la tradurrò a mano e per bene.**

## Indice

0. [Introduzione](#introduzione)
1. [Il Nostro Caso](#il-nostro-caso)
2. [Passi Già Intrapresi e Dove Possiamo Migliorare](#passi-già-intrapresi-e-dove-possiamo-migliorare)
   - [Separazione per Ambiente, Funzionalità e Fix](#separazione-per-ambiente-funzionalità-e-fix)
   - [Hotfix Rapidi](#possibilità-per-hotfix-rapidi)
   - [Utilizzo di un Monorepo](#utilizzo-di-un-monorepo)
   - [Standard per il Version Tagging](#standard-per-il-version-tagging)
   - [Test Continuativi e Merge Requests Automatici](#test-continuativi-e-merge-requests-automatici)
3. [Possibili Soluzioni](#possibili-soluzioni)
4. [Workflow Finale](#workflow-finale)

## Introduzione

Questo repository è stato creato per spiegare e progettare quello che per me sarebbe il miglior, più semplice e veloce modo
di organizzare il flusso di lavoro per una **web application a microservizi**.

Lo sto creando anche per meglio interiorizzare il processo e delineare tutti i requisiti, problemi e soluzioni per il mio caso d'uso. Tutte le sezioni di questo documento sono soggette a modifiche.

## Il nostro caso

Abbiamo un'applicazione composta da **molti** microservizi (circa 40-50), e abbiamo bisogno di un flusso di lavoro semplice ed efficiente per:

- Gestire più ambienti di sviluppo (dev, staging, produzione)
- Garantire la **migliore** qualità del codice possibile ad ogni passo del nostro SDLC
- Rilasciare il nostro codice su staging/produzione e abilitare **rollbacks veloci e affidabili**
- Fare hotfix al volo per risolvere bug urgenti o problemi in produzione
- Automatizzare qualsiasi passo possibile lungo il percorso, per prevenire errori umani
- Rendi la vita dei nostri sviluppatori più facile

## Passi già intrapresi e dove possiamo migliorare

Abbiamo già intrapreso i seguenti passi:

- Pipeline CI/CD per l'integrazione automatica del codice e il rilascio
- Versionamento semantico di base

Nelle sezioni seguenti, considereremo tutti i miglioramenti possibili al nostro flusso di lavoro e i problemi che derivano dall'integrazione di questi processi.

### Separazione per ambiente, funzionalità e fix

Separando i nostri rami per ambiente (un ramo per dev, un ramo per staging, un ramo per produzione), possiamo guardare direttamente al codice che gira sui diversi ambienti, rendendo più facile il debug e il tracciamento dei problemi.

- Creare un ramo per ogni funzionalità E fix/hotfix può migliorare il nostro flusso di lavoro. Quando uniamo questi rami nel ramo principale, dovremmo mantenere la cronologia Git e non unire tutti i commit, per fornire una comprensione più chiara di ogni fase del nostro ciclo.
  In caso di problemi, è più facile fare il debug e risolverli.
- Ogni ramo dovrebbe riflettere direttamente il codice che gira sull'ambiente corrispondente. Questo approccio consente anche di fare hotfix in ambienti non dev come la produzione e staging.

### Possibilità per hotfix rapidi

Abbiamo bisogno di introdurre la possibilità di fare correzioni rapide nel caso in cui troviamo un bug critico in qualsiasi sezione del nostro SDLC.
Potremmo rendere il processo CI/CD più semplice per un rilascio più veloce, saltando direttamente i nostri ambienti dev o staging.

### Utilizzo di un monorepo

Nel mio caso, il progetto è diviso in _diversi_ repository, anche se i microservizi sono strettamente interconnessi tra loro. Potremmo migliorare il nostro flusso di lavoro creando un monorepo per il nostro progetto, anche se questo non è privo di svantaggi. Non usare un monorepo pone anche la sfida di tracciare le dipendenze (sia esterne che interne).

- Un approccio ibrido valido potrebbe essere quello di **tenere le nostre dipendenze e librerie condivise in un monorepo** per tracciarle. Questo semplificherebbe il tracciamento e l'aggiornamento delle versioni delle nostre dipendenze. Tramite l'uso di un bot autenticato su un registro o repository privato, potremmo automatizzare il processo di aggiornamento delle dipendenze di ogni microservizio senza preoccupazioni, definendo possibili problemi di sicurezza legati a dipendenze vulnerabili.

### Standard per il version tagging

Abbiamo bisogno di essere allineati sullo standard per il version tagging. Potremmo utilizzare
[**Semantic Versioning**](https://semver.org/). Ecco una breve descrizione su come funzionerebbe per noi.

- Nome versione **X.Y.Z**, dove
  - **X** DEVE essere incrementato se vengono introdotte modifiche non compatibili con la versione precedente nell'API pubblica. PUÒ anche includere modifiche di livello minore o patch. Le versioni patch e minor devono essere azzerate quando viene incrementata la versione principale.
  - **Y** DEVE essere incrementato se vengono introdotte nuove funzionalità compatibili con la versione precedente nell'API pubblica. DEVE essere incrementato se qualsiasi funzionalità dell'API pubblica è contrassegnata come obsoleta. PUÒ essere incrementato se vengono introdotte funzionalità sostanziali o miglioramenti nel codice privato. PUÒ includere modifiche di patch. La versione patch DEVE essere azzerata quando viene incrementata la versione minor.
  - **Z** DEVE essere incrementato se vengono introdotte solo correzioni di bug compatibili con la versione precedente. Una correzione di bug è definita come una modifica interna che risolve comportamenti errati.
    Questo potrebbe essere facilmente RISERVATO per **hotfix** su staging o produzione.
- Etichettare le nostre versioni in base all'ambiente in cui vengono rilasciate potrebbe anche essere preso in considerazione (per esempio v0.0.1-staging), sebbene potrebbe causare confusione quando si applicano gli hotfix e si tracciano le dipendenze. **Semantic Versioning** dovrebbe essere sufficiente, e fornisce facilmente al team una comprensione di quale versione del nostro codice è in esecuzione. I task di Gradle/Maven o le pipeline CI/CD per l'aggiornamento automatico delle versioni devono essere scritti per ridurre gli errori umani.
  L'uso di problemi GitLab, rilasci e milestone per tracciare i nostri rilasci/hotfix potrebbe anche migliorare il nostro flusso di lavoro fornendo una prospettiva chiara e diretta sullo stato del nostro codice e sui problemi urgenti.
- Al momento, non stiamo usando il version tagging per il nostro chart finale Helm. Penso che dovremmo iniziare a farlo per tenere traccia meglio delle nostre modifiche e per abilitare rollback più facili.

### Test Continuativi e Merge Requests Automatici

I nostri processi CI/CD potrebbero essere migliorati per eseguire test su OGNI ramo e Merge Request. Questo dovrebbe individuare bug ed errori di build prima, migliorando la qualità del codice in ogni fase del percorso (anche se ciò comporta maggiori costi riguardo a runner e computazione). Le Merge Requests dovrebbero assicurarsi che il codice superi un controllo di qualità, e solo allora venga unito. Dal dev allo staging può essere automatico, ma dallo staging alla produzione possiamo richiedere un input manuale.

## Possibili Soluzioni

Questa è una bozza su come affronterei tutti i problemi considerati sopra:

1. **Separazione dei rami per ambiente, funzionalità** - crea un ramo per ogni ambiente (dev, staging, produzione), funzionalità e hotfix.
   Modifica i nostri processi CI/CD per adattarli a ogni ramo, per legare direttamente i nostri rami agli ambienti.
   - Strumenti automatici per la promozione a staging o produzione devono essere creati o recuperati per ridurre gli errori umani e per tracciare meglio i nostri cambiamenti.
   - I rami hotfix devono seguire questa convenzione: `hotfix/<ISSUE_ID>`
2. **Processo CI/CD più semplice per gli hotfix** - crea un processo CI/CD più semplice per gli hotfix. Etichettando un commit su staging o produzione con _hotfix_ verrà automaticamente
   incrementata la versione minor e poi il codice verrà rilasciato una volta che i test sono passati.
3. **Organizzazione delle librerie interne e delle dipendenze in un monorepo** - questo migliorerebbe il loro tracciamento e ridurrebbe gli errori. Ciò richiederebbe molto sforzo al momento.
4. **Utilizzo di Semantic Versioning** - utilizzare semantic versioning per tracciare le modifiche del nostro codice, usando Z (X.Y.Z) solo per gli hotfix.
   Creazione di task Maven/Gradle e pipeline CI/CD per l'aggiornamento automatico delle versioni dei nostri microservizi e dei chart Helm.
5. **Abilitazione del processo CI/CD ad ogni passo del percorso** - eseguire test e build per OGNI ramo per individuare i bug prima.
6. **Utilizzo di ArgoCD e RenovateBot** - configurare _ArgoCD_ per osservare il nostro template finale Helm (e fare il commit del nostro
   template Helm ogni passo del percorso in un repository) renderebbe più facili i rollback, e fornirebbe una comprensione più chiara e veloce dello stato
   attuale del nostro codice. L'utilizzo di _ArgoCD_ favorisce anche (**GitOps**)[https://www.redhat.com/en/topics/devops/what-is-gitops#gitops-workflows].
   L'utilizzo di _RenovateBot_ sembra anche essenziale per tracciare tutte le nostre dipendenze e individuare eventuali problemi di sicurezza,
   migliorando la sicurezza della nostra applicazione e riducendo gli errori umani, mantenendo allineato ogni microservizio.
   - Un lavoro notturno di Renovate o il suo deployment su un pod Kubernetes o macchina virtuale è sufficiente e una soluzione a basso costo.

## Workflow Finale

Ecco il processo SDLC proposto:

1. **Sviluppo delle funzionalità**

   - Gli sviluppatori creano rami per le funzionalità: `feature/<nome-funzionalità>`.
   - CI esegue test unitari, analisi statica e build per il ramo.
   - Al termine, crea una Merge Request al ramo `dev`.
   - CI convalida la Merge Request: controlli di qualità del codice, test e build.

2. **Ambiente di sviluppo**

   - Le funzionalità unite vengono distribuite su `dev` automaticamente.
   - Le pipeline CI/CD eseguono test di integrazione, test end-to-end, incremento versione e deployment.
   - I bug vengono risolti nei rami `dev` o `fix/<id-issue>`.

3. **Ambiente di staging**

   - Le funzionalità considerate stabili vengono promosse a `staging` tramite Merge Request.
   - CI/CD distribuisce su staging ed esegue i test.
   - Viene eseguita una QA manuale e vengono preparate le note di rilascio.

4. **Ambiente di produzione**

   - Le modifiche approvate vengono unite a `production` tramite Merge Request.
   - CI/CD distribuisce su produzione con un tag di versione semantico (ad esempio, `vX.Y.Z`).
   - I sistemi di monitoraggio e logging garantiscono la stabilità.

5. **Processo di Hotfix**

   - Per problemi critici, crea `hotfix/<id-issue>` da `production`.
   - CI convalida e distribuisce direttamente su produzione dopo che i test sono passati.
   - L'hotfix viene ripristinato su `staging` e `dev` per sincronizzare gli ambienti.

6. **Gestione delle dipendenze**

   - Utilizza RenovateBot per gli aggiornamenti delle dipendenze in un monorepo condiviso.
   - Aggiornamenti automatici delle dipendenze ogni notte o continuamente.

7. **Meccanismo di rollback**

   - ArgoCD traccia i template Helm e i commit per un rollback facile.
   - Le distribuzioni fallite possono essere ripristinate allo stato precedente funzionante.

8. **Gestione delle versioni**

   - Semantic Versioning: incrementa il patch per gli hotfix, il minor per le nuove funzionalità, il major per i cambiamenti incompatibili.
   - CI automatizza gli aggiornamenti delle versioni e i tag di rilascio.

9. **Miglioramento Continuo**
   - Tutti i passi vengono registrati in GitLab Issues e Milestones.
   - Retrospettive regolari per perfezionare i flussi di lavoro.
