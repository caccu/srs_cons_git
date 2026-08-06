---
{"dg-publish":true,"permalink":"/wiki/analyses/analysis-2026-08-05-stato-scaduto-spiegato-semplice/","title":"Stato SCADUTO — Spiegato in Modo Semplice","tags":["scaduto","bat-03","guida-semplice","sia","asr","cliente","batch-02","cdu-17"],"dg-note-properties":{"title":"Stato SCADUTO — Spiegato in Modo Semplice","aliases":["SCADUTO spiegato semplice"],"type":"analysis","tags":["scaduto","bat-03","guida-semplice","sia","asr","cliente","batch-02","cdu-17"],"created":"2026-08-05","updated":"2026-08-05","sources":["2026-03-02-conspref-srs-v1-revised","2019-03-20-acc-del-cdu-01-servizi-acquisizione"],"related":["[[analysis-2026-08-05-stato-scaduto-comunicazione-sia|Stato SCADUTO — versione tecnica (BAT-03)]]","[[analysis-2026-05-27-punti-aperti-spiegati|Punti Aperti — Spiegati in Modo Semplice]]","[[batch-processes|Processi Batch — BATCH-01, BATCH-02, BATCH-03]]","[[ciclo-vita-consenso|Ciclo di Vita del Consenso]]","[[alternativa-batch-03-pull|Alternativa BATCH-03 — PULL CDU-17]]"]}}
---


# Stato SCADUTO — Spiegato in Modo Semplice

Versione in linguaggio piano del documento tecnico [[wiki/analyses/analysis-2026-08-05-stato-scaduto-comunicazione-sia\|Stato SCADUTO — semantica e comunicazione asincrona ai SIA/ASR]]. Serve a preparare la call dedicata con CSI sul punto aperto **BAT-03**.

**Chi legge questo documento** non ha bisogno di conoscere il dettaglio tecnico: qui si spiega **cosa succede**, **perché è un problema** e **quali sono le tre strade possibili**.

---

## 1. Il concetto in due righe

Quando l'informativa sulla privacy che il cittadino ha accettato **scade**, il suo consenso non viene cancellato: resta valido, ma viene marcato come **"SCADUTO"**, che significa *«la scelta del cittadino vale ancora, però deve accettare la nuova versione dell'informativa»*.

Il problema è che **nessuno avvisa le aziende sanitarie** che questo è successo.

---

## 2. Un'analogia

Immaginiamo il consenso come un **modulo firmato allo sportello**.

- Il cittadino ha firmato dicendo «sì, acconsento».
- Dopo un po' la Regione aggiorna il testo dell'informativa (nuova versione, nuove clausole).
- Da quel momento il modulo firmato dal cittadino è ancora **valido nella sostanza** — lui ha detto sì e continua a valere sì — ma è **agganciato a un testo superato**: prima o poi il cittadino dovrà rifirmare sulla versione nuova.

Lo stato SCADUTO è esattamente questa condizione: **il "sì" è ancora buono, la carta su cui è scritto è da rinnovare.**

---

## 3. Cosa cambia rispetto al sistema attuale — attenzione, è un tranello

Questo è il punto più delicato di tutto il documento.

Nel **vecchio sistema** la parola "scaduto" veniva usata per marcare un modulo **superato, da ignorare**: quando il cittadino cambiava idea, il vecchio foglio veniva timbrato "scaduto" e archiviato, e ne nasceva uno nuovo.

Nel **nuovo sistema** la stessa parola significa **il contrario**: SCADUTO è il modulo **attualmente in vigore**, quello da guardare, semplicemente in attesa che il cittadino riaccetti l'informativa aggiornata.

| | Vecchio sistema | Nuovo sistema |
|---|---|---|
| Cosa vuol dire "scaduto" | Documento **archiviato**, da ignorare | Documento **valido e in corso** |
| Cosa deve fare chi lo legge | Scartarlo | Usarlo normalmente |

> ⚠️ **Il rischio concreto:** un'azienda sanitaria che riusa la logica del vecchio sistema vedrebbe la parola "scaduto" e **butterebbe via un consenso che invece è valido**. Il paziente risulterebbe senza consenso pur avendolo dato.

Va aggiunto un dettaglio: nel vecchio sistema questa parola **non usciva mai** verso le aziende sanitarie, restava un'informazione interna. Nel nuovo sistema, per la prima volta, le aziende la vedono. Quindi non hanno mai avuto occasione di imparare il significato nuovo.

**Conclusione: qualunque strada si scelga, va mandata una comunicazione esplicita alle aziende sanitarie su questo cambio di significato.** È indipendente da tutto il resto.

---

## 4. Il problema aperto (punto BAT-03)

Mettiamo insieme tre fatti:

1. Quando un consenso diventa SCADUTO, **non parte nessun avviso** alle aziende sanitarie.
2. Il servizio che espone l'elenco completo dei consensi (lo "snapshot", il CDU-17) **contiene solo i consensi attivi**: i consensi scaduti semplicemente non compaiono in quell'elenco.
3. Il passaggio a SCADUTO è **massivo**: quando scade un'informativa, tutti i consensi collegati cambiano stato **nella stessa notte**, potenzialmente decine di migliaia insieme.

Risultato: un'azienda sanitaria che tiene una **copia locale** dei consensi — cioè che ha scaricato l'elenco per non doverlo richiedere ogni volta — **non ha modo di accorgersi** che quei consensi sono cambiati. L'unico modo sarebbe richiedere lo stato **un cittadino alla volta**, il che non è praticabile su grandi numeri.

**La domanda da porre a CSI è una sola:**

> Lo stato SCADUTO deve essere un **avviso che parte** verso le aziende sanitarie, oppure resta un'informazione che l'azienda **va a leggere quando le serve**?

Da questa risposta discende tutto il resto.

---

## 5. Le tre strade possibili

### Strada A — Non avvisiamo nessuno, chi ha bisogno chiede

L'azienda sanitaria, ogni volta che deve usare un consenso, **lo chiede al momento** al sistema regionale, invece di fidarsi della propria copia.

- 👍 Nessuno sviluppo aggiuntivo, nessun costo, nessuna modifica ai documenti già approvati.
- 👍 Ha una logica di fondo solida: siccome in SCADUTO **il consenso vale ancora**, per l'attività clinica quotidiana non cambia nulla. Cambia solo che, prima o poi, al cittadino verrà richiesta la firma sull'informativa aggiornata.
- 👎 Non funziona per le aziende che tengono una copia locale — cioè proprio quelle che si sono attrezzate meglio.

### Strada B — Mettiamo a disposizione un "elenco delle novità" (raccomandata) ⭐

Il sistema regionale espone un elenco consultabile del tipo *«ecco cosa è cambiato da ieri a oggi»*. L'azienda sanitaria lo va a leggere quando vuole, con la frequenza che preferisce, e aggiorna la propria copia.

- 👍 **Coerente con l'impostazione già decisa e confermata dal committente il 20/07/2026**: il sistema regionale è il "centro stella" che mette a disposizione i dati, le aziende vengono a prenderli. Nessun invio, nessun onere infrastrutturale sulla Regione.
- 👍 Riusa tutto quello che è già stato progettato per il servizio snapshot: stessa sicurezza, stesso modo di sfogliare i risultati, stessa gestione degli errori.
- 👍 Risolve da solo il caso «l'azienda era ferma per manutenzione»: quando torna su, legge le novità arretrate. Nessuna coda di rispedizioni da gestire.
- 👍 Regge bene il picco notturno: ognuno legge quando è pronto, senza congestionare nulla.
- 👎 Va progettato e sviluppato (impegno contenuto: è un'estensione di un servizio già previsto).
- 👎 È l'azienda sanitaria a doversi organizzare per leggerlo con regolarità.

### Strada C — Mandiamo un avviso a domicilio

Il sistema regionale **invia attivamente** un messaggio a ciascuna azienda sanitaria a ogni cambio di stato, riusando i canali di notifica già esistenti, attivabile azienda per azienda.

- 👍 Non serve creare un servizio nuovo: i canali esistono già e CSI ha confermato il 20/07/2026 che vanno comunque "svecchiati", quindi la finestra per aggiungerci questo dato è aperta.
- 👍 Le aziende con copia locale restano allineate senza fare nulla.
- 👎 Contraddice quanto scritto oggi nei documenti approvati (che dicono espressamente che questa variazione non viene notificata alle aziende) → richiede una modifica documentale.
- 👎 **Genera un picco enorme e concentrato**: la scadenza di un'informativa può produrre decine di migliaia di avvisi in una sola notte, con tutti i problemi di dimensionamento e di rispedizione dei messaggi non consegnati.
- 👎 Rimette in carico alla Regione l'onere dell'invio, che era già stato scartato in passato proprio per questo motivo.

---

## 6. Cosa proponiamo

**Strada B come meccanismo, Strada A come comportamento predefinito** per le aziende che non tengono una copia locale. La Strada C solo se CSI chiede espressamente l'invio attivo.

Il motivo è di coerenza: la Strada B è l'unica delle tre che **rispetta l'impostazione architetturale già decisa e confermata dal committente**, riusa infrastruttura già progettata e non riapre una discussione (l'onere di invio a carico della Regione) che era già stata chiusa.

**In tutti e tre i casi resta obbligatoria** la comunicazione alle aziende sanitarie sul cambio di significato della parola "scaduto" (§3).

---

## 7. Le domande da portare alla call CSI

| # | Domanda | A cosa serve la risposta |
|---|---|---|
| 1 | SCADUTO è un **avviso da mandare** o un **dato da leggere su richiesta**? | È la domanda madre: decide fra A, B e C |
| 2 | Quali aziende sanitarie tengono una **copia locale** dei consensi? | Se nessuna, la Strada A basta e non serve altro |
| 3 | Se serve avvisare: **invio attivo** o **elenco delle novità** da consultare? | Coerenza con il modello "centro stella" |
| 4 | Il tracciato dei messaggi esistenti può ospitare questo dato? | CSI ha lasciato aperti i nomi dei campi: la finestra è aperta ora |
| 5 | Le aziende sanitarie sanno che "scaduto" ha **cambiato significato**? | È il rischio più concreto: scartare un consenso valido |
| 6 | Serve un **preavviso** di scadenza (N giorni prima) alle aziende? | Permetterebbe di distribuire il carico invece di subire il picco |
| 7 | Ogni quanto gira la procedura notturna che produce gli SCADUTO? | Determina quanto tempo passa prima che l'azienda possa accorgersene |
| 8 | Per quanto tempo un consenso resta consultabile in stato SCADUTO? | Serve alle aziende per impostare la gestione della propria copia |

---

## 8. Cosa succede se non si decide

Il progetto va avanti comunque: tecnicamente il sistema funziona anche così. Il rischio non è di blocco, è di **disallineamento silenzioso**. Le aziende sanitarie continuerebbero a lavorare su una copia dei consensi che, dopo ogni scadenza di informativa, si allontana progressivamente dalla realtà — **senza che nessuno se ne accorga**, perché non c'è nessun errore, nessun messaggio, nessun segnale.

È il tipo di problema che non si vede in collaudo e si manifesta in esercizio, mesi dopo.

---

## 9. Per approfondire

| Documento | Contenuto |
|---|---|
| [[wiki/analyses/analysis-2026-08-05-stato-scaduto-comunicazione-sia\|Stato SCADUTO — versione tecnica]] | Stessa analisi con riferimenti puntuali a SRS, servizi, tabelle e ADR |
| [[wiki/concepts/batch-processes\|Processi Batch]] | Come funziona la procedura notturna che produce lo stato SCADUTO |
| [[wiki/concepts/ciclo-vita-consenso\|Ciclo di Vita del Consenso]] | Tutti gli stati possibili di un consenso e i passaggi fra l'uno e l'altro |
| [[wiki/analyses/analysis-2026-05-27-punti-aperti-spiegati\|Punti Aperti — Spiegati in Modo Semplice]] | Gli altri punti ancora da chiudere con CSI, stesso registro |
