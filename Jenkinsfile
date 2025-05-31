pipeline {
    agent any

    environment {
        // Se usi un DB esterno, puoi impostare qui le variabili di connessione
        //MYSQL_ROOT_PASSWORD = 'root'
        //MYSQL_DATABASE = 'onlinebookstore'
        JAVA_HOME = 'C:\\Program Files\\Java\\jdk...' // aggiorna se serve
    }

    tools {
        maven 'Maven'
    }

    stages {
        stage('Checkout Codice') {
            steps {
                git url: 'https://github.com/MactarSck/SistemiSicurezza.git', branch: 'master'
            }
        }

        // RIMOSSE le fasi Start MySQL, Attendi MySQL e Inizializza DB

        stage('Build con Maven') {
            steps {
                bat 'mvn clean install -B'
            }
        }

        stage('Analisi SonarQube') {
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    bat 'mvn -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=MactarSck_SistemiSicurezza -Dsonar.token=%SONAR_TOKEN%'
                }
            }
        }

        stage('Dependency Check') {
            steps {
                echo 'Salto Dependency Check perché richiede Docker'
                // Se vuoi fare Dependency Check senza Docker, devi installare localmente l'OWASP Dependency Check CLI
                // oppure rimuovere completamente questa fase
            }
        }

        stage('Pubblica Report') {
            steps {
                // Se non generi report di Dependency Check, puoi togliere o adattare questa fase
                echo 'Nessun report da archiviare'
            }
        }
    }

    post {
        always {
            echo 'Pulizia completata, niente Docker da rimuovere'
        }
    }
}
