# Progetto di Sicurezza dei Sistemi e Privacy AA 2024/2025

## 1 Presentazione
Questo progetto mira a realizzare una pipeline CI/CD sicura (SSDLC - Secure Software Development Life Cycle) applicata a un’applicazione Java. Il repository utilizzato è onlinebookstore, che simula un sistema di gestione per una libreria online.

La pipeline è stata configurata utilizzando GitHub Actions per automatizzare il processo di integrazione continua e delivery continuo, integrando i seguenti strumenti:

Maven: per la gestione del ciclo di vita del progetto e per la fase di build;

SonarQube: per eseguire una scansione SAST (Static Application Security Testing) finalizzata all’analisi della qualità del codice e alla rilevazione di vulnerabilità e code smell;

OWASP Dependency-Check: per effettuare un’analisi SCA (Software Composition Analysis) sulle librerie esterne utilizzate, rilevando vulnerabilità note;

Caching: per migliorare l’efficienza della pipeline tramite il caching delle dipendenze Maven e dei pacchetti di SonarQube;


## 2 Configurazione della Pipeline di CI/CD
La pipeline di CI/CD è stata implementata utilizzando GitHub Actions, scelta che ha permesso di automatizzare l’intero processo di integrazione e delivery direttamente all’interno del repository GitHub.


La pipeline si articola in quattro job principali:

#### Setup Database: 
configura un servizio MySQL tramite container Docker integrato in GitHub Actions, attende la disponibilità del database e lo inizializza con le tabelle necessarie e dati di test casuali. Questo garantisce un ambiente di test consistente e ripetibile per le fasi successive.

#### Build Project: 
effettua il checkout del codice, configura Java JDK 17 (Temurin), attiva il caching delle dipendenze Maven per velocizzare le build successive, ed esegue la compilazione del progetto con Maven (mvn clean install). L’output del build, ovvero il file WAR, viene archiviato come artefatto della pipeline.

#### SonarQube Analysis:
esegue una scansione SAST (Static Application Security Testing) usando il plugin Maven per SonarQube. Questa fase analizza il codice sorgente alla ricerca di vulnerabilità, bug e code smell, fornendo un quadro della qualità e sicurezza del codice. Viene utilizzato un token segreto per autenticarsi con il server SonarQube.

#### Dependency Check: 
utilizza l’azione ufficiale OWASP Dependency-Check per effettuare un’analisi SCA (Software Composition Analysis) sulle librerie di terze parti usate dal progetto. Questa analisi identifica vulnerabilità note presenti nelle dipendenze esterne, contribuendo a prevenire rischi di sicurezza derivanti da componenti non aggiornati o insicuri.


### Strumenti integrati per l’analisi
SonarQube: integrato tramite plugin Maven, fornisce un’analisi statica approfondita del codice sorgente.

OWASP Dependency-Check: integrato come GitHub Action, analizza le dipendenze di progetto per rilevare vulnerabilità note.

La pipeline viene eseguita automaticamente a ogni push sul repository. Grazie alla sequenza dei job e ai loro needs, ogni fase dipende dal completamento con successo della precedente, assicurando che il codice venga compilato solo se il database è pronto, che le analisi siano eseguite solo se la build è andata a buon fine, e così via.

Il caching delle dipendenze Maven e della cache di SonarQube migliora l’efficienza della pipeline, riducendo i tempi di esecuzione. L’upload degli artefatti consente di conservare e distribuire i risultati delle build e delle analisi.


Artifact Archiving: per salvare gli artefatti di build (file WAR) e i report di analisi, permettendo un successivo download e revisione;

Controllo su servizi esterni: tramite l’utilizzo di un servizio MySQL Dockerizzato per garantire l’inizializzazione e la preparazione dell’ambiente di test.

L’integrazione di questi strumenti consente di garantire una maggiore affidabilità e sicurezza del software attraverso l’automatizzazione di analisi statiche e controlli sulle dipendenze, migliorando la qualità generale del progetto e facilitando un processo di sviluppo più sicuro e controllato.
