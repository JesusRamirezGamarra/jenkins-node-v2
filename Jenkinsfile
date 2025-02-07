pipeline {
    agent none

    stages {
        stage('Instalar dependencias...') {
            agent {
                docker {
                    image 'node:18-alpine'
                }
            }
            steps {
                echo 'Listando todas las carpetas y archivos...'
                sh 'npm install'
            }
        }

        stage('Ejecutar tests...') {
            agent {
                docker {
                    image 'node:18-alpine'
                }
            }
            steps {
                echo 'Listando todas las carpetas y archivos...'
                sh 'npm run test'
            }
        }

        stage('Construir y pushear imagen a dockerhub') {
            when {
                branch 'develop'
            }

            agent {
                docker {
                    image 'docker:latest'
                }
            }

            environment {
                    DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
                    DOCKER_REPO = 'jesusramirezgamarra/jenkins-node'
            }
            steps {
                sh '''
                echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                docker build -t $DOCKER_REPO:latest .
                docker push $DOCKER_REPO:latest
                '''
            }
        }
    }
   
    post {
        success {
            mail to: 'lcruzfarfan@gmail.com',
                 subject: 'Pipeline ejecucion correcta',
                 body: "El pipeline ${env.JOB_NAME} con el build numero ${env.BUILD_NUMBER} ha finalizado de manera correcta"
        }
    }
}