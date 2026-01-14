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
                    echo ""
                    echo "Verificando package.json:"
                    cat package.json
                '''
            }
        }

        stage('Construir Imagen Docker') {
            steps {
                script {
                    def dockerPath = tool name: 'Dockertool', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
                    env.PATH = "${dockerPath}:${env.PATH}"
                }
                sh '''
                    echo "Construyendo imagen Docker..."
                    docker build -t hola-mundo-node:latest .
                '''
            }
        }

        stage('Ejecutar tests') {
            steps {
                sh '''
                    echo "Ejecutando tests dentro del contenedor..."
                    docker run --rm hola-mundo-node:latest npm test
                '''
            }
        }

        stage('Detener contenedor previo') {
            steps {
                sh '''
                    echo "Deteniendo contenedores previos si existen..."
                    docker stop hola-mundo-node || true
                    docker rm hola-mundo-node || true
                '''
            }
        }

        stage('Ejecutar Contenedor Node.js') {
            steps {
                sh '''
                    echo "Iniciando contenedor de la aplicación..."
                    docker run -d --name hola-mundo-node -p 3000:3000 hola-mundo-node:latest
                    
                    echo "Esperando que el contenedor inicie..."
                    sleep 5
                    
                    echo "Verificando estado del contenedor:"
                    docker ps | grep hola-mundo-node
                    
                    echo "Logs del contenedor:"
                    docker logs hola-mundo-node
                '''
            }
        }

        stage('Verificar aplicación') {
            steps {
                sh '''
                    echo "Verificando que la aplicación responda..."
                    sleep 2
                    docker exec hola-mundo-node wget -q -O- http://localhost:3000 || echo "La aplicación aún no responde"
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline ejecutado exitosamente!'
            sh '''
                echo ""
                echo "==================================="
                echo "Contenedores en ejecución:"
                docker ps
                echo ""
                echo "Aplicación disponible en: http://localhost:3000"
                echo "==================================="
            '''
        }
        failure {
            echo '❌ El pipeline ha fallado. Revisa los logs para más detalles.'
            sh '''
                echo "Logs del contenedor (si existe):"
                docker logs hola-mundo-node || echo "No hay logs disponibles"
            '''
        }
        always {
            echo "Limpieza completada"
        }
    }
}