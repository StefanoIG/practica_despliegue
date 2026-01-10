pipeline {
    agent any

    tools {
        nodejs "Node25"
        dockerTool 'Dockertool'
    }

    stages {
        stage('Construir Imagen Docker') {
            steps {
                script {
                    // Usar la ruta completa de docker
                    def dockerPath = tool name: 'Dockertool', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
                    env.PATH = "${dockerPath}:${env.PATH}"
                }
                sh 'docker --version'
                sh 'docker build -t hola-mundo-node:latest .'
            }
        }

        stage('Ejecutar Contenedor Node.js') {
            steps {
                sh '''
                    # Detener y eliminar cualquier contenedor previo
                    docker stop hola-mundo-node || true
                    docker rm hola-mundo-node || true

                    # Ejecutar el contenedor de la aplicación
                    docker run -d --name hola-mundo-node -p 3000:3000 hola-mundo-node:latest
                '''
            }
        }
    }
}