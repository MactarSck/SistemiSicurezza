pipeline {
    agent any

    environment {
        MYSQL_ROOT_PASSWORD = 'root'
        MYSQL_DATABASE = 'onlinebookstore'
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

        stage('Start MySQL in Docker') {
            steps {
                bat '''
                docker run -d --name mysql-db ^
                    -e MYSQL_ROOT_PASSWORD=%MYSQL_ROOT_PASSWORD% ^
                    -e MYSQL_DATABASE=%MYSQL_DATABASE% ^
                    -p 3306:3306 ^
                    --health-cmd="mysqladmin ping -h 127.0.0.1 --silent" ^
                    --health-interval=10s ^
                    --health-timeout=5s ^
                    --health-retries=5 ^
                    mysql:5.7
                '''
            }
        }

        stage('Attendi MySQL') {
            steps {
                bat '''
                for /l %%i in (1,1,30) do (
                  docker exec mysql-db mysqladmin ping -h127.0.0.1 -uroot -proot --silent && (
                    echo MySQL pronto!
                    goto :break
                  )
                  echo Attendo MySQL...
                  timeout /t 2
                )
                :break
                '''
            }
        }

        stage('Inizializza DB') {
            steps {
                bat '''
                docker exec -i mysql-db mysql -uroot -proot onlinebookstore < init.sql
                '''
                // Oppure metti i comandi SQL in un file init.sql nel repo e li esegui così
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
                bat '''
                mkdir reports
                docker run --rm ^
                  -v %cd%:/src ^
                  owasp/dependency-check ^
                  --project "onlinebookstore" ^
                  --scan /src ^
                  --format HTML ^
                  --out /src/reports ^
                  --failOnCVSS 7 ^
                  --enableRetired
                '''
            }
        }

        stage('Pubblica Report') {
            steps {
                archiveArtifacts artifacts: 'reports/*.html', fingerprint: true
            }
        }
    }

    post {
        always {
            echo 'Pulizia container MySQL'
            bat 'docker rm -f mysql-db || exit 0'
        }
    }
}
