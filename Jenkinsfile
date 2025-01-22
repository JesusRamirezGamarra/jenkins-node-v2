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

        stage('Testear conexion SSH con servidor DigitalOcean') {
            steps {
                sshagent(['droplet-ssh-key']) {
                    sh '''
                    ssh -vvv -o StrictHostKeyChecking=no -i C:/Users/jesus/.ssh/id_rsa root@165.22.186.185 "echo conexion correcta"
                    '''
                }
            }
        }


        // stage('Testear conexion SSH con servidor DigitalOcean con puerto 2222') {
        //     when {
        //         branch 'develop'
        //     }
        //     agent any
        //     steps {
        //         sshagent(['droplet-ssh-key']) {
        //             sh '''
        //             ssh -p 2222 -o StrictHostKeyChecking=no root@165.22.186.185 "echo conexion correcta"
        //             '''
        //         }
        //     }
        // }


        // stage('Testear conexion SSH con servidor DigitalOcean') {
        //     when {
        //         branch 'develop'
        //     }
        //     agent any
        //     steps {
        //         sshagent(['droplet-ssh-key']) {
        //             sh '''
        //             ssh -vvv -o StrictHostKeyChecking=no root@165.22.186.185 "echo conexion correcta"
        //             '''
        //         }
        //     }
        // }

        stage('Testear conexion SHH con servidor digital Ocean') {
            when {
                branch 'develop'
            }
            agent any

            steps {
                sshagent(['droplet-ssh-key']) {
                    sh 'ssh -o StrictHostKeyChecking=no root@165.22.186.185 "echo conexion correcta"'
                    //sh 'ssh root@165.22.186.185 "echo conexion correcta"'
                } 
            } 
        }
    }

    post {
        success {
            mail to: 'luciojesusramirezgamarra@gmail.com',
                subject: "Pipeline ${env.JOB_NAME} ejecucion correcta",
                body: """
                Hola,

                El pipeline '${env.JOB_NAME}' (Build #${env.BUILD_NUMBER}) ha finalizado de manera correcta

                Los detalles se pueden revisar en el siguiente enlace:
                ${env.BUILD_URL}

                Saludos,
                Jenkins Server
                """
        }
    }
}