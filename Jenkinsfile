pipeline {
    agent any

    environment {
        IMAGEN = "luciagruiz/polls"
        USUARIO = 'dockerhub-id'
    }

    stages {
        stage('Test en contenedor') {
            agent {
                docker {
                    image 'python:3.12'
                    args '-u root'
                }
            }
            steps {
                sh '''
                    pip install -r requirements.txt
                    python manage.py test
                '''
            }
        }

        stage('Creación y subida de la imagen') {
            steps {
                script {
                    def newApp = docker.build("${IMAGEN}:${BUILD_NUMBER}")
                    docker.withRegistry('', USUARIO) {
                        newApp.push()
                    }
                    sh "docker rmi ${IMAGEN}:${BUILD_NUMBER}"
                }
            }
        }

        stage('Despliegue en el VPS') {
            steps {
                sshagent(credentials: ['kyogre']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no debian@kyogre.luciagruiz.com '
                            cd django_tutorial &&
                            git pull &&
                            docker-compose down &&
                            docker-compose pull &&
                            docker-compose up -d
                        '
                    '''
                }
            }
        }
    }

    post {
        success {
            mail to: 'luciagruiz9@gmail.com',
                 subject: 'Jenkins Pipeline Exitoso',
                 body: "La ejecución del pipeline fue correcta. Build: ${env.BUILD_URL}"
        }
        failure {
            mail to: 'luciagruiz9@gmail.com',
                 subject: 'Jenkins Pipeline Fallido',
                 body: "Ha fallado alguna etapa del pipeline. Revisa Jenkins: ${env.BUILD_URL}"
        }
    }
}
