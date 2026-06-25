pipeline {
    agent any
    tools {
        maven 'MAVEN_3_9_16'
        jdk 'JDK_26'
    }
    environment {
        REGISTRY_USER = "jhector632" 
        IMAGE_NAME    = "dockerhub-retail-store-u20231c540" 
        TAG           = "${env.BUILD_NUMBER}" 
    }

    stages {
        stage ('Compile Project') {
            steps {
                withMaven(maven : 'MAVEN_3_9_16') {
                    sh 'mvn clean compile'
                }
            }
        }

        stage('Validate Checkstyle') {
            steps {
                withMaven(maven: 'MAVEN_3_9_16') {
                    sh 'mvn checkstyle:check'
                }
            }
        }

        stage('Validate Unit Tests') {
            steps {
                withMaven(maven: 'MAVEN_3_9_16') {
                    sh 'mvn test'
                }
            }
        }

        stage('Validate Test Coverage') {
            steps {
                withMaven(maven: 'MAVEN_3_9_16') {
                    sh 'mvn clean verify jacoco:report'
                    sh 'mvn jacoco:check'
                }
            }
        }

        stage ('SonarQube Analysis') {
            steps {
                // 1. Enviar el código a analizar a SonarQube
                // Se reemplazó '/' por ':' en el projectKey para evitar el error de validación
                withSonarQubeEnv('MiSonarServer') {
                    sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=jhector632_dockerhub:retail-store-u20231c540'
                }
                
                // 2. Pausar el pipeline y esperar la respuesta del Webhook de SonarQube
                script {
                    timeout(time: 10, unit: 'MINUTES') { 
                        def qg = waitForQualityGate()
                        
                        // 3. Evaluar el estado del Quality Gate
                        if (qg.status != 'OK') {
                            error "El pipeline se ha detenido porque el código no superó el Quality Gate de SonarQube. Estado: ${qg.status}"
                        }
                    }
                }
            }
        }

        stage('Construir y Publicar Imagen Docker') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'DOCKER_HUB_CREDENTIALS', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        echo "Iniciando sesión en Docker Hub..."
                        sh "echo '${DOCKER_PASS}' | docker login -u '${DOCKER_USER}' --password-stdin"

                        echo "Construyendo y publicando imagen optimizada AMD64..."
                        
                        sh "docker buildx build --platform linux/amd64 -t ${REGISTRY_USER}/${IMAGE_NAME}:${TAG} -t ${REGISTRY_USER}/${IMAGE_NAME}:latest --push ."
                    }
                }
            }
        }
    }
}
