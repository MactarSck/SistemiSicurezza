#  Progetto di Sicurezza dei Sistemi e Privacy AA 2024/2025

## 1 Presentazione
Questo progetto mira a realizzare una pipeline CI/CD sicura (SSDLC - Secure Software Development Life Cycle) applicata a un’applicazione Java. Il repository utilizzato è onlinebookstore, che simula un sistema di gestione per una libreria online.

La pipeline è stata configurata utilizzando GitHub Actions per automatizzare il processo di integrazione continua e delivery continuo, integrando i seguenti strumenti:

Maven: per la gestione del ciclo di vita del progetto e per la fase di build;

SonarQube: per eseguire una scansione SAST (Static Application Security Testing) finalizzata all’analisi della qualità del codice e alla rilevazione di vulnerabilità e code smell;

OWASP Dependency-Check: per effettuare un’analisi SCA (Software Composition Analysis) sulle librerie esterne utilizzate, rilevando vulnerabilità note;

Caching: per migliorare l’efficienza della pipeline tramite il caching delle dipendenze Maven e dei pacchetti di SonarQube;

Slack: per notificare automaticamente l’esito della pipeline al gruppo di sviluppo


## 2 Configurazione della Pipeline di CI/CD
La pipeline di CI/CD è stata implementata utilizzando GitHub Actions, scelta che ha permesso di automatizzare l’intero processo di integrazione e delivery direttamente all’interno del repository GitHub.


La pipeline si articola in cinque job principali:

#### Setup Database: 
configura un servizio MySQL tramite container Docker integrato in GitHub Actions, attende la disponibilità del database e lo inizializza con le tabelle necessarie e dati di test casuali. Questo garantisce un ambiente di test consistente e ripetibile per le fasi successive.

#### Build Project: 
effettua il checkout del codice, configura Java JDK 17 (Temurin), attiva il caching delle dipendenze Maven per velocizzare le build successive, ed esegue la compilazione del progetto con Maven (mvn clean install). L’output del build, ovvero il file WAR, viene archiviato come artefatto della pipeline.

#### SonarQube Analysis:
esegue una scansione SAST (Static Application Security Testing) usando il plugin Maven per SonarQube. Questa fase analizza il codice sorgente alla ricerca di vulnerabilità, bug e code smell, fornendo un quadro della qualità e sicurezza del codice. Viene utilizzato un token segreto per autenticarsi con il server SonarQube.

#### Dependency Check: 
utilizza l’azione ufficiale OWASP Dependency-Check per effettuare un’analisi SCA (Software Composition Analysis) sulle librerie di terze parti usate dal progetto. Questa analisi identifica vulnerabilità note presenti nelle dipendenze esterne, contribuendo a prevenire rischi di sicurezza derivanti da componenti non aggiornati o insicuri.

#### Notify (Slack Notification)
Alla fine del processo, viene eseguito un job che invia una notifica automatica su Slack al team di sviluppo, indicando che la pipeline è stata completata con successo. L’URL del webhook è salvato in un secret GitHub (SLACK_WEBHOOK_URL) per garantire la sicurezza.


### Strumenti integrati per l’analisi
SonarQube: integrato tramite plugin Maven, fornisce un’analisi statica approfondita del codice sorgente.

OWASP Dependency-Check: integrato come GitHub Action, analizza le dipendenze di progetto per rilevare vulnerabilità note.

La pipeline viene eseguita automaticamente a ogni push sul repository. Grazie alla sequenza dei job e ai loro needs, ogni fase dipende dal completamento con successo della precedente, assicurando che il codice venga compilato solo se il database è pronto, che le analisi siano eseguite solo se la build è andata a buon fine, e così via.

Il caching delle dipendenze Maven e della cache di SonarQube migliora l’efficienza della pipeline, riducendo i tempi di esecuzione. L’upload degli artefatti consente di conservare e distribuire i risultati delle build e delle analisi.

Slack: integrato tramite webhook e GitHub Actions, per comunicare automaticamente lo stato della pipeline al team.

Artifact Archiving: per salvare gli artefatti di build (file WAR) e i report di analisi, permettendo un successivo download e revisione;

Controllo su servizi esterni: tramite l’utilizzo di un servizio MySQL Dockerizzato per garantire l’inizializzazione e la preparazione dell’ambiente di test.

L’integrazione di questi strumenti consente di garantire una maggiore affidabilità e sicurezza del software attraverso l’automatizzazione di analisi statiche e controlli sulle dipendenze, migliorando la qualità generale del progetto e facilitando un processo di sviluppo più sicuro e controllato.

## CODICE PIPELINE

### Stage 1 – Check-out del codice
 ```yaml
 - name: Checkout codice
   uses: actions/checkout@v3

 ```
Presente in tutti i job: build, sonar, dependency-check

Questa fase recupera il codice sorgente dal repository GitHub affinché i job successivi (build, analisi, ecc.) possano operare sui file più aggiornati del progetto.

### Stage 2 – Build
 ```yaml
 - name: Configura JDK 17 (Temurin)
        uses: actions/setup-java@v3
        with:
          java-version: 17
          distribution: temurin

      - name: Cache dipendenze Maven
        uses: actions/cache@v4
        with:
          path: ~/.m2/repository
          key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
          restore-keys: ${{ runner.os }}-m2

      - name: Build progetto (Maven)
        run: mvn clean install -B

 ```
Job: build

In questa fase il codice Java viene compilato usando Maven. Prima viene installata la JDK 17, poi viene attivata una cache per velocizzare le build future, infine viene eseguita la compilazione con mvn clean install.

### Stage 3 – Scansione SAST (Static Application Security Testing)

 ```yaml
- name: Cache pacchetti SonarQube
        uses: actions/cache@v4
        with:
          path: ~/.sonar/cache
          key: ${{ runner.os }}-sonar
          restore-keys: ${{ runner.os }}-sonar

      - name: Esegui SonarQube Scanner
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: mvn -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=MactarSck_SistemiSicurezza

 ```
Usa SonarQube per eseguire un’analisi statica del codice sorgente. Individua problemi come codice duplicato, vulnerabilità, bug logici, e fornisce metriche di qualità. Serve per prevenire vulnerabilità direttamente nel codice scritto dagli sviluppatori.

###  Stage 4 e 5 – Scansione SCA (Software Composition Analysis) - Check dei gate di qualità e sicurezza

 ```yaml
 - name: Esegui Dependency Check
        uses: dependency-check/Dependency-Check_Action@main
        continue-on-error: true
        env:
          JAVA_HOME: /opt/jdk
        with:
          project: onlinebookstore
          path: '.'
          format: HTML
          out: reports
          args: >
            --failOnCVSS 7
            --enableRetired

```
Job: dependency-check

Questa fase esegue la scansione delle librerie di terze parti usate nel progetto (dipendenze) per verificare la presenza di vulnerabilità note (CVEs). Il parametro --failOnCVSS 7 consente di fallire la build se vengono trovate vulnerabilità con severità ≥ 7, ma continue-on-error impedisce che blocchi la pipeline.

###  Stage 6 – Archiviazione locale

```yaml
- name: Carica artefatto WAR
  uses: actions/upload-artifact@v4
  with:
    name: artefatto-app
    path: target/*.war

```
 Job: build

 ```yaml
- name: Carica report Dependency Check
  uses: actions/upload-artifact@v4
  with:
    name: Report-Dependency-Check
    path: reports

```
Job: dependency-check

Gli artefatti generati dalla build (come file .war) o dai report di scansione vengono salvati all’interno della pipeline, così da poter essere scaricati e analizzati successivamente, anche manualmente.

###  Stage 7 – Notifica Slack

 ```yaml
- name: Invia notifica a Slack
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
  run: |
    curl -X POST -H 'Content-type: application/json' --data "{
      \"text\": \"✅ CI/CD completata per *onlinebookstore* su branch *${{ github.ref_name }}*.\nJobs completati: Setup DB, Build, SonarQube, Dependency Check.\nControlla i report se necessario.\"
    }" $SLACK_WEBHOOK_URL

 ```
Job: notify

Esegue una richiesta POST al webhook Slack per informare il gruppo che la pipeline è stata completata con successo.


## 3 Analisi delle vulnerabilità
 











