
pipeline {
    agent any

    tools {
        jdk 'JAVA_HOME'
        maven 'M2_HOME'
    }

    triggers {
        pollSCM('H/5 * * * *')
    }

    stages {
        stage('Commit') {
            steps {
                sh 'git log -1 --pretty=format:"Hash: %h%nAuteur: %an%nMessage: %s"'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test unitaire') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'target/*.jar',
                             fingerprint: true
            echo 'Pipeline réussi : le JAR a été archivé.'
        }
        failure {
            echo 'Pipeline échoué : consultez les logs et les résultats des tests.'
        }
    }
}
