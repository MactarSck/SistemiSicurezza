pipeline {
    agent any

    environment {
        MYSQL_ROOT_PASSWORD = 'root'
        MYSQL_DATABASE = 'onlinebookstore'
        JAVA_HOME = '/opt/jdk'
    }

    tools {
        maven 'Maven'  // qui va fuori da stages
    }

    stages {
        stage('Checkout Codice') {
            steps {
                git url: 'https://github.com/MactarSck/SistemiSicurezza.git', branch: 'master'
            }
        }

        // ... gli altri stage restano uguali

        stage('Build con Maven') {
            steps {
                sh 'mvn clean install -B'
            }
        }

        // ...
    }

    post {
        always {
            echo 'Pulizia container MySQL'
            sh 'docker rm -f mysql-db || true'
        }
    }
}
