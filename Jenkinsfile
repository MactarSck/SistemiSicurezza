pipeline {
    agent any

    environment {
        MYSQL_ROOT_PASSWORD = 'root'
        MYSQL_DATABASE = 'onlinebookstore'
        JAVA_HOME = '/opt/jdk'
    }

    stages {
        stage('Checkout Codice') {
            steps {
                git url: 'https://github.com/MactarSck/SistemiSicurezza.git', branch: 'master'
            }
        }

        stage('Start MySQL in Docker') {
            steps {
                sh '''
                docker run -d --name mysql-db \
                    -e MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_PASSWORD \
                    -e MYSQL_DATABASE=$MYSQL_DATABASE \
                    -p 3306:3306 \
                    --health-cmd="mysqladmin ping -h 127.0.0.1 --silent" \
                    --health-interval=10s \
                    --health-timeout=5s \
                    --health-retries=5 \
                    mysql:5.7
                '''
            }
        }

        stage('Attendi MySQL') {
            steps {
                sh '''
                for i in {1..30}; do
                  if docker exec mysql-db mysqladmin ping -h127.0.0.1 -uroot -proot --silent; then
                    echo "MySQL pronto!"
                    break
                  fi
                  echo "Attendo MySQL..."
                  sleep 2
                done
                '''
            }
        }

        stage('Inizializza DB') {
            steps {
                sh '''
                docker exec -i mysql-db mysql -uroot -proot onlinebookstore <<EOF
                CREATE TABLE IF NOT EXISTS books(
                    barcode VARCHAR(100) PRIMARY KEY,
                    name VARCHAR(100),
                    author VARCHAR(100),
                    price INT,
                    quantity INT
                );
                CREATE TABLE IF NOT EXISTS users(
                    username VARCHAR(100) PRIMARY KEY,
                    password VARCHAR(100),
                    firstname VARCHAR(100),
                    lastname VARCHAR(100),
                    address TEXT,
                    phone VARCHAR(100),
                    mailid VARCHAR(100),
                    usertype INT
                );
                INSERT INTO books VALUES(
                    '9780134190563',
                    'The Go Programming Language',
                    'Alan A. A. Donovan and Brian W. Kernighan',
                    400,
                    8
                );
                INSERT INTO users VALUES(
                    'shashi',
                    'shashi',
                    'Shashi',
                    'Raj',
                    'Bihar',
                    '1236547089',
                    'shashi@gmail.com',
                    2
                );
                COMMIT;
                EOF
                '''
            }
        }
        tools {
                 maven 'Maven'
            }

        stage('Build con Maven') {
            steps {
                sh 'mvn clean install -B'
            }
        }

        stage('Analisi SonarQube') {
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh 'mvn -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=MactarSck_SistemiSicurezza -Dsonar.token=$SONAR_TOKEN'
                }
            }
        }

        stage('Dependency Check') {
            steps {
                sh '''
                mkdir -p reports
                docker run --rm \
                  -v $(pwd):/src \
                  owasp/dependency-check \
                  --project "onlinebookstore" \
                  --scan /src \
                  --format HTML \
                  --out /src/reports \
                  --failOnCVSS 7 \
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
            sh 'docker rm -f mysql-db || true'
        }
    }
}
