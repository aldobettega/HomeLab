# Indice della Tesi

## 1. Introduzione
* **1.1 L'azienda ospitante:** Presentazione di Kirey Group e del contesto aziendale in cui si è svolto il tirocinio.
* **1.2 L'idea del progetto:** Introduzione al problema del Vulnerability Assessment moderno e necessità di una prioritizzazione basata sul rischio reale e non solo sulla severità teorica.
* **1.3 Organizzazione dell'elaborato:** Guida alla lettura e panoramica dei contenuti dei capitoli successivi.

## 2. Processi e Metodologie
* **2.1 Metodologia di lavoro:** Descrizione dell'approccio utilizzato (es. framework Agile, sprint, daily meeting) e della modalità di lavoro in smart working.
* **2.2 Pianificazione delle attività:** programma settimanale dello stage con descrizione dei task affrontati e dei milestone raggiunti.

## 3. Analisi dei Requisiti e del Dominio
* **3.1 Studio del dominio:** Analisi della letteratura (paper accademici) sul Vulnerability Assessment e sui limiti delle metriche tradizionali. Introduzione ai concetti chiave (CVSS, punteggi probabilistici EPSS e catalogo CISA KEV).
* **3.2 Casi d'uso:** Definizione delle funzionalità principali del sistema dal punto di vista dell'utente finale (es. Start Assessment, Get Assessment Report).
* **3.3 Tracciamento dei requisiti:** Mappatura tra gli obiettivi di business/progetto e le funzionalità tecniche da sviluppare.

## 4. Tecnologie e Strumenti Utilizzati
* **4.1 Stack Backend:** Python 3.12, framework e librerie principali.
* **4.2 Stack Frontend:** TypeScript, Angular.
* **4.3 Strumenti a supporto:** Strumenti di versioning, CI/CD e project management.

## 5. Ambiente di Test e Setup del Laboratorio
* **5.1 Architettura del laboratorio:** Creazione dell'ambiente isolato con macchine virtuali.
* **5.2 Configurazione del target e dello scanner:** Setup della macchina vulnerabile (Metasploitable) e della virtual appliance dello scanner (Qualys). Configurazione del network e del routing locale.

## 6. Progettazione Architetturale e Implementazione
* **6.1 Architettura del sistema:** 
  * **6.1.1 Backend (Ports & Adapters):** Adozione dell'architettura esagonale per disaccoppiare la logica di dominio dalle dipendenze esterne (scanner e provider di enrichment).
  * **6.1.2 Frontend (Data-Driven UI):** Interazione diretta con le API REST. Scelta progettuale di delegare la responsabilità del mapping interamente al backend (assenza voluta di ViewModel e DTO intermedi lato frontend).
* **6.2 Ciclo di vita del software (Pipeline di Analisi):** Spiegazione dei passaggi sequenziali: Scansione, Normalizzazione, Arricchimento (Enrichment), Prioritizzazione e Generazione del Vulnerability Report.
* **6.3 Decisioni architetturali e Design Pattern:** Analisi delle sfide e delle soluzioni. Adozione del *Bulk Processing* per ottimizzare le chiamate alle API esterne e abbattere la latenza.
* **6.4 Codifica e convenzioni:** Organizzazione del codice sorgente. Utilizzo di `dataclasses` per il dominio puro e `Pydantic` per gli schemi API. Convenzioni di naming (snake_case nel backend, camelCase nel frontend).
* **6.5 Verifica, validazione e gestione degli errori:** Strategie di testing, tolleranza ai fallimenti dei provider esterni, gestione resiliente dei valori nulli accettati dal sistema.

## 7. Il Ruolo dell'Intelligenza Artificiale (Explainability)
* **7.1 AI per la spiegazione, non per la decisione:** Scelta architetturale di relegare l'AI a un layer di *Explainability*, senza che questa influenzi il calcolo matematico della priorità operativa.
* **7.2 Integrazione e prompt engineering:** Interazione con il provider AI (es. Google Gemini) per la ricezione dei dati di contesto formattati e la generazione di output strutturati (Structured Output con Pydantic).

## 8. Sfide e Problematiche Incontrate
* **8.1 Limiti infrastrutturali:** Difficoltà legate alle risorse hardware limitate per l'esecuzione del laboratorio locale.
* **8.2 Vincoli sulle API esterne:** Limitazioni dovute all'uso del servizio gratuito dell'intelligenza artificiale (es. rate limiting) e conseguente decisione di analizzare un sottoinsieme limitato (MVP su 5 vulnerabilità) per dimostrare il proof of concept.

## 9. Conclusioni
* **9.1 Raggiungimento degli obiettivi:** Analisi dei risultati ottenuti rispetto ai requisiti iniziali.
* **9.2 Competenze acquisite:** Valutazione della crescita professionale (approccio metodologico, nuove tecnologie apprese, comprensione della Cyber Threat Intelligence).
* **9.3 Valutazione personale dello stage:** Considerazioni finali sull'esperienza formativa lavorativa in Kirey Group.

## Bibliografia e Sitografia
* Testi di riferimento.
* Paper accademici sul Vulnerability Management e Risk Prioritization.
* Documentazione ufficiale delle tecnologie e degli standard usati (CVSS, FIRST EPSS, CISA KEV, ecc.).