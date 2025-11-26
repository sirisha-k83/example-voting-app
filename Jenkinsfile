pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        SONARQUBE_SERVER = 'sonar_server'
        SONAR_PROJECT_KEY = 'newtest'
        SONAR_PROJECT_NAME = 'newtest'

        DOCKER_REPOSITORY = 'sirishak83'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/sirisha-k83/example-voting-app.git'
            }
        }

    stage('SonarQube Analysis') {
      steps {
        script {
            def scannerHome = tool 'Sonar_Scanner'
            
            withCredentials([string(credentialsId: 'sonar_token', variable: 'SONAR_TOKEN_SECRET')]) { 
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh """
                        ${scannerHome}/bin/sonar-scanner \\
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \\
                        -Dsonar.projectName=${SONAR_PROJECT_NAME} \\
                        -Dsonar.login=${SONAR_TOKEN_SECRET} \\
                        -Dsonar.sources=vote,result,worker
                    """
                }
            }
        }
    }
}

        stage('Quality Gate Check') {
            steps {
                echo "Skipping quality gate check"
            }
        }

        stage('TRIVY FS Scan') {
            steps {
                sh "trivy fs . > trivyfs.txt"
                archiveArtifacts artifacts: 'trivyfs.txt', onlyIfSuccessful: true
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', url: '') {

                        // Build Docker images using Dockerfiles
                        sh "docker build -t ${DOCKER_REPOSITORY}/vote-app:latest ./vote"
                        sh "docker build -t ${DOCKER_REPOSITORY}/result-app:latest ./result"
                        sh "docker build -t ${DOCKER_REPOSITORY}/worker-app:latest ./worker"

                        // Push images
                        sh "docker push ${DOCKER_REPOSITORY}/vote-app:latest"
                        sh "docker push ${DOCKER_REPOSITORY}/result-app:latest"
                        sh "docker push ${DOCKER_REPOSITORY}/worker-app:latest"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh """
                        # Bring down old stack
                        docker compose down || true

                        # Deploy latest build
                        docker compose up -d
                    """
                }
            }
        }
    }
}
