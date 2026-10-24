pipeline {
    agent any

    tools {
        jdk 'JAVA_HOME'
        maven 'M2_HOME'
    }

    environment {
        DOCKERHUB_USER = 'nargese'
        IMAGE = "${DOCKERHUB_USER}/student-management"
    }

    triggers {
        pollSCM('H/5 * * * *')
    }

    stages {
        stage('Git') {
            steps {
                checkout scm
                script {
                    env.BRANCHE = (env.GIT_BRANCH ?: 'main').replaceAll('^origin/', '')
                    env.TAG = "${env.BRANCHE}-${env.BUILD_NUMBER}"
                }
                sh 'git log -1 --pretty=format:"%h - %an : %s"'
                echo "Branche : ${env.BRANCHE} — tag de l'image : ${env.TAG}"
            }
        }

        stage('Build') {
            steps {
                sh 'chmod +x mvnw'
                sh './mvnw -B clean package -DskipTests'
            }
        }

        stage('Test unitaire') {
            steps {
                sh './mvnw -B test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Analyse SonarQube') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                        ./mvnw -B sonar:sonar \
                          -Dsonar.projectKey=student-management-nargese_5cc1_g1 \
                          -Dsonar.projectName=student-management-nargese_5cc1_g1
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${IMAGE}:${TAG} ."
            }
        }

        stage('Docker push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_PASS'
                )]) {
                    sh 'echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin'
                    sh "docker push ${IMAGE}:${TAG}"

                    script {
                        if (env.BRANCHE == 'main') {
                            sh "docker tag ${IMAGE}:${TAG} ${IMAGE}:latest"
                            sh "docker push ${IMAGE}:latest"
                        }
                    }

                    sh 'docker logout'
                }
            }
        }

        stage('Déploiement') {
            when {
                expression { env.BRANCHE == 'main' }
            }
            steps {
                sh 'docker compose pull app'
                sh 'docker compose up -d'
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            echo 'Pipeline réussi : tests, analyse SonarQube et Quality Gate validés.'
        }
        failure {
            echo 'Pipeline échoué : consultez les logs, les tests et le Quality Gate.'
        }
        always {
            sh "docker image rm -f ${IMAGE}:${TAG} || true"
        }
    }
}
