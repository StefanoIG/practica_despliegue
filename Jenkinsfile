pipeline {
    agent any

    tools {
        dockerTool 'Dockertool'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verificar archivos') {
            steps {
                sh '''
                    echo "Contenido del workspace:"
                    ls -la
                    echo "Verificando package.json:"
                    cat package.json || echo "package.json no encontrado"
                '''
            }
        }

        stage('Instalar dependencias') {
            steps {
                script {
                    def dockerPath = tool name: 'Dockertool', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
                    env.PATH = "${dockerPath}:${env.PATH}"
                }
                sh '''
                    docker run --rm -v ${WORKSPACE}:/app -w /app node:25-alpine npm install
                '''
            }
        }

        stage('Ejecutar tests') {
            steps {
                sh '''
                    docker run --rm -v ${WORKSPACE}:/app -w /app node:25-alpine npm test
                '''
            }
        }

        stage('Construir Imagen Docker') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                sh 'docker --version'
                sh 'docker build -t hola-mundo-node:latest .'
            }
        }

        stage('Ejecutar Contenedor Node.js') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
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

    post {
        failure {
            echo 'El pipeline ha fallado. Revisa los logs para más detalles.'
        }
        success {
            echo 'Pipeline ejecutado exitosamente!'
        }
    }
}