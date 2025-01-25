pipeline {
    agent none

    environment {
        SONARQUBE_TOKEN = credentials('sonar-token')
        CONTAINER_NAME_PROJECT = 'node_project_LJRG'
    }

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

        stage("Listar carpetas y archivos del REPo") {
            agent {
                docker {
                    image 'node:18-alpine'
                }
            }
            steps {
                sh 'ls -la'
            }
        }

        stage("sonarQube Analisis ..."){
            agent any
            steps{
                script{
                    def scannerHome = tool 'SonarScanner';
                    withSonarQubeEnv() {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }

        }

        // stage("sonarQube analis .........."){
        //     agent any
        //     steps{
        //         script{
        //             sonarScanner{
        //                 installationName: 'sonarQube-server',
        //                 additionalArguments: "-Dsonar.login=${SONARQUBE_TOKEN}"
                    
        //             // def scannerHome = tool 'sonarQube';
        //             // withSonarQubeEnv('sonarQube') {
        //             //     sh "${scannerHome}/bin/sonar-scanner"
        //             // }
        //         }
        //     }
        // }

        // stage("sonarQube Analisis ...") {
        //     agent {
        //         docker {
        //             image 'node:18-alpine'
        //         }
        //     }
        //     steps {
        //         sh '''
        //         if [ ! -f ./node_modules/.bin/sonar-scanner ]; then
        //             echo "Sonar scanner no instalado, instalando dependencias..."
        //             npm install
        //         fi
        //         chmod +x ./node_modules/.bin/sonar-scanner
        //         '''
        //         script {
        //             withSonarQubeEnv("sonarQube") {
        //                 sh './node_modules/.bin/sonar-scanner'
        //             }
        //         }
        //     }
        // }
        // stage("sonarQube Analisis ...") {
        //     agent {
        //         docker {
        //             image 'node:18-alpine'
        //         }
        //     }
        //     steps {
        //         sh 'chmod +x ./node_modules/.bin/sonar-scanner'
        //         script {
        //             withSonarQubeEnv("sonarqube-server") {
        //                 sh './node_modules/.bin/sonar-scanner'
        //             }
        //         }
        //     }
        // }
    //  https://2b6e-38-25-17-255.ngrok-free.app 
        stage("sonarQube quality gate..."){
            agent any
            steps{
                waitForQualityGate abortPipeline: true
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
        stage('Testear conexion SHH con servidor digital Ocean') {
            when {
                branch 'develop'
            }
            agent any

            steps {
                sshagent(['droplet-ssh-key']) {
                    sh 'ssh -o StrictHostKeyChecking=no root@142.93.115.84 "echo conexion correcta"'
                }
            }
        }

        stage('Remover container en ejecucion antes de su actualizacion...') {
            when {
                branch 'develop'
            }
            agent any
            steps {
                sh """
                    docker rm -f ${CONTAINER_NAME_PROJECT} || true
                """
            }

        }


        // stage('Desplegar proyecto node a Digital ocean') {
        //     when {
        //         branch 'develop'
        //     }
        //     agent any

        //     environment {
        //             DOCKER_REPO = 'jesusramirezgamarra/jenkins-node'
        //     }

        //     steps {
        //         sshagent(['droplet-ssh-key']) {
        //             sh '''
        //                 ssh -o StrictHostKeyChecking=no root@142.93.115.84 "
        //                 docker pull $DOCKER_REPO:latest &&
        //                 docker run -d --name node_project_LJRG -p 8080:3000 $DOCKER_REPO:latest
        //                 "
        //             '''
        //         }
        //     }
        // }
        stage('Desplegar proyecto node a Digital ocean') {
            when {
                branch 'develop'
            }
            agent any

            environment {
                DOCKER_REPO = 'jesusramirezgamarra/jenkins-node'
            }

            steps {
                sshagent(['droplet-ssh-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no root@142.93.115.84 "
                        docker ps -q --filter name=node_project_LJRG | xargs -r docker stop &&
                        docker ps -a -q --filter name=node_project_LJRG | xargs -r docker rm &&
                        docker pull $DOCKER_REPO:latest &&
                        docker run -d --name node_project_LJRG -p 8080:3000 $DOCKER_REPO:latest
                        "
                    '''
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