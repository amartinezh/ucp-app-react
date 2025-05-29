pipeline {
    agent any
    tools {
        nodejs 'Node_24'  // Configurado en Global Tools
        sonarScanner 'SonarQubeScanner' // Configurado en Global Tools
    }
    environment {
        SONAR_PROJECT_KEY = 'ucp-app-react'
        SONAR_PROJECT_NAME = 'UCP React App'
    }
    stages {
        // Etapa 1: Checkout
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/amartinezh/ucp-app-react.git'
            }
        }

        // Nueva etapa: Análisis de SonarQube
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                        -Dsonar.sources=src \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.login=${SONAR_AUTH_TOKEN} \
                        -Dsonar.javascript.node=${NODEJS_HOME}/bin/node \
                        -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
                    '''
                }
            }
        }

        // Etapa 2: Build
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
                sh 'npm run test:coverage' // Asegúrate de que tu package.json tenga este script
            }
        }

        // Etapa 3: Pruebas Paralelizadas
        stage('Pruebas en Paralelo') {
            parallel {
                // Pruebas en Chrome
                stage('Pruebas Chrome') {
                    steps {
                        script {
                            try {
                                sh 'npm test -- --browser=chrome --watchAll=false --ci --reporters=jest-junit'
                                junit 'junit-chrome.xml'
                            } catch (err) {
                                echo "Pruebas en Chrome fallaron: ${err}"
                                currentBuild.result = 'UNSTABLE'
                            }
                        }
                    }
                }
                // Pruebas en Firefox
                stage('Pruebas Firefox') {
                    steps {
                        script {
                            try {
                                sh 'npm test -- --browser=firefox --watchAll=false --ci --reporters=jest-junit'
                                junit 'junit-firefox.xml'
                            } catch (err) {
                                echo "Pruebas en Firefox fallaron: ${err}"
                                currentBuild.result = 'UNSTABLE'
                            }
                        }
                    }
                }
            }
        }


        // Etapa 4: Deploy Simulado
        stage('Deploy a Producción (Simulado)') {
            steps {
                script {
                    // Crear carpeta "prod" y copiar build
                    sh 'mkdir -p prod && cp -r build/* prod/'
                    echo "¡Deploy simulado exitoso! Archivos copiados a /prod"
                }
            }
        }
    }
    post {
        always {
            // Publicar reportes HTML (opcional)
            publishHTML target: [
                allowMissing: true,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'prod',
                reportFiles: 'index.html',
                reportName: 'Demo Deploy'
            ]
            
            // Agregar notificación de calidad de SonarQube
            script {
                def qg = waitForQualityGate()
                if (qg.status != 'OK') {
                    error "Calidad no aprobada: ${qg.status}"
                }
            }

            mail(
                to: 'amartinezh@gmail.com',
                subject: "Build Status: ${currentBuild.currentResult}",
                body: "Job: ${env.JOB_NAME}\nEstado: ${currentBuild.currentResult}\nURL: ${env.BUILD_URL}"
            )
            
            // Limpiar workspace
            cleanWs()
        }
    }
}
