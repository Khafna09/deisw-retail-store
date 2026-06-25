pipeline {
  agent any
  tools {
    maven 'MAVEN_3_9_16'
    jdk 'JDK_26'
  }
	environment {
    REGISTRY_USER = "jhector632" 
    IMAGE_NAME    = "retail-store-u20231c540"
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

	 stage('Construir y Publicar Imagen Docker') {
    steps {
        withCredentials([usernamePassword(credentialsId: 'DOCKER_HUB_CREDENTIALS', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            script {
                echo "Iniciando sesión en Docker Hub..."
                sh "echo '${DOCKER_PASS}' | docker login -u '${DOCKER_USER}' --password-stdin"

                echo "Construyendo imagen optimizada AMD64..."
                // Observa cómo se forma el nombre: ${REGISTRY_USER}_dockerhub/${IMAGE_NAME}:${TAG}
                sh "docker buildx build --platform linux/amd64 -t ${REGISTRY_USER}_dockerhub/${IMAGE_NAME}:${TAG} -t ${REGISTRY_USER}_dockerhub/${IMAGE_NAME}:latest --push ."
            }
        }
    }
}

	  /*stage('Construir  y Publicar  Imagen Docker') {
            steps {
                script {

					echo "Construyendo imagen híbrida/compatible con servidores de producción (AMD64)..."
					// Usamos 'buildx' para asegurar que la imagen de salida sea estrictamente para plataformas de 64 bits estándar
					sh "docker buildx build --platform linux/amd64 -t ${IMAGE_NAME}:${TAG} --load ."
					sh "docker buildx build --platform linux/amd64 -t ${IMAGE_NAME}:latest --load ."
                    
                    echo "Imagen construida exitosamente."
                }
            }
        }*/


    /*stage ('package Project') {
        steps {
            withMaven(maven : 'MAVEN_3_9_11') {
                sh 'mvn package'
            }
        }
    }*/


    }
}
