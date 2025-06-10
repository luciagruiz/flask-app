pipeline {
    agent any

    environment {
        IMAGEN = "luciagruiz/flask-app"
        USUARIO = 'dockerhub-id'
    }

    stages {
        stage("Test en contenedor") {
            agent {
                docker {
                    image "python:3"
                    args '-u root:root'
                }
            }
            stages {
                stage('Clone') {
                    steps {
                        git branch:'main',url:'https://github.com/luciagruiz/flask-app.git'
                    }
                }
                stage('Install') {
                    steps {
                        sh 'pip install -r app/requirements.txt'
                    }
                }
                stage('Test')
                {
                    steps {
                        sh 'pytest app/test_app.py'
                    }
                }
            }
        }

        stage('Creación y subida de la imagen') {
            agent any
            stages {
                stage('Build') {
                    steps {
                        script {
                            newApp = docker.build "$IMAGEN:$BUILD_NUMBER"
                        }
                    }
                }
                stage('Deploy') {
                    steps {
                        script {
                            docker.withRegistry( '', USUARIO ) {
                                newApp.push()
                                newApp.push("latest")
                            }
                        }
                    }
                }
                stage('Clean Up') {
                    steps {
                        sh "docker rmi $IMAGEN:$BUILD_NUMBER"
                        }
                }
            }
        }
        
        stage ('Despliegue') {
            agent any
            stages {
                stage ('Despliegue en el VPS'){
                    steps{
                        sshagent(credentials : ['kyogre']) {
                        sh 'ssh -o StrictHostKeyChecking=no debian@kyogre.luciagruiz.com. "cd flask-app && git pull && docker-compose down && docker pull luciagruiz/flask-app:latest && docker-compose up -d && docker image prune -f"'
                        }
                    }
                }
            }
        }
    }    
    
    post {
        always {
        mail to: 'luciagruiz9@gmail.com',
        subject: "Despliegue de $IMAGEN: ${currentBuild.fullDisplayName}",
        body: "El despliegue ${env.BUILD_URL} de $IMAGEN ha finalizado: ${currentBuild.result}"
        }
    }
}
