pipeline {
    agent any

    environment {
        JAVA_HOME = 'C:\\Program Files\\Java\\jdk-21'
        PATH = "${JAVA_HOME}\\bin;${env.PATH}"
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
                withCredentials([string(credentialsId: 'NVD_API_KEY', variable: 'NVD_API_KEY')]) {
                    bat 'mvn org.owasp:dependency-check-maven:check -Dformat=HTML -Ddependency-check.nvd.apiKey=%NVD_API_KEY%'
                }
            }
        }

        stage('Archivia Report') {
            steps {
                archiveArtifacts artifacts: 'target/dependency-check-report.html', fingerprint: true
            }
        }
    }

    post {
        always {
            echo 'Pulizia completata, niente Docker da rimuovere'
        }
    }
}
