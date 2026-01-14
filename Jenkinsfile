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
                    docker run --rm \
                        -v "${WORKSPACE}":/app:rw \
                        -w /app \
                        --user root \
                        node:25-alpine \
                        sh -c "ls -la && npm install"
                '''
            }
        }

        stage('Ejecutar tests') {
            steps {
                sh '''
                    docker run --rm \
                        -v "${WORKSPACE}":/app:rw \
                        -w /app \
                        --user root \
                        node:25-alpine \
                        npm test
                '''
            }
        }

        stage('Construir Imagen Docker') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                sh 'docker --version'
                sh '''
                    cd "${WORKSPACE}"
                    docker build -t hola-mundo-node:latest .
                '''
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
                    
                    # Verificar que el contenedor esté corriendo
                    sleep 3
                    docker ps | grep hola-mundo-node
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
            sh 'docker ps'
        }
        always {
            echo "Limpieza de recursos..."
        }
    }
}