pipeline {
    agent any

    triggers {
        pollSCM('H/5 * * * *')
    }

    environment {
        DOCKER_IMAGE = 'nargese/student-management'
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Commit') {
            steps {
                sh 'git log -1 --pretty=format:"Hash: %h%nAuteur: %an%nMessage: %s"'
            }
        }

        stage('Build') {
            steps {
                sh 'chmod +x mvnw'
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Test unitaire') {
            steps {
                sh './mvnw test'
            }
            post {
                always {
                    junit allowEmptyResults: true,
                          testResults: 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                    docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhubcredentials',
                        usernameVariable: 'DOCKERHUB_USER',
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$DOCKERHUB_TOKEN" | docker login -u "$DOCKERHUB_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker push ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
            archiveArtifacts artifacts: 'target/*.jar',
                             fingerprint: true,
                             allowEmpty: true
        }
        success {
            echo 'Pipeline réussi : le JAR a été archivé.'
        }
        failure {
            echo 'Pipeline échoué : consultez les logs et les résultats des tests.'
        }
    }
}
